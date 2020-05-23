# Shell脚本笔记


- [变量](#%e5%8f%98%e9%87%8f)
  - [系统给定的特殊变量](#%e7%b3%bb%e7%bb%9f%e7%bb%99%e5%ae%9a%e7%9a%84%e7%89%b9%e6%ae%8a%e5%8f%98%e9%87%8f)
  - [自定义变量](#%e8%87%aa%e5%ae%9a%e4%b9%89%e5%8f%98%e9%87%8f)
    - [判断变量是否为空](#%e5%88%a4%e6%96%ad%e5%8f%98%e9%87%8f%e6%98%af%e5%90%a6%e4%b8%ba%e7%a9%ba)
- [语法](#%e8%af%ad%e6%b3%95)
- [基本算法](#%e5%9f%ba%e6%9c%ac%e7%ae%97%e6%b3%95)
  - [判断是否为数字](#%e5%88%a4%e6%96%ad%e6%98%af%e5%90%a6%e4%b8%ba%e6%95%b0%e5%ad%97)
  - [判断上一条命令是否执行成功](#%e5%88%a4%e6%96%ad%e4%b8%8a%e4%b8%80%e6%9d%a1%e5%91%bd%e4%bb%a4%e6%98%af%e5%90%a6%e6%89%a7%e8%a1%8c%e6%88%90%e5%8a%9f)
  - [判断文件或文件夹是否存在](#%e5%88%a4%e6%96%ad%e6%96%87%e4%bb%b6%e6%88%96%e6%96%87%e4%bb%b6%e5%a4%b9%e6%98%af%e5%90%a6%e5%ad%98%e5%9c%a8)
  - [将多段文字写入文件](#%e5%b0%86%e5%a4%9a%e6%ae%b5%e6%96%87%e5%ad%97%e5%86%99%e5%85%a5%e6%96%87%e4%bb%b6)
  - [判断是否为空](#%e5%88%a4%e6%96%ad%e6%98%af%e5%90%a6%e4%b8%ba%e7%a9%ba)
- [Reference](#reference)

## 变量
### 系统
```
\$0	当前脚本的名字  
\$n	传递给脚本或者函数的参数，n表示第几个参数 
\$#	传递给脚本或函数的参数个数
\$*	传递给脚本或函数的所有参数 
\$@	传递给脚本或者函数的所有参数 
\$$	当前shell脚本进程的PID 
\$?	函数返回值，或者上个命令的退出状态 
\$BASH	BASH的二进制文件问的路径 
\$BASH_ENV	BASH的启动文件 
\$BASH_VERSINFO[n]	BASH版本信息，有六个元素 
\$BASH_VERSION	BASH版本号 
\$EDITOR	脚本所调用的默认编辑器 
\$EUID	当前有效的用户ID 
\$FUNCNAME	当前函数名 
\$GROUPS	当前用户所属组 
\$HOME	当前用户家目录 
\$HOSTTYPE	主机类型 
\$LINENO	当前行号 
\$OSTYPE	操作系统类型 
\$PATH	PATH路径 
\$PPID	当前shell进程的父进程ID 
\$PWD	当前工作目录 
\$SECONDS	当前脚本运行秒数 
\$TMOUT	不为0时，超过指定的秒将退出shell 
\$UID	当前用户ID 
```

### 自定义变量
```
#! /bin/bash
#‘=’前后不能有空格
string="I am shell"
num=5
 
echo "a=${a},string=${string}"
```

传出变量给外部使用
```
export variable="" 
```
#### 判断变量是否为空
- 
## 语法
1. if
   ```
   if [statement]; then
     do ...
   elif [statement]; then
     do ...
   else
     do ...
   fi
   ```
2. for loop
   ```
   for((i=1;i<=10;i++));  
   do   
   echo $(expr $i \* 3 + 1);  
   done  
   ```

3. 函数 返回字符串
   ```
   get_str()
   {
   	echo "string"
   }

   echo `get_str` #写法一
   echo $(get_str) #写法二
   ```

## 基本算法
### 判断是否为数字
   ```
   if [ "$1" -gt 0 ] 2>/dev/null ;then 
     # echo "$1 is number." 
     return 0
   else 
     # echo 'no.' 
     return 1
   fi
   ```

### 判断上一条命令是否执行成功
   ```
   if [ $? -ne 0 ]; then
       echo "failed"
   else
       echo "succeed"
   fi
   ```  
   ``` 
   -eq 等于  
   -ne	不等于  
   -gt 大于  
   -lt	小于  
   ge	大于等于  
   le	小于等于  
   ```

### 判断文件或文件夹是否存在
   ```
   #如果文件夹不存在，创建文件夹
   if [ ! -d "/myfolder" ]; then
     mkdir /myfolder
   fi

   # 判断文件
   if[ ! -f "$myfile" ];
   ```
### 将多段文字写入文件
   ```
   #! /bin/bash
   cat>filename.txt<<EOF
   hello world
   代码改变世界 Coding Changes the World
   100 \$
   她买了张彩票，中了3,300多万美元。
   She bought a lottery ticket and won more than\$ 33 million.
   EOF # end of the file
   ```

### 判断是否为空
  ```
  m=""
  # 判断结果为空
  # method 1
  if [ ! -n "$m"];then
  # method 2
  if [ ! $m]; then
  # method 3
  if [ test -z "$m"]; then
  # method 4
  if ["$m"=""]; then
  ```
   
## Reference
- [将多段文字写入文件](https://www.cnblogs.com/zqifa/p/shell-file-1.html)
- [判断文件或文件夹是否存在](https://www.cnblogs.com/emanlee/p/3583769.html)
- [判断是否为数字](https://blog.csdn.net/sodaslay/article/details/53325199)

