---
layout: post
title: 在UNIX平台安装Node.js
---
### 在SUSE Linux 11安装nodejs ### 
官网下载tar.xz文件，并复制解压: 

    xz -d xxxx.tar.xz 
    mv node /opt/node 

在PATH中添加/opt/node/bin 

测试: 

    node --version 

### 在AIX7.1安装nodejs ### 
在nodejs.org和ibm官网下载以下文件： 
> ibm-4.3.1.0-node-v4.3.1-aix-ppc64.bin 
> libgcc-4.8.3-1.aix7.1.ppc.rpm 
> libstdc++-4.8.3-1.aix7.1.ppc.rpm 

安装库文件: 

    rpm -ivh libgcc-4.8.3-1.aix7.1.ppc.rpm 
    rpm -ivh libstdc++-4.8.3-1.aix7.1.ppc.rpm 
    
安装node: 

    chmod 777 ibm-4.3.1.0-node-v4.3.1-aix-ppc64.bin 
    ./ibm-4.3.1.0-node-v4.3.1-aix-ppc64.bin 
    
在PATH中添加参数: 
> export LD_LIBRARY_PATH=/opt/freeware/lib64 
> export PATH=$PATH:/ibm/node/bin 

测试: 

    node --version 

-EOF-
