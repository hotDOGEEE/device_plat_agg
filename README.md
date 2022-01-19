# Device_Plat_Agg

本片内容仅针对linux环境，测试系统为Centos8，目的是部署多个项目只需要这一篇文档，并且基本不用查其他的教程。

对atx,provider,和ws-scrcpy，目前常用设备平台服务部署在一台机器上的环境整理部署方案

两个平台有优有劣，内容共涉及四个板块。

因使用这些内容往往需要进行修改，除atx-server2本身可以本地构建容器化运行外，其他的内容推荐直接使用源码运行部署（要经常修改导致重新打包耗费时间，再加上内容多确实没有源码直接运行好管理一些）。

比起官方文档那么正式，我们这个是为了解决环境问题而存在的，所以格式会有一些变动。

每个部分的内容会先把需要安装的环境内容列出来

## 共用内容

python3.8：yum install python38

nvm: curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

adb :https://developer.android.google.cn/studio/releases/platform-tools?hl=zh-cn 从此处下载

## atx-server2

初始部署

```
git clone https://github.com/openatx/atxserver2.git
yum install docker-ce
service docker start
pip3 install docker-compose
```

解决访问问题（关闭防火墙）

```
systemctl stop firewalld.service
systemctl disable firewalld.service
reboot(关闭防火墙后需重启系统)
```

运行

```
cd atxserver2
docker-compose up
```
如果遇到不能访问的问题检查docker映射设置，尝试直接使用
```
docker ps -a
docker start rethinkdb_imageid
docker start web_imageid
```

代码有修改后更新

```
docker-compose up --force-recreate --build -d
```

不过这样会带来一个问题，每次都会重新打包后运行，历史容器可以自己做一个脚本手动清除

## atxserver2-android-provider

安装nvm

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

安装成功后重启终端即可使用。

安装nodejs,文档没有更新，只说了nodejs8才适用，但实际上nodejs8中太高的版本依然不行，这边使用8.9.4可以成功部署

```
git clone https://github.com/openatx/atxserver2-android-provider.git
cd atxserver2-android-provider
nvm install 8.9.4
npm install
pip3 install -r requirements.txt
python3 main.py --server localhost:4000
```

## atxserver2-ios-provider

linux上使用tidevice是最好的方案，不用依赖brew中的特定库

前提内容

```
git clone https://github.com/openatx/atxserver2-ios-provider
cd atxserver2-ios-provider
pip3 install -r requirements.txt
pip3 install -U "tidevice[openssl]"
yum install usbmuxd
```

启动
新接入的苹果设备记得要信任电脑
```
SERVER_URL="http://localhost:4000"
tidevice applist | grep "WebDriverAgent"
WDA_BUNDLE_PATTERN="(bundle name)"
python3 main.py -s $SERVER_URL --use-tidevice --wda-bundle-pattern $WDA_BUNDLE_PATTERN
```

## ws-scrcpy

```
git clone https://github.com/NetrisTV/ws-scrcpy.git
cd ws-scrcpy
nvm install 10.9.0
npm start
```

这个看着是最简单的，但他nodejs没怎么做好的样子，很多版本运行遇到环境问题会很麻烦，目前使用12遇到的问题最少。
