# Simple Live 编译说明书

## 项目概述

Simple Live 是一个跨平台的直播观看应用，支持以下直播平台：
- 虎牙直播
- 斗鱼直播
- 哔哩哔哩直播
- 抖音直播

支持的应用平台：
- Android
- iOS
- Windows
- MacOS
- Linux
- Android TV

## 编译环境

- Flutter: 3.38
- Java JDK: 17
- Windows 10/11 (用于构建Windows和Android安装包)

## 编译步骤

### 1. 克隆项目

```bash
git clone https://github.com/jiangnanxue369-bot/dart_simple_live.git
cd dart_simple_live
```

### 2. 配置构建环境

1. 安装 Flutter 3.38
2. 安装 Java JDK 17
3. 配置 Flutter 环境变量

### 3. 构建安装包

#### 手动构建

```bash
# 安装依赖
flutter pub get

# 构建 Android APK
flutter build apk --release

# 构建 Windows 可执行文件
flutter build windows --release
```

#### 使用 GitHub Actions 自动构建

1. 推送以 `v` 开头的标签，例如：
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. GitHub Actions 会自动开始构建流程
3. 构建完成后，在 GitHub Actions 页面下载构建产物

### 4. 安装和使用

#### Android 安装

1. 下载 `app-release.apk` 文件
2. 在 Android 设备上，进入 **设置** > **安全** > **允许安装未知来源的应用**
3. 找到下载的 APK 文件并点击安装
4. 系统会提示"安装被阻止"，点击 **更多详情**
5. 然后点击 **仍然安装** 按钮

#### Windows 运行

1. 下载并解压 `release-builds` 文件
2. 进入 `build/windows/x64/runner/Release/` 目录
3. 双击 `simple_live_app.exe` 文件运行应用

## 常见问题及解决方案

### 1. APK 无签名无法安装

**解决方案**：修改 `simple_live_app/android/app/build.gradle.kts` 文件，使用 debug 签名配置：

```kotlin
buildTypes {
    release {
        // Signing with the debug keys for now, so `flutter run --release` works.
        signingConfig = signingConfigs.getByName("debug")
        isMinifyEnabled = true
        isShrinkResources = true
        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
}
```

### 2. 签名配置冲突

**解决方案**：使用 `maybeCreate` 方法创建签名配置，避免重复创建：

```kotlin
signingConfigs {
    // Create debug signing config if it doesn't exist
    maybeCreate("debug")

    if (keystorePropertiesFile.exists()) {
        create("release") {
            keyAlias = keystoreProperties["keyAlias"] as String
            keyPassword = keystoreProperties["keyPassword"] as String
            storeFile = keystoreProperties["storeFile"]?.let { file(it) }
            storePassword = keystoreProperties["storePassword"] as String
            isV1SigningEnabled = true
            isV2SigningEnabled = true
        }
    }
}
```

## 自动构建配置

项目配置了 GitHub Actions 工作流，会在以下情况自动构建：
1. 推送以 `v` 开头的标签
2. 手动触发工作流
3. 每3天检测一次原作者是否有代码更新，如果有就自动执行编译

## 构建产物

构建完成后，会生成以下产物：
- `app-release.apk`：Android 安装包
- Windows 可执行文件：位于 `build/windows/x64/runner/Release/` 目录

## 注意事项

1. 首次构建可能需要较长时间，因为需要下载和安装依赖
2. 构建产物将保存7天，期间可以随时下载
3. 使用 debug 签名的 APK 在安装时需要允许安装未知来源的应用
4. 自动构建会每3天检测一次原作者是否有代码更新，如果有就自动执行编译
