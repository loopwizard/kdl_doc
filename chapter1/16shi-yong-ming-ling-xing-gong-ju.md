#  一. 简介

KDL提供的命令行工具，可以满足用户在命令行进行训练任务、模型服务、tensorboard的创建及查询功能等功能。

**注意由于Kubernetes命名的限制，所有models、tensorboards资源名称必须符合\[a-z0-9\]\(\[-a-z0-9\]\*\[a-z0-9\]正则表达式，不可以用下划线，不可以超过25个字符。**

# 二.  安装命令行工具

```bash
wget https://ks3-cn-beijing.ksyun.com/ai-infra/kdl-cli/cloud_ml_common.tar.gz
wget https://ks3-cn-beijing.ksyun.com/ai-infra/kdl-cli/cloud_ml_sdk.tar.gz
tar -zxvf cloud_ml_common.tar.gz
tar -zxvf cloud_ml_sdk.tar.gz
```

需要先安装cloud\_ml\_common再安装cloud\_ml\_sdk：

```bash
cd cloud_ml_common
sudo pip install -r requirements.txt
sudo python setup.py install

cd ../cloud_ml_sdk
sudo pip install -r requirements.txt
sudo python setup.py install
```

# 三. 初始化环境

用户需要获取金山云的access key和secret key，以及kdl的enpoint:

```
$ cloudml init
Please input access key: your_access_key
Please input secret key (will not be echoed): 
Please input cloudml endpoint[default: https://ai-beta:8000]: http://ai.beta.ksyun.com

Test access with supplied credentials? [y/N]: N

Save settings? [y/N]: y
Successfully initialize config file in path: /home/user/.config/ksc/config
```



# 四. cloudml命令

可以通过cloudml -h查看支持的命令：

| 命令名 | 描述 | 示例 |
| :--- | :--- | :--- |
| init | 初始化环境变量 | cloudml init |
| org\_id | 查询当前用户的组织结构id | cloudml org\_id |
| flavors | 查询kdl支持的资源套餐 | cloudml flavors list |
| jobs | 创建、查询模型训练任务 | cloudml jobs list |
| models | 创建、查询模型服务 | cloudml models list |
| tensorboard | 创建、查询tensorboard | cloudml tensorboard list |
| quota | 查询当前用户的quota信息 | cloudml quota list |

#  五. 训练任务相关命令

训练任务相关命令可以通过cloudml jobs -h查看。

## 1. 训练任务列表

```
cloudml jobs list
```

支持分页及按状态过滤

```
cloudml jobs list -cp 1 -pg 5 -st error
```

## 2. 提交训练任务

### 1\). 通过参数提交

```
 cloudml jobs submit -n cli_linear_test -m trainer.task -u ks3://ai-train-demo/tf-1.0/linear/trainer-1.0.tar.gz -fi cpu.basic
```

### 2\). 通过JSON文件提交

训练任务json描述文件cli\_linear\_demo.json：

```json
{
  "raw_job_name": "cli_linear_demo",
  "module_name": "trainer.task",
  "trainer_uri": "ks3://ai-train-demo/tf-1.0/linear/trainer-1.0.tar.gz",
  "job_args": "--max_epochs 100 --model_path ks3://ai-train-demo/tf-1.0/linear/model",
  "flavor_id": "cpu.basic"
}
```

提交任务：

```
cloudml jobs submit -f cli_linear_demo.json
```

### 3\). 提交训练任务命令参数说明





