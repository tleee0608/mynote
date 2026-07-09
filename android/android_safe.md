# 安卓学习plan
## 第一周： 基础 + 静态分析
**目标：能独立看懂一个 APK 的 Manifest 和 Java 伪代码。** \
Day 1：Android APK 结构、Manifest、四大组件 \
Day 2：权限系统、dangerous/special/normal 权限\
Day 3：安装 adb、JADX、Apktool\
Day 4：解包一个 CTF APK\
Day 5：JADX 搜索敏感 API 和字符串\
Day 6：写第一份静态分析报告\
Day 7：复盘 + 整理自己的 checklist

## 第二周：动态分析 + 网络分析
**目标：能运行 App 并记录行为证据** \
Day 1：模拟器 / 测试机环境 \
Day 2：adb install、pm、am、logcat \
Day 3：观察启动、点击、授权后的行为变化 \
Day 4：安装 mitmproxy 或 Burp \
Day 5：抓取 CTF App 网络流量 \
Day 6：用 MobSF 自动分析 \
Day 7：手工结果与 MobSF 报告对照 

## 第三周：攻击技术与平台滥用
**目标：能把攻击技术转成检测规则和防护建议。** \
Day 1：UI deception、overlay、WebView 风险\
Day 2：Accessibility Abuse\
Day 3：BroadcastReceiver、Service、Provider 风险\
Day 4：动态加载、加壳、混淆识别\
Day 5：持久化、隐藏、反分析\
Day 6：网络 C2 与数据外传识别\
Day 7：整理威胁建模表\

## 第四周：恶意家族与报告写作
**目标：能写出完整 Android 样本分析报告。**\
Day 1：阅读一个恶意家族页面\
Day 2：整理家族能力矩阵\
Day 3：整理权限和行为对应关系\
Day 4：整理 IOC\
Day 5：整理 MITRE ATT&CK Mobile 映射\
Day 6：写完整报告\
Day 7：复盘报告并补充检测建议

## Day 1 Android APK 结构、AndroidManifest.xml、四大组件

### Android APK 是什么
APK 可以理解为 Android App 的安装包。做安全分析时，你看到的一个 .apk 文件，里面通常包含这些核心部分：
| 文件 / 目录               | 作用                         | 安全分析时看什么              |
| --------------------- | -------------------------- | --------------------- |
| `AndroidManifest.xml` | 应用配置清单                     | 包名、权限、组件、入口点、导出组件     |
| `classes.dex`         | Java / Kotlin 编译后的 DEX 字节码 | 主要业务逻辑、敏感 API、恶意行为    |
| `res/`                | 编译后的资源目录                   | 布局、图片、字符串、XML 配置      |
| `resources.arsc`      | 编译后的资源索引                   | 字符串、样式、资源 ID 映射       |
| `assets/`             | 原始资源文件                     | 配置、模型、隐藏脚本、加密 payload |
| `lib/`                | Native so 库                | C/C++ 逻辑、加壳、反调试、加密算法  |
| `META-INF/` 或签名块      | 签名相关信息                     | 证书、签名、是否被二次打包         |


APK
├── AndroidManifest.xml     应用说明书
├── classes.dex             程序代码
├── res/                    界面和资源
├── assets/                 原始资源
├── lib/                    native so 库
├── resources.arsc          资源索引
└── 签名相关文件             身份与完整性

做 Android 安全分析时，优先顺序通常是先看 `Manifest → 再看 DEX 代码 → 再看资源/字符串 → 再看 native so → 最后动态验证`

### AndroidManifest.xml 是什么

AndroidManifest.xml 是 Android App 的核心配置文件。它告诉系统：

这个 App 的包名是什么；\
它有哪些组件；\
它需要哪些权限；\
哪个 Activity 是启动入口；\
哪些组件能被其他 App 调用；\
它支持哪些设备能力；\
它是否允许备份、调试、明文流量等配置。
一个简化的 Manifest 长这样：
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.demo">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />

    <application
        android:allowBackup="true"
        android:usesCleartextTraffic="true"
        android:theme="@style/AppTheme">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>

        <service
            android:name=".UploadService"
            android:exported="false" />

        <receiver
            android:name=".BootReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>

        <provider
            android:name=".UserProvider"
            android:authorities="com.example.demo.provider"
            android:exported="false" />

    </application>
</manifest>
\
```
#### 有几个重要的标签
1. package="com.example.demo"
   表示包名，通常来说包名应该是和APP名字相匹配
2. `<uses-permission>`
   可以用来声明APP需要哪些权限
   ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_SMS" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
   ```
   | 权限组合                                   | 可能行为       |
    | -------------------------------------- | ---------- |
    | `INTERNET`                             | 联网通信       |
    | `READ_SMS` + `RECEIVE_SMS`             | 读取短信、验证码风险 |
    | `READ_CONTACTS` + `INTERNET`           | 通讯录上传风险    |
    | `ACCESS_FINE_LOCATION` + `INTERNET`    | 定位上传风险     |
    | `SYSTEM_ALERT_WINDOW`                  | 悬浮窗、覆盖界面风险 |
    | `BIND_ACCESSIBILITY_SERVICE`           | 辅助功能滥用风险   |
    | `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` | 后台保活风险     |
    | `RECEIVE_BOOT_COMPLETED`               | 开机自启动风险    |
    安全分析时，权限不是单独判断的，而是要看“权限组合”
3. `<application>`
   应用级配置，常见的重点属性是：
   | 属性                                    | 含义                | 风险            |
    | ------------------------------------- | ----------------- | ------------- |
    | `android:allowBackup="true"`          | 允许备份 App 数据       | 可能导致敏感数据被备份导出 |
    | `android:debuggable="true"`           | 允许调试              | 正式版中出现较危险     |
    | `android:usesCleartextTraffic="true"` | 允许 HTTP 明文流量      | 可能导致数据明文传输    |
    | `android:networkSecurityConfig`       | 网络安全配置            | 可能放宽证书校验      |
    | `android:name`                        | 自定义 Application 类 | 常见初始化入口       |
    | `android:theme`                       | 主题                | 可用于伪装 UI      |
    | `android:label`                       | App 名称            | 可伪装成系统或银行 App |

### Android 四大组件
Activity
Service
BroadcastReceiver
ContentProvider

#### Activity：界面组件
Activity代表一个用户界面页面
例如：`<activity android:name=".MainActivity" />`
