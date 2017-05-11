# 线性回归模型导出示例

作业训练时，将模型导出的代码如下

```py
# 创建builder
builder = saved_model_builder.SavedModelBuilder(export_path)

# 创建signature_def_map
regression_inputs = utils.build_tensor_info(X)
regression_outputs = utils.build_tensor_info(predict_op)

regression_signature = signature_def_utils.build_signature_def(
    inputs={signature_constants.REGRESS_INPUTS: regression_inputs}, 
    outputs={signature_constants.REGRESS_OUTPUTS: regression_outputs},
    method_name=signature_constants.REGRESS_METHOD_NAME)

tensor_info_x = utils.build_tensor_info(X)
tensor_info_y = utils.build_tensor_info(predict_op)

prediction_signature = signature_def_utils.build_signature_def( 
    inputs={'x': tensor_info_x}, 
    outputs={'y': tensor_info_y}, 
    method_name=signature_constants.PREDICT_METHOD_NAME)

legacy_init_op = tf.group(tf.tables_initializer(), name='legacy_init_op')

# add_meta_graph_and_variables
builder.add_meta_graph_and_variables(
    sess, [tag_constants.SERVING], 
    signature_def_map={
        'predict_y': prediction_signature, 
        signature_constants.DEFAULT_SERVING_SIGNATURE_DEF_KEY: regression_signature,
    }, 
    legacy_init_op=legacy_init_op)

builder.save()
```











