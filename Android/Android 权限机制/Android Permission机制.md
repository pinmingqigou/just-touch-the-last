#跨进程调用四大组件#
- 完全暴露：在manifest中，四大组件节点配置属性：**android:exported=”true”**，即将此组件完全暴露出去，其他应用可以使用。
> 当配置了intentFilter，exported属性默认为true，可强制设为false；当没有配置intentFilter，exported属性默认为false

- 权限提示暴露：在manifest中，四大组件节点配置属性：**android:permission=”xxx.xxx.xx”**（自定义权限），跨进程使用此组件时，必须在manifest的application节点配置**<uses-permission android:name="xxx.xxx.xx" />**
- 私有暴露：这种情况，多数出在一个公司的两个产品。在两个产品的manifest的根节点设置属性**android:sharedUserId="xxx.xxx.xxx"**一致，从而将两个软件的UID强制设置为一样的，同时使用相同签名即可使两个应用数据互通。特别的，当使用系统自带软件的shareUserID，例如Contact，那么无须第三方签名，默认用的是系统签名


#UID机制#
- Android中每个程序有且仅有一个uid（userId）
- 默认情况下，Android会给每个程序分配一个普通级别互不相同的uid
- 不同的Application可共用一个uid
- 各级别uid：root，system，app