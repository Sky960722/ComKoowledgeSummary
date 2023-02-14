# Linux的使用者管理

## 总结

1. 用户标识符：UID与GID

   1. 文件和目录只会UID和GID，通过/etc/passwd与/etc/group，找到对应的UID与GID的账号与组名
   2. /etc/passwd,/etc/shadow,/etc/group三个文件分别存放账号，用户组信息

2. 账号管理

   ~~~
   id：查询相关的UID/GID信息
   useradd:增加用户
   passwd：设置密码
   groupadd：新增用户组
   groupdel：删除用户组
   ~~~

3. 用户身份切换

   1. su：最简单的身份切换命令，可以进行任何身份的切换
   2. sudo：当前用户以其他用户身份执行命令（通常是root）（需要对/etc/sudoers进行配置）

   ~~~
   # %wheel        ALL=(ALL)       NOPASSWD: ALL
   这个就是配置内容，使用sudo命令时，不用输入密码
   ~~~

   
