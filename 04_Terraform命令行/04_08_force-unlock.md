

# 1 force-unlock

手动解除状态锁。

这个命令不会修改你的基础设施，它只会删除当前工作区对应的状态锁。具体操作步骤取决于使用的Backend。本地状态文件无法被其他进程解锁。

## 1.1 用法

terraform force-unlock LOCK_ID [DIR]

参数：
- -force=true：解锁时不提示确认

需要注意的是，就像我们在状态管理篇当中介绍过的那样，每一个状态锁都有一个锁ID。Terraform为了确保我们解除正确的状态锁，所以会要求我们显式输入锁ID。

一般情况下我们不需要强制解锁，只有在Terraform异常终止，来不及解除锁时需要我们手动强制解除锁。错误地解除状态锁可能会导致状态混乱，所以请小心使用。


Das ist ein Schutzmechanismus, falls zwei parallel an diesem TF-Projekt arbeiten und einer mitten im apply steckt. 
Aber der Lock bleibt, wenn TF mal ungünstig abbricht.