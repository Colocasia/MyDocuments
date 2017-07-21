
[TOC]

# **Centos7  编译安装GCC 6.2.0**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.下载GCC 6.2.0</font> ** ##

可以从[GCC的FTP地址下载](http://ftp.gnu.org/gnu/gcc)你需要的版本，我下载的是6.2.0<br>

``` 
wget http://ftp.gnu.org/gnu/gcc/gcc-6.2.0.tar.gz
```
<br>

## **<font color=#191970 size=5>2.安装依赖项</font> ** ##

执行如下命令，下载编译必需的包。<br>
```
tar -zvxf gcc-6.2.0.tar.gz
cd gcc-6.2.0
./contrib/download_prerequisites
```
如果此过程有错误，可以把错误包删除重新执行此命令来解决。<br>


## **<font color=#191970 size=5>3.新建编译中间文件夹</font> ** ##

``` 
mkdir gcc_build_6.2
```

## **<font color=#191970 size=5>4.构建工程</font> ** ##

``` 
cd gcc_build_6.2
../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
```

## **<font color=#191970 size=5>5.开始编译</font> ** ##

``` 
make -j$(nproc)
```
如果编译出错，执行make clean后直接make，只是编译过程会比较慢。<br>

## **<font color=#191970 size=5>6.安装GCC</font> ** ##

``` 
make install
```

安装完成后

``` 
gcc -v
```

可看到版本信息，如果仍为旧版本，添加软连接

```
ln -s /usr/local/gcc-6.2.0/bin/gcc /usr/bin/gcc
```

再次使用命令查看版本

```
gcc -v
```

大功告成