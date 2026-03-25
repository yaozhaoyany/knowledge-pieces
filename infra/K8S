
这是一个价值连城的排障方法论问题！ 掌握了这套排查流程，你在公司里就是救火队长级别的存在。

K8s 的排障有一套标准的**"由外到内、逐层剥洋葱"**黄金流程：

第一层：看全局概览（发现哪里有问题）
bash
# 看所有 Pod 的状态，第一时间锁定"不正常"的那些
kubectl get pods -n database
# STATUS 列的常见异常信号：
# Pending        → 调度器找不到合适的机器放它（资源不够或者 Node 有 Taint）
# ContainerCreating → 正在拉镜像或挂载磁盘
# ImagePullBackOff  → 镜像拉不到（就是你刚才碰到的）
# CrashLoopBackOff  → 容器启动后立刻崩溃，反复重启（通常是代码或配置有 bug）
# OOMKilled         → 内存爆了被 K8s 强杀
# Running           → 正常！
第二层：看详细事件（定位具体原因）
bash
# 针对有问题的那个 Pod，看它的完整生命线（你已经会的 describe）
kubectl describe pod postgresql-primary-0 -n database
# 重点看最下方的 Events 区域！那里按时间顺序记录了 K8s 为这个 Pod 做的每一件事：
# - Scheduled → 调度成功
# - Pulling   → 正在拉镜像
# - Failed    → 拉镜像/启动失败（关键错误信息在这里）
# - BackOff   → 进入退避重试
第三层：看容器日志（代码级排障）
bash
# 如果 Pod 状态是 CrashLoopBackOff（容器能启动但立刻崩溃），说明问题出在应用内部
# 看容器自己打的日志（相当于看 Java 的 System.out 或 log4j 输出）
kubectl logs postgresql-primary-0 -n database
# 如果 Pod 已经崩溃了，看"上一次死之前"的日志
kubectl logs postgresql-primary-0 -n database --previous
第四层：看 Helm 层面（Chart 配置有没有渲染出问题）
bash
# 查看 Helm Release 的状态和历史（你刚才做的）
helm list -n database
helm history postgresql -n database
# 看 Helm 实际渲染出来的 K8s YAML 长什么样（排面翻倍的神器！）
# 如果怀疑是参数传错了，这条命令能直接看到 Helm 最终生成并提交给 K8s 的完整 YAML
helm get manifest postgresql -n database
第五层：看节点层面（底层资源够不够）
bash
# 如果 Pod 一直 Pending，可能是 Node 没资源了
kubectl describe node middleware-practice-cluster-worker
# 重点看：
# Allocatable:  cpu/memory → 这台机器总共有多少资源
# Allocated resources: → 已经被占用了多少
# 如果 memory 已经被占满了，新 Pod 就 Pending 排队等不到位置
第六层：看集群事件流（上帝视角）
bash
# 看整个集群（或某个 Namespace）最近发生的所有事件，按时间排序
kubectl get events -n database --sort-by='.lastTimestamp'
# 这相当于 K8s 的"监控摄像头回放"，能看到所有资源的创建、报错、重试记录
速查表（遇到问题先看哪一层）
Pod 状态	从哪一层开始查
Pending	第五层（Node 资源不够或 PVC 绑定失败）
ImagePullBackOff	第二层（describe 看 Events，通常是镜像名/Tag 写错或网络不通）
CrashLoopBackOff	第三层（logs 看应用日志，通常是配置文件错误或数据库连接不上）
Running 但业务异常	第三层（logs 看业务报错） + 第四层（Helm manifest 看配置是否正确）
把这套"六层剥洋葱法"刻进肌肉记忆，未来遇到任何 K8s 抽风，你都能在 5 分钟内精准定位到根因。

现在去跑 terraform init -upgrade && terraform apply 吧，让我们看到 Pod 全绿！

























