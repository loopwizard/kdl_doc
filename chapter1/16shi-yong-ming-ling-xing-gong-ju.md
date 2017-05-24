* ##### [安装命令行工具](#安装命令行工具)

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



