
---
title: Objection Hook
published: 2025-10-23
description: Objection是一款类似于frida的工具。
image: ''
tags: ['hook', 'code']
category: 代码日常
draft: false
lang: zh-CN
---
    介绍：

> Objection 提供了一个用户友好的命令行界面（REPL - Read-Eval-Print Loop），其中包含预构建的命令，简化了常见任务，这使得它对于可能不擅长编写复杂 [Frida]("https://github.com/frida/frida") 脚本的安全专业人员来说更易于上手，支持安卓和iOS。

> 注意：设备需要开启frida server

## 命令

#### 启动命令

```
objection -g 包名 explore
```

#### Hook 命令

#### 基础 Hook 命令

- **Hook 某个类的所有方法：**
    
    ```
    android hooking watch class <完整类名>
    ```
    
- **Hook 特定方法：**
    
    ```
    android hooking watch method <完整方法签名> [--dump-args] [--dump-return] [--dump-backtrace]
    ```
    
    - **示例：Hook Activity 的 `onCreate`：**
        
        ```
        android hooking watch method android.app.Activity.onCreate --dump-args
        ```
        
- **Hook 特定类的方法：**
    
    ```
    android hooking watch class_method com.example.app.MainActivity.login --dump-args --dump-backtrace --dump-return
    ```
    

#### 常用 Hook 场景

- **查看类的方法：**
    
    ```
    android hooking list class_methods com.example.app.User
    ```
    
- **Hook HTTP 请求：**
    
    ```
    android hooking watch class_method okhttp3.OkHttpClient.newCall --dump-args
    ```
    
- **Hook 数据库操作：**
    
    ```
    android hooking watch class_method net.sqlcipher.database.SQLiteDatabase.insert --dump-args
    ```
    

**高级 Hook 操作**

- **保存 Hook 结果：**
    
    ```
    android hooking watch class_method com.example.app.API.request --dump-args --dump-return > hook_log.txt
    ```
    
- **设置方法返回值：**
    
    ```
    android hooking set return_value com.huluxia.gametools.AuthChecker.isLoggedIn true
    ```
    

#### 绕过命令

- **SSL Pinning 绕过：**
    
    ```
    android sslpinning disable
    ```
    
- **Root 检测绕过：**
    
    ```
    android root disable
    ```
    
- **调试检测绕过：**
    
    ```
    android hooking watch method android.app.Application.isDebuggable --dump-args --dump-return
    ```
    
- **文件操作监控：**
    
    ```
    android hooking watch class java.io.File --dump-args --dump-return
    ```
    
- **数据库操作监控：**
    
    ```
    android hooking watch class net.sqlcipher.database.SQLiteDatabase
    ```

> 开启 API 模式

```
objection -g com.example.targetapp explore --enable-api --api-port 8888
```

测试方法

```
(com.example.targetapp)# android hooking run class_method com.example.targetapp.SignInManager.performSignIn
```

> 通过 `curl` 命令发送 HTTP 请求来触发签到
   我们将使用 `curl` 这个命令行工具来发送 HTTP 请求。

1. **确定请求的 URL：**
    
    - Objection API 监听在 `http://127.0.0.1:8888` (默认情况下，如果你在本地电脑运行 Objection)。
    - 调用特定方法的是 `/rpc/invoke/<方法名>`。根据文档，执行 `android hooking run class_method` 对应的内部 API 名称是 `androidHookingRunClassMethod`。
    - 所以 URL 是：`http://127.0.0.1:8888/rpc/invoke/androidHookingRunClassMethod`
2. **确定请求方法和参数：**
    
    - 因为 `androidHookingRunClassMethod` 需要参数（类名和方法名），所以必须使用 **POST 请求**。
    - `Content-Type` 必须是 `application/json`。
    - **JSON 请求体**需要包含 `className` (目标类名) 和 `methodName` (目标方法名)。如果方法有参数，还需要 `args` 数组。
    
    所以，一个调用 **无参数** 方法 `com.example.targetapp.SignInManager.performSignIn` 的 JSON 请求体看起来是这样的：
    
    JSON
    
    ```
    {
        "className": "com.example.targetapp.SignInManager",
        "methodName": "performSignIn",
        "args": []
    }
    ```
    
    （注意：`args` 字段必须是一个数组，即使没有参数也要传入一个空数组 `[]`）
    
3. **在新的终端窗口中执行 `curl` 命令：**
    
    Bash
    
    ```
    curl -X POST "http://127.0.0.1:8888/rpc/invoke/androidHookingRunClassMethod" \
    -H "Content-Type: application/json" \
    -d '{ "className": "com.example.targetapp.SignInManager", "methodName": "performSignIn", "args": [] }'
    ```
    
    - `-X POST`：指定使用 POST 请求方法。
    - `"http://127.0.0.1:8888/rpc/invoke/androidHookingRunClassMethod"`：API 的完整 URL。
    - `-H "Content-Type: application/json"`：设置请求头，告诉服务器我们发送的是 JSON 数据。
    - `-d '{ ... }'`：发送的 JSON 数据。

**执行这个 `curl` 命令后：**

- **如果成功，** 你会在 `curl` 所在的终端看到一个 JSON 响应，通常包含 `{"status": "ok", "data": ...}` 或方法的返回值。
- 同时，在 **Objection 启动的那个终端**里，你可能会看到类似 `Agent injected and responds ok!` 这样的日志（如果之前没有）。
- **最重要的是，目标 Android 应用程序中 `com.example.targetapp.SignInManager.performSignIn()` 方法会被执行！** 你可能会在应用界面上看到签到成功的提示，或者在 `adb logcat` 中看到相关的日志输出。