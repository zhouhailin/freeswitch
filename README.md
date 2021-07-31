# FreeSWITCH 源代码分析

源代码分析和注释不修改源代码，只添加相关注释说明，源代码行数会受影像

## 编译安装

### Clone源代码

#### 最新代码

    git clone https://github.com/signalwire/freeswitch

#### 指定分支

    git clone -b v1.6 https://github.com/signalwire/freeswitch

### 源代码编译安装

#### 首次

    ./bootstrap.sh && ./configure && make && make install

#### 增量

##### 所有模块

    make && make install

##### 指定模块

    # 指定模块调整
    make mod_sofia-install

    # 如果核⼼的代码调整
    make core-install

## 源码目录

## 源码分析

主要包含启动、模块、呼叫

### 启动

### 模块

### 呼叫

