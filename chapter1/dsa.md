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

### 引入依赖

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

```
长时间运行的训练任务需要有失败处理策略，可以使用Tensorflow提供的checkpoint功能在合适的时机对作业进行checkpoint,以确保重新启动作业时可以继续运行而不是重新运行，KDL支持用户将checkpoint数据写入ks3文件系统：
```

```py
saver = tf.train.Saver()
with tf.Session() as sess:
      sess.run(init_op)
      ...
      saver.save(sess, FLAGS.checkpoint_path + '/model.ckpt', global_step=i)
```

### Export Model

训练作业运行完成后我们需要将模型数据导出至ks3文件系统，并启动模型服务对外提供模型预测。具体请参考“模型服务”具体章节。

## linear训练任务完整代码

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
from tensorflow.python.util import compat

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

def main():
  if tf.gfile.Exists(FLAGS.output_path):
    tf.gfile.DeleteRecursively(FLAGS.output_path)
  tf.gfile.MakeDirs(FLAGS.checkpoint_path)
  # Create train data
  train_X = np.linspace(-1, 1, 100)
  train_Y = 2 * train_X + 10.33
  learning_rate = FLAGS.learning_rate
  start_training_time = datetime.datetime.now()

  print("Use the optimizer: {}".format(FLAGS.optimizer))
  if FLAGS.optimizer == "sgd":
    optimizer = tf.train.GradientDescentOptimizer(learning_rate)
  elif FLAGS.optimizer == "adadelta":
    optimizer = tf.train.AdadeltaOptimizer(learning_rate)
  elif FLAGS.optimizer == "adagrad":
    optimizer = tf.train.AdagradOptimizer(learning_rate)
  elif FLAGS.optimizer == "adam":
    optimizer = tf.train.AdamOptimizer(learning_rate)
  elif FLAGS.optimizer == "ftrl":
    optimizer = tf.train.FtrlOptimizer(learning_rate)
  elif FLAGS.optimizer == "rmsprop":
    optimizer = tf.train.RMSPropOptimizer(learning_rate)
  else:
    print("Unknow optimizer: {}, exit now".format(FLAGS.optimizer))
    exit(1)
    
  # Run standalone training
  if os.environ.get('TF_CONFIG', "") == "":

    # Define the model
    keys_placeholder = tf.placeholder(tf.int32, shape=[None, 1])
    keys = tf.identity(keys_placeholder)
    X = tf.placeholder("float", shape=[None, 1])
    Y = tf.placeholder("float", shape=[None, 1])
    w = tf.Variable(0.0, name="weight")
    b = tf.Variable(0.0, name="bias")
    global_step = tf.Variable(0, name='global_step', trainable=False)
    loss = tf.reduce_sum(tf.square(Y - tf.multiply(X, w) - b))
    train_op = optimizer.minimize(loss, global_step=global_step)
    predict_op = tf.multiply(X, w) + b
    tf.summary.scalar('loss', loss)
    tf.summary.scalar('training/hptuning/metric', loss)
    summary_op = tf.summary.merge_all()
    init_op = tf.global_variables_initializer()
    saver = tf.train.Saver()

    with tf.Session() as sess:
      sess.run(init_op)
      print("Save tensorboard files into: {}".format(FLAGS.output_path))
      writer = tf.summary.FileWriter(FLAGS.output_path, sess.graph)

      print("Run training with epoch number: {}".format(FLAGS.max_epochs))
      for i in range(FLAGS.max_epochs):
        for (x, y) in zip(train_X, train_Y):
          x = np.array([[x]])
          y = np.array([[y]])
          sess.run(train_op, feed_dict={X: x, Y: y})

        if i % FLAGS.checkpoint_period == 0:
          x = np.array([[train_X[0]]])
          y = np.array([[train_Y[0]]])
          summary_value, loss_value, step = sess.run(
              [summary_op, loss, global_step],
              feed_dict={X: x,
                         Y: y})
          writer.add_summary(summary_value, step)
          print("Epoch: {}, loss: {}".format(i, loss_value))

          #checkpoint
          saver.save(sess, FLAGS.checkpoint_path + '/model.ckpt', global_step=i)
      #finish SummaryWriter
      writer.close()
      
      end_training_time = datetime.datetime.now()
      print("[{}] End of standalone training.".format(end_training_time -
                                                      start_training_time))
      print("Get the model, w: {}, b: {}".format(sess.run(w), sess.run(b)))
      #export_inputs_signature = {"keys": keys_placeholder, "X": X}
      #export_outputs_signature = {"keys": keys, "predict": predict_op}
      #export_model(sess, export_inputs_signature, export_outputs_signature)
            # Export model
      export_path_base = FLAGS.model_path
      export_path = os.path.join(
          compat.as_bytes(export_path_base),
          compat.as_bytes(str(FLAGS.model_version)))
      print 'Exporting trained model to', export_path
      if tf.gfile.Exists(export_path):
        tf.gfile.DeleteRecursively(export_path)
      builder = saved_model_builder.SavedModelBuilder(export_path)

      # Build the signature_def_map.
      regression_inputs = utils.build_tensor_info(X)
      regression_outputs = utils.build_tensor_info(predict_op)

      regression_signature = signature_def_utils.build_signature_def(
          inputs={signature_constants.REGRESS_INPUTS: regression_inputs},
          outputs={
              signature_constants.REGRESS_OUTPUTS: regression_outputs
          },
          method_name=signature_constants.REGRESS_METHOD_NAME)

      tensor_info_x = utils.build_tensor_info(X)
      tensor_info_y = utils.build_tensor_info(predict_op)
 
      prediction_signature = signature_def_utils.build_signature_def(
          inputs={'x': tensor_info_x},
          outputs={'y': tensor_info_y},
          method_name=signature_constants.PREDICT_METHOD_NAME)

      legacy_init_op = tf.group(tf.tables_initializer(), name='legacy_init_op')
      builder.add_meta_graph_and_variables(
          sess, [tag_constants.SERVING],
          signature_def_map={
              'predict_y':
                  prediction_signature,
              signature_constants.DEFAULT_SERVING_SIGNATURE_DEF_KEY:
                        regression_signature,
      },
      legacy_init_op=legacy_init_op)

    builder.save()

    print 'Done exporting!'

if __name__ == "__main__":
  main()
```



