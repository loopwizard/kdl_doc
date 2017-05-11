# 线性回归模型python客户端

下载线性回归模型python client示例代码：

```bash
wget https://ks3-cn-beijing.ksyun.com/ai-train-demo/tf-1.0/linear/python_predict_client.tar.gz
```

将client代码中模型服务的名字修改为2.2节中启动的模型服务的名字“demo”

```py
request.model_spec.name = 'demo'
```

运行client脚本：

```bash
python linear_client.py —server=120.92.91.217:30048 —num_tests=10
```

