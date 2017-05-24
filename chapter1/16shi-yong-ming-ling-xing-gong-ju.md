# 一. 简介

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

# 五. 训练任务相关命令

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

| 参数名 | 参数全名 | 描述 | 数据类型 | 是否必填 | 举例 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| -n | --raw\_job\_name | 任务名称 | string | yes | linear\_test |
| -m | --module\_name | 任务代码的python模块名称 | string | yes | trainer.task |
| -u | --trainer\_uri | 任务代码包ks3路径 | string | yes | ks3://ai-train-demo/tf-1.0/linear/trainer-1.0.tar.gz |
| -fi | --flavor\_id | 资源套餐id | string | yes | cpu.basic |
| -f | --filename | 任务描述文件路径 | string | no | ../linear\_test.json |
| -a | --job\_args | 任务参数 | string | no | --max\_epochs 100 --optimizer sgd |
| -pc | --prepare\_command | 训练准备命令，训练任务运行前执行 | string | no | pip isntall six && apt-get install libssl-dev |
| -fc | --finish\_command | 训练清理命令，训练任务运行后执行 | string | no | rm run.log |
| -ct | --create\_tensorboard | 是否为当前任务创建tensorboard | string | no | true或false |
| -tn | --tensorboard\_name | tensorboard名称 | string | no | linear\_tensorboard |
| -ld | --logdir | tensorboard目录 | string | no | ks3://ai-train-demo/tf-1.0/linear/tensorboard/ |
| -ek | --ks3\_encryption\_key | ks3上的加密训练数据的加密秘钥,必须是长度为16/24/32的字符串 | string | no | 1234567890123456 |

2. 查看训练任务详细信息



