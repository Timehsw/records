最近折腾了一下服务器，比较了一下Ubuntu和CentOS，最终还是发现CentOS比较顺手。但是，Ubuntu自带了Python2.7和Python3.6，CentOS却只安装了Python2.7，一点都不人性化啊有木有！？
然后开始Google，捣鼓CentOS下Python3的安装方法。网上多数是用下载源码编译的方法安装，比较折腾。
在重装了几次系统后，最终发现了一种使用yum安装的极简方法。
准备工作
首先更新一下yum：
```
sudo yum -y update
```


该 -y 标志用于提醒系统我们知道我们正在进行更改，免去终端提示我们要确认再继续（可以不添加该标志）。

然后安装yum-utils，一组扩展和补充yum的实用程序和插件：
```
sudo yum -y install yum-utils
```

最后，我们将安装CentOS开发工具，用于允许您从源代码构建和编译软件：
```
sudo yum -y groupinstall development
```
安装Python3
安装EPEL：
```
sudo yum -y install epel-release
```
复制代码安装IUS软件源：
```
sudo yum -y install https://centos7.iuscommunity.org/ius-release.rpm
```
安装Python3.6：
```
sudo yum -y install python36u
```
安装pip3：
```
sudo yum -y install python36u-pip
```
可以检查一下安装情况，分别执行命令查看：
```
python3.6 -V
pip3.6 -V
```

添加软链接
到此，可以说是安装完成了，在 /usr/lib/目录下可以看到Python3.6的文件夹。
那就创建一个软链接，使用python3去使用Python3.6吧：
```

ln -s /usr/bin/python3.6 /usr/bin/python3
```

pip3.6同理：
```
ln -s /usr/bin/pip3.6 /usr/bin/pip3
```