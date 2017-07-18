
[TOC]

# **Centos7  编译安装Clang 3.9.1**#

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.升级cmake</font> ** ##


``` 
yum autoremove cmake
wget https://cmake.org/files/v3.7/cmake-3.7.1.tar.gz
tar xzf cmake-3.7.1.tar.gz
cd cmake-3.7.1
./bootstrap
gmake -j$(nproc)
make install
cd ..
rm -fr cmake*
```

## **<font color=#191970 size=5>2.安装pip</font> ** ##

```
sudo yum -y install epel-release
sudo yum -y install python-pip
sudo yum clean all
pip install distribute
```


## **<font color=#191970 size=5>3.安装依赖库</font> ** ##

``` 
wget http://llvm.org/releases/3.9.1/llvm-3.9.1.src.tar.xz
wget http://llvm.org/releases/3.9.1/cfe-3.9.1.src.tar.xz
wget http://llvm.org/releases/3.9.1/compiler-rt-3.9.1.src.tar.xz
wget http://llvm.org/releases/3.9.1/clang-tools-extra-3.9.1.src.tar.xz
tar xf llvm-3.9.1.src.tar.xz
mv llvm-3.9.1.src llvm
cd llvm/tools
tar xf ../../cfe-3.9.1.src.tar.xz
mv cfe-3.9.1.src clang
cd clang/tools
tar xf ../../../../clang-tools-extra-3.9.1.src.tar.xz
mv clang-tools-extra-3.9.1.src extra
cd ../../../projects
tar xf ../../compiler-rt-3.9.1.src.tar.xz
mv compiler-rt-3.9.1.src compiler-rt
cd ../..
mkdir llvm-build
cd llvm-build
cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/home/data/software/clang3.9.1 -DLLVM_OPTIMIZED_TABLEGEN=1 ../llvm
make -j$(nproc)
make install
```



