### Gradle 获取项目信息

#### 前言

因为之前公司内部互传 `xxx.apk` 都是通过*即时通讯软件*或者*邮件*来达成的。这样可能会导致同一个项目对应的 apk，内部会有多个版本存在，而且因为互传比较随意的原因，很难查到具体的某个 apk 到底是什么版本，是基于什么时候的代码编译的。所以，为了更好地管理 apk，以及在出现问题时可以根据其内部的配置文件追踪到更多有用的信息（Git 分支、Commit Id等信息） ，公司内部创建了一个管理各个项目 apk 的后台。

这个后台需要每个项目对应的 apk 在自己内部持有一个信息文件（ `asserts/appinfo.xml`），以获取这个 apk 的各项信息。为了满足这个需求，就有了以下的脚本（定义在 `build.gradle`）。

**以下脚本已弃用，换成插件的形式来搜集应用的信息，请看更新内容。**

#### build.gradle 脚本

以下是完整的一个 build.gradle 文件，里面包含了完成上述任务的相关脚本：

```groovy
import groovy.xml.Namespace
import groovy.xml.XmlUtil
import java.text.SimpleDateFormat
import java.util.regex.Pattern

apply plugin: 'com.android.application'
apply plugin: 'maven'

def vn = '1.06.00'
def vc = vn.replace('.', '').toInteger()

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    signingConfigs {
        release {
            keyAlias 'iptv'
            keyPassword '709394'
            storeFile file('./release.keystore')
            storePassword '709394'
        }
    }

    defaultConfig {
        applicationId "linkin.com.apkinfo"
        minSdkVersion 17
        targetSdkVersion 23
        versionCode vc
        versionName vn

    }

    buildTypes {
        release {
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        debug {
        }
    }

    productFlavors {
        domestic {
            manifestPlaceholders = [LINKIN_CHANNEL_VALUE: "domestic"]
        }

        oversea {
            manifestPlaceholders = [LINKIN_CHANNEL_VALUE: "oversea"]
        }
    }

    lintOptions {
        abortOnError false
    }
}

/************************ 搜集项目信息 - 开始 ************************/

// 脚本的版本号
def scriptVersion = '1.00.05'

/**************** 方法定义 - 开始 ****************/

// 包名
def pkgName() {
    return String.valueOf(android.defaultConfig.applicationId)
}

// 版本名
def versionName() {
    return String.valueOf(android.defaultConfig.versionName)
}

// 版本号
def versionCode() {
    return String.valueOf(android.defaultConfig.versionCode)
}

// 最低支持的 Sdk 版本
def minSdk() {
    return String.valueOf(android.defaultConfig.minSdkVersion.mApiLevel)
}

// 当前所处的 Git 分支名称
def gitBranch() {
    return "git rev-parse --abbrev-ref HEAD".execute().text.trim()
}

// 当前打包时对应的 Commit Id
def gitCommitId() {
    if (changesNotCommit()) {
        return null
    } else {
        return "git rev-parse HEAD".execute().text.trim()
    }
}

// 是否为系统应用（是否属于系统用户组）
def isSystemApp() {
    return "android.uid.system".equals(userId())
}

// 获取用户组
def userId() {
    def manifestFile = file(project.projectDir.absolutePath + '/src/main/AndroidManifest.xml')
    // 需要 import groovy.xml.Namespace
    def ns = new Namespace("http://schemas.android.com/apk/res/android", "android")
    def xml = new XmlParser().parse(manifestFile)
    return xml.attributes()[ns.sharedUserId].toString()
}

// 判断是否还有变更没有提交
def changesNotCommit() {
    def output = "git status --porcelain".execute().text.readLines()
    println "commit lines ${output.size()}"
    if (output.isEmpty()) return false
    return !(output.size() == 1 && output.get(0).contains('appinfo.xml'))
}

// 获取渠道号
def channelId() {
    def matcher = releaseMatcher()
    def channelId = matcher.find() ? matcher.group(1).toLowerCase() : null
    if (channelId != null) {
        return channelId.trim().length() == 0 ? null : channelId
    }
    return null
}

// 判断是否为 release 版本
def isRelease() {
    return releaseMatcher().find()
}

// 获取当前的编译版本是否可能为 release 版本
def releaseMatcher() {
    def source = gradle.startParameter.taskRequests.first().args.first().toString()
    def pattern = Pattern.compile("assemble(.*?)(Release)");
    def matcher = pattern.matcher(source);
    return matcher
}

// 获取项目对公司模块的依赖集合
def linkinDeps() {
    def result = "gradlew.bat --daemon app:dependencies --configuration compile".execute().text.toString()
    def lines = result.readLines()
            .findAll { it.contains("com.linkin") }
            .collect { it.substring(it.indexOf('com.linkin')).split(' ')[0]}

    def map = new HashMap()
    for (String line : lines) {
        String[] array = line.split(':')
        if (array.size() == 3) {
            map.put(array[0], array[1] + ':' + array[2])
        }
    }

    def dependencies = new ArrayList()
    for (String key : map.keySet()) {
        dependencies.add("${key}:${map.get(key)}")
    }

    return dependencies
}

/**************** 方法定义 - 结束 ****************/

/**************** 任务定义 - 开始 ****************/

// 搜集项目信息的任务
task createAppInfo << {

    if (!isRelease() && !isSystemApp()) {
        println 'not release compile or system app'
        return
    }

    // 创建记录项目基本信息的文件，以供后台系统读取
    def assets = project.projectDir.absolutePath + '/src/main/assets/'
    def apkInfo = assets + "appinfo.xml"

    def directory = new File(assets)
    if (!directory.exists()) {
        directory.mkdirs()
    }

    def file = new File(apkInfo)
    file.createNewFile()

    // 获取最基本的项目信息
    def packageName = pkgName()
    def versionName = versionName()
    def versionCode = versionCode()
    def channelId = channelId()
    def minSdk = minSdk()
    def gitBranch = gitBranch()
    def gitCommitId = gitCommitId()
    def isSystemApp = isSystemApp()

    // 记录上述的基本信息，可自定义 project 的 name，这里默认用项目的名字
    def info = "<project name='${rootProject.name}'>\n" +
            "\t<appinfo_script_version>${scriptVersion}</appinfo_script_version>\n" +
            "\t<package_name>${packageName}</package_name>\n" +
            "\t<version_name>${versionName}</version_name>\n" +
            "\t<version_code>${versionCode}</version_code>\n" +
            "\t<channel_id>${channelId}</channel_id>\n" +
            "\t<min_sdk>${minSdk}</min_sdk>\n" +
            "\t<git_branch>${gitBranch}</git_branch>\n" +
            "\t<git_commit_id>${gitCommitId}</git_commit_id>\n" +
            "\t<system_app>${isSystemApp}</system_app>\n" +
            "\t<dependencies>\n\t</dependencies>\n" +
            "</project>"

    def root = new XmlSlurper().parseText(info)

    // 获取依赖模块的信息
    linkinDeps().each {
        //noinspection GroovyAssignabilityCheck
        def d = it.split(':')
        root.dependencies.appendNode {
            //noinspection GroovyAssignabilityCheck
            dependency(name: d[1]) {
                package_name d[0]
                version_name d[2]
            }
        }
        // import groovy.xml.XmlUtil
        file.write(XmlUtil.serialize(root), 'UTF-8')
    }
}

// 检查是否有混淆的标识文件，没有则创建它
task createProguardMark << {
    def proguardPath = project.projectDir.absolutePath + '/src/main/java/com/linkin/proguard/';
    File dir = new File(proguardPath)
    File mark = new File(proguardPath + 'Mark.java')
    if (!dir.exists()) {
        dir.mkdirs()
    }
    if (!mark.exists()) {
        mark.createNewFile()
        // import java.text.SimpleDateFormat
        def dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        def now = dateFormat.format(new Date())
        def source = "package com.linkin.proguard;\n" +
                "\n" +
                "/**\n" +
                " * 后台判断是否已经混淆的标识类，其中不需要任何的代码或逻辑\n" +
                " * <p/>\n" +
                " * Auto created by build.gradle(${scriptVersion}) on ${now}\n" +
                " */\n" +
                "public class Mark {\n" +
                "}"
        mark.write(source, 'UTF-8')
    }
}

/**************** 任务定义 - 结束 ****************/

// 在信息搜集前，先确定混淆的标志类已经创建
createAppInfo.dependsOn createProguardMark

// 在编译前，首先执行“创建信息文件”的任务
preBuild.dependsOn createAppInfo

/************************ 搜集项目信息 - 结束 ************************/

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.2.1'
    compile 'com.google.code.gson:gson:2.4'
    compile 'com.linkin.tvlayout:tvlayout:1.0.13'
    compile 'com.jakewharton:butterknife:7.0.1'
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3.1'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3.1'
    testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3.1'
    compile project(':common')
}
```

关键的说明已经在注释里面表达清楚，这里不再哆嗦。简单来说，就是把 `/** 搜集项目信息 - 开始 **/` 到 `/** 搜集项目信息 - 结束 **/` 之间的内容拷贝到具体的项目中的 `build.gradle` 文件即可。

#### 混淆标识

虽然混淆与否可以通过读取 `build.gradle` 的 `minifyEnabled` 是否为 `true` 来判断，但是为了避免“意外（`minifyEnabled` 为 `true`，但实际并没有混淆）”，现在后台采取的方式是，对 `xxx.apk` 进行反编译，并查找标识文件`com.linkin.proguard.Mark.java`，如果找到，则表示没有混淆，反之，则认为已经混淆。

所以，每个项目都需要加入一个 `Mark.java` 的类文件，并放置到`com.linkin.proguard`这个包下，以协助后台区分具体的某个 apk 是否有混淆。这个`Mark.java`不需要任何内容。如：

```java
package com.linkin.proguard;

/**
 * 后台判断是否已经混淆的标识类，其中不需要任何的代码或逻辑
 * <p/>
 * Auto created by build.gradle(1.00.01) on 2016-03-14
 */
public class Mark {
}
```

> 附：
> 
> 上述脚本中已经有任务负责处理这个事情，**不需要手工去创建这个类**，这部分的说明只是为了提前告知项目的负责人，本脚本会在 `com.linkin.proguard` 的包中创建一个 `Mark.java` 的文件。

#### 更新记录

> version 1.00.10

支持对 `productFlovors` 配置信息的读取——假如 `build.gradle` 有如下的配置：

```groovy
    defaultConfig {
        applicationId "com.linkin.focustest"
        minSdkVersion 17
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    productFlavors {
        domestic {
            applicationId "com.linkin.focustest.domestic"
            versionCode 10700
            versionName "domestic.1.07.00"
        }

        oversea {
            applicationId "com.linkin.focustest.oversea"
            versionCode 10800
            versionName "oversea.1.08.00"
        }
    }
```

在这个版本之前，如果选择 `release` 版本和 `domestic`，旧版本生成的 `apkinfo.xml` 中会记录 `com.linkin.focustest`、`1` 和 `1.0`；而新的这个版本中，会记录正确的值 `com.linkin.focustest.domestic`、`10700` 和 `domestic.1.07.00`。

> version 1.00.08

已将本脚本的功能集成到 `ApkInfoPlugin` 插件中，所以不再需要本脚本，旧的项目可清除上述的内容。至于 `ApkInfoPlugin` 的安装，方法如下：

在 `build.gradle` 中配置以下内容：

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    defaultConfig {
        applicationId "com.linkin.focustest"
        minSdkVersion 17
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    productFlavors {
        domestic {

        }

        oversea {

        }
    }

    lintOptions {
        abortOnError false
    }

}

/**************** 配置 ApkInfoPlugin - 开始 ****************/

buildscript {
    repositories {
        maven {
            url "http://172.16.0.120:8089/nexus/content/repositories/releases/"
        }
    }
    dependencies {
        classpath 'com.vsoontech.plugin.apkinfo:apkinfo:1.00.08'
    }
}
apply plugin: 'apkinfo'

/**************** 配置 ApkInfoPlugin - 结束 ****************/

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.2.0'
    compile 'com.android.support:support-v4:23.2.0'
    compile 'com.linkin.base:base:1.01.58'
    compile 'com.linkin.tvlayout:tvlayout:1.0.18'
}

```

> version 1.00.05

1. 修复获取渠道号可能会出现的空指针异常导致的编译错误；
2. 搜集依赖信息增加 `--daemon` 参数（以守护进程的方式来运行，避免重复创建虚拟机，减少耗时）；
2. 完善 `appinfo.xml` 的生成策略——在编译 `release` 版本或者如果是 `system app` 的情况下会生成。

> version 1.00.04

1. 修复获取模块依赖信息时导致编译异常的问题；
2. 调整 `appinfo.xml` 的生成策略，目前只有在编译 `release` 版本的时候才会生成。

> version 1.00.03

1. 调整模块依赖的信息范围（目前只搜集属于公司内部的模块依赖）；
2. 包含子模块的依赖信息。

> version 1.00.02

1. 增加获取渠道号的方法；
2. 根据动态获取的渠道号设置 `channel_id` 节点的值。

> version 1.00.01

1. 增加脚本的版本信息（后面更新记录将以版本为标签）；
2. 在 `appinfo.xml` 中增加脚本版本的信息，对应的节点是 `appinfo_script_version`；
3. 增加自动生成混淆的标志类 `Mark.java` 文件的任务。


> 2016-03-14

1. 考虑到开发的实际需求，取消本地编译 `release` 版本时对版本号需要 `00` 结尾的约束（但是上传服务器时，需要以 `00` 结尾）；
2. 考虑的开发的实际需求，取消本地编译 `release` 版本对是否完成提交的约束（但是上传服务器时，需要完成了提交）。

> 2016-03-11

1. 增加对中文内容的支持。

> 2016-03-10

1. 修复 `needCommit()` 方法的一个错误。

> 2016-03-09

1. 增加 `release` 版本的编译约束：
    - 改动必须已经提交（`git commit`）；
    - 版本名必须以`00` 结尾；
2. 优化脚本代码。