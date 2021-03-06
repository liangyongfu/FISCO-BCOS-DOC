# 鲲鹏平台编译并运行FISCO-BCOS 2.6.0
## 一：申请鲲鹏服务器（已经有鲲鹏服务器略过此步）
### 1. 注册
打开 [鹏城实验室官网](https://dw.pcl.ac.cn/#/home/index) 进行账户注册，注册过程中需要填写真实名称、工作单位、邮箱，收到激活邮件后需要手动点击激活连接地址激活账户 
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/1.png)   

### 2. 登录
登陆注册的账户以后点击 "开发者云" 按钮 
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/2.png)  

### 3. 填写申请订单
在打开的页面中自己填写 "需求申请" 订单，按照自己实际项目需求规划硬件配置、系统版本、使用多长时间 
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/3.png)   

### 4. 等待通知
需求订单填写无误提交以后，等待官方批准，批准完成以后会收到批准结果邮件通知  
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/4.png)  

### 5. 通过申请
登录鲲鹏服务器查看服务器信息，至此鲲鹏服务器准备完成  
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/5.png)  

## 二：在鲲鹏服务器安装基础软件
### 1. 更新软件 
以 Centos 为例，使用 yum 进行更新（此过程会下载更新包，请保持网络畅通并耐心等待）
```.
yum update 
```

![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/6.png) 

### 2. 查看基础工具版本
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/7.png) 

### 3. 安装基础依赖 
```
yum install flex patch bison gmp-static 
```

![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/8.png) 

### 4. 升级gcc 到9.3  

 * 下载gcc-9.3.0.tar.gz 
 ```
 wget http://mirror.hust.edu.cn/gnu/gcc/gcc-9.3.0/gcc-9.3.0.tar.gz
 ```

![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/9.png)

 * 解压文件 
```
tar -xf gcc-9.3.0.tar.gz 
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/10.png)

 * 安装gcc 依赖
```
./contrib/download_prerequisites
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/11.png)

 * 加--noverify选项禁止边下边验证
```
./contrib/download_prerequisites --no-verify
```


 * 创建预编译目录
```
mkdir build && cd build
```

 * 设置编译选项并编译
```
../configure --prefix=/usr/local/gcc-9.3.0 --enable-bootstrap --enable-checking=release --enable-languages=c,c++ --disable-multilib 
```

 * 编译并安装
编译生成makefile文件
```
make && make install
```

#### 4.1 安装后的设置
 * 设置环境变量 
```
touch /etc/profile.d/gcc.sh sudo chmod 777 /etc/profile.d/gcc.sh 
sudo echo -e '\nexport PATH=/usr/local/gcc-9.3.0/bin:$PATH\n' >> /etc/profile.d/gcc.sh && source /etc/profile.d/gcc.sh

```
 * 设置头文件 
```
sudo ln -sv /usr/local/gcc/include/ /usr/include/gcc 
```
 * 设置库文件 
```
touch /etc/ld.so.conf.d/gcc.conf sudo chmod 777 /etc/ld.so.conf.d/gcc.conf sudo echo -e "/usr/local/gcc/lib64" >> /etc/ld.so.conf.d/gcc.conf 
```

 * 加载动态连接库 
```
sudo ldconfig -v ldconfig -p |grep gcc 
```

#### 4.2 测试版本号
 * 测试
```  
gcc -v
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/12.png)

### 5. 安装鲲鹏版本jdk-1.8
 * 安装 jdk
 从 [oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 下载 jdk-1.8

![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/13.png)
 * 解压文件到/usr/local中
```
tar -xf jdk-8u261-linux-arm64-vfp-hflt.tar.gz  -C /usr/local/
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/14.png)
 * 设置环境变量
```
vim /etc/profile

// 在文件最后添加：
export JAVA_HOME=/usr/local/jdk1.8.0_261
export PATH=$JAVA_HOME/bin:$PATH

//使变量生效
source /etc/profile
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/15.png)

## 三：编译FISCO-BCOS 2.6.0源码
本文中用码云仓库映射到GitHub 仓库来同步代码，加快下载速度
### 1. 下载源码
 * 克隆 FISCO BCOS 代码
```
cd /home 
git clone https://gitee.com/hailong99/FISCO-BCOS-2.6.git
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/16.png)
### 2. 切换到2.6.0 分支
```
git branch -a
git checkout remotes/origin/release-2.6.0
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/17.png)
### 3. 源码编译前配置
```
cd /home/FISCO-BCOS
mkdir -p build && cd build

// CentOS请执行此命令，其他系统不需要
source /opt/rh/devtoolset-7/enable 

cmake3 .. -DARCH_NATIVE=on
```
### 4. 编译源码
```
make
```

#### 4.1 解决编译GroupSigLib 报错
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/18.png)
 * 解决方法：
```
cp/usr/share/automake-1.13/config.guess /home/FISCO-BCOS-2.6/deps/src/GroupSigLib/deps/src/pbc_sig/config.guess
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/19.png)
### 5. 查看编译结果
 * 编译完成效果
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/20.png)
 * 查看编译结果的版本号
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/21.png)
 * 查看二进制可执行文件类型
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/22.png)

## 四 : 鲲鹏平台运行2群组3机构6节点底层FISCO-BCOS联盟链服务
### 1. 创建联盟链底层二进制可执行文件存放目录
```
mkdir -p /home/soft/fisco-bcos-bin/
```

### 2. 复制编译的fisco-bcos 文件到创建的目录中
```
cp ./fisco-bcos /home/soft/fisco-bcos-bin/
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/23.png)
### 3. 运行一键搭建2群组3机构6节点底层FISCO-BCOS联盟链服务脚本
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/24.png)
build-Bcos_fgmnode.sh
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/25.png)
### 4. 脚本运行完成效果
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/26.png)
### 5. 查看节点状态
```
ps -aux |grep bcos
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/27.png)
### 6. 查看共识状态
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/28.png)
## 五 : 鲲鹏平台安装FISCO-BCOS 控制台
说明：控制台程序依赖 java-1.8 需要提前安装好鲲鹏版本（arrch64）的java-1.8

### 1. 下载控制台程序
```
cd ../generator-A/
./generator --download_console ./ --cdn
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/29.png)
### 2. 启动控制台
```
cd console/
./start.sh
```
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/30.png)
## 六 : 鲲鹏平台通过控制台查看FISCO-BCOS 2.6.0 版本
 * 查看 FISCO 版本
![](../../../images/articles/kunpeng_platform_compiles_and_runs_fisco-bcos-2.6.0/31.png)