## 抓包引擎

- Linux：Libpcap9
- Windows: winpcap10

## 基本使用方法

- 启动

- 选择网卡

- 混杂模式

  设置wireshark工作模式

  - 普通模式：只会对自己mac的流量进行解包
  - 混杂模式：只要是经过网卡的流量包都会监控，无论是不是自己的mac地址

- 实时抓包

- 保存和分析捕获文件

- 首选项

## 筛选器

- 抓包筛选器（capture filters）：初次过滤
- 显示筛选器（display filters）：二次过滤

流程：network -> capture filters -> capture engine -> display filters

## 常见协议包

## 流量解密

### 私钥解密

条件：拥有私钥证书

缺点：无法解密ECDHE进行秘钥交换的加密流量