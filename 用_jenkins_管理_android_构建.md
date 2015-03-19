### 用 Jenkins 管理 Android 构建

### 建立主从(可选)

打开`Manage Jenkins -> Manage Nodes`，建立一个新的节点，这里命名为 "build-server-68"。

打开该节点的配置页，主要注意几个地方：

- Labels，后面会在配置项目的构建时用到。这里填写了 Slave01
- Launch method，默认为`Launch slave agents on Unix machines via SSH`


这里有两点相关的注意事项。

我们先明确一主一从的情况下，存在三对 SSH连接

1. master -> slave
2. master -> git server （在此文描述的示例中未发挥作用）
3. slave -> git server


我们并不是在自己的办公电脑上 ssh 到 slave 上，并以此时的用户身份访问 git server，当我们在 Jenkins 上发起一个构建请求时，master 是以 `Jenkins` 的用户身份先 ssh 到 slave 上（与上述的`Launch method`设置有关）。

这就要求：  

- slave 上需要创建用户 `Jenkins`
- 关于这三对连接使用的证书，前两对没什么疑问，都是使用 master 包含 SSH私钥 的证书。
但第三对连接，也需要使用相同的证书，而不是另外建立一个包含 slave 私钥的证书。

另外一点注意事项是`环境变量`和`工具位置`：

试验发现，通过 Jenkins 使用Gralde时，并不会将操作系统的环境变量传递到 gradle 脚本, slave 也不会使用 master 设置好的的环境变量。

为了后面的 Android 构建过程，我们在这里准备好 ANDROID_HOME 这个环境变量。

这里因为 slave 上的 git 安装位置与 master 上不同，所以也需要在`Tool Locations`下额外指定一下。

### 安装/配置 Gradle 插件

插件安装很简单，就不赘述了。

安装好后，打开`Manage Jenkins -> Configure System`，找到`Gradle`节

1. 这里选择自动安装，勾选`Install automatically`。自动安装的好处是，当未来变更或增加 slave 时，不需要手动去 slave 上安装 gralde
2. 可能因为墙的原因，从 Gradle 官网下载安装文件可能有问题。  
在网上搜索到一个下载 Gradle 安装压缩包的[墙内替换连接](http://get.jenv.mvnsearch.org/download/gradle/gradle-2.2.1.zip)，所以我们删除掉`Install from Gradle.org`这个默认的`Installer`，添加一个`Installer`, `Extract \*.zip/*.tar.gz`。  
下载地址填好，`Label` 和 `Subdirectory of extracted archive`这里填写了 Gralde 的版本名: `gradle-2.2.1`

### 配置构建项目

前面建立了 slave，如何指定构建过程发生在该 slave 上呢？

在配置页中找到`Restrict where this project can be run`并勾选，`Label Expression`中填写`Slave01`(输入时会得到自动提示)。

`Build` 节中：

- 添加一个调用 Gradle 脚本的 `build step`。这里使用 `Use Gradle Wrapper`(好处自行搜索)
- Tasks 中输入希望执行的 Gradle Task, 这里输入了 assembleRelease，用来打出所有正式的渠道包
- Root Build script 使用相对于 git 根目录的相对路径，来制定 gradle脚本存放的位置


### 构建过程中的问题

查看项目构建的控制台输出，看到一个`aapt`相关的错误，搜索得知，是因为 slave 机器上的CentOS版本是5，其携带的 gcc 版本为 4.1.2，不能满足 aapt 的依赖要求。

通过 yum 升级 gcc，因为操作系统版本的限制，不能升级到 gcc 的更高版本。

解决方法有两个：

1. 升级 CentOS 到6以上的版本 
2. 下载 gcc source及依赖的三个包(gmp, mpc, mpfr) ，自行 make 	

> 试验2未果，默认 make 出的 gcc, target 是 x86_64-redhat-linux，而 这里 aapt 需要的是 32bit 的 gcc)。此处结论不详，需要查阅更多的参考资料。

