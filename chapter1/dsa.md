# 本地编写训练任务代码

开始编写训练任务代码，代码规范与社区版Tensorflow一致并符合Python模块标准，不依赖额外的KDL API。

第一个示例liner

## 创建模块

```bash
    mkdir trainer
    touch trainer/__init__.py
    touch trainer/task.py
```

## 编写代码

```bash
     vim trainer/task.py
```

###  引入依赖

```py
#!/usr/bin/env python

import datetime
import json
import numpy as np
import os
import sys
import tensorflow as tf
from tensorflow.contrib.session_bundle import exporter
from tensorflow.python.saved_model import builder as saved_model_builder
from tensorflow.python.saved_model import signature_constants
from tensorflow.python.saved_model import signature_def_utils
from tensorflow.python.saved_model import tag_constants
from tensorflow.python.saved_model import utils
from tensorflow.python.util import compact
```

### 用户自定义参数

```py
flags = tf.app.flags
flags.DEFINE_integer('max_epochs', 100, 'Number of steps to run trainer.')
flags.DEFINE_string('checkpoint_path', './checkpoint/',
                    'The checkpoint directory')
flags.DEFINE_string("output_path", "./tensorboard/",
                    "indicates training output")
flags.DEFINE_integer('checkpoint_period', 100,
                     'Number of epochs to save checkpoint.')
flags.DEFINE_integer('model_version', 1, 'Version number of the model.')
flags.DEFINE_string('model_path', './model/', 'The model directory')
flags.DEFINE_float('learning_rate', 0.01, 'Initial learning rate.')
flags.DEFINE_string("optimizer", "sgd", "Optimizer to train")
FLAGS = flags.FLAGS
```

**我们定义的参数将会在KDL控制台创建训练任务时传入，并应用到我们的训练任务代码中。**如checkpoint_path，_指定了我们的训练任务在运行过程中checkpoint时使用的文件目录；outputpath参数用以输出summary信息;model\_path以及model\_version指定了训练完成后输出模型数据的文件目录及模型版本。

我们的训练任务在kdl中作为标准Python模块被引入与执行，**用户自定义参数在执行时被设置**：

```py
...
os.chdir(module_dir)
subprocess.call("python ./setup.py install", shell=True)
...
runpy.run_module(module_name, run_name="__main__")
...
```

当然，也可以在本地执行：

```
python -m trainer.task
```

### 输出Summary信息

我们可以通过Tensorflow标准方式将summary信息输出至本地磁盘或者ks3文件系统：

```py
 tf.summary.scalar('loss', loss)
 tf.summary.scalar('training/hptuning/metric', loss)
 summary_op = tf.summary.merge_all()
 init_op = tf.global_variables_initializer()
 with tf.Session() as sess:
    sess.run(init_op)
    print("Save tensorboard files into: {}".format(FLAGS.output_path))
    writer = tf.summary.FileWriter(FLAGS.output_path, sess.graph)
    summary_value, loss_value, step = sess.run(
              [summary_op, loss, global_step],
              feed_dict={X: x,
                         Y: y})
    writer.add_summary(summary_value, step)
```

### Checkpoint

        长时间运行的训练任务需要有失败处理策略，可以使用Tensorflow提供的checkpoint功能在合适的时机对作业进行checkpoint,以确保重新启动作业时可以继续运行而不是重新运行，KDL支持用户将checkpoint数据写入ks3文件系统：

```py
saver = tf.train.Saver()
with tf.Session() as sess:
      sess.run(init_op)
      ...
      saver.save(sess, FLAGS.checkpoint_path + '/model.ckpt', global_step=i)
```

### Export Model

        训练作业运行完成后我们需要将模型数据导出至ks3文件系统，并启动模型服务对外提供模型预测。具体请参考“模型服务”具体章节。

