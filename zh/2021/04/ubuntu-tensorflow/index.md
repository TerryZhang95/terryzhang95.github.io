# Ubuntu下Tensorflow环境搭建


## Docker方案
1. 安装[docker](https://docs.docker.com/install/)

2. [安装nvida gpu驱动](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)

使用docker 安装不需要安装cuda toolkit，如果事先有安装过的话，可能在重复安装nvida driver的时候会报错

文档推荐使用[package manager](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html)：

选择ubuntu系统执行
```
sudo apt-get install linux-headers-$(uname -r)

distribution=$(. /etc/os-release;echo $ID$VERSION_ID | sed -e 's/\.//g')

wget https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/cuda-$distribution.pin

sudo mv cuda-$distribution.pin /etc/apt/preferences.d/cuda-repository-pin-600

sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/7fa2af80.pub

echo "deb http://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64 /" | sudo tee /etc/apt/sources.list.d/cuda.list

sudo apt-get update

sudo apt-get -y install cuda-drivers
```

3. 安装[nvida docker支持](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

```
# 如果已安装了docker可以跳过
curl https://get.docker.com | sh \
  && sudo systemctl --now enable docker

distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update

sudo apt-get install -y nvidia-docker2

sudo systemctl restart docker

# docker会自动拉取cuda镜像
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```
执行完成会看到gpu的运行状态

```
...
   0  GeForce RTX 3070    On   | 00000000:01:00.0  On |                  N/A |
| 30%   33C    P8    19W / 220W |    957MiB /  7948MiB |     32%      Default |
...
```

4. 安装tensorflow docker 并运行

```
# 拉取最新版本
sudo docker pull tensorflow/tensorflow

# 启动容器
sudo docker run [-it] [--rm] [-p hostPort:containerPort] tensorflow/tensorflow[:tag] [command]

# 在CPU下启动
sudo docker run -it --rm tensorflow/tensorflow \
   python -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

回顾一些docker常用命令：

```
docker ps # 当前运行容器

docker ps -a # 查看所有容器

docker images # 查看已有镜像
```

5. 在gpu下使用

```
lspci | grep -i nvidia

sudo docker run --gpus all --rm nvidia/cuda:11.2.2-base nvidia-smi

```

需要添加cuda的tag,不然执行docker run时报错找不到镜像
```
docker: Error response from daemon: manifest for nvidia/cuda:latest not found: manifest unknown: manifest unknown.
```

```
# 启动
sudo docker run --gpus all -it --rm tensorflow/tensorflow:latest-gpu \
   python -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

# tensorflow镜像中使用bash shell命令
sudo docker run --gpus all -it tensorflow/tensorflow:latest-gpu bash
```

6. 使用tensorflow

使用docker ps，看不到正在运行的容器，说明无法在命令行直接运行python脚本import tensorflow这样，看来对docker 的应用还得再巩固一下，如果想要运行python脚本时，需要将文件移植到容器内编译才行

或者使用jupyter：
```
sudo docker run -it -p 8888:8888 tensorflow/tensorflow:nightly-jupyter
```

## anaconda方案
目前最简单方案

1. [下载anaconda](https://www.anaconda.com/products/individual/download-success)

下载为sh脚本，直接bash运行，一路yes

```
# 激活anaconda 环境
conda config --set auto_activate_base True
source ~/.bashrc
```

2. 安装 tensorflow

```
# cpu版本
conda create -n tf tensorflow
conda activate tf

# gpu版本
conda create -n tf-gpu tensorflow-gpu
conda activate tf-gpu
```

在命令行执行，需要activate，当环境变为(tf-gpu)的时候可直接编译程序
