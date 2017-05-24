## 查看作业详情

如图所示：

![](/assets/job_list.png)

# 基本信息

作业基本信息包含作业的名称、状态、参数等信息:![](/assets/job_base_info.png)

如果是超参数调优任务，还会显示超参数调优相关的参数信息，点击“参数名称”显示超参数调优结果对比情况：![](/assets/hyper_result.png)

# 作业资源使用情况

处于“运行中”的作业，用户可以查看资源利用情况:![](/assets/job_metrics.png)

# 查看作业日志

KDL提供作业日志查看功能，用户可以通过日志定位作业异常：![](/assets/job_log.png)

# 绑定tensorboard

在训练任务运行的过程中，用户可以通过tensorboard查看作业的更多信息。KDL中为作业绑定tensorboard有两种方式

1. 创建训练任务时同时创建tensorboard并绑定。
2. 单独创建tensorboard,并在训练任务完成后进行绑定。



