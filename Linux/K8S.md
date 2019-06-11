Linux 容器基础的两种技术：

Namespace 和 Cgroup，容器的本质是一种特殊的进程。

Namespace作用是隔离，进程只能看到该Namespace内的世界，而Cgroups的作用是限制。

Kubernetes Master:

API Server 、Scheduler、Controller、etcd