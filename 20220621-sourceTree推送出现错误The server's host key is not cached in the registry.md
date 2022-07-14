# 20220621-sourceTree推送出现错误The server's host key is not cached in the registry

如果你的gitHub上使用的是`.ssh/id_rsa`的密钥，而你的sourceTree是使用自带工具生成的密钥(.ppk格式)，就有可能出现两个密钥不一致。

这时候需要在 工具-选项里面，统一使用ssh密钥，选择`.ssh/`目录下的`id_rsa`文件即可

![image-20220621111310090](https://s2.loli.net/2022/06/21/bmlygv4CFjUeNkO.png)

![image-20220621111248142](https://s2.loli.net/2022/06/21/2iG3fmoOT8CcgME.png)