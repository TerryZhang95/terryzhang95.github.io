# Docker学习笔记


# 安装
## ubuntu 安装docker
```
sudo apt-get update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt-get update
apt-cache policy docker-ce # 验证docker repo添加成功
sudo apt-get install docker-ce # 安装docker
sudo systemctl status docker # 验证docker安装成功
```
## ubuntu 安装 docker-compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version # 验证
```

# docker 环境遇到的问题
- Got permission denied while trying to connect to the Docker daemon socket
  - 需要给docker进程777的权限
  ```
  sudo chmod 777 /var/run/docker.sock
  ```
- macos上开通对/var/的file sharing权限
  - 在Docker->Preference->resources->File Sharing中，添加路径，只能通过folder添加，不能输入，那要怎么添加系统路径呢？
  - 解决这个问题用了我一个小时的时间查找。。。
  - 点击添加后，在folder界面，点击“cmd+shift+G”，输入路径即可
  
5d420170f0ea
# Reference
- [Install Docker Compose](https://docs.docker.com/compose/install/)
- [How To Install and Use Docker on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04)
- [Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
](https://stackoverflow.com/questions/47854463/got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket-at-uni)
