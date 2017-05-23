# 打包训练任务代码

目前KDL只支持tar.gz格式的代码包，我们可以用setuptools进行打包：

```py
cat << EOF > setup.py
import setuptools
setuptools.setup(name='trainer', version='1.0', packages=['trainer'])
EOF
```

打包：

```py
python setup.py sdist --format=gztar
```

dist目录中生成我们的代码包

```
├── dist
│   └── trainer-1.0.tar.gz
├── setup.py
├── trainer
│   ├── __init__.py
│   └── task.py
└── trainer.egg-info
    ├── dependency_links.txt
    ├── PKG-INFO
    ├── SOURCES.txt
    └── top_level.txt
```







