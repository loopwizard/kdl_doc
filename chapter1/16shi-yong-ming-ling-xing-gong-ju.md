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



