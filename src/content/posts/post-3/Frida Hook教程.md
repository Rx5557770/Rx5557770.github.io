
---
title: Frida Hook教程
published: 2025-10-23
description: Frida是Hook之路必备工具。
image: ''
tags: ['code', 'hook', 'frida']
category: 代码日常
draft: false
lang: zh-CN
---
    [Frida]("https://github.com/frida/frida") 是一款开源的动态插桩工具，可以插入一些代码到原生App的内存空间去动态地监视和修改其行为，支持Windows、Mac、Linux、Android或者iOS，从安卓层面来讲，可以实现`Java`层和`Native`层`hook`操作。

## 知识科普

Java (以及其他面向对象语言如 Kotlin) 允许类中存在多个同名方法，只要它们的参数类型或参数数量不同，这被称为**方法重载**。

## 配置

> firda电脑的版本要和安卓server运行的版本一致,否则会连接不上.

升级命令:`pip install --upgrade frida-tools`
重启adb命令:

```
adb kill-server
adb start-server
adb devices
```

1. 安装电脑端
   * `pip install frida-tools`
2.  查看手机架构,然后下载对应[Server]("https://github.com/frida/frida/releases")
    * `adb shell getprop ro.product.cpu.abi`
3. 下载后通过adb命令推送到安卓`/data/local/tmp/`目录
    * `adb push /path/to/your/frida-server /data/local/tmp/`
4.  添加执行权限
    - `adb shell`
    * `chmod 777 /data/local/tmp/frida-server`
5. 后台运行
   - `/data/local/tmp/frida-server &`
6.  验证
    * `frida-ps -Uai`

## 模式分析

#### Attach模式

> 在目标App已经启动的情况下，Frida通过ptrace注入程序从而执行Hook的操作.
>
适合于已经运行的App，不会重新启动App，对用户体验影响较小


`frida -U 进程名 -l hook.js`
#### Spawn模式

> 将启动App的权利交由Frida来控制，即使目标App已经启动，在使用Frida注入程序时还是会重新启动App
> 
   适合于需要在App启动时即进行注入的场景，可以在App启动时即捕获其行为

`frida -U -f 进程名 -l hook.js`

## 基本结构

```python
import frida  #导入frida模块
import sys    #导入sys模块

jscode = """  #从此处开始定义用来Hook的javascript代码
    Java.perform(function(){  
        var MainActivity = Java.use('com.example.testfrida.MainActivity'); //获得MainActivity类
        MainActivity.testFrida.implementation = function(){ //Hook testFrida函数，用js自己实现
            send('Statr! Hook!'); //发送信息，用于回调python中的函数
            return 'Change String!' //劫持返回值，修改为我们想要返回的字符串
        }
    });
"""

def on_message(message,data): #js中执行send函数后要回调的函数
    print(message)

process = frida.get_remote_device().attach('com.example.testfrida') #得到设备并劫持进程com.example.testfrida（该开始用get_usb_device函数用来获取设备，但是一直报错找不到设备，改用get_remote_device函数即可解决这个问题）
script = process.create_script(jscode) #创建js脚本
script.on('message',on_message) #加载回调函数，也就是js中执行send函数规定要执行的python函数
script.load() #加载脚本
sys.stdin.read()
```

## Hook技巧

#### 绕过root检测

```
Java.perform(function () {
    const Build = Java.use("android.os.Build");
    Build.TAGS.value = "release-keys";
    Build.FINGERPRINT.value = "generic/google/sdk_gphone_x86/generic_x86:8.1.0/OSM1.180201.007/4581226:userdebug/release-keys";
});
```

#### 延迟Hook

```
setTimeout(function () {
    Java.perform(function () {
        // 延迟执行的 Hook 代码
    });
}, 3000);
```

#### 调用栈打印

```
Java.perform(function () {
    const Exception = Java.use("java.lang.Exception");
    const Log = Java.use("android.util.Log");
    
    const targetMethod = TargetClass.targetMethod;
    targetMethod.implementation = function () {
        const stack = Exception.$new().getStackTrace();
        console.log("调用栈:", stack.toString());
        return targetMethod.apply(this, arguments);
    };
});
```

## Hook模板

```
function main(){
    Java.perform(function(){
        hookTest1();
    });
}
setImmediate(main);
```

#### Hook普通方法、打印参数和修改返回值
```
Java.perform(function () {
    // 1. 获取目标类
    const Calculator = Java.use("com.example.Calculator");

    // 2. Hook add() 方法
    Calculator.add.implementation = function (a, b) {
        // 打印参数
        console.log(`调用 add(${a}, ${b})`);

        // 调用原方法（可选）
        const originalResult = this.add(a, b);
        console.log(`原方法返回值: ${originalResult}`);

        // 修改返回值
        const fakeResult = 100;
        console.log(`修改返回值: ${fakeResult}`);

        return fakeResult;
    };
});
```

#### Hook静态方法

```
Java.perform(function () {
    const Utils = Java.use("com.example.Utils");
    
    Utils.staticMethod.implementation = function (param) {
        console.log("静态方法被调用，参数:", param);
        return "hacked_static_result";
    };
});
```

#### Hook方法重载

> 在 Java 中，方法重载（Overload）允许同一个类中存在多个同名但参数不同的方法。Frida 提供了 `.overload()` 方法来精确 Hook 特定的重载方法。

```
Java.perform(function () {
    const Calculator = Java.use("com.example.Calculator");

    // Hook add(int a, int b)
    Calculator.add.overload('int', 'int').implementation = function (a, b) {
        console.log(`add(int, int): ${a} + ${b}`);
        return a + b + 100; // 修改返回值
    };

    // Hook add(int a, int b, int c)
    Calculator.add.overload('int', 'int', 'int').implementation = function (a, b, c) {
        console.log(`add(int, int, int): ${a} + ${b} + ${c}`);
        return a * b * c; // 完全修改逻辑
    };

    // Hook add(String a, String b)
    Calculator.add.overload('java.lang.String', 'java.lang.String').implementation = function (a, b) {
        console.log(`add(String, String): "${a}" + "${b}"`);
        return "[Hooked] " + a + b;
    };
});

```

#### Hook构造函数

```
Java.perform(function () {
    const User = Java.use("com.example.User");

    User.$init.implementation = function (name) {
        console.log(`创建用户: ${name}`);
        this.$init("Hacked Name"); // 修改构造函数参数
    };
});
```

#### Hook对象字段值

```
Java.perform(function () {
    const User = Java.use("com.example.User");
    
    // 修改实例字段
    Java.choose("com.example.User", {
        onMatch: function (instance) {
            instance.name.value = "HackedName";
            console.log("修改用户名:", instance.name.value);
        },
        onComplete: function () {}
    });
    
    // 修改静态字段
    User.staticField.value = "hacked_static_value";
});
```

#### 动态创建对象并调用方法

```
Java.perform(function () {
    const String = Java.use("java.lang.String");
    const newStr = String.$new("Hello Frida");
    console.log("新字符串:", newStr.toString());
    
    const ArrayList = Java.use("java.util.ArrayList");
    const list = ArrayList.$new();
    list.add("item1");
    console.log("列表内容:", list.toString());
});
```

## Native 层 Hook

#### Hook 导出函数

```
Interceptor.attach(Module.findExportByName("libnative.so", "native_function"), {
    onEnter: function (args) {
        console.log("native_function 被调用");
        console.log("参数1:", args[0].toInt32());
        console.log("参数2:", args[1].readUtf8String());
    },
    onLeave: function (retval) {
        console.log("返回值:", retval.toInt32());
        retval.replace(0); // 修改返回值
    }
});
```

#### Hook 未导出函数

```
const funcPtr = Module.findBaseAddress("libnative.so").add(0x1234);
Interceptor.attach(funcPtr, {
    onEnter: function (args) {
        console.log("未导出函数被调用");
    }
});
```

#### Hook JNI方法

```
const JNINativeInterface = Java.vm.getEnv().handle;
const nativeFuncPtr = JNINativeInterface.GetMethodID;
Interceptor.replace(nativeFuncPtr, new NativeCallback(function (env, clazz, name, sig) {
    console.log(`JNI GetMethodID: ${name} ${sig}`);
    return nativeFuncPtr(env, clazz, name, sig);
}, 'pointer', ['pointer', 'pointer', 'pointer', 'pointer']);
```

