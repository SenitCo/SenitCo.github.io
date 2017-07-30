---
title: Linux服务器下安装TensorFlow
date: 2017-07-20
categories: Compile & Installation 
tags: Sofeware
---
&emsp;&emsp;简单介绍在Linux服务器的个人目录下安装TensorFlow。TensorFlow的安装方式有多种，基于Pip的安装、基于Docker的安装、基于VirtualEnv的安装、基于Anaconda的安装，以及从源码编译安装，这些在[官网](https://www.tensorflow.org/install/install_linux)均有介绍，这里简单记录下基于Anaconda安装的方法。
<!-- more -->

### 安装Anaconda
&emsp;&emsp;Anaconda是一个集成许多第三方科学计算库的Python科学计算环境，Anaconda使用conda 作为自己的包管理工具，同时具有自己的计算环境，类似Virtualenv。和Virtualenv一样,不同Python工程需要的依赖包，conda将其存储在不同的地方。TensorFlow上安装的Anaconda不会对之前安装的Python包进行覆盖。
- 进入Anaconda官网[下载页面](https://www.continuum.io/downloads)，选择合适版本直接下载，或者在个人终端目录下，使用wget命令，示例如下：
```
wget https://repo.continuum.io/archive/Anaconda2-4.4.0-Linux-x86_64.sh
```
- 下载到本地后运行安装脚本
```
bash Anaconda2-4.4.0-Linux-x86_64.sh
```
- 安装完成后在~/.bashrc文件中添加环境变量
```
export PATH="/$HOME/anaconda2/bin:$PATH"
```

此外，还需在~/.zshrc文件添加相关路径，否则在后面执行conda命令时，可能会出现错误信息：zsh: command not found: conda。在.zshrc文件的 #User configuration 处追加
```
export PATH = "$PATH:$HOME/anaconda/bin"
```
添加完路径后分别执行以下命令使之生效
```
source ~/.bashrc
source ~/.zshrc
```

### 安装TensorFlow
- 创建conda环境，命名为tensorflow
```
conda create -n tensorflow﻿​
#也可指定Python版本
conda create -n tensorflow python=2.7
```
- 激活并进入创建的conda环境
```
source activate tensorflow
```
- 下载并安装TensorFlow
```
pip install --ignore-installed --upgrade tfBinaryURL﻿​ 
```
- [tfBinaryURL](https://www.tensorflow.org/install/install_linux#TF_PYTHON_URL)须根据平台环境进行选择。例如，对于Python2.7，GPU版本为CUDA8.0的平台，可安装如下版本
```
pip install --ignore-installed --upgrade \
https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.2.1-cp27-cp27mu-manylinux1_x86_64.whl 
```
- 在不确定安装版本的情况下也可直接使用如下命令安装
```
pip install tensorflow 		#CPU版本
pip install tensorflow-gpu	#GPU版本
```
- 退出虚拟环境
```
source deactivate tensorflow
```

### 测试
- 用source activate指令进入tensorflow环境，执行Python解释器
```
python
```
- 在Python环境内，逐条输入以下语句
```
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> print(sess.run(hello))
```
- 如果成功打印下面语句，说明安装成功
```
Hello, TensorFlow!
```

### 注意事项
&emsp;&emsp;安装GPU版本的TensorFlow时，需要使用NVIDIA的显卡，并安装和配置CUDA和CUDNN环境。
&emsp;&emsp;一般对于多用户使用的服务器，系统主目录下都会安装有Python解释器(甚至是多个版本)。对于大多数用户而言都不具备管理员权限，在利用pip命令安装一些Python依赖库时会失败，因此在个人目录下安装集成的Anaconda环境，可以有效地与系统自带的Python解释器隔离，前提是在.bashrc文件中添加路径
```
export PATH="$HOME/anaconda2/bin:$PATH"
```
并执行以下命令使之生效
```
source ~/.bashrc
```
这样每次执行python命令都是在个人目录下的Anaconda环境中。如果要与系统Python环境随时切换，可通过给命令起别名的方式，即别名声明alias。
```
alias python27="/usr/bin/python2.7"  	#系统Python环境
alias python36="/usr/bin/python3.6" 
alias pyana="/home/myname/anaconda2/bin/python2.7"	#个人Python环境，精确到版本路径
```
使用系统自带的Python时，执行Python27或者Python36命令即可；使用Anaconda时，执行pyana或者python命令。将上述命令添加到.bashrc文件中，这样每次开机都不需要重新输入。

conda的简单命令
```
conda create -n [name]      #创建名为name的conda环境，如tensorflow

source activate [name]  	#激活并进入创建的环境

source deactivate [name]    #退出名为name的环境，回到系统默认环境

conda remove -n [name] --all   #删除创建的conda环境 

conda info -envs    #查看所安装环境列表，创建的环境都在`~/anaconda2/envs/`目录下面

conda list      #查看已经安装的包

conda install [packagename]        #安装具体的包，加-n [name]可以安装到指定环境

conda list -n [name]      #name环境下安装了哪些包

conda update -n [name] [packagename]     #升级name环境的名为packagename的包

conda remove -n [name] [packagename]     #删除name环境的名为packagename的包
```

### reference
- https://www.tensorflow.org/install/install_linux
- https://github.com/jikexueyuanwiki/tensorflow-zh/blob/master/SOURCE/get_started/os_setup.md
- https://wxinlong.github.io/2017/02/23/InstallTensorflow/
- https://stackoverflow.com/questions/31615322/zsh-conda-pip-installs-command-not-found
- http://blog.csdn.net/zhangxinyu11021130/article/details/64125058