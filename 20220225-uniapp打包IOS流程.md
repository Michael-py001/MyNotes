# 20220225-uniapp打包IOS流程

## 申请苹果证书

1. 申请证书教程 https://ask.dcloud.net.cn/article/152
2. 一共有两套证书，
   - **发布证书 Distribution**，用来发布审核上架用的，使用此证书打包的APP只能在苹果商店下载安装
   - **开发证书 Development** 开发阶段用来调试用的，用此证书打包的APP可以直接在手机上安装
3. 每套证书里面包含两种文件
   - .mobileprovision后缀的**描述文件**
   - .p12/.cer后缀的私钥证书
   - 打包不同版本就用不同的文件
4. Bundle ID 相当于安卓的包名，比如com.xxx.app 
5. .mobileprovision描述文件，添加测试设备/添加权限，都需要重新生成。

## 打包常见错误

1.  doesn't support the Sign in with Apple capability / doesn't include the com.apple.developer.applesignin entitlement 这种报错是因为uniapp的manifest.json文件勾选了苹果登录，但是苹果开发者后台这个应用的权限没有勾选

​			![image-20220226143827571](https://s2.loli.net/2022/02/26/En7RLlYFBIGJti1.png)

解决方案：

- 在uniapp的app模块中取消苹果登录，开发者后台里面也不要勾选



## 苹果手机怎么支持微信登录

> 详细教程 https://uniapp.dcloud.io/api/plugins/universal-links

### 一、配置IOS平台通用链接(UniversalLinks)

登录[苹果开发者网站](https://developer.apple.com/)，在“Certificates, Identifiers & Profiles”页面选择“Identifiers”中选择对应的App ID，确保开启Associated Domains服务 ![img](https://vkceyugu.cdn.bspapp.com/VKCEYUGU-f184e7c3-1912-41b2-b81f-435d1b37c7b4/2eae9f97-2c4a-4dc8-97e8-e665b0c660e1.png)

**开启Associated Domains服务后需要重新生成profile文件，提交云端打包时使用**

### 二、自动生成通用链接

我使用的是uniapp推荐的方案，使用云服务空间来生成

需要绑定域名，在阿里云/腾讯云的域名管理页面，添加记录值

配置域名教程：https://uniapp.dcloud.io/uniCloud/hosting?id=domain

接下来就在这自动生成就🆗了
![img](https://s2.loli.net/2022/02/26/hZH2SMRczFjsaiu.jpg)

### 三、在第三方开发平台配置通用链接

- 微信
  打开微信[开放平台](https://open.weixin.qq.com/)，在“管理中心”页面的“移动应用”下找到已经申请的应用（没有申请应用请点击“创建移动应用”新建应用），点击“查看”打开应用详情页面。 在“开发信息”栏后点击修改，在“iOS应用”下的“Universal Links”项中配置应用的通用链接，如下图所示： ![img](https://s2.loli.net/2022/02/26/XsKuA3l2ECGPwyO.png)

## 打包好的APP怎么安装到手机上

1. ### 安装爱思助手

   1.1 安装驱动

   ​		1.1.1 安装驱动失败
   
   ​								打开控制面板-程序-卸载，搜索Apple，把相关的程序全部卸载，然后重启，即								可安装驱动。
   
   1.2 连接好手机，点击应用，左上角有一个导入安装包，把打包好的文件打开即可安装。
   
2. ### 使用蒲公英应用分发平台(推荐)

   > https://www.pgyer.com/ 官网，需要注册个人开发者

​			

## 要不要加上苹果登录？

看下官方的描述：

> https://developer.apple.com/cn/app-store/review/guidelines/#sign-in-with-apple

- 4.8 通过 Apple 登录

  如果 app 使用第三方或社交登录服务 (例如，Facebook 登录、Google 登录、通过 Twitter 登录、通过 LinkedIn 登录、通过 Amazon 登录或微信登录) 来对其进行设置或验证这个 app 的用户主帐户，则该 app 必须同时提供“通过 Apple 登录”作为同等选项。用户的主帐户是指在 app 中建立的、用于标识身份、登录和访问功能和相关服务的帐户。

  在以下情况下，不要求提供“通过 Apple 登录”选项：

  - 您的 app 仅使用公司自有的帐户设置和登录系统。
  - 您的 app 是一款教育、企业或商务 app，要求用户使用现有的教育或企业帐户登录。
  - 您的 app 使用政府或行业支持的公民身份系统或电子身份证来鉴定用户身份。
  - 您的 app 是特定第三方服务的客户端，用户需要使用他们的邮件、社交媒体或其他第三方帐户直接登录才能访问内容。

按这样的要求，如果有微信登录，那必须要有苹果登录，否则无法上架，但是有实际已经上架的案例，并没有加上苹果登录，只有微信登录，也能上架苹果商店，那就不得而知了，看实际情况吧。