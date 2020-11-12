# Cellbang IDE

Cellbang IDE 是 [Malagu](https://github.com/cellbang/malagu) + [Theia](https://theia-ide.org/) 实现的 Web IDE，可以部署在 Serverless 平台上。这是一个模板项目，使用该模板项目，我们可以快速部署一个运行在 Serverless 平台上的 Web IDE。本模板以默认部署到阿里云函数计算平台。

## 效果

在线演示网址：[https://ide.cellbang.com#/tmp](http://ide.cellbang.com)。只有 `/tmp` 目录下可写，其他目录都是只读。

![ide.jpg](https://i.loli.net/2020/10/23/F7LzKlMfaob5Gj4.jpg)

## 原理

从效果图，可以发现我们的 Web IDE 与 Theia IDE 几乎一模一样。原因是我们采用 Theia + Malagu 实现的。Malagu 框架的前后端一体设计灵感来自 Theia IDE，所以 Malagu 可以无缝与 Theia IDE 进行融合。我们甚至可以使用 Malagu 作为基础框架开发 Theia IDE 的扩展。Theia IDE 的扩展与 Malagu 的组件几乎一样。

目前，Serverless 平台对长连接双向通信支持不是很友好，所以 Malagu 在集成 Theia IDE 的时候，将大部分长连接的实现替换了短连接单向通信。对于 Teminal 等对双向通信有强要求的模块，我们并没有适配。

## 使用

本模板最为核心的部分是 `.malagu` 目录，该目录事代码目录，可以直接上传到函数计算平台上，并为之创建 HTTP 触发器和绑定自定义域名即可。

接下来，我们使用 Malagu 的方式一键部署本项目。

#### 一、安装命令行

```bash
# 指定淘宝镜像源下载会更快
npm install -g yarn --registry https://registry.npm.taobao.org 
npm install -g @malagu/cli --registry https://registry.npm.taobao.org 
```

#### 二、初始化 IDE 模板

```bash
malagu init demo https://github.com/cellbang/ide.git
```

#### 三、修改 `malagu.yml` 配置


```yaml
malagu:
  name: IDE
  fc-adapter:
    service:
      # 需要，则替换为你自己的 VPC 配置，否则删除
      vpcConfig:
        vpcId: vpc-12345
        vSwitchIds: [ vsw-12345 ]
        securityGroupId: sg-12345
      # 需要，则替换为你自己的 NAS 配置，否则删除
      nasConfig:
        userId: 10003
        groupId: 10003
        mountPoints:
          - 
            serverAddr: 12345-vnc76.cn-hangzhou.nas.aliyuncs.com:/
            mountDir: /mnt/code
      # 如果没有配置 VPC，可以不需要角色，否则请配置好你自己的角色，并至少拥有 AliyunECSNetworkInterfaceManagementAccess 权限策略
      role: acs:ram::${malagu['fc-adapter'].profile.accountId}:role/role12345
    function:
      memorySize: 1024
      instanceConcurrency: 100
    customDomain:
      # 配置为你自己的域名
      name: 12345.com

```

#### 四、部署项目到函数计算

```bash
cd demo           # 进入项目根目录
yarn run deploy   # 部署项目到函数计算，如果你是第一次使用，且没有使用过 Funcraft 工具，会提示你输入阿里云账号的 AKSK 信息
```

## 使用场景

#### 场景一：可视化操作 NAS 的文件系统，无需挂载 ECS

目前的一个典型的使用场景是可视化上传、下载和编辑 NAS 里面的文件。虽然函数计算对请求的大小有 6 MB 限制，但是本项目采用文件分片上传策略解决了该问题。

![IDE 演示.gif](https://i.loli.net/2020/10/23/9xEkpyvBOwXHUnq.gif)
