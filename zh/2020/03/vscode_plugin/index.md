# VScode实用插件


**陆续更新中**

## Arduino
- specify the arduino path
User.setting -> arduino configuration  
Cmd path 是启动脚本所在路径，是在arduino应用文件中的路径  
Path 是应用所在路径，是在local目录下的路径  
- 右下角 Select 编译器，板子和端口号  
编译器和板子可以回到arduino中查看  
![vscode-arduino-右下标](/images/vscode-arduino1.png) 
- 右上角 操作  
![vscode-arduino-右上标](/images/vscode-arduino2.png)  
分别是：上传，验证，open changes
- 验证
生成暂时的.vscode文件夹，存储配置信息

## SFTP
- 远程连接服务器
- 使用vscode编译服务器端程序
- 设置
  ```
  {
    "comment": "comment",
    "host": "ip",
    "port": 22,
    "username": "root",
    "privateKeyFile": "_Path_to_the_key_"
  }
  ```
- 缺点
  - 到现在我都没找到配置文件搁哪，遂卸

## Remote-SSH
- 比SFTP更好用
- 配置文件清晰
- terminal中直接进入远程的terminal路径
- 缺点
  - 不能远程联入mac系统
