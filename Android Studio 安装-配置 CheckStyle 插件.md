#### 安装 JDK

因为现在的 `CheckStyle` 插件需要 `JDK 8` 的环境，所以在安装这个插件前，先配置好 `JDK 8` ：

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-23/63810028.jpg)

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-23/15891011.jpg)

#### 安装插件

`Settings` -> `Plugins` -> `search "checkstyle"`

![install checkstyle plugin](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-22/87531921.jpg)

安装完需要重启 Android Studio。

#### 配置插件

`Settings` -> `搜索 "checkstyle"` -> `添加配置文件` -> `选中添加的配置文件` -> `OK`

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-22/39560181.jpg)

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-22/13025907.jpg)

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-22/58247160.jpg)

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-22/23598285.jpg)

上述提到的配置文件(`ipmacro_check_style.xml`)，可以从本文档的同一级目录中下载到本地。

#### 效果

当编写代码过程中出现了违反`《Java 编程规范》`的代码时，会有`色块`出现在问题代码附近，鼠标移近，可看到相关的提示：

![](http://7xrgtg.com1.z0.glb.clouddn.com/16-3-22/41283927.jpg)

**希望大家能重视这些提示！希望大家能重视这些提示！希望大家能重视这些提示！**