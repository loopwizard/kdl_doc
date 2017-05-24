# 简介

KDL提供的命令行工具，可以满足用户在命令行进行训练任务、模型服务、tensorboard的创建及查询功能等功能。

**注意由于Kubernetes命名的限制，所有models、tensorboards资源名称必须符合\[a-z0-9\]\(\[-a-z0-9\]\*\[a-z0-9\]正则表达式，不可以用下划线，不可以超过25个字符。**

# 安装命令行工具

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

# 初始化环境

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

# 命令说明

## cloudml命令

可以通过cloudml -h查看支持的命令：

| 命令名 | 描述 |
| :--- | :--- |
| init | 初始化环境变量 |
| org\_id | 查询当前用户的组织结构id |
| flavors | 查询kdl |

# 提交训练任务



