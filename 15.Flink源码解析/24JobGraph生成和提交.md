# JobGraph 的生成与提交流程解析

## 什么是 JobGraph

JobGraph 是 Flink 中 StreamGraph 进一步转换生成的逻辑执行计划，主要负责将 StreamGraph 中的算子进行合并优化，但尚未形成最终的物理执行计划（物理执行计划由 JobMaster 负责生成）。本文将详细解析从 StreamGraph 到 JobGraph 的转换过程及 JobGraph 的提交机制。

## StreamingJobGraphGenerator：StreamGraph 到 JobGraph 的转换器

`StreamingJobGraphGenerator`是专门用于将`StreamGraph`转换为`JobGraph`的核心类，其核心入口方法如下：

```java
public static JobGraph createJobGraph(
        ClassLoader userClassLoader, StreamGraph streamGraph, @Nullable JobID jobID) {
    // 为每个作业创建一个线程池用于序列化操作
    final ExecutorService serializationExecutor =
            Executors.newFixedThreadPool(
                    Math.max(
                            1,
                            Math.min(
                                    Hardware.getNumberCPUCores(),
                                    streamGraph.getExecutionConfig().getParallelism())),
                    new ExecutorThreadFactory("flink-operator-serialization-io"));
    try {
        return new StreamingJobGraphGenerator(
                        userClassLoader, streamGraph, jobID, serializationExecutor)
                .createJobGraph();
    } finally {
        serializationExecutor.shutdown();
    }
}
```

### createJobGraph () 方法详解

`createJobGraph()`方法是转换过程的核心实现，主要包含以下关键步骤：

```java
private JobGraph createJobGraph() {
        preValidate();
        jobGraph.setJobType(streamGraph.getJobType());
        jobGraph.setDynamic(streamGraph.isDynamic());

        jobGraph.enableApproximateLocalRecovery(
                streamGraph.getCheckpointConfig().isApproximateLocalRecoveryEnabled());

        // Generate deterministic hashes for the nodes in order to identify them across
        // submission iff they didn't change.
        Map<Integer, byte[]> hashes =
                defaultStreamGraphHasher.traverseStreamGraphAndGenerateHashes(streamGraph);

        // Generate legacy version hashes for backwards compatibility
        List<Map<Integer, byte[]>> legacyHashes = new ArrayList<>(legacyStreamGraphHashers.size());
        for (StreamGraphHasher hasher : legacyStreamGraphHashers) {
            legacyHashes.add(hasher.traverseStreamGraphAndGenerateHashes(streamGraph));
        }

        setChaining(hashes, legacyHashes);

        if (jobGraph.isDynamic()) {
            setVertexParallelismsForDynamicGraphIfNecessary();
        }

        // Note that we set all the non-chainable outputs configuration here because the
        // "setVertexParallelismsForDynamicGraphIfNecessary" may affect the parallelism of job
        // vertices and partition-reuse
        final Map<Integer, Map<StreamEdge, NonChainedOutput>> opIntermediateOutputs =
                new HashMap<>();
        setAllOperatorNonChainedOutputsConfigs(opIntermediateOutputs);
        setAllVertexNonChainedOutputsConfigs(opIntermediateOutputs);

        setPhysicalEdges();

        markSupportingConcurrentExecutionAttempts();

        validateHybridShuffleExecuteInBatchMode();

        setSlotSharingAndCoLocation();

        setManagedMemoryFraction(
                Collections.unmodifiableMap(jobVertices),
                Collections.unmodifiableMap(vertexConfigs),
                Collections.unmodifiableMap(chainedConfigs),
                id -> streamGraph.getStreamNode(id).getManagedMemoryOperatorScopeUseCaseWeights(),
                id -> streamGraph.getStreamNode(id).getManagedMemorySlotScopeUseCases());

        configureCheckpointing();

        jobGraph.setSavepointRestoreSettings(streamGraph.getSavepointRestoreSettings());

        final Map<String, DistributedCache.DistributedCacheEntry> distributedCacheEntries =
                JobGraphUtils.prepareUserArtifactEntries(
                        streamGraph.getUserArtifacts().stream()
                                .collect(Collectors.toMap(e -> e.f0, e -> e.f1)),
                        jobGraph.getJobID());

        for (Map.Entry<String, DistributedCache.DistributedCacheEntry> entry :
                distributedCacheEntries.entrySet()) {
            jobGraph.addUserArtifact(entry.getKey(), entry.getValue());
        }

        // set the ExecutionConfig last when it has been finalized
        try {
            jobGraph.setExecutionConfig(streamGraph.getExecutionConfig());
        } catch (IOException e) {
            throw new IllegalConfigurationException(
                    "Could not serialize the ExecutionConfig."
                            + "This indicates that non-serializable types (like custom serializers) were registered");
        }
        jobGraph.setJobConfiguration(streamGraph.getJobConfiguration());

        addVertexIndexPrefixInVertexName();

        setVertexDescription();

        // Wait for the serialization of operator coordinators and stream config.
        try {
            FutureUtils.combineAll(
                            vertexConfigs.values().stream()
                                    .map(
                                            config ->
                                                    config.triggerSerializationAndReturnFuture(
                                                            serializationExecutor))
                                    .collect(Collectors.toList()))
                    .get();

            waitForSerializationFuturesAndUpdateJobVertices();
        } catch (Exception e) {
            throw new FlinkRuntimeException("Error in serialization.", e);
        }

        if (!streamGraph.getJobStatusHooks().isEmpty()) {
            jobGraph.setJobStatusHooks(streamGraph.getJobStatusHooks());
        }

        return jobGraph;
    }
```

1. **生成算子哈希 ID**
   为 StreamGraph 中的算子生成确定性哈希，用于跨提交识别未变更的算子

   ```java
   Map<Integer, byte[]> hashes =
           defaultStreamGraphHasher.traverseStreamGraphAndGenerateHashes(streamGraph);
   
   // 生成遗留版本的哈希以保证向后兼容性
   List<Map<Integer, byte[]>> legacyHashes = new ArrayList<>(legacyStreamGraphHashers.size());
   for (StreamGraphHasher hasher : legacyStreamGraphHashers) {
       legacyHashes.add(hasher.traverseStreamGraphAndGenerateHashes(streamGraph));
   }
   ```

2. **算子链化（Chaining）**
   将 StreamGraph 中的 Operator 连接形成算子链，优化执行效率

   ```java
   setChaining(hashes, legacyHashes);
   ```

3. **动态图并行度设置（如需要）**

   ```java
   if (jobGraph.isDynamic()) {
       setVertexParallelismsForDynamicGraphIfNecessary();
   }
   ```

4. **输出配置与物理边设置**

   ```java
   final Map<Integer, Map<StreamEdge, NonChainedOutput>> opIntermediateOutputs =
           new HashMap<>();
   
   // 设置非链式输出配置
   setAllOperatorNonChainedOutputsConfigs(opIntermediateOutputs);
   setAllVertexNonChainedOutputsConfigs(opIntermediateOutputs);
   
   // 设置物理连接边
   setPhysicalEdges();
   ```

5. **其他关键配置**

   ```java
   markSupportingConcurrentExecutionAttempts();
   validateHybridShuffleExecuteInBatchMode();
   setSlotSharingAndCoLocation();
   setManagedMemoryFraction(...);
   configureCheckpointing();
   jobGraph.setSavepointRestoreSettings(streamGraph.getSavepointRestoreSettings());
   ```

6. **分布式缓存设置**

   ```java
   final Map<String, DistributedCache.DistributedCacheEntry> distributedCacheEntries =
           JobGraphUtils.prepareUserArtifactEntries(
                   streamGraph.getUserArtifacts().stream()
                           .collect(Collectors.toMap(e -> e.f0, e -> e.f1)),
                   jobGraph.getJobID());
   
   for (Map.Entry<String, DistributedCache.DistributedCacheEntry> entry :
           distributedCacheEntries.entrySet()) {
       jobGraph.addUserArtifact(entry.getKey(), entry.getValue());
   }
   ```

7. **执行配置与序列化**

   ```java
   jobGraph.setExecutionConfig(streamGraph.getExecutionConfig());
   jobGraph.setJobConfiguration(streamGraph.getJobConfiguration());
   
   // 等待算子协调器和流配置的序列化完成
   FutureUtils.combineAll(
           vertexConfigs.values().stream()
                   .map(
                           config ->
                                   config.triggerSerializationAndReturnFuture(
                                           serializationExecutor))
                   .collect(Collectors.toList()))
           .get();
   
   waitForSerializationFuturesAndUpdateJobVertices();
   ```

## JobGraph 的提交过程

### AbstractSessionClusterExecutor：会话集群执行器

生成的 JobGraph 通过`AbstractSessionClusterExecutor`客户端进行提交：

```java
public CompletableFuture<JobClient> execute(
        @Nonnull final Pipeline pipeline,
        @Nonnull final Configuration configuration,
        @Nonnull final ClassLoader userCodeClassloader)
        throws Exception {
    // 将Pipeline转换为JobGraph
    final JobGraph jobGraph =
            PipelineExecutorUtils.getJobGraph(pipeline, configuration, userCodeClassloader);

    try (final ClusterDescriptor<ClusterID> clusterDescriptor =
            clusterClientFactory.createClusterDescriptor(configuration)) {
        final ClusterID clusterID = clusterClientFactory.getClusterId(configuration);
        checkState(clusterID != null);

        final ClusterClientProvider<ClusterID> clusterClientProvider =
                clusterDescriptor.retrieve(clusterID);
        ClusterClient<ClusterID> clusterClient = clusterClientProvider.getClusterClient();
        return clusterClient
                .submitJob(jobGraph)
                .thenApplyAsync(
                        FunctionUtils.uncheckedFunction(
                                jobId -> {
                                    ClientUtils.waitUntilJobInitializationFinished(
                                            () -> clusterClient.getJobStatus(jobId).get(),
                                            () -> clusterClient.requestJobResult(jobId).get(),
                                            userCodeClassloader);
                                    return jobId;
                                }))
                .thenApplyAsync(
                        jobID ->
                                (JobClient)
                                        new ClusterClientJobClientAdapter<>(
                                                clusterClientProvider,
                                                jobID,
                                                userCodeClassloader))
                .whenCompleteAsync((ignored1, ignored2) -> clusterClient.close());
    }
}
```

### RestClusterClient：REST 客户端提交实现

`RestClusterClient`负责将客户端的 JobGraph 和相关 jar 包提交到集群，集群中的`JobSubmitHandler`类负责接收这些数据：

```java
@Override
public CompletableFuture<JobID> submitJob(@Nonnull JobGraph jobGraph) {
    // 序列化JobGraph到临时文件
    CompletableFuture<java.nio.file.Path> jobGraphFileFuture =
            CompletableFuture.supplyAsync(
                    () -> {
                        try {
                            final java.nio.file.Path jobGraphFile =
                                    Files.createTempFile(
                                            "flink-jobgraph-" + jobGraph.getJobID(), ".bin");
                            try (ObjectOutputStream objectOut =
                                    new ObjectOutputStream(
                                            Files.newOutputStream(jobGraphFile))) {
                                objectOut.writeObject(jobGraph);
                            }
                            return jobGraphFile;
                        } catch (IOException e) {
                            throw new CompletionException(
                                    new FlinkException("Failed to serialize JobGraph.", e));
                        }
                    },
                    executorService);

    // 准备提交请求
    CompletableFuture<Tuple2<JobSubmitRequestBody, Collection<FileUpload>>> requestFuture =
            jobGraphFileFuture.thenApply(
                    jobGraphFile -> {
                        List<String> jarFileNames = new ArrayList<>(8);
                        List<JobSubmitRequestBody.DistributedCacheFile> artifactFileNames =
                                new ArrayList<>(8);
                        Collection<FileUpload> filesToUpload = new ArrayList<>(8);

                        filesToUpload.add(
                                new FileUpload(
                                        jobGraphFile, RestConstants.CONTENT_TYPE_BINARY));

                        // 添加用户JAR包
                        for (Path jar : jobGraph.getUserJars()) {
                            jarFileNames.add(jar.getName());
                            filesToUpload.add(
                                    new FileUpload(
                                            Paths.get(jar.toUri()),
                                            RestConstants.CONTENT_TYPE_JAR));
                        }

                        // 添加分布式缓存文件
                        for (Map.Entry<String, DistributedCache.DistributedCacheEntry>
                                artifacts : jobGraph.getUserArtifacts().entrySet()) {
                            final Path artifactFilePath =
                                    new Path(artifacts.getValue().filePath);
                            try {
                                // 仅上传本地 artifacts
                                if (!artifactFilePath.getFileSystem().isDistributedFS()) {
                                    artifactFileNames.add(
                                            new JobSubmitRequestBody.DistributedCacheFile(
                                                    artifacts.getKey(),
                                                    artifactFilePath.getName()));
                                    filesToUpload.add(
                                            new FileUpload(
                                                    Paths.get(artifactFilePath.getPath()),
                                                    RestConstants.CONTENT_TYPE_BINARY));
                                }
                            } catch (IOException e) {
                                throw new CompletionException(
                                        new FlinkException(
                                                "Failed to get the FileSystem of artifact "
                                                        + artifactFilePath
                                                        + ".",
                                                e));
                            }
                        }

                        final JobSubmitRequestBody requestBody =
                                new JobSubmitRequestBody(
                                        jobGraphFile.getFileName().toString(),
                                        jarFileNames,
                                        artifactFileNames);

                        return Tuple2.of(
                                requestBody, Collections.unmodifiableCollection(filesToUpload));
                    });

    // 发送提交请求
    final CompletableFuture<JobSubmitResponseBody> submissionFuture =
            requestFuture.thenCompose(
                    requestAndFileUploads -> {
                        LOG.info(
                                "Submitting job '{}' ({}).",
                                jobGraph.getName(),
                                jobGraph.getJobID());
                        return sendRetriableRequest(
                                JobSubmitHeaders.getInstance(),
                                EmptyMessageParameters.getInstance(),
                                requestAndFileUploads.f0,
                                requestAndFileUploads.f1,
                                isConnectionProblemOrServiceUnavailable(),
                                (receiver, error) -> {
                                    if (error != null) {
                                        LOG.warn(
                                                "Attempt to submit job '{}' ({}) to '{}' has failed.",
                                                jobGraph.getName(),
                                                jobGraph.getJobID(),
                                                receiver,
                                                error);
                                    } else {
                                        LOG.info(
                                                "Successfully submitted job '{}' ({}) to '{}'.",
                                                jobGraph.getName(),
                                                jobGraph.getJobID(),
                                                receiver);
                                    }
                                });
                    });

    // 清理临时文件
    submissionFuture
            .exceptionally(ignored -> null) // 忽略错误
            .thenCompose(ignored -> jobGraphFileFuture)
            .thenAccept(
                    jobGraphFile -> {
                        try {
                            Files.delete(jobGraphFile);
                        } catch (IOException e) {
                            LOG.warn("Could not delete temporary file {}.", jobGraphFile, e);
                        }
                    });

    return submissionFuture
            .thenApply(ignore -> jobGraph.getJobID())
            .exceptionally(
                    (Throwable throwable) -> {
                        throw new CompletionException(
                                new JobSubmissionException(
                                        jobGraph.getJobID(),
                                        "Failed to submit JobGraph.",
                                        ExceptionUtils.stripCompletionException(throwable)));
                    });
}
```

## 总结

JobGraph 作为 StreamGraph 与物理执行计划之间的中间表示，在 Flink 作业执行流程中扮演着重要角色。其核心功能是对算子进行链化优化，为后续的物理执行计划生成奠定基础。

JobGraph 的生成主要由`StreamingJobGraphGenerator`完成，而提交过程则通过`AbstractSessionClusterExecutor`和`RestClusterClient`等组件实现，最终由集群的`JobSubmitHandler`接收处理。

JobGraph 的`setChaining`方法和`intermediateData`数据结构与其内部实现密切相关，而 JobMaster 会基于 JobGraph 构建真正的物理执行计划，其中中间输出过程与 NIO 和数据分区机制相关联。理解 JobGraph 的生成与提交流程，有助于深入掌握 Flink 作业的执行原理。
