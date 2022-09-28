# Musl C 库

标签： libc C C++

---

当前OpenHarmony使用Musl作为基础libc库构建整个系统,Musl库作为系统的基石,我们需要对其兼容性、稳定性、性能要全面评估或改进,结合系统需求、横向对比bionic等libc库,做到增强Musl库功能或性能。基于这个目的,对Musl进行分析和使用，尽量做到全面理解库

> * 介绍Musl库, 如何源码下载、编译、调试
> * 重点库模块, ldso, memory, etc.

## 什么是Musl库
[musl](https://musl.libc.org/) is an implementation of the C standard library built on top of the Linux system call API, including interfaces defined in the base language standard, POSIX, and widely agreed-upon extensions. musl is **lightweight**, **fast**, **simple**, **free**, and strives to be correct in the sense of **standards-conformance** and **safety**.

### 1. 下载&编译&安装
* [Musl in GitHub](https://git.musl-libc.org/cgit/musl)
* [下载tar.gz](http://musl.libc.org/releases/musl-1.2.2.tar.gz)
```shell
# git clone https://git.musl-libc.org/cgit/musl
curl -LO http://musl.libc.org/releases/musl-1.2.2.tar.gz
tar vxf musl-1.2.3.tar.gz
cd musl-1.2.3

# ./configure --help
# --help	查看../configure的使用帮助
# --prefix=DIR	指定安装目录。默认为/usr/local/musl
# --host=HOST	设置目标程序运行的CPU平台一般不需要设置，除非你想要交叉编译默认与宿主机一样
# --enable-FEATURE[=yes|no]	yes：开启FEATURE  no： 关闭FEATURE
# --enable-static[=yes|no]	是否生成静态库
# --enable-shared[=yes|no]	是否生成动态库
# --enable-warnings[=yes|no]	是否开启编译器警告
# --enable-debug[=yes|no]	是否带上debug符号

# prefix这里为空
./configure --prefix= CFLAGS='-O2 -v'
sudo make install

# 可以安装到指定目录
DESTDIR=/tmp/musl make install
```
生成的目标
```shell
-rw-rw-r-- 1       7504 9月   7 23:12 crt1.o
-rw-rw-r-- 1       2608 9月   7 23:12 crti.o
-rw-rw-r-- 1       2568 9月   7 23:12 crtn.o
-rw-rw-r-- 1   12542464 9月   7 23:13 libc.a
-rw-rw-r-- 1          8 9月   7 23:13 libcrypt.a
-rwxrwxr-x 1    4186448 9月   7 23:13 libc.so
-rw-rw-r-- 1          8 9月   7 23:13 libdl.a
-rw-rw-r-- 1          8 9月   7 23:13 libm.a
-rw-rw-r-- 1          8 9月   7 23:13 libpthread.a
-rw-rw-r-- 1          8 9月   7 23:13 libresolv.a
-rw-rw-r-- 1          8 9月   7 23:13 librt.a
-rw-rw-r-- 1          8 9月   7 23:13 libutil.a
-rw-rw-r-- 1          8 9月   7 23:13 libxnet.a
-rw-rw-r-- 1        621 9月   7 23:13 musl-gcc.specs
-rw-rw-r-- 1      16080 9月   7 23:12 rcrt1.o
-rw-rw-r-- 1       7512 9月   7 23:12 Scrt1.o
```
安装之后, 假设DE
```
/tmp/musl
├── bin
│   └── musl-gcc
├── include [98 entries exceeds filelimit, not opening dir]
└── lib
    ├── crt1.o
    ├── crti.o
    ├── crtn.o
    ├── ld-musl-x86_64.so.1 -> /lib/libc.so
    ├── libc.a
    ├── libcrypt.a
    ├── libc.so
    ├── libdl.a
    ├── libm.a
    ├── libpthread.a
    ├── libresolv.a
    ├── librt.a
    ├── libutil.a
    ├── libxnet.a
    ├── musl-gcc.specs
    ├── rcrt1.o
    └── Scrt1.o
```
> * **从软连接信息 ld-musl-x86_64.so.1 -> /usr/local/musl/lib/libc.so 来看，libc.so本身就是链接器**

### 2. 调试
* 增加调试信息
```shell
./configure --prefix=/usr/local/musl --enable-debug=yes
```
* 安装GDB Musl小插件:
```shell
# Requirements:（环境需求）
#    Python 3.5.2+
#    GDB 7.11.1+ with python3 support
# 通过：
# p __malloc_context 查看meta 堆管理器结构体
git clone https://github.com/xf1les/muslheap.git
echo "source /path/to/muslheap.py" >> ~/.gdbinit
```
### 3. 使用Musl开发
* 简单C源码 
```C
/* main.c */
#include <stdio.h>

int main() {
    printf("Hello musl libc\n");
    return 0;
}
```
* 编译链接(*静态链接*)
  -static表示链接的库是静态库，这里要链接libc.a。
  -nostdlib表示不链接系统的标准库文件，因为他要链接musl-libc自己实现的标准库文件。/usr/lib/x86_64-linux-musl/crt1.o、/usr/lib/x86_64-linux-musl/crti.o、/usr/lib/x86_64-linux-musl/crtn.o就是musl-libc自己实现的标准库文件。

```shell
# MUSL_HOME=/tmp/musl/
clang main.c \
    -static -nostdinc -nostdlib \
    -I${MUSL_HOME}/include \
    -L${MUSL_HOME}/lib \
    ${MUSL_HOME}/lib/crt1.o \
    ${MUSL_HOME}/lib/crti.o \
    ${MUSL_HOME}/lib/crtn.o \
    -lc
```
* 编译链接(*动态链接*）
  相比static链接，去掉了-static，并使用[patchelf](https://github.com/NixOS/patchelf)修改连接器
```shell
# MUSL_HOME=/tmp/musl/
clang main.c \
    -nostdinc -nostdlib \
    -I${MUSL_HOME}/include \
    -L${MUSL_HOME}/lib \
    ${MUSL_HOME}/lib/crt1.o \
    ${MUSL_HOME}/lib/crti.o \
    ${MUSL_HOME}/lib/crtn.o \
    -lc -o hello_musl
patchelf --set-interpreter /tmp/musl/lib/libc.so ./hello_musl
```
### 4. C 库模块

