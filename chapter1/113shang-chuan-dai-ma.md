# 上传代码包及数据包至KS3

## 上传代码包

将我们的代码包上传至ks3中，如图所示：

## ![](/assets/upload_code_pkg.png)

## 上传数据包

KDL支持训练作业使用KS3上的训练、测试数据。**支持加密数据读取；对压缩文件支持不足，需要用户在作业代码中自行解压**。

以mnist示例的训练、测试数据为例

### 控制台上传作数据

如图所示，上传至某个目录并将访问控制设置为公开：

![](/assets/upload_mnist_data.png)

### 使用KS3 API上传数据

#### 安装KS3 python sdk

请参考KS3 Python SDK文档[https://docs.ksyun.com/read/latest/65/\_book/sdk/python.html](https://docs.ksyun.com/read/latest/65/_book/sdk/python.html)

#### 上传非加密数据

```bash
[]# ls input_data/
t10k-images-idx3-ubyte.gz  t10k-labels-idx1-ubyte.gz  train-images-idx3-ubyte.gz  train-labels-idx1-ubyte.gz
```

```
vim upload_mnist_data.py
```

```py
from ks3.connection import Connection as Ks3Connection
import base64

ak = "..."
sk = "..." 
host = "ks3-cn-beijing.ksyun.com"
bucket_name = "..."

def test_put_mnist_data():
    filenames = ["t10k-images-idx3-ubyte.gz","t10k-labels-idx1-ubyte.gz",
                 "train-images-idx3-ubyte.gz","train-labels-idx1-ubyte.gz"]
    obj_prefix = "tf-1.0/mnist/input_data/"
    local_file_prefix = "./input_data/"
    try:
        connection = Ks3Connection(ak,sk,host=host)
        bucket = connection.get_bucket(bucket_name)
        for filename in filenames:
            key = bucket.new_key(obj_prefix+filename) 
            response = key.set_contents_from_filename(local_file_prefix+filename)
            print response.status,response.msg,response.read() 
    except Exception as e:
        print(e)
if __name__ == '__main__':
    test_put_mnist_data()
```

上传

```
python upload_minist_data.py
```

#### 上传加密数据

ks3对上传加密数据提供秘钥托管、自定义秘钥两种方式，需要在上传时提供相应header。

##### 秘钥托管

```
vim upload_mnist_data_with_hosting_encryption.py
```

代码如下：

```py
from ks3.connection import Connection as Ks3Connection
import base64

ak = "..."
sk = "..." 
host = "ks3-cn-beijing.ksyun.com"
bucket_name = "..."
filenames = ["t10k-images-idx3-ubyte.gz","t10k-labels-idx1-ubyte.gz",
    "train-images-idx3-ubyte.gz","train-labels-idx1-ubyte.gz"]
obj_prefix = "tf-1.0/mnist/hosting_encryption_input_data/"
local_file_prefix = "./input_data/"

headers = {"x-kss-server-side-encryption":"AES256"}

def test_post_with_hosting_encryption():
    try:
        connection = Ks3Connection(ak,sk,host=host)
        bucket = connection.get_bucket(bucket_name)
        for filename in filenames:
            key = bucket.new_key(obj_prefix+filename) 
            response = key.set_contents_from_filename(local_file_prefix+filename,
                        headers=headers)
            print response.status,response.msg,response.read() 
    except Exception as e:
        print e 

def test_get_with_header():

if __name__ == '__main__':
    test_post_with_hosting_encryption()
```

##### 自定义秘钥

```
vim upload_mnist_data_with_custom_encryption.py
```

代码如下：

```

```



