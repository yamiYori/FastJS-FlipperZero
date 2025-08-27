# FastJS-FlipperZero JavaScript 功能和库总结

## 概述

FastJS-FlipperZero 是一个专为 Flipper Zero 设备设计的 JavaScript 运行时环境。它基于 mJS (Mongoose JavaScript 引擎)，提供了丰富的硬件访问接口和功能模块，让开发者能够使用 JavaScript 编写 Flipper Zero 应用程序。

## 核心 JavaScript 特性

### 1. 基础语言支持
- **JavaScript 引擎**: 基于 mJS，支持 ES5 标准的核心特性
- **数据类型**: Number, String, Boolean, Object, Array, ArrayBuffer
- **控制流**: if/else, for, while, switch 等
- **函数**: 函数声明、表达式、闭包
- **对象**: 对象字面量、属性访问、方法调用

### 2. 全局函数和对象

#### 内置全局函数
```javascript
// 输出调试信息
print("Hello World");

// 延时函数（毫秒）
delay(1000);

// 字符串转整数（支持进制）
parseInt("123", 10);
parseInt("FF", 16);

// 模块加载
let gpio = require("gpio");
let storage = require("storage");
```

#### Console 对象
```javascript
console.log("信息日志");
console.warn("警告信息");
console.error("错误信息");
console.debug("调试信息");
```

#### 全局变量
```javascript
__filename  // 当前脚本文件路径
__dirname   // 当前脚本目录路径
```

## 可用模块和库

### 1. 设备信息模块 (flipper)
获取 Flipper Zero 设备信息：

```javascript
let flipper = require("flipper");

// 获取设备型号
let model = flipper.getModel();

// 获取设备名称
let name = flipper.getName();

// 获取电池电量（百分比）
let battery = flipper.getBatteryCharge();

// 获取固件信息
let vendor = flipper.firmwareVendor;      // "flipperdevices"
let sdkVersion = flipper.jsSdkVersion;    // [0, 3]
```

### 2. 存储模块 (storage)
文件系统操作：

```javascript
let storage = require("storage");

// 打开文件
let file = storage.openFile("/ext/test.txt", "w");

// 写入数据
file.write("Hello World");

// 读取数据
let content = file.read("ascii", 100);

// 文件操作
file.seekAbsolute(0);     // 绝对定位
file.seekRelative(10);    // 相对定位
file.tell();              // 获取当前位置
file.size();              // 获取文件大小
file.eof();               // 是否到达文件末尾
file.close();             // 关闭文件

// 目录和文件管理
storage.exists("/ext/myfile.txt");
storage.remove("/ext/myfile.txt");
storage.mkdir("/ext/mydir");
storage.rmdir("/ext/mydir");

// 获取文件信息
let info = storage.stat("/ext/myfile.txt");
// info 包含: path, isDirectory, size
```

### 3. GPIO 模块 (gpio)
硬件 GPIO 控制：

```javascript
let gpio = require("gpio");

// 获取 GPIO 引脚
let pin = gpio.get("PA7");  // 支持的引脚: PA4, PA6, PA7, PB2, PB3, PC0, PC1, PC3

// 配置为输出模式
pin.init({
    direction: "out",        // "in" 或 "out"
    outMode: "push_pull"     // "push_pull" 或 "open_drain"
});

// 数字输出
pin.write(true);   // 高电平
pin.write(false);  // 低电平

// 配置为输入模式
pin.init({
    direction: "in",
    inMode: "plain_digital", // "analog", "plain_digital", "interrupt", "event"
    pull: "up"               // "up", "down", "no"
});

// 数字输入
let state = pin.read();

// 模拟输入
let voltage = pin.readAnalog();

// PWM 输出（仅支持特定引脚）
pin.startPwm(1000, 50);  // 频率1000Hz，占空比50%
pin.stopPwm();

// 中断处理
pin.interrupt((state) => {
    print("Pin state changed:", state);
});
```

### 4. 通知模块 (notification)
LED 和蜂鸣器控制：

```javascript
let notify = require("notification");

// 预定义通知
notify.success();   // 成功提示音和绿灯
notify.error();     // 错误提示音和红灯

// LED 闪烁
notify.blink("red", "short");    // 短暂红灯闪烁
notify.blink("green", "long");   // 长时间绿灯闪烁
notify.blink("blue", "short");   // 短暂蓝灯闪烁

// 支持的颜色: "red", "green", "blue", "yellow", "cyan", "magenta"
// 支持的类型: "short", "long"
```

### 5. BadUSB 模块 (badusb)
USB HID 键盘模拟：

```javascript
let badusb = require("badusb");

// 设置键盘布局
badusb.setup({
    layout: "QWERTY"  // 支持不同键盘布局
});

// 输入文本
badusb.print("Hello World!");

// 按键操作
badusb.press("ENTER");
badusb.press("CTRL", "c");     // 组合键
badusb.press("ALT", "TAB");

// 支持的特殊键
// "CTRL", "SHIFT", "ALT", "GUI"
// "ENTER", "TAB", "ESC", "SPACE", "BACKSPACE", "DELETE"
// "UP", "DOWN", "LEFT", "RIGHT"
// "F1"-"F24", "HOME", "END", "PAGEUP", "PAGEDOWN"

// 释放所有按键
badusb.quit();
```

### 6. 串口通信模块 (serial)
UART 串口通信：

```javascript
let serial = require("serial");

// 设置串口参数
serial.setup("usart", 115200, {
    dataBits: "8",      // "6", "7", "8", "9"
    parity: "none",     // "none", "even", "odd"
    stopBits: "1"       // "0.5", "1", "1.5", "2"
});

// 发送数据
serial.write("AT\r\n");
serial.writeBytes([0x41, 0x54, 0x0D, 0x0A]);

// 读取数据
let response = serial.read(10);          // 读取最多10字节
let available = serial.bytesAvailable(); // 可读字节数

// 期待特定响应
serial.expect("OK", 1000);  // 等待"OK"响应，超时1秒

// 支持的串口: "usart", "lpuart"
```

### 7. 数学模块 (math)
扩展数学函数：

```javascript
let math = require("math");

// 三角函数
math.sin(1.57), math.cos(0), math.tan(0.785);
math.asin(1), math.acos(0), math.atan(1);
math.atan2(1, 1);

// 双曲函数
math.sinh(1), math.cosh(1), math.tanh(1);
math.asinh(1), math.acosh(2), math.atanh(0.5);

// 指数和对数
math.exp(1), math.log(10), math.pow(2, 3);
math.sqrt(16), math.cbrt(27);

// 舍入函数
math.ceil(3.2), math.floor(3.8), math.trunc(-3.8);
math.abs(-5), math.sign(-10);

// 比较函数
math.max(10, 20), math.min(10, 20);
math.isEqual(0.1 + 0.2, 0.3);  // 浮点数比较

// 其他函数
math.random();    // 0-1之间随机数
math.clz32(8);    // 32位整数前导零个数

// 常量
math.PI, math.E, math.EPSILON;
```

### 8. GUI 模块 (gui)
图形用户界面：

```javascript
let gui = require("gui");

// 创建视图调度器
let viewDispatcher = gui.viewDispatcher();

// 创建各种视图
let submenu = gui.submenu();
let textBox = gui.textBox();
let dialog = gui.dialog();
let loading = gui.loading();
let emptyScreen = gui.emptyScreen();

// 子菜单配置
submenu.addItem("Option 1", 0);
submenu.addItem("Option 2", 1);
submenu.setHeader("Select Option");

// 文本框配置
textBox.setText("Display text content");
textBox.setFont("text");     // "text", "hex"
textBox.setFocus("start");   // "start", "end"

// 对话框配置
dialog.setHeader("Confirmation");
dialog.setText("Are you sure?");
dialog.setLeft("Cancel");
dialog.setRight("OK");

// 事件处理
viewDispatcher.addEventListener("navigation", (event) => {
    if (event === "back") {
        // 处理返回事件
    }
});

// 视图切换
viewDispatcher.switchTo(submenu);
```

### 9. 事件循环模块 (event_loop)
异步事件处理：

```javascript
let eventLoop = require("event_loop");

// 创建定时器
let timer = eventLoop.timer("periodic", 1000);  // 1秒周期定时器

// 事件监听
eventLoop.subscribe(timer, (event, timer) => {
    print("Timer fired!");
    return [event, timer];  // 返回参数供下次调用
});

// 运行事件循环
eventLoop.run();

// 停止事件循环
eventLoop.stop();
```

## SDK 兼容性检查

```javascript
// 检查 SDK 版本兼容性
let status = sdkCompatibilityStatus(0, 3);  // 检查版本 0.3
if (!isSdkCompatible(0, 3)) {
    print("SDK version incompatible!");
}

// 检查特定功能支持
if (doesSdkSupport(["gpio-pwm", "gui-widget"])) {
    print("Required features supported");
}

// 强制检查功能（会弹出用户确认对话框）
checkSdkFeatures(["baseline", "gpio-pwm"]);
```

## 支持的功能特性

当前 SDK 支持以下功能特性：
- `baseline` - 基础功能
- `gpio-pwm` - GPIO PWM 输出
- `gui-widget` - GUI 小部件
- `serial-framing` - 串口数据帧
- `gui-widget-extras` - 扩展 GUI 组件

## 使用注意事项

1. **内存限制**: mJS 引擎有内存限制，避免创建大型对象或长期运行的循环
2. **硬件访问**: GPIO 操作需要正确的引脚配置，避免短路
3. **文件路径**: 使用绝对路径访问文件，路径前缀通常为 `/ext/`
4. **错误处理**: 使用 try-catch 或检查返回值处理错误
5. **资源释放**: 及时关闭文件句柄和释放资源

## 示例应用

### 简单的 LED 闪烁程序
```javascript
let gpio = require("gpio");
let led = gpio.get("PC3");

led.init({ direction: "out", outMode: "push_pull" });

for (let i = 0; i < 10; i++) {
    led.write(true);
    delay(500);
    led.write(false);
    delay(500);
}
```

### 文件读写示例
```javascript
let storage = require("storage");

// 写入文件
let file = storage.openFile("/ext/data.txt", "w");
file.write("Hello Flipper!");
file.close();

// 读取文件
file = storage.openFile("/ext/data.txt", "r");
let content = file.read("ascii", 100);
file.close();

print("File content:", content);
```

### 串口通信示例
```javascript
let serial = require("serial");

serial.setup("usart", 9600);
serial.write("AT\r\n");

delay(100);
let response = serial.read(10);
print("Response:", response);
```

这个 JavaScript 运行时为 Flipper Zero 设备提供了丰富的编程接口，让你可以创建功能强大的应用程序，从简单的 GPIO 控制到复杂的用户界面应用。