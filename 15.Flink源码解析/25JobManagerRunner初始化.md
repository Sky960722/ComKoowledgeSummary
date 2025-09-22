# JobManager 初始化流程解析

## JobManager 核心角色

- JobManager 是 Flink 集群中的核心进程（在 Standalone 模式、YARN、K8s 等部署模式下均存在），主要负责资源管理、任务调度和作业生命周期管理。
- 其内部包含多个子组件，核心组件为 ResourceManager 和 JobMaster，二者均运行在 JobManager 进程中。
- 本文将从客户端提交任务后的接收环节开始，解析 JobManager 的初始化流程。

## JobSubmitHandler：接收任务提交

`JobSubmitHandler` 类专门负责接收客户端提交的任务，核心逻辑在 `handleRequest` 方法中：

```java
@Override
protected CompletableFuture<JobSubmitResponseBody> handleRequest(
        @Nonnull HandlerRequest<JobSubmitRequestBody> request,
        @Nonnull DispatcherGateway gateway)
        throws RestHandlerException {
    // 处理上传文件
    final Collection<File> uploadedFiles = request.getUploadedFiles();
    final Map<String, Path> nameToFile =
            uploadedFiles.stream()
                    .collect(Collectors.toMap(File::getName, Path::fromLocalFile));

    // 校验文件唯一性
    if (uploadedFiles.size() != nameToFile.size()) {
        throw new RestHandlerException(
                String.format(
                        "上传文件数量与预期不符。预期: %s 实际: %s",
                        nameToFile.size(),
                        uploadedFiles.size()),
                HttpResponseStatus.BAD_REQUEST);
    }

    final JobSubmitRequestBody requestBody = request.getRequestBody();

    // 校验JobGraph文件是否存在
    if (requestBody.jobGraphFileName == null) {
        throw new RestHandlerException(
                String.format(
                        "%s字段不可省略或为null。",
                        JobSubmitRequestBody.FIELD_NAME_JOB_GRAPH),
                HttpResponseStatus.BAD_REQUEST);
    }

    // 加载JobGraph
    CompletableFuture<JobGraph> jobGraphFuture = loadJobGraph(requestBody, nameToFile);

    // 获取需上传的JAR和artifact文件
    Collection<Path> jarFiles = getJarFilesToUpload(requestBody.jarFileNames, nameToFile);
    Collection<Tuple2<String, Path>> artifacts =
            getArtifactFilesToUpload(requestBody.artifactFileNames, nameToFile);

    // 上传JobGraph相关文件并完成初始化
    CompletableFuture<JobGraph> finalizedJobGraphFuture =
            uploadJobGraphFiles(gateway, jobGraphFuture, jarFiles, artifacts, configuration);
    
    // 提交作业
    CompletableFuture<Acknowledge> jobSubmissionFuture =
            finalizedJobGraphFuture.thenCompose(
                    jobGraph -> gateway.submitJob(jobGraph, timeout));

    // 返回作业提交结果
    return jobSubmissionFuture.thenCombine(
            jobGraphFuture,
            (ack, jobGraph) -> new JobSubmitResponseBody("/jobs/" + jobGraph.getJobID()));
}
```

## Dispatcher：作业提交与管理

Dispatcher 是作业提交的核心协调者，其 `submitJob` 方法负责作业提交的校验与处理：

```java
@Override
public CompletableFuture<Acknowledge> submitJob(JobGraph jobGraph, Time timeout) {
    final JobID jobID = jobGraph.getJobID();
    log.info("接收作业提交 '{}' ({})。", jobGraph.getName(), jobID);
    
    return isInGloballyTerminalState(jobID)
            .thenComposeAsync(
                    isTerminated -> {
                        // 校验作业状态：已终止则拒绝提交
                        if (isTerminated) {
                            log.warn(
                                    "忽略作业提交 '{}' ({})，因为作业已处于全局终止状态。",
                                    jobGraph.getName(),
                                    jobID);
                            return FutureUtils.completedExceptionally(
                                    DuplicateJobSubmissionException.ofGloballyTerminated(
                                            jobID));
                        } 
                        // 校验作业是否已存在
                        else if (jobManagerRunnerRegistry.isRegistered(jobID)
                                || submittedAndWaitingTerminationJobIDs.contains(jobID)) {
                            return FutureUtils.completedExceptionally(
                                    DuplicateJobSubmissionException.of(jobID));
                        } 
                        // 校验资源配置
                        else if (isPartialResourceConfigured(jobGraph)) {
                            return FutureUtils.completedExceptionally(
                                    new JobSubmissionException(
                                            jobID,
                                            "当前不支持部分顶点配置资源的作业，该限制将在未来版本移除。"));
                        } 
                        // 执行实际提交逻辑
                        else {
                            return internalSubmitJob(jobGraph);
                        }
                    },
                    getMainThreadExecutor());
}
```

`internalSubmitJob` 方法进一步处理作业提交的核心流程：

```java
private CompletableFuture<Acknowledge> internalSubmitJob(JobGraph jobGraph) {
    applyParallelismOverrides(jobGraph);
    log.info("提交作业 '{}' ({})。", jobGraph.getName(), jobGraph.getJobID());

    // 标记为待处理作业
    submittedAndWaitingTerminationJobIDs.add(jobGraph.getJobID());
    
    // 持久化并运行作业
    return waitForTerminatingJob(jobGraph.getJobID(), jobGraph, this::persistAndRunJob)
            .handle((ignored, throwable) -> handleTermination(jobGraph.getJobID(), throwable))
            .thenCompose(Function.identity())
            .whenComplete(
                    (ignored, throwable) ->
                            // 移除已处理的作业标记
                            submittedAndWaitingTerminationJobIDs.remove(jobGraph.getJobID()));
}
```

`persistAndRunJob` 方法负责作业的持久化与 JobMasterRunner 的创建：

```java
private void persistAndRunJob(JobGraph jobGraph) throws Exception {
    // 持久化JobGraph
    jobGraphWriter.putJobGraph(jobGraph);
    // 初始化作业客户端过期时间
    initJobClientExpiredTime(jobGraph);
    // 创建并运行JobMasterRunner
    runJob(createJobMasterRunner(jobGraph), ExecutionType.SUBMISSION);
}
```

## JobMasterRunner 的创建与启动

`createJobMasterRunner` 方法通过工厂类创建 JobMasterRunner：

```java
private JobManagerRunner createJobMasterRunner(JobGraph jobGraph) throws Exception {
    Preconditions.checkState(!jobManagerRunnerRegistry.isRegistered(jobGraph.getJobID()));
    // 通过工厂类创建JobManagerRunner
    return jobManagerRunnerFactory.createJobManagerRunner(
            jobGraph,
            configuration,
            getRpcService(),
            highAvailabilityServices,
            heartbeatServices,
            jobManagerSharedServices,
            new DefaultJobManagerJobMetricGroupFactory(jobManagerMetricGroup),
            fatalErrorHandler,
            failureEnrichers,
            System.currentTimeMillis());
}
```

`runJob` 方法启动 JobMasterRunner 并完成注册：

```java
private void runJob(JobManagerRunner jobManagerRunner, ExecutionType executionType)
        throws Exception {
    // 启动JobManager
    jobManagerRunner.start();
    // 注册JobManagerRunner
    jobManagerRunnerRegistry.register(jobManagerRunner);
    
    // 处理JobMaster结束后的清理逻辑
    final JobID jobId = jobManagerRunner.getJobID();

    final CompletableFuture<CleanupJobState> cleanupJobStateFuture =
            jobManagerRunner
                    .getResultFuture()
                    .handleAsync(
                            (jobManagerRunnerResult, throwable) -> {
                                Preconditions.checkState(
                                        jobManagerRunnerRegistry.isRegistered(jobId)
                                                && jobManagerRunnerRegistry.get(jobId)
                                                        == jobManagerRunner,
                                        "运行中的作业条目必须与JobManagerRunner的生命周期绑定。");

                                if (jobManagerRunnerResult != null) {
                                    return handleJobManagerRunnerResult(
                                            jobManagerRunnerResult, executionType);
                                } else {
                                    return CompletableFuture.completedFuture(
                                            jobManagerRunnerFailed(
                                                    jobId, JobStatus.FAILED, throwable));
                                }
                            },
                            getMainThreadExecutor())
                    .thenCompose(Function.identity());

    // 作业终止后的清理
    final CompletableFuture<Void> jobTerminationFuture =
            cleanupJobStateFuture.thenCompose(
                    cleanupJobState ->
                            removeJob(jobId, cleanupJobState)
                                    .exceptionally(
                                            throwable ->
                                                    logCleanupErrorWarning(jobId, throwable)));

    // 处理未捕获的异常
    FutureUtils.handleUncaughtException(
            jobTerminationFuture,
            (thread, throwable) -> fatalErrorHandler.onFatalError(throwable));
    registerJobManagerRunnerTerminationFuture(jobId, jobTerminationFuture);
}
```

## JobMasterServiceLeadershipRunnerFactory

该工厂类负责构建 JobMaster，为 JobMasterRunner 启动 JobMaster 做准备：

```java
@Override
public JobManagerRunner createJobManagerRunner(
        JobGraph jobGraph,
        Configuration configuration,
        RpcService rpcService,
        HighAvailabilityServices highAvailabilityServices,
        HeartbeatServices heartbeatServices,
        JobManagerSharedServices jobManagerServices,
        JobManagerJobMetricGroupFactory jobManagerJobMetricGroupFactory,
        FatalErrorHandler fatalErrorHandler,
        Collection<FailureEnricher> failureEnrichers,
        long initializationTimestamp)
        throws Exception {

    checkArgument(jobGraph.getNumberOfVertices() > 0, "作业不能为空");

    // 创建JobMaster配置
    final JobMasterConfiguration jobMasterConfiguration =
            JobMasterConfiguration.fromConfiguration(configuration);

    // 获取作业结果存储
    final JobResultStore jobResultStore = highAvailabilityServices.getJobResultStore();

    // 获取JobManager leader选举服务
    final LeaderElection jobManagerLeaderElection =
            highAvailabilityServices.getJobManagerLeaderElection(jobGraph.getJobID());
    
    // 创建Slot池服务调度器工厂（负责资源分配）
    final SlotPoolServiceSchedulerFactory slotPoolServiceSchedulerFactory =
            DefaultSlotPoolServiceSchedulerFactory.fromConfiguration(
                    configuration, jobGraph.getJobType(), jobGraph.isDynamic());

    // 校验响应式模式与调度器类型的兼容性
    if (jobMasterConfiguration.getConfiguration().get(JobManagerOptions.SCHEDULER_MODE)
            == SchedulerExecutionMode.REACTIVE) {
        Preconditions.checkState(
                slotPoolServiceSchedulerFactory.getSchedulerType()
                        == JobManagerOptions.SchedulerType.Adaptive,
                "响应式模式需要自适应调度器");
    }

    // 注册类加载器租约
    final LibraryCacheManager.ClassLoaderLease classLoaderLease =
            jobManagerServices
                    .getLibraryCacheManager()
                    .registerClassLoaderLease(jobGraph.getJobID());

    // 获取用户代码类加载器
    final ClassLoader userCodeClassLoader =
            classLoaderLease
                    .getOrResolveClassLoader(
                            jobGraph.getUserJarBlobKeys(), jobGraph.getClasspaths())
                    .asClassLoader();

    // 创建JobMaster服务工厂
    final DefaultJobMasterServiceFactory jobMasterServiceFactory =
            new DefaultJobMasterServiceFactory(
                    jobManagerServices.getIoExecutor(),
                    rpcService,
                    jobMasterConfiguration,
                    jobGraph,
                    highAvailabilityServices,
                    slotPoolServiceSchedulerFactory,
                    jobManagerServices,
                    heartbeatServices,
                    jobManagerJobMetricGroupFactory,
                    fatalErrorHandler,
                    userCodeClassLoader,
                    failureEnrichers,
                    initializationTimestamp);

    // 创建JobMaster服务进程工厂
    final DefaultJobMasterServiceProcessFactory jobMasterServiceProcessFactory =
            new DefaultJobMasterServiceProcessFactory(
                    jobGraph.getJobID(),
                    jobGraph.getName(),
                    jobGraph.getCheckpointingSettings(),
                    initializationTimestamp,
                    jobMasterServiceFactory);

    // 返回JobMaster服务领导力运行器
    return new JobMasterServiceLeadershipRunner(
            jobMasterServiceProcessFactory,
            jobManagerLeaderElection,
            jobResultStore,
            classLoaderLease,
            fatalErrorHandler);
}
```

## JobMasterServiceLeadershipRunner

`JobMasterServiceLeadershipRunner` 继承了 `JobManagerRunner` 和 `LeaderContender`，核心逻辑在 `grantLeadership` 方法中，负责获取领导力后启动 JobMaster：

```java
@Override
public void grantLeadership(UUID leaderSessionID) {
    runIfStateRunning(
            // 启动JobMasterServiceProcess
            () -> startJobMasterServiceProcessAsync(leaderSessionID),
            "启动新的JobMasterServiceProcess");
}

@GuardedBy("lock")
private void startJobMasterServiceProcessAsync(UUID leaderSessionId) {
    sequentialOperation =
            sequentialOperation.thenCompose(
                    unused ->
                            jobResultStore
                                    .hasJobResultEntryAsync(getJobID())
                                    .thenCompose(
                                            hasJobResult -> {
                                                if (hasJobResult) {
                                                    return handleJobAlreadyDoneIfValidLeader(
                                                            leaderSessionId);
                                                } else {
                                                    // 创建新的JobMaster服务进程
                                                    return createNewJobMasterServiceProcessIfValidLeader(
                                                            leaderSessionId);
                                                }
                                            }));
    handleAsyncOperationError(sequentialOperation, "无法启动作业管理器。");
}

private CompletableFuture<Void> createNewJobMasterServiceProcessIfValidLeader(
        UUID leaderSessionId) {
    return runIfValidLeader(
            leaderSessionId,
            () ->
                    // 执行JobMaster服务进程的创建
                    ThrowingRunnable.unchecked(
                            () -> createNewJobMasterServiceProcess(leaderSessionId))
                    .run(),
            "创建新的JobMaster服务进程");
}

@GuardedBy("lock")
private void createNewJobMasterServiceProcess(UUID leaderSessionId) throws FlinkException {
    Preconditions.checkState(jobMasterServiceProcess.closeAsync().isDone());

    LOG.info(
            "{} 为作业 {} 授予领导力（leader id: {}）。创建新的{}。",
            getClass().getSimpleName(),
            getJobID(),
            leaderSessionId,
            JobMasterServiceProcess.class.getSimpleName());
    
    // 创建JobMaster服务进程
    jobMasterServiceProcess = jobMasterServiceProcessFactory.create(leaderSessionId);
    
    // 配置异步回调逻辑
    forwardIfValidLeader(
            leaderSessionId,
            jobMasterServiceProcess.getJobMasterGatewayFuture(),
            jobMasterGatewayFuture,
            "来自JobMasterServiceProcess的JobMasterGatewayFuture");
    forwardResultFuture(leaderSessionId, jobMasterServiceProcess.getResultFuture());
    confirmLeadership(leaderSessionId, jobMasterServiceProcess.getLeaderAddressFuture());
}
```

## 最终的 JobMaster 创建

`DefaultJobMasterServiceProcess` 和 `DefaultJobMasterServiceFactory` 完成 JobMaster 的最终创建：

```java
// DefaultJobMasterServiceProcess 构造方法
public DefaultJobMasterServiceProcess(
        JobID jobId,
        UUID leaderSessionId,
        JobMasterServiceFactory jobMasterServiceFactory,
        Function<Throwable, ArchivedExecutionGraph> failedArchivedExecutionGraphFactory) {
    this.jobId = jobId;
    this.leaderSessionId = leaderSessionId;
    // 创建JobMasterService
    this.jobMasterServiceFuture =
            jobMasterServiceFactory.createJobMasterService(leaderSessionId, this);

    // 处理创建结果
    jobMasterServiceFuture.whenComplete(
            (jobMasterService, throwable) -> {
                if (throwable != null) {
                    final JobInitializationException jobInitializationException =
                            new JobInitializationException(
                                    jobId, "无法启动JobMaster。", throwable);

                    LOG.debug(
                            "作业 {} 在leader id {} 下的JobMasterService初始化失败。",
                            jobId,
                            leaderSessionId,
                            jobInitializationException);

                    resultFuture.complete(
                            JobManagerRunnerResult.forInitializationFailure(
                                    new ExecutionGraphInfo(
                                            failedArchivedExecutionGraphFactory.apply(
                                                    jobInitializationException)),
                                    jobInitializationException));
                } else {
                    registerJobMasterServiceFutures(jobMasterService);
                }
            });
}

// DefaultJobMasterServiceFactory 创建JobMaster
@Override
public CompletableFuture<JobMasterService> createJobMasterService(
        UUID leaderSessionId, OnCompletionActions onCompletionActions) {

    return CompletableFuture.supplyAsync(
            FunctionUtils.uncheckedSupplier(
                    () -> internalCreateJobMasterService(leaderSessionId, onCompletionActions)),
            executor);
}

private JobMasterService internalCreateJobMasterService(
        UUID leaderSessionId, OnCompletionActions onCompletionActions) throws Exception {

    // 创建JobMaster实例
    final JobMaster jobMaster =
            new JobMaster(
                    rpcService,
                    JobMasterId.fromUuidOrNull(leaderSessionId),
                    jobMasterConfiguration,
                    ResourceID.generate(),
                    jobGraph,
                    haServices,
                    slotPoolServiceSchedulerFactory,
                    jobManagerSharedServices,
                    heartbeatServices,
                    jobManagerJobMetricGroupFactory,
                    onCompletionActions,
                    fatalErrorHandler,
                    userCodeClassloader,
                    shuffleMaster,
                    lookup ->
                            new JobMasterPartitionTrackerImpl(
                                    jobGraph.getJobID(), shuffleMaster, lookup),
                    new DefaultExecutionDeploymentTracker(),
                    DefaultExecutionDeploymentReconciler::new,
                    BlocklistUtils.loadBlocklistHandlerFactory(
                            jobMasterConfiguration.getConfiguration()),
                    failureEnrichers,
                    initializationTimestamp);
    
    // 启动JobMaster
    jobMaster.start();

    return jobMaster;
}
```

JobMaster 创建后，会与 ResourceManager 建立通信并协调资源分配，后续将详细解析这部分逻辑。
