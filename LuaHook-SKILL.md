---
name: luahook-code-snippets
description: 当用户需要 LuaHook/AndroLua 代码示例、Hook 写法、Intent/网络/文件/界面/系统能力调用时，优先从本技能代码库中检索最小可用片段并按需组合，输出可直接运行的 Lua 代码。
---

# LuaHook 代码技能库

# LuaHook 完整文档手册

**生成时间**: 2026-02-03 14:48:13
**页面总数**: 22
**数据来源**: https://luahook.kulipai.top/

<!-- 页面 1: 001_packaged.md -->

## 目录

- [打包 | LuaHook](#打包-luahook)
- [Hook 使用说明文档 | LuaHook](#hook-使用说明文档-luahook)
- [lpparam（LoadPackageParam）介绍 | LuaHook](#lpparamloadpackageparam介绍-luahook)
- [字段操作（Field 操作） | LuaHook](#字段操作field-操作-luahook)
- [导入和寻找类 | LuaHook](#导入和寻找类-luahook)
- [类的构造与方法调用 | LuaHook](#类的构造与方法调用-luahook)
- [内存管理 | LuaHook](#内存管理-luahook)
- [内存读写 | LuaHook](#内存读写-luahook)
- [字符串操作 | LuaHook](#字符串操作-luahook)
- [指针操作 (LuaPointer) | LuaHook](#指针操作-luapointer-luahook)
- [模块与符号 | LuaHook](#模块与符号-luahook)
- [Hook 接口 | LuaHook](#hook-接口-luahook)
- [Native函数调用 | LuaHook](#native函数调用-luahook)
- [兼容性说明 | LuaHook](#兼容性说明-luahook)
- [JSON | LuaHook](#json-luahook)
- [Task | LuaHook](#task-luahook)
- [📂 LuaFile | LuaHook](#luafile-luahook)
- [🌐 LuaHttp | LuaHook](#luahttp-luahook)
- [Luaj++ | LuaHook](#luaj-luahook)
- [监听音量按键 | LuaHook](#监听音量按键-luahook)
- [SO文件动态注入 | LuaHook](#so文件动态注入-luahook)
- [编写脚本设置页面 | LuaHook](#编写脚本设置页面-luahook)

---

## 打包 | LuaHook

**页面地址**: [https://luahook.kulipai.top/packaged.html](https://luahook.kulipai.top/packaged.html)

**处理顺序**: 第1个

---

## 打包​

### 1.克隆指定分支​

克隆一个LuaHook的simplify分支到本地

```bash

git clone -b simplify https://github.com/KuLiPai/LuaHook.git

```

### 2.打开项目并定位文件​

用Android Studio或其他IDE打开项目

等待依赖加载完毕

找到app/src/main/java/com/kulipai/luahook/LuaCode.kt文件

### 3.编写 Lua 代码​

在上图红色方块中写入lua代码，并保存

### 4.修过应用信息​

在app/build.gradle.kts里修改如下内容

applicationId包名

versionCode版本号

versionName版本名

> 请不要修改namespace = "com.kulipai.luahook"会导致hook失效

如果你的lua代码使用宿主资源注入扩展，为防止资源 ID 互相冲突，你需要修改资源 ID。（app/build.gradle.kts第54-56行左右）

> 注意提供的示例资源 ID 值仅供参考，不可使用0x7f，默认为0x64，为了防止当前宿主存在多个 Xposed 模块，建议自定义你自己的资源 ID。

修改app名称和图标

在app\src\main\AndroidManifest.xml里修改

可以直接将@string/app_name改成app名字符串

### 5.编译和打包​

使用Gradle或 IDE 的编译功能来构建程序，生成最终的 APK 安装包

Android Studio点这里如图

然后加载或新建你的签名文件

最后选release进行编译

最终生成的apk在app\release\app-release.apk

（初次编译时间可能较长，请耐心等待，如果失败尝试重新编译，还有问题可以联系作者）

---

<!-- 页面 2: 002_manual__hook.md -->

## Hook 使用说明文档 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/hook.html](https://luahook.kulipai.top/manual/hook.html)

**处理顺序**: 第2个

---

## Hook 使用说明文档​

本模块提供了四种 Hook 接口形式：

- hook：用于 Hook 指定的方法

- hookctor：用于 Hook 构造函数（构造方法）

- hookAll：用于 Hook 某个类中所有同名方法（重载方法）

- replace：用于完全替换指定方法的实现

所有 Hook 形式均支持新式配置表语法，这种语法提高了可读性并使其参数更明确。旧的函数参数列表语法依然保留，以保持向后兼容性。

before和after钩子函数（以及replace的替换函数）均接收一个参数it（即MethodHookParam），其中包含当前 Hook 的上下文信息。

### 1.hook—— Hook 单个方法​

hook用于对单个 Java 方法进行拦截。它支持前置 (before) 和后置 (after) 钩子。

#### 推荐新语法​

新语法通过一个 Luatable来组织 Hook 的所有参数，提高了可读性和灵活性。before和after钩子都是可选的，你只写需要的即可。

用法一：通过类名、方法名和参数类型 Hook 方法

```lua

hook {

class = "com.xx",           -- 必填：目标类的全限定名字符串 或 Class 类型的类对象

--classloader = lpparam.classLoader, -- 可选：用于加载类的 ClassLoader，不填默认 lpparam.classLoader

method = "f1",              -- 必填：要 Hook 的方法名

params = {"int", "String", findClass("com.my.class", lpparam.classLoader)}, -- 可选：一个 table，包含参数类型字符串或 Class 对象

before = function(it)

-- before hook：在原方法执行前执行

print("进入 f1 方法前")

end,

after = function(it)

-- after hook：在原方法执行后执行

print("离开 f1 方法后，返回值:", it.result)

end,

}

```

用法二：直接 HookMethod对象

如果你已经通过DexKit或其他方式获取了Method实例，可以直接传入。

```lua

local myMethod = someMethodRetrievalFunction("com.xx", "f1", "String") -- 假设这是获取 Method 对象的方法

hook {

method = myMethod,          -- 必填：Method 类型的量

before = function(it)

print("Method 对象 Hook 前")

end,

-- after 可选，此处省略

}

```

#### 旧语法​

旧语法作为兼容性选项保留。

用法一：通过类名、类加载器、方法名、参数类型列表 Hook 方法

```lua

hook("com.xx",

lpparam.classLoader,

"f1",

"int", "String", "com.my.class", -- 参数类型可变参数

function(it)

-- before

end,

function(it)

-- after

end

)

```

用法二：直接 Hook 方法对象

```lua

hook(method, -- 直接传入 Method 对象

function(it)

-- before

end,

function(it)

-- after

end

)

```

### 2.hookctor—— Hook 构造方法（Constructor）​

hookctor用于拦截类的构造方法。它没有method属性，因为目标总是构造函数。

#### 推荐新语法​

```lua

hookctor {

class = "com.xx",           -- 必填：目标类的全限定名字符串 或 Class 类型的类对象

classloader = lpparam.classLoader, -- 可选：用于加载类的 ClassLoader，不填默认 lpparam.classLoader

params = {"int", "String"}, -- 可选：一个 table，包含构造函数的参数类型

before = function(it)

print("构造函数调用前")

end,

after = function(it)

print("构造函数调用后")

end,

}

```

#### 旧语法​

```lua

hookctor("com.xx",

lpparam.classLoader,

"int", "String", -- 构造函数参数类型

function(it)

-- before

end,

function(it)

-- after

end

)

```

### 3.hookAll—— Hook 所有同名方法​

hookAll用于 Hook 某个类中所有同名（重载）方法。它不支持method属性或直接传入Method对象，因为其目标是所有重载。

#### 推荐新语法​

```lua

hookAll {

class = "com.xx",           -- 必填：目标类的全限定名字符串 或 Class 类型的类对象

classloader = lpparam.classLoader, -- 可选：用于加载类的 ClassLoader，不填默认 lpparam.classLoader

before = function(it)

print("Hook 所有重载方法前，当前方法:", it.method.getName())

end,

after = function(it)

print("Hook 所有重载方法后，当前方法:", it.method.getName(), "返回值:", it.result)

end,

}

```

#### 旧语法（函数参数列表形式）​

```lua

hookAll("com.xx",

lpparam.classLoader,

function(it)

-- before

end,

function(it)

-- after

end

)

```

### 4.replace—— 完全替换方法实现​

replace方法用于完全取代目标方法的原始实现。一旦方法被replace，其原有的功能将不再执行，而是由你传入的替换函数来接管。replace没有before和after钩子，因为它本身就是终极替换。

#### 推荐新语法（配置表形式）​

```lua

replace {

class = "com.xx",           -- 必填：目标类的全限定名字符串 或 Class 类型的类对象

classloader = lpparam.classLoader, -- 可选：用于加载类的 ClassLoader，不填默认 lpparam.classLoader

method = "f1",              -- 必填：要替换的方法名

params = {"int", "String"}, -- 可选：一个 table，包含参数类型字符串或 Class 对象

replace = function(it)

-- 这是替换后的方法体，完全取代原方法

print("f1 方法被完全替换了！接收参数:", table.concat(it.args, ", "))

-- 替换函数的返回值将成为被替换方法的最终返回值

return "新的返回值"

end,

}

```

用法二：直接替换Method对象

```lua

local targetMethod = someMethodRetrievalFunction("com.yy", "calculate")

replace {

method = targetMethod,      -- 必填：Method 类型的量

replace = function(it)

print("calculate 方法被替换，原始 this 对象:", it.thisObject)

return it.args[0] + it.args[1] -- 假设原方法是计算两个参数的和

end,

}

```

#### 旧语法（函数参数列表形式）​

```lua

replace("com.xx",

lpparam.classLoader,

"f1",

"int", "String",

function(it)

-- 这是替换后的方法体

print("f1 方法被替换了！参数:", table.concat(it.args, ", "))

return 123 -- 返回值会成为被替换方法的最终返回值

end

)

```

### it参数说明（即MethodHookParam）​

> 可以简单地理解成函数的相关信息。

无论是hook的before/after钩子，还是replace方法的替换函数，都会接收到一个it参数（即MethodHookParam实例），它提供了当前 Hook 或替换操作的上下文信息：

- it.method: 当前 Hook/替换的java.lang.reflect.Method或Constructor对象。你可以通过it.method.getName()获取方法名，it.method.getDeclaringClass()获取所属类等。

- it.args：一个 Luatable，包含了目标方法的参数数组。it.args[0]：获取第一个参数。#it.args：获取参数个数。it.args[0] = 1：修改参数值。此操作仅在before钩子中对后续的原方法执行有效。在replace函数中修改it.args不会影响任何原始方法调用（因为原方法不会被执行）。

- it.args[0]：获取第一个参数。

- #it.args：获取参数个数。

- it.args[0] = 1：修改参数值。此操作仅在before钩子中对后续的原方法执行有效。在replace函数中修改it.args不会影响任何原始方法调用（因为原方法不会被执行）。

- it.result：it.result：获取返回值。主要在after钩子中用于获取原方法的执行结果。it.result = true：修改返回值。在before钩子中赋值it.result将会直接跳过原方法的执行，并使用it.result的值作为该方法的最终返回值。在after钩子中赋值it.result将会覆盖原方法已经产生的结果，作为该方法的最终返回值。在replace方法中，直接return替换函数的值即可作为最终返回值，it.result在此场景下不用于设定返回值。

- it.result：获取返回值。主要在after钩子中用于获取原方法的执行结果。

- it.result = true：修改返回值。在before钩子中赋值it.result将会直接跳过原方法的执行，并使用it.result的值作为该方法的最终返回值。在after钩子中赋值it.result将会覆盖原方法已经产生的结果，作为该方法的最终返回值。在replace方法中，直接return替换函数的值即可作为最终返回值，it.result在此场景下不用于设定返回值。

- 在before钩子中赋值it.result将会直接跳过原方法的执行，并使用it.result的值作为该方法的最终返回值。

- 在after钩子中赋值it.result将会覆盖原方法已经产生的结果，作为该方法的最终返回值。

- 在replace方法中，直接return替换函数的值即可作为最终返回值，it.result在此场景下不用于设定返回值。

- it.thisObject：当前方法所属的对象实例（即 Java 中的this）。对于非静态方法，你可以通过it.thisObject访问该对象的字段、调用其其他方法等。对于静态方法或构造函数（在after构造函数中it.thisObject代表新创建的对象实例），it.thisObject可能为nil或代表类本身。

- 对于非静态方法，你可以通过it.thisObject访问该对象的字段、调用其其他方法等。

- 对于静态方法或构造函数（在after构造函数中it.thisObject代表新创建的对象实例），it.thisObject可能为nil或代表类本身。

---

<!-- 页面 3: 003_manual__lpparam.md -->

## lpparam（LoadPackageParam）介绍 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/lpparam.html](https://luahook.kulipai.top/manual/lpparam.html)

**处理顺序**: 第3个

---

## lpparam（LoadPackageParam）介绍​

lpparam是 Xposed 模块中用于代表当前 Hook 宿主应用上下文的参数对象，完整类型为XC_LoadPackage.LoadPackageParam。

可以理解为：当前被 Hook 的 App 运行时信息容器。

它通常作为 Hook 回调函数中的参数，用于获取宿主包的信息与加载相关类。

#### 常见字段说明​

| 字段名 | 类型 | 含义说明 |

| --- | --- | --- |

| `appInfo` | `ApplicationInfo` | 宿主 App 的安装信息对象（可获取路径、签名等） |

| `classLoader` | `ClassLoader` | 当前宿主的默认类加载器（**Hook 类核心入口**） |

| `isFirstApplication` | `boolean` | 是否首个加载的app，在加载真实的app前，部分手机(小米)会额外加载应用包管理程序和什么程序来着，要用这个属性才能确保是加载了目标的app |

| `packageName` | `String` | 宿主应用的包名 |

| `processName` | `String` | 当前进程名（可用于区分主/子进程） |

#### 常用用法总结​

```lua

-- 获取类加载器，用于 findClass 或 hook 类

local loader = lpparam.classLoader

-- 获取宿主包名，用于判断目标 App

local pkg = lpparam.packageName

-- 获取安装包路径，适用于读取宿主资源或路径验证

local apkPath = lpparam.appInfo.sourceDir

```

---

<!-- 页面 4: 004_manual__field.md -->

## 字段操作（Field 操作） | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/field.html](https://luahook.kulipai.top/manual/field.html)

**处理顺序**: 第4个

---

## 字段操作（Field 操作）​

在 Hook 过程中，除了方法的调用，我们也经常需要对类的字段进行读取或修改。本模块提供以下四个函数，用于访问对象或类的字段信息：

#### 1.getField—— 获取实例字段​

```lua

getField(实例, "字段名")

```

##### 功能说明：​

- 获取对象实例中的成员字段（包括public、private、protected等所有修饰符）

##### 示例：​

```lua

local activityTitle = getField(activityInstance, "mTitleTextView")

```

#### 2.getStaticField—— 获取静态字段​

```lua

getStaticField(类, "字段名")

```

##### 功能说明：​

- 适用于访问类级别的static字段

##### 示例：​

```lua

local sdkInt = getStaticField(Build, "SDK_INT")

```

#### 3.setField—— 修改实例字段​

```lua

setField(实例, "字段名", 新值)

```

##### 功能说明：​

- 动态设置对象的字段值

- 支持私有字段修改

##### 示例：​

```lua

setField(view, "mPaddingTop", 20)

```

#### 4.setStaticField—— 修改静态字段​

```lua

setStaticField(类, "字段名", 新值)

```

##### 功能说明：​

- 用于修改静态字段的值

##### 示例：​

```lua

setStaticField(Builds, "SDK_INT", 99)

```

---

<!-- 页面 5: 005_manual__import.md -->

## 导入和寻找类 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/import.html](https://luahook.kulipai.top/manual/import.html)

**处理顺序**: 第5个

---

## 导入和寻找类​

1. 查找类（findClass）

2. 导入类（import/imports）

#### 1.findClass—— 查找类​

```lua

local clazz = findClass("类全名", 加载器)

```

##### 参数说明：​

- 类全名：Java 类的全限定名，例如"android.app.Activity"

- 加载器（可选）：指定使用的类加载器，默认为lpparam.classLoader

#### 2.imports—— 导入类（宿主或模块）​

```lua

imports "android.os.Build"

local device = Build.DEVICE

```

##### 功能说明：​

- 将 Java 类导入为全局变量，可直接使用类名访问其静态字段或方法

- 搜索顺序：优先从宿主应用中加载类若未找到，再从模块本身中加载类

- 优先从宿主应用中加载类

- 若未找到，再从模块本身中加载类

#### 3.import—— 导入模块类（支持通配符）​

```lua

import "java.lang.String"

import "java.util.*"

```

##### 功能说明：​

- 仅用于导入模块 APK 自带的类

- 支持通配符\*，一次导入整个包中的所有类

---

<!-- 页面 6: 006_manual__invoke.md -->

## 类的构造与方法调用 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/invoke.html](https://luahook.kulipai.top/manual/invoke.html)

**处理顺序**: 第6个

---

## 类的构造与方法调用​

#### 1. 类的构造​

```lua

local a = clazz()

local b = claxx(1, "aaa")

```

- 直接通过类()进行实例化

- 构造函数参数可直接传入

#### 2. 方法调用​

##### 调用静态方法​

```lua

clazz.func()

```

- 使用类.方法()调用静态函数

##### 调用非静态方法​

```lua

clazz().func()

```

- 使用实例.方法()调用非静态函数

##### 调用私有方法​

```lua

invoke(类或实例, "方法名", 参数...)

```

- 使用invoke调用私有方法或无法直接访问的方法

- 支持静态方法和实例方法，参数直接追加在后面

---

<!-- 页面 7: 007_manual__native__memory_alloc.md -->

## 内存管理 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/memory_alloc.html](https://luahook.kulipai.top/manual/native/memory_alloc.html)

**处理顺序**: 第7个

---

## 内存管理​

native.memory提供了手动管理堆内存的能力。请务必注意内存泄漏问题，手动申请的内存必须手动释放。

### 申请内存 (alloc)​

申请指定大小的内存块。

```lua

native.memory.alloc(size) -> LuaPointer

```

- 参数:size(number) —— 字节数。

- 返回:LuaPointer对象。如果size <= 0则返回nil或空指针对象。

示例：

```lua

local ptr = native.memory.alloc(1024) -- 申请 1KB

if ptr.is_null() then

log("内存申请失效")

end

```

### 申请字符串内存 (alloc_utf8_string)​

申请一块内存，并将 Lua 字符串按照 UTF-8 编码写入，末尾自动添加\0。

```lua

native.memory.alloc_utf8_string(str) -> LuaPointer

```

- 参数:str(string) —— 内容。

- 返回: 指向字符串首地址的LuaPointer。

示例：

```lua

local str_ptr = native.memory.alloc_utf8_string("Hello World")

-- 此时内存中为：48 65 6C 6C 6F 20 57 6F 72 6C 64 00

```

### 释放内存 (free)​

释放由alloc系列函数申请的内存。

```lua

native.memory.free(ptr)

```

- 参数:ptr(LuaPointer | number) —— 要释放的内存首地址。

警告:

1. 只能释放自己申请的内存。

2. 禁止对同一块内存重复释放（Double Free）。

3. 禁止释放未经申请的野指针。

示例：

```lua

local buf = native.memory.alloc(16)

-- 使用 buf ...

native.memory.free(buf)

```

---

<!-- 页面 8: 008_manual__native__memory_io.md -->

## 内存读写 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/memory_io.html](https://luahook.kulipai.top/manual/native/memory_io.html)

**处理顺序**: 第8个

---

## 内存读写​

native.memory提供了一套全面的内存读写接口，支持整数、浮点数和原始字节数组。

所有接口的addr参数均支持：

- LuaPointer对象

- number(地址数值)

- string(十六进制字符串)

### 读写整数​

#### 基础类型表​

| 位宽 | 类型 | 读取接口 | 写入接口 | 范围/备注 |

| --- | --- | --- | --- | --- |

| **8位** | 无符号 | `read_u8` | `write_u8` | 0 ~ 255 |

| 有符号 | `read_s8` | `write_s8` | -128 ~ 127 | |

| **16位** | 无符号 | `read_u16` | `write_u16` | 0 ~ 65535 |

| 有符号 | `read_s16` | `write_s16` | -32768 ~ 32767 | |

| **32位** | 无符号 | `read_u32` | `write_u32` | Lua number (UInt32) |

| 有符号 | `read_s32` | `write_s32` | Lua number (Int32) | |

| **64位** | 无符号 | `read_u64` | `write_u64` | **返回 LuaPointer**以保精度 |

| 有符号 | `read_s64` | `write_s64` | **返回 LuaPointer**以保精度 | |

| **指针** | 指针宽 | `read_ptr` | `write_ptr` | 自动适配 32/64 位架构 |

#### 接口定义​

读取：

```lua

native.memory.read_u32(addr, [offset]) -> number

```

- offset: 可选，基于addr的偏移量，默认为 0。

写入：

```lua

native.memory.write_u32(addr, value, [offset]) -> boolean

```

- value: 要写入的数值。

- 返回:true表示写入成功。

示例：

```lua

local buf = native.memory.alloc(16)

-- 写入

native.memory.write_u32(buf, 0x1234, 0)

native.memory.write_s64(buf, 0x1122334455667788, 4)

-- 读取

local val = native.memory.read_u32(buf, 0) -- 0x1234

local val64 = native.memory.read_s64(buf, 4) -- 返回 LuaPointer

native.memory.free(buf)

```

### 读写浮点数​

| 类型 | 读取接口 | 写入接口 |

| --- | --- | --- |

| Float (32位) | `read_f32` | `write_f32` |

| Double (64位) | `read_f64` | `write_f64` |

示例：

```lua

native.memory.write_f32(addr, 3.14, 0)

local pi = native.memory.read_f32(addr, 0)

```

### 读写字节数组​

用于批量读取或写入原始二进制数据。

#### 读取数组 (read_byte_array)​

```lua

native.memory.read_byte_array(addr, size, [offset]) -> table | nil

```

- size: 读取字节数。

- 返回: Lua table，包含[1..size]个0-255的整数。

#### 写入数组 (write_byte_array)​

```lua

native.memory.write_byte_array(addr, table, [offset]) -> boolean

```

- table: 包含字节数据的 Lua table。

示例：

```lua

-- 写入 HEX: 41 42 43

native.memory.write_byte_array(addr, {0x41, 0x42, 0x43})

-- 读取

local bytes = native.memory.read_byte_array(addr, 3)

-- bytes[1] == 65

```

---

<!-- 页面 9: 009_manual__native__memory_string.md -->

## 字符串操作 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/memory_string.html](https://luahook.kulipai.top/manual/native/memory_string.html)

**处理顺序**: 第9个

---

## 字符串操作​

native.memory提供了一组工具来处理 Native 内存中的字符串，涵盖 C 风格字符串和长度前缀字符串。

### C 风格字符串 (\0结尾)​

#### 读取 (read_cstr)​

```lua

native.memory.read_cstr(addr, [max_len]) -> string | nil

native.memory.read_cstr_utf8(addr, [max_len]) -> string | nil

```

- 区别：read_cstr按原始字节读取，read_cstr_utf8尝试做 UTF-8 解码。

- max_len: 可选，最大读取长度（防溢出），默认 512。

- 返回: Lua字符串，遇到\0截断。

#### 写入 (write_cstr)​

```lua

native.memory.write_cstr(addr, str) -> boolean

native.memory.write_cstr_utf8(addr, str) -> boolean

```

- 会自动在字符串末尾补\0。

示例：

```lua

local buf = native.memory.alloc(64)

native.memory.write_cstr_utf8(buf, "你好, LuaHook")

local txt = native.memory.read_cstr_utf8(buf)

log(txt) -- "你好, LuaHook"

native.memory.free(buf)

```

### 长度前缀字符串 (Len-Prefixed)​

处理首字节为长度的自定义字符串格式。

#### 读取 (read_lp_utf8)​

```lua

native.memory.read_lp_utf8(addr, [max_len]) -> string | nil

```

- 格式:[Length (1 byte)] [Body ...]

- 读取第一个字节len，然后读取后续len个字节并转为 UTF-8。

#### 写入 (write_lp_utf8)​

```lua

native.memory.write_lp_utf8(addr, str, [max_len]) -> boolean

```

- 写入长度字节，随后写入内容。

### 自动识别 (read_auto_utf8)​

```lua

native.memory.read_auto_utf8(addr, [max_len]) -> string | nil

```

逻辑：

1. 检查首字节是否在[1..0x7F]范围内。

2. 尝试该长度的内存内容是否符合 UTF-8 打印字符特征。

3. 若符合则按长度前缀解析；否则回退到read_cstr模式。

---

<!-- 页面 10: 010_manual__native__pointer.md -->

## 指针操作 (LuaPointer) | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/pointer.html](https://luahook.kulipai.top/manual/native/pointer.html)

**处理顺序**: 第10个

---

## 指针操作 (LuaPointer)​

LuaPointer是 NativeHook 中的核心对象，用于封装和操作内存地址。它比简单的 Lua number 更安全，支持链式调用，且能保持 64 位精度。

### 创建指针​

#### 使用native.ptr​

```lua

local p = native.ptr(0x12345678)

local p2 = native.ptr("0x1A2B3C")

```

#### 使用native.memory.alloc​

```lua

local p3 = native.memory.alloc(32)

```

### 基础操作​

#### 判空​

```lua

p.is_null()  -- 检查是否为 Null (0)

p.not_null() -- 检查是否非空

```

#### 转换​

```lua

p.to_hex()  -- 转为大写 HEX 字符串 (无 0x 前缀)

p.to_int()  -- 转为 number (可能丢失 64 位精度)

p.to_long() -- 返回自身 (兼容性接口)

```

### 指针运算​

所有运算返回新的LuaPointer对象，原对象不变。

#### 偏移计算​

```lua

local p2 = p.add(0x10) -- 加法

local p3 = p.sub(0x04) -- 减法

```

#### 位运算​

```lua

p.and(mask) -- 按位与

p.or(mask)  -- 按位或

p.xor(mask) -- 按位异或

p.shl(bits) -- 左移

p.shr(bits) -- 右移

```

#### 比较​

```lua

if p.equals(other_ptr) then

log("地址相同")

end

```

#### 赋值 (Set)​

```lua

-- 创建一个指向新地址的指针对象

local new_ptr = p.set(0x5566)

```

### 链式读写​

LuaPointer对象自身携带了所有native.memory.\*的读写方法。调用时，默认地址为指针本身，offset参数依然可用。

```lua

local p = native.memory.alloc(64)

-- 等价于 native.memory.write_u32(p, 100, 0)

p.write_u32(100)

-- 支持偏移：等价于 native.memory.write_u32(p, 200, 4)

p.write_u32(200, 4)

-- 链式解引用

-- 读取 p 处的指针，然后偏移 0x10，再读取一个 u32

local val = p.read_ptr().add(0x10).read_u32()

```

#### 常用快捷方法​

| 方法 | 描述 |

| --- | --- |

| `p.deref([offset])` | 解引用。读取`p`处的指针值。等价于`p.read_ptr(offset)` |

| `p.hexdump([size])` | 打印内存 Hex Dump，方便调试。默认 size=256 |

---

<!-- 页面 11: 011_manual__native__module.md -->

## 模块与符号 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/module.html](https://luahook.kulipai.top/manual/native/module.html)

**处理顺序**: 第11个

---

## 模块与符号​

在 Hook 之前，通常需要先获取目标 SO (Shared Object) 的内存基址，或直接查找导出函数地址。

### 获取模块基址 (native.module_base)​

获取已加载模块的起始地址。

```lua

native.module_base(module_name) -> LuaPointer

```

- 参数:module_name(string) —— 完整模块名，如libil2cpp.so。

- 返回: 模块基址。若未找到模块，通常返回0(null)。

别名兼容：native.get_module_base(name)与此功能相同。

示例：

```lua

local base = native.module_base("libunity.so")

if base.not_null() then

log("Unity Base: " .. base.to_hex())

end

```

### 解析导出符号 (native.resolve_symbol)​

查找 SO 导出表（Export Table）中的符号地址。

```lua

native.resolve_symbol(module_name, symbol_name) -> number

```

- 参数:module_name: 模块名。symbol_name: 符号名（C++ 函数可能是 Mangled Name）。

- module_name: 模块名。

- symbol_name: 符号名（C++ 函数可能是 Mangled Name）。

- 返回: 符号的绝对地址 (number)。未找到返回 0。

示例：

```lua

local fopen_addr = native.resolve_symbol("libc.so", "fopen")

log("fopen address: " .. string.format("0x%X", fopen_addr))

```

### 高级查找 (getModuleBase)​

此接口保留用于更复杂的/proc/self/maps匹配。

```lua

native.getModuleBase(name, filter) -> number

```

- Feature 1: 支持查找 BSS 段。name传入libfoo.so:bss，可查找该 SO 的 BSS 段地址。

- name传入libfoo.so:bss，可查找该 SO 的 BSS 段地址。

- Feature 2:filter过滤。传入权限字符串（如r-xp）来精确匹配内存段。

- 传入权限字符串（如r-xp）来精确匹配内存段。

```lua

-- 获取 libfoo.so 的 r-xp 段

local code_base = native.getModuleBase("libfoo.so", "r-xp")

```

---

<!-- 页面 12: 012_manual__native__hook.md -->

## Hook 接口 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/hook.html](https://luahook.kulipai.top/manual/native/hook.html)

**处理顺序**: 第12个

---

## Hook 接口​

native.hook是 NativeLayor 的核心功能，支持对任意内存地址的函数进行 Inline Hook。

### 接口定义​

```lua

native.hook(target_addr, config) -> boolean

```

- target_addr: 目标函数绝对地址。可以通过基址+偏移算得，或通过符号解析得到。

- config: Hook 配置表。

#### Config 结构​

```lua

{

ret = "void",       -- [可选] 返回值类型

argc = 0,           -- [可选] 参数个数（影响栈参数读取）

onEnter = function(ctx) ... end, -- [可选] 进入回调

onLeave = function(retval) ... end -- [可选] 离开回调

}

```

- ret 类型支持:"int","ptr","pointer","float","double","void"。

### onEnter 回调​

函数执行前调用。

```lua

onEnter = function(ctx)

-- ctx 是上下文对象

end

```

#### 上下文对象 (ctx)​

| 属性 | 描述 |

| --- | --- |

| `ctx[i]` | 获取第`i`个通用寄存器 (GPR) 参数。**注意：这是寄存器索引，不是参数索引**。ARM64 下`ctx[0]`~`ctx[7]`对应参数 1~8。 |

| `ctx.raw` | 访问原始寄存器数组。 |

| `ctx.fpr` | 浮点寄存器数组。 |

| `ctx.stack` | 栈参数数组。 |

修改参数示例：

```lua

-- 修改第一个整型参数 (ARM64 x0)

ctx[0] = 100

-- 修改第二个参数指针偏移

ctx[1] = ctx[1].add(0x20)

```

### onLeave 回调​

函数执行后调用。

```lua

onLeave = function(retval)

-- retval: 原始返回值

-- return: (可选) 新的返回值

end

```

修改返回值示例：

```lua

onLeave = function(orig_ret)

log("Original return: " .. orig_ret)

return 1 -- 强制返回 1

end

```

### 完整示例​

假设我们要 Hooklibgame.so偏移0x1234处的函数int add(int a, int b)。

```lua

local base = native.module_base("libgame.so")

if base.is_null() then return end

native.hook(base.add(0x1234), {

ret = "int",

onEnter = function(ctx)

local a = ctx[0].to_int() -- ARM64 第1个参数在 x0

local b = ctx[1].to_int() -- ARM64 第2个参数在 x1

log("add called with: " .. a .. ", " .. b)

-- 强行把 b 改成 999

ctx[1] = 999

end,

onLeave = function(ret)

log("result: " .. ret)

return ret * 2 -- 结果翻倍

end

})

```

---

<!-- 页面 13: 013_manual__native__invoke.md -->

## Native函数调用 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/invoke.html](https://luahook.kulipai.top/manual/native/invoke.html)

**处理顺序**: 第13个

---

## Native函数调用​

### 1. 创建Native函数对象​

使用native.new_function为Native函数地址创建一个包装器。

```lua

local func = native.new_function(address, returnType, paramTypes)

```

- address: Native函数的内存地址（Long 类型）。

- returnType: 返回类型字符串。支持的类型有：void、int、float、double和pointer。

- paramTypes: 包含参数类型字符串的表。支持的类型有：int、float、double和pointer。

示例：

```lua

local base = native.resolve_symbol("libexample.so", "some_exported_func")

local func = native.new_function(base, "void", {"int", "int"})

```

### 2. 调用函数​

函数对象创建完成后，即可直接调用。

```lua

func(arg1, arg2...)

```

- 直接传递参数。

- 返回值与指定的returnType匹配。

示例：

```lua

func(10, 20)

```

---

<!-- 页面 14: 014_manual__native__legacy.md -->

## 兼容性说明 | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/native/legacy.html](https://luahook.kulipai.top/manual/native/legacy.html)

**处理顺序**: 第14个

---

## 兼容性说明​

随着 LuaHook 版本的迭代，Native 接口经历了多次重构。本页汇总了旧接口的去留情况以及升级指南。

### 已移除的旧接口​

以下接口在最新版中已被移除或不推荐使用，请使用native.memory替代：

| 旧接口 | 替代方案 | 说明 |

| --- | --- | --- |

| `native.read(ptr, size)` | `native.memory.read_byte_array` | 读取字节数组 |

| `native.write(ptr, data)` | `native.memory.write_byte_array` | 写入字节数组 |

| `native.readDword(ptr)` | `native.memory.read_u32` | 读 32位整数 |

| `native.readFloat(ptr)` | `native.memory.read_f32` | 读 Float |

| `native.readByte(ptr)` | `native.memory.read_u8` | 读 Byte |

(对应 write 系列同理)

### 保留的兼容接口​

为了兼容旧脚本，以下顶级接口仍被保留，但建议新脚本使用规范的新接口：

- native.get_module_base推荐使用native.module_base。

- 推荐使用native.module_base。

- native.readPoint(ptr, offsets)多级指针读取工具。逻辑: 从ptr开始，依次读取偏移并解引用，直到最后一个偏移只相加不解引用。lua-- 等价于 \*(base + 0x10) + 0x20localval=native.readPoint(base, {0x10,0x20})

- 多级指针读取工具。

- 逻辑: 从ptr开始，依次读取偏移并解引用，直到最后一个偏移只相加不解引用。

- lua-- 等价于 \*(base + 0x10) + 0x20localval=native.readPoint(base, {0x10,0x20})

### 注意事项​

1. 64位精度: Lua 的number(double) 无法精确表示所有 64 位整数。处理指针地址或 64 位数值时，请务必使用read_u64/read_s64返回的LuaPointer对象。

2. Hook 稳定性: Hook 可能会导致目标应用崩溃（如修改了关键寄存器或访问非法内存）。编写 Hook 时请务必进行充分测试。

---

<!-- 页面 15: 015_manual__json.md -->

## JSON | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/json.html](https://luahook.kulipai.top/manual/json.html)

**处理顺序**: 第15个

---

## JSON​

用于对字符串进行json编码解码

#### json.decode(String)​

返回值:LuaTable

#### json.encode(LuaTable)​

返回值:String

---

<!-- 页面 16: 016_manual__task.md -->

## Task | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/task.html](https://luahook.kulipai.top/manual/task.html)

**处理顺序**: 第16个

---

## Task​

用于在指定时间后异步执行一段 Lua 函数。

#### Task(function [, delay])​

参数:

- function: 需要执行的 Lua 函数

- delay(可选): 延迟时间（毫秒），默认为0

返回值:

- nil

说明:在后台启动一个协程，等待delay毫秒后调用function。

#### 示例​

```lua

-- 立即执行一次

Task(function()

print("Hello from Task!")

end)

-- 2秒后执行

Task(function()

print("This runs after 2 seconds")

end, 2000)

-- 连续调度多个

for i = 1, 3 do

Task(function()

print("Task #" .. i)

end, i * 1000) -- 1秒后、2秒后、3秒后依次执行

end

```

---

<!-- 页面 17: 017_manual__file.md -->

## 📂 LuaFile | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/file.html](https://luahook.kulipai.top/manual/file.html)

**处理顺序**: 第17个

---

## 📂 LuaFile​

文件操作库，用于读写、删除、复制、移动文件等。

#### file.isFile(path)​

返回值: boolean是否为文件。

#### file.isDir(path)​

返回值: boolean是否为文件夹。

#### file.isExists(path)​

返回值: boolean文件或文件夹是否存在。

#### file.read(path)​

返回值: string | nil读取文件文本内容。

#### file.readBytes(path)​

返回值: string | nil读取文件二进制内容（按字节）。

#### file.write(path, content)​

返回值: boolean写入文本，覆盖原文件。

#### file.writeBytes(path, content)​

返回值: boolean写入二进制，覆盖原文件。

#### file.append(path, content)​

返回值: boolean追加写入文本。

#### file.appendBytes(path, content)​

返回值: boolean追加写入二进制。

#### file.copy(from, to)​

返回值: boolean复制文件，若存在则覆盖。

#### file.move(from, to)​

返回值: boolean移动文件，若存在则覆盖。

#### file.rename(path, newName)​

返回值: boolean重命名文件。

#### file.delete(path)​

返回值: boolean删除文件。

#### file.getName(path)​

返回值: string获取文件名。

#### file.getSize(path)​

返回值: number获取文件大小（字节数）。

---

<!-- 页面 18: 018_manual__http.md -->

## 🌐 LuaHttp | LuaHook

**页面地址**: [https://luahook.kulipai.top/manual/http.html](https://luahook.kulipai.top/manual/http.html)

**处理顺序**: 第18个

---

## 🌐 LuaHttp​

HTTP 请求库，用于 GET / POST / 下载 / 上传。

#### http.get(url [, headers, cookie, timeout], callback)​

返回值: nil发送 GET 请求。

- url: 请求地址

- headers(可选): 表头（LuaTable）

- cookie(可选): Cookie 字符串

- timeout(可选): 超时时间（毫秒，默认 10000）

- callback: 回调函数，格式function(success, body, code)

#### http.post(url, body [, headers, cookie, timeout], callback)​

返回值: nil发送 POST 请求。

- body: 请求体（字符串或 LuaTable）

- 其余参数与http.get相同。

#### http.download(url, path [, headers, cookie, timeout], callback)​

返回值: nil下载文件并保存到指定路径。

- path: 保存路径

- callback:function(success, pathOrError, code)

#### http.upload(url, path [, headers, cookie, timeout], callback)​

返回值: nil上传文件。

- path: 文件路径

- callback:function(success, body, code)

#### 示例​

```lua

-- GET 请求

http.get("https://httpbin.org/get", function(success, body, code)

print(success, code)

print(body)

end)

-- POST 请求

http.post("https://httpbin.org/post", { name = "kulipai" }, function(success, body, code)

print("POST:", success, code)

end)

-- 下载文件

http.download("https://example.com/file.zip", "/sdcard/file.zip", function(success, path, code)

if success then

print("下载完成:", path)

else

print("失败:", path, code)

end

end)

-- 上传文件

http.upload("https://example.com/upload", "/sdcard/file.zip", function(success, body, code)

print("上传结果:", success, code)

end)

```

---

<!-- 页面 19: 019_luaj__.md -->

## Luaj++ | LuaHook

**页面地址**: [https://luahook.kulipai.top/luaj++.html](https://luahook.kulipai.top/luaj++.html)

**处理顺序**: 第19个

---

## Luaj++​

### 简介​

nirenr的luaj改版，增加对java更好的支持，以下内容来自NeLuaj提供的Luaj++参考手册，希望对您了解luaj++有帮助

### 入口文件​

Activitymain.lua

Serviceservice.lua

AccessibilityServiceaccessibility.lua

NotificationListenerServicenotification.lua

WallpaperServicewallpaper.lua

服务可以使用setLuaDir(dir)设置运行目录，setEnabled(context)打开启动服务设置界面，getInstance()获取服务实例。

### 可省略非必要关键字​

- 省略then

```lua

if a then

end

-->

if a

end

```

- 省略do

```lua

while a do

end

-->

while a

end

```

- 省略in

```lua

for k,v in pairs(t) do

end

-->

for k,v pairs(t)

end

```

- 省略function

```lua

local function a()

end

-->

local a()

end

```

- 支持switch

```lua

switch a

case 1,3,5,7,9

print(1)

case 2,4,6,8

print(2)

case 0

print(0)

default

print(nil)

end

```

- 支持when

```lua

a = when a

case 1,3,5,7,9

return 1

case 2,4,6,8

return 2

case 0

return 0

default

return nil

end

```

- 支持continue

```lua

for n = 1,10

if n%2 == 0

continue

end

print(n)

end

```

### 支持foreach​

```lua

for k,v : t

end

for k,v in t

end

```

### 支持defer​

defer后语句将在函数结束时运行 多个defer将按照后入先出原则运行。

### 支持?操作符​

```lua

?a print(1)`print(2)

a = ?a print(1)`print(2)

```

### 支持三目 if​

```lua

b = if a 1 else 2

print(b)

```

### 支持try-catch-finally​

```lua

try

error("err")

catch(e)

print("catch", e)

finally

print("finally")

end

```

** 支持lambda，可以使用反斜杠代替lambda关键字 **

```lua

lambda a,b->a+b

lambda a,b=>print(a+b)

lambda a,b:print(a+b)

lambda () -> print("lambda")

```

### 支持import​

```lua

import "package"

--将导入包并设置为局部变量

import "java.lang.String"

--返回值为 javaClass

import "java.lang.*"

--返回值为 javaPackage

import str "java.lang.String"

--设置别名

import "java.lang. *", "java.io.* "

--一次性导入多个包或类

```

支持module

module自带环境，默认设置环境表的metatable为自己

module "name"

支持自赋值local

local:print

将全局print设置为局部print

运算符优化

```lua

!= 可代替 ~=

！ 可代替 not

&& 可代替 and

|| 可代替 or

```

### 支持位运算​

- 按位与 a=1&2

- 按位或 a=1|2

- 按位异或 a=1~2

- 右移 a=1>>8

- 左移 a=8<<2

- 按位非 a=~2

### 支持64位整数​

```lua

i=0xffffffffff

```

支持+= -= \*= /= %= ^= //= &= |= ~= <<= >>= ..=运算

```lua

a+=1

a-=1

a*=1

a/=1

```

### 调用java优化​

- javaClass 拓展函数/属性

```lua

Object.array{} -- 创建数组

print(Object.new) -- 类的构造器

print(Object.class) -- 获取类本身

Object.override{} -- 覆盖方法

```

- 直接()构建实例或实现接口,抽象类

```lua

b = ArrayList()

m = HashMap()

i = interface {

methodName=function(arg)

end

}

c = abstract {

methodName=function(super, arg)

end

}

```

- 支持覆盖方法

```lua

list = ArrayList.override {

function add(superCall, arg)

superCall(arg)

end

}()

list = ArrayList {

add = function(s, a)

end

}

```

- 支持元方法

```lua

function Button:print()

print(self)

end

Button(this):print()

```

- 支持批量设置属性

```lua

Button(this) {

text="test",

enabled=false

}

```

- 直接创建数组

```lua

i=int[10]

i=int{1,2,3}

i=Integer[10]

```

- java 方法使用.调用

```lua

b.add(!)

```

- is 方法简写

```lua

view.isActivated()

-->

view.Activated

```

- java getter/setter优化

```lua

b.setText("")

-->

b.text=""

m.abc=1

t=b.getText()

-->

t=b.text

t=m.abc

```

- 语法糖示例

```lua

mBtn.setOnClickListener(View.OnClickListener {

onClick = function(v)

print(v)

end

})

--> 忽略接口类型

mBtn.setOnClickListener({

onClick = function(v)

print(v)

end

})

--> 简写函数式接口

mBtn.setOnClickListener(function(v)

print(v)

end)

--> 简写此类方法

mBtn.onClick = function(v)

print(v)

end

```

- 数组操作

```lua

使用#获取Java常见数据类型的长度

使用 ["索引"] 或 .索引 直接访问数组

```

- 索引优化

```lua

t={}

t."end"=123

t.end=123

```

- 非必要的接口类型可省略

```lua

mViewPager.addOnPageChangeListener(

ViewPager.OnPageChangeListener{

onPageSelected = function(_)

end

}

)

-->

mViewPager.addOnPageChangeListener{

onPageSelected = function(_)

end

}

```

- 函数式接口可简写

```lua

obj.run(Runnable {

run = function()

-- do something

end

})

-->

obj.run(function()

-- do something

end)

```

支持增强型字符串格式化

a/A有符号十六进制浮点数，

b布尔值，

B无符号byte类型，

c char类型，数字转文字，

i/d有符号整数类型，字符串转十进制，

I无符号int整数，

e/E/f/g/G有符号浮点数，

o八进制有符号整数，

L无符号长整数，

u/U字符串转u码，

x/X十六进制有符号整数，字符串转hex，

r解析字符串转义，

q格式化为合法字符串形式，

s转字符串，

lurl编码，

---

<!-- 页面 20: 020_example__monitor-volume-buttons.md -->

## 监听音量按键 | LuaHook

**页面地址**: [https://luahook.kulipai.top/example/monitor-volume-buttons.html](https://luahook.kulipai.top/example/monitor-volume-buttons.html)

**处理顺序**: 第20个

---

## 监听音量按键​

Hook了Android系统中所有Activity的onKeyDown方法，主要用于监听并处理音量键的按下事件：

- 监听目标：所有Activity的按键事件

- 特别关注：音量+键（KeyCode 24）和音量-键（KeyCode 25）

- 执行时机：在系统处理按键事件后触发（后置Hook）

- 典型应用：修改音量键功能、添加快捷操作等

### 代码概览​

```lua

hook{

class="android.app.Activity",

method="onKeyDown",

params={"int","android.view.KeyEvent"},

before=function(it)

local context = it.thisObject

local keyCode = it.args[0]

if keyCode == 24 then

-- 音量+键处理

end

if keyCode == 25 then

-- 音量-键处理

end

end

}

```

通过上下文context可以进行UI相关的操作

| 操作 | 代码示例 | 说明 |

| --- | --- | --- |

| 显示Toast | `Toast.makeText(context, "text", Toast.LENGTH_SHORT).show()` | 弹出提示 |

| 弹对话框 | `AlertDialog.Builder(context).setTitle(...).show()` | 显示系统对话框 |

| 获取屏幕尺寸 | `context.getResources().getDisplayMetrics().widthPixels` | 屏幕宽度/高度 |

| 获取状态栏高度 | `context.getResources().getIdentifier("status_bar_height", "dimen", "android")` | 读取系统状态栏高度 |

### 关键参数​

- it.args[0]对应方法的第一个参数（KeyCode）

- it.thisObject获取当前Activity上下文

### 使用场景​

- 修改默认音量键行为

- 实现自定义快捷键功能

- 禁用特定按键功能

- 记录用户按键操作

### Android 按键 KeyCode 对照表​

| 按键名称 | KeyCode | 常量名 | 备注 |

| --- | --- | --- | --- |

| **音量控制** | | | |

| 音量+ | 24 | `KEYCODE_VOLUME_UP` | |

| 音量- | 25 | `KEYCODE_VOLUME_DOWN` | |

| 静音 | 164 | `KEYCODE_VOLUME_MUTE` | |

| **导航键** | | | |

| 返回键 | 4 | `KEYCODE_BACK` | |

| 主页键 | 3 | `KEYCODE_HOME` | 需要系统级权限 |

| 最近任务 | 187 | `KEYCODE_APP_SWITCH` | |

| 菜单键 | 82 | `KEYCODE_MENU` | |

| **媒体控制** | | | |

| 播放/暂停 | 85 | `KEYCODE_MEDIA_PLAY_PAUSE` | |

| 下一曲 | 87 | `KEYCODE_MEDIA_NEXT` | |

| 上一曲 | 88 | `KEYCODE_MEDIA_PREVIOUS` | |

| 停止播放 | 86 | `KEYCODE_MEDIA_STOP` | |

| **数字键** | | | |

| 0-9 | 7-16 | `KEYCODE_0`-`KEYCODE_9` | |

| **字母键** | | | |

| A-Z | 29-54 | `KEYCODE_A`-`KEYCODE_Z` | |

| **功能键** | | | |

| 电源键 | 26 | `KEYCODE_POWER` | 需要系统级权限 |

| 相机键 | 27 | `KEYCODE_CAMERA` | |

| 搜索键 | 84 | `KEYCODE_SEARCH` | |

| **方向键** | | | |

| 上 | 19 | `KEYCODE_DPAD_UP` | |

| 下 | 20 | `KEYCODE_DPAD_DOWN` | |

| 左 | 21 | `KEYCODE_DPAD_LEFT` | |

| 右 | 22 | `KEYCODE_DPAD_RIGHT` | |

| 确定 | 23 | `KEYCODE_DPAD_CENTER` | |

| **游戏控制** | | | |

| L1/R1 (肩键) | 102/103 | `KEYCODE_BUTTON_L1`/`R1` | |

| A/B/X/Y | 96-99 | `KEYCODE_BUTTON_A`-`Y` | |

| **特殊按键** | | | |

| 截图 | 120 | `KEYCODE_SYSRQ` | Android 4.0+ |

| 语音助手 | 231 | `KEYCODE_VOICE_ASSIST` | |

| 外接键盘Enter | 66 | `KEYCODE_ENTER` | |

### 触摸事件对照表​

| 事件类型 | 说明 |

| --- | --- |

| `ACTION_DOWN` | 手指按下 (代码值: 0) |

| `ACTION_UP` | 手指抬起 (代码值: 1) |

| `ACTION_MOVE` | 手指移动 (代码值: 2) |

| `ACTION_CANCEL` | 事件取消 (代码值: 3) |

> 注：完整常量定义见KeyEvent官方文档

---

<!-- 页面 21: 021_example__inject-so-file.md -->

## SO文件动态注入 | LuaHook

**页面地址**: [https://luahook.kulipai.top/example/inject-so-file.html](https://luahook.kulipai.top/example/inject-so-file.html)

**处理顺序**: 第21个

---

## SO文件动态注入​

### 功能概述​

Hook了Android应用启动时的Application.attach()方法，用于动态加载指定SO文件：

- 注入目标：应用启动时的Context初始化阶段

- 核心功能：动态加载指定路径的SO文件

- 执行时机：Context绑定完成后（后置Hook）

- 典型应用：热修复、模块注入、Native代码动态加载

### 核心特性​

- ✅无侵入式注入：通过Hook技术实现

- ✅动态加载：运行时加载SO文件

- ✅错误处理：完善的异常捕获机制

- ✅多场景适配：支持自定义路径和权限管理

### 代码概览​

```lua

imports "java.lang.System"

-- 把so文件复制到

-- /data/data/包名/files/名称.so

-- 授权777

local soName = "libcloud.so"

hook {

class = "android.app.Application",

classLoader = lpparam.classLoader,

method = "attach",

params = {"android.content.Context"},

after = function(after)

local context = after.args[0]

local packageName = invoke(context, "getPackageName")

log("当前包名: "..packageName)

local path = "/data/data/"..packageName.."/files/"..soName

log("尝试加载: "..path)

local loader = invoke(after.thisObject, "getClassLoader")

local ok, err = pcall(function()

System.load(path)

end)

if ok then

log(soName.." 加载成功")

else

log("加载失败: "..tostring(err))

end

end

}

```

### System.load()​

System.load是 Java 中用于加载本地库（Native Library）的一个静态方法，它允许 Java 程序调用本地代码（通常是用 C/C++ 编写的代码）。

方法定义

```java

@SystemApi

public static void load(String pathName) {

Runtime.getRuntime().load(Reflection.getCallerClass(), pathName);

}

```

工作原理

System.load方法加载指定的本地库到Java虚拟机的地址空间中

如果库中定义了JNI_OnLoad函数，该方法会被调用

加载成功后，Java 代码可以通过 JNI (Java Native Interface) 调用本地方法

---

<!-- 页面 22: 022_example__write-setting.md -->

## 编写脚本设置页面 | LuaHook

**页面地址**: [https://luahook.kulipai.top/example/write-setting.html](https://luahook.kulipai.top/example/write-setting.html)

**处理顺序**: 第22个

---

## 编写脚本设置页面​

### 1.::set::注解的作用​

::set::是一个特殊的注解，用于标识脚本文件定义了一个可作为独立界面的设置页面。在 LuaHook 运行时环境中，该注解帮助脚本引擎识别并正确加载和处理这些设置页面脚本。

### 2. 设置页面代码概览​

以下是一个典型的 Lua 设置页面脚本结构，它展示了一个包含文本视图和按钮的基础界面：

```lua

::set::

function setActivity()

-- 导入必要的 Java 类，用于 UI 组件和功能

import "com.kulipai.luahook.util.*"

import "java.lang.*"

import "android.widget.Toast"

import "android.widget.*"

-- 定义一个全局的 print 函数，方便调试输出

function print(...)

local text = table.concat({ ... }, " ")

Toast.makeText(this, tostring(text), Toast.LENGTH_SHORT).show()

end

-- 使用 Lua 表定义界面布局

local layout = {

LinearLayout, -- 根布局为线性布局

gravity = "center", -- 内容居中

id = "filelayout", -- 布局ID

orientation = 1, -- 垂直排列 (1=垂直，0=水平)

layout_width = "fill", -- 宽度填满父视图

layout_height = "fill", -- 高度填满父视图

{

TextView, -- 文本视图

id = "tv", -- 控件ID

textIsSelectable = true, -- 文本可长按复制

gravity = "center", -- 文本居中

layout_width = "fill", -- 宽度填满父视图

text = "测试文本", -- 默认显示文本

},

{

Button, -- 按钮视图

id = "btn", -- 控件ID

layout_marginTop = "16dp", -- 上边距 16dp

layout_marginBottom = "16dp", -- 下边距 16dp

gravity = "center", -- 内容居中

layout_width = "fill", -- 宽度填满父视图

text = "测试按钮", -- 按钮文本

},

}

-- 加载并设置布局到当前 Activity

activity.setContentView(loadlayout(layout))

-- 绑定按钮点击事件

btn.onClick = function()

tv.setText("我爱LuaHook") -- 修改文本内容

end

end

```

### 3. 关键点说明​

- loadlayout函数: 此函数负责将 Lua 表格定义的布局结构转换为 Android 实际的视图对象，并将其渲染到屏幕上。

- 事件绑定: 通过控件ID.onClick这种简洁的方式可以直接为 UI 控件绑定事件监听器，例如btn.onClick = function() ... end。

- 布局属性:layout_width/layout_height: 可以设置为"fill"(填满父视图) 或"wrap"(包裹内容)。gravity: 控制视图内容的对齐方式。orientation: 仅适用于线性布局 (LinearLayout)，决定子视图的排列方向（垂直或水平）。

- layout_width/layout_height: 可以设置为"fill"(填满父视图) 或"wrap"(包裹内容)。

- gravity: 控制视图内容的对齐方式。

- orientation: 仅适用于线性布局 (LinearLayout)，决定子视图的排列方向（垂直或水平）。

- 单位: 在 Android 布局中，推荐使用dp(density-independent pixels) 作为尺寸单位，以确保在不同密度的屏幕上显示效果一致。

### 4. 页面跳转与参数传递​

LuaHook 提供了两种主要方式来实现设置页面的跳转：通过封装Intent启动ScriptSetActivity或直接使用injectActivity方法。无论是哪种方式，最终都是通过ScriptSetActivity作为统一的入口来加载和显示 Lua 设置页面。参数传递则通过标准的Intent.putExtra机制完成。

#### 4.1 通过封装Intent实现跳转​

为了简化页面跳转逻辑，建议封装一个辅助函数，集中处理Intent的构建和启动。这种方式提供了更明确的控制，尤其是当你需要传递复杂参数或进行其他Intent配置时。

```lua

local function startScriptActivity(context, scriptName, arg)

import "android.content.ComponentName"

import "android.content.Intent"

local intent = Intent()

-- 明确指定目标 Activity 组件为 LuaHook 的 ScriptSetActivity

intent.setComponent(ComponentName("com.kulipai.luahook",

"com.kulipai.luahook.activity.ScriptSetActivity"))

-- 通过 Intent.putExtra 传递参数。

-- 'arg' 是一个示例键名，你可以根据需要定义不同的键。

intent.putExtra("arg", arg)

-- 构建脚本文件的完整路径，这是 ScriptSetActivity 加载 Lua 脚本的关键

local packageName = context.getPackageName()

intent.putExtra("path", "/data/local/tmp/LuaHook/AppScript/" .. packageName .. "/" .. scriptName .. ".lua")

-- 启动目标 Activity

context.startActivity(intent)

end

```

#### 4.2 通过injectActivity方法实现跳转​

injectActivity方法是 LuaHook 提供的一种更直接的跳转方式，它通常用于在运行时注入并启动新的 Activity。其底层实现原理与封装Intent启动ScriptSetActivity类似，都是将 Lua 代码或文件路径作为参数传递给ScriptSetActivity。

injectActivity的基本用法如下：

```lua

-- injectActivity(当前的activity实例, lua代码字符串/lua文件路径)

-- 目前主要支持 lua代码字符串

injectActivity(currentActivity, "print('Hello from injected Activity')")

```

重要提示：

- 目前injectActivity主要支持直接传入Lua 代码字符串。这意味着你需要将整个设置页面的 Lua 逻辑作为字符串传递。

- 如果需要传递参数，无论使用startScriptActivity还是injectActivity，都仍然需要通过Intent.putExtra这种原始方法来传递。这是因为ScriptSetActivity统一通过Intent来接收所有必要的启动信息和参数。

#### 4.3 宿主页面调用示例​

以下示例展示了如何在宿主应用的 Lua 脚本中调用封装的startScriptActivity函数来跳转到设置页面，并传递一个字符串参数。如果你选择直接使用injectActivity，逻辑会类似，但需要将整个::set::定义的 Lua 代码作为字符串传入。

```lua

hookcotr(

"com.tencent.mm.pluginsdk.ui.chat.ChatFooter", -- 目标类

loader,

"android.content.Context",

"android.util.AttributeSet",

"int",

function(it) end,

function(it)

local button = getField(it.thisObject, "w") -- 获取宿主应用中的一个按钮实例

local context = invoke(button, "getContext") -- 获取当前上下文

button.onLongClick = function() -- 按钮长按事件

-- 调用封装的函数，跳转到名为 "当前脚本名称" 的设置页面，并传递参数 "你好"

startScriptActivity(context, "当前脚本名称", "你好")

-- 如果使用 injectActivity，大致会是这样（需要将 ::set:: 函数转换为字符串）：

-- local luaCode = [[

-- ::set::

-- function setActivity()

--    -- ... 你的设置页面代码 ...

-- end

-- ]]

-- injectActivity(context, luaCode)

end

end

)

```

#### 4.4 设置页面接收参数示例​

在::set::定义的设置页面脚本中，无论你是通过startScriptActivity还是injectActivity跳转过来，都可以通过this.getIntent().getExtras()方法来获取传递过来的参数：

```lua

::set::

function setActivity()

import "com.kulipai.luahook.util.*"

import "java.lang.*"

import "android.widget.Toast"

import "android.widget.*"

function print(...)

local text = table.concat({ ... }, " ")

Toast.makeText(this, tostring(text), Toast.LENGTH_SHORT).show()

end

-- 获取从 Intent 中传递过来的字符串参数 "arg"

local argStr = this.getIntent().getExtras().getString("arg")

local layout = {

LinearLayout,

gravity = "center",

id = "filelayout",

orientation = 1,

layout_width = "fill",

layout_height = "fill",

{

TextView,

id = "tv",

textIsSelectable = true,

gravity = "center",

layout_width = "fill",

text = "测试文本",

},

{

Button,

id = "btn",

layout_marginTop = "16dp",

layout_marginBottom = "16dp",

gravity = "center",

layout_width = "fill",

text = "测试按钮",

},

}

activity.setContentView(loadlayout(layout))

btn.onClick = function()

-- 将接收到的参数设置到文本视图中

tv.setText(argStr)

end

end

```

### 5. 未来展望（待实现）​

目前，页面跳转和参数传递主要依赖于ScriptSetActivity作为统一入口，并通过Intent.putExtra来实现。未来计划将支持更灵活的跳转方式和参数传递机制，包括：

- 通过::set::传递 Lua 函数: 允许直接传递 Lua 函数引用，实现更高级的回调和交互。

- 通过 Lua 文件路径传递: 简化大型 Lua 脚本的组织和加载，避免长字符串。

这些改进将进一步提升 LuaHook 在开发自定义设置页面方面的便利性和强大功能。

---

## 使用规则

1. 先匹配用户需求到最接近的代码片段，再进行最小化改造，避免重写。
2. 输出代码时保持可运行，补齐必要 import、上下文变量和错误处理。
3. 多片段组合时遵循 KISS、DRY、YAGNI：仅保留当前需求必需代码。
4. 需要修改系统行为、敏感权限或高风险操作时，先显式提示风险再给实现。

## 代码片段索引

## 来源：笔记.txt

### 打印

```lua
print(打印内容)
```

### 控件被单击

```lua
function 控件ID.onClick()
--事件
end

控件ID.onClick=function()
--事件
end
```

### 控件被长按

```lua
控件ID.onLongClick=function()
--事件
end

function 控件ID.onLongClick()
--事件
end
```

### 控件可视，不可视或隐藏

```lua
--控件可视
控件ID.setVisibility(View.VISIBLE)
--控件不可视
控件ID.setVisibility(View.INVISIBLE)
--控件隐藏
控件ID.setVisibility(View.GONE)
```

### 提示框

```lua
import "android.content.DialogInterface"
local dl=AlertDialog.Builder(activity)
.setTitle("提示框标题")
.setMessage("提示框内容")
.setPositiveButton("按钮标题",DialogInterface
.OnClickListener{
onClick=function(v)
--事件
end
})
.setNegativeButton("按钮标题",nil)
.create()
dl.show()
```

### 读写文件

```lua
--读文件
local file=io.input("地址")
local str=io.read("*a")
io.close()
print(str)
--写文件
local file=io.output("地址")
io.write(写入内容)
io.flush()
io.close()
```

### 加载框示例

```lua
local dl=ProgressDialog.show(activity,nil,'登录中')
dl.show()
local a=0
local tt=Ticker()
tt.start()
tt.onTick=function()
a=a+1
if a==3 then
dl.dismiss()
tt.stop()
end
end
```

### 标题栏菜单按钮

```lua
tittle={"分享","帮助","皮肤","退出"}
function onCreateOptionsMenu(menu)
for k,v in ipairs(tittle) do
if tittle[v] then
local m=menu.addSubMenu(v)
for k,v in ipairs(tittle[v]) do
m.add(v)
end
else
local m=menu.add(v)
m.setShowAsActionFlags(1)
end
end
end
function onMenuItemSelected(id,tittle)
if y[tittle.getTitle()] then
y[tittle.getTitle()]()
end
end

y={}
y["帮助"]=function()
--事件
end

--菜单
function onCreateOptionsMenu(menu)
menu.add("打开").onMenuItemClick=function(a)

end
menu.add("新建").onMenuItemClick=function(a)

end
end
```

### 关闭对话框

```lua
--将dl.show赋值
dialog=dl.show()
--在某按钮点击后关闭这个对话框
function zc.onClick()
dialog.dismiss()
end
```

### 判断是否有网络

```lua
local wl=activity.getApplicationContext().getSystemService(Context.CONNECTIVITY_SERVICE).getActiveNetworkInfo();
if wl== nil then
print("无法连接到服务器")
end
```

### 沉浸状态栏

```lua
--这个需要系统SDK21以上才能用
if Build.VERSION.SDK_INT >= 21 then
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS).setStatusBarColor(0xff4285f4);
end
--这个需要系统SDK19以上才能用
if Build.VERSION.SDK_INT >= 19 then
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
end
```

### 复制文本到剪贴板

```lua
--先导入包
import "android.content.*"
activity.getSystemService(Context.CLIPBOARD_SERVICE).setText(文本)
```

### 安卓跳转动画

```text
android.R.anim.accelerate_decelerate_interpolator
android.R.anim.accelerate_interpolator
android.R.anim.anticipate_interpolator
android.R.anim.anticipate_overshoot_interpolator
android.R.anim.bounce_interpolator
android.R.anim.cycle_interpolator
android.R.anim.decelerate_interpolatoandroid.R.anim.r
android.R.anim.fade_in
android.R.anim.fade_out
android.R.anim.linear_interpolator
android.R.anim.overshoot_interpolator
android.R.anim.slide_in_left
android.R.anim.slide_out_right
```

### TextView文本可选择复制

```lua
--代码中设置
t.TextIsSelectable=true
--布局表中设置
textIsSelectable=true
```

### 取随机数

```text
math.random(最小值,最大值)
```

### 延迟

```lua
--这个会卡进程，配合线程使用
Thread.sleep(延迟时间)
--这个不会卡进程
--500指延迟500毫秒
task(500‚function()
--延迟之后执行的事件
end)
```

### 定时器

```lua
--timer定时器
t=timer(function()
--事件
end,延迟,间隔,初始化)
--暂停timer定时器
t.Enable=false
--启动timer定时器
t.Enable=true

--Ticker定时器
ti=Ticker()
ti.Period=间隔
ti.onTick=function()
--事件
end
--启动Ticker定时器
ti.start()
--停止Ticker定时器
ti.stop()
```

### 获取本地时间

```lua
--格式的时间
os.date("%Y-%m-%d %H:%M:%S")
--本地时间总和
os.clock()
《《EditText文本被改变事件
控件ID.addTextChangedListener{
onTextChanged=function(s)
--事件
end
}
```

### 字符串操作

```lua
--字符串转大写
string.upper(字符串)
--字符串转小写
string.lower(字符串)
--字符串替换
string.gsub(字符串,被替换的字符,替换的字符,替换次数)
```

### 设置控件大小

```lua
--设置宽度
linearParams = 控件ID.getLayoutParams()
linearParams.width =宽度
控件ID.setLayoutParams(linearParams)
--同理设置高度
linearParams = 控件ID.getLayoutParams()
linearParams.height =高度
控件ID.setLayoutParams(linearParams)
```

### 载入窗口传参

```lua
activity.newActivity("窗口名",{参数})

--渐变动画效果的，中间是安卓跳转动画代码
activity.newActivity("窗口名",android.R.anim.fade_in,android.R.anim.fade_out,{参数})
```

### EditText只能输数字

```lua
import "android.text.InputType"
import "android.text.method.DigitsKeyListener"
控件ID.setInputType(InputType.TYPE_CLASS_NUMBER)
控件ID.setKeyListener(DigitsKeyListener.getInstance("0123456789"))
```

### 窗口全屏

```lua
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN)
```

### 关闭当前窗口

```lua
activity.finish()
```

### 按两次返回键退出

```lua
参数=0
function onKeyDown(code,event)
if string.find(tostring(event),"KEYCODE_BACK") ~= nil then
if 参数+2 > tonumber(os.time()) then
activity.finish()
else
 Toast.makeText(activity,"再按一次返回键退出" , Toast.LENGTH_SHORT )
.show()
参数=tonumber(os.time())
end
return true
end
end
```

### 取字符串中间

```text
string.match("左测试测试右","左(.-)右")
```

### 判断文件是否存在

```lua
--先导入io包
import "java.io.*"
file,err=io.open("路径")
print(err)
if err==nil then
print("存在")
else
print("不存在")
end
```

### 判断文件夹是否存在

```lua
--先导入io包
import "java.io.*"
if File(文件夹路径).isDirectory()then
print("存在")
else
print("不存在")
end
```

### 窗口回调事件

```lua
function onActivityResult()
--事件
end
```

### 隐藏标题栏

```lua
activity.ActionBar.hide()
```

### 自定义布局对话框

```lua
local dl=AlertDialog.Builder(activity)
.setTitle("自定义布局对话框")
.setView(loadlayout(layout))
dl.show()
```

### 列表下滑到最底事件

```lua
list.setOnScrollListener{
onScrollStateChanged=function(l,s)
if list.getLastVisiblePosition()==list.getCount()-1 then
--事件
end
end}
```

### 标题栏返回按钮

```lua
activity.getActionBar().setDisplayHomeAsUpEnabled(true)
```

### 列表长按事件

```lua
ID.setOnItemLongClickListener(AdapterView.OnItemLongClickListener{
onItemLongClick=function(parent, v, pos,id)
--事件
end
})
```

### 列表点击事件

```lua
ID.setOnItemClickListener(AdapterView.OnItemClickListener{
onItemClick=function(parent, v, pos,id)
--事件
end
})
```

### 关于V4的圆形下拉刷新

```lua
--设置下拉刷新监听事件
swipeRefreshLayout.setOnRefreshListener(this);
--设置进度条的颜色
swipeRefreshLayout.setColorSchemeColors(Color.RED, Color.BLUE, Color.GREEN);
--设置圆形进度条大小
swipeRefreshLayout.setSize(SwipeRefreshLayout.LARGE);
--设置进度条背景颜色
swipeRefreshLayout.setProgressBackgroundColorSchemeColor(Color.DKGRAY);
--设置下拉多少距离之后开始刷新数据
swipeRefreshLayout.setDistanceToTriggerSync(50);
```

### 活动中的回调

```lua
function main(...)
--...是newActivity传递过来的参数。
print("入口函数",...)
end

function onCreate()
print("窗口创建")
end

function onStart()
print("活动开始")
end

function onResume()
print("返回程序")
end

function onPause()
print("活动暂停")
end

function onStop()
print("活动停止")
end

function onDestroy()
print("程序已退出")
end

function onResult(name,...)
--name：返回的活动名称
--...：返回的参数
print("返回活动",name,...)
end

function onCreateOptionsMenu(menu)
--menu：选项菜单。
menu.add("菜单")
end

function onOptionsItemSelected(item)
--item：选中的菜单项
print(item.Title)
end

function onConfigurationChanged(config)
--config：配置信息
print("屏幕方向关闭")
end

function onKeyDown(keycode,event)
--keycode：键值
--event：事件
print("按键按下",keycode)
end

function onKeyUp(keycode,event)
--keycode：键值
--event：事件
print("按键抬起",keycode)
end

function onKeyLongPress(keycode,event)
--keycode：键值
--event：事件
print("按键长按",keycode)
end

function onTouchEvent(event)
--event：事件
print("触摸事件",event)
end
```

### 对话框Dialog

```lua
--简单对话框
AlertDialog.Builder(this).setTitle("标题")
.setMessage("简单消息框")
.setPositiveButton("确定",nil)
.show();

--带有三个按钮的对话框
AlertDialog.Builder(this)
.setTitle("确认")
.setMessage("确定吗？")
.setPositiveButton("是",nil)
.setNegativeButton("否",nil)
.setNeutralButton("不知道",nil)
.show();

--带输入框的
AlertDialog.Builder(this)
.setTitle("请输入")
.setIcon(android.R.drawable.ic_dialog_info)
.setView(EditText(this))
.setPositiveButton("确定", nil)
.setNegativeButton("取消", nil)
.show();

--单选的
AlertDialog.Builder(this)
.setTitle("请选择")
.setIcon(android.R.drawable.ic_dialog_info)
.setSingleChoiceItems({"选项1","选项2","选项3","选项4"}, 0,
DialogInterface.OnClickListener() {
 onClick(dialog,which) {
dialog.dismiss();
 }
}
)
.setNegativeButton("取消", null)
.show();

--多选的
AlertDialog.Builder(this)
.setTitle("多选框")
.setMultiChoiceItems({"选项1","选项2","选项3","选项4"}, null, null)
.setPositiveButton("确定", null)
.setNegativeButton("取消", null)
.show();

--列表的
AlertDialog.Builder(this)
.setTitle("列表框")
.setItems({"列表项1","列表项2","列表项3"},nil)
.setNegativeButton("确定",nil)
.show();

--图片的
img = ImageView(this);
img.setImageResource(R.drawable.icon);
AlertDialog.Builder(this)
.setTitle("图片框")
.setView(img)
.setPositiveButton("确定",nil)
.show();
```

### 删除ListView中某项

```text
adp.remove(pos)
```

### 打开某APP

```lua
--导入包
import "android.content.*"

intent = Intent();
componentName = ComponentName("com.androlua","com.androlua.Welcome");
intent.setComponent(componentName);
activity.startActivity(intent);
```

### 设置横屏竖屏

```lua
--横屏
activity.setRequestedOrientation(0);
--竖屏
activity.setRequestedOrientation(1);
```

### 设置控件图片

```lua
--设置的图片也可以输入路径
ID.setImageBitmap(loadbitmap("图片.png"))
```

### 禁用编辑框

```lua
--代码中设置
editText.setFocusable(false);
--布局表中设置
Focusable=false;
```

### 隐藏滑条

```lua
--横向
horizontalScrollBarEnabled=false;
--竖向
VerticalScrollBarEnabled=false;
```

### 图片着色

```lua
--代码中设置
ID.setColorFilter(0xffff0000)
--布局表中设置
ColorFilter="#ffff0000"；
```

### 获取IMEI号

```lua
import "android.content.*"
--导入包

imei=activity.getSystemService(Context.TELEPHONY_SERVICE).getDeviceId();
print(imei)

--别忘了添加权限"READ_PHONE_STATE"
```

### 分享文字

```lua
import "android.content.*"

text="分享的内容"
intent=Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_SUBJECT, "分享");
intent.putExtra(Intent.EXTRA_TEXT, text);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
activity.startActivity(Intent.createChooser(intent,"分享到:"));
```

### 发送短信

```lua
--导入包
import "android.content.*"
import "android.net.*"

uri = Uri.parse("smsto:15800001234");
intent = Intent(Intent.ACTION_SENDTO, uri);
intent.putExtra("sms_body","你好")
intent.setAction("android.intent.action.VIEW");
activity.startActivity(intent);
```

### 拔号

```lua
import "android.content.*"
import "android.net.*"
--导入包
uri = Uri.parse("tel:15800001234");
intent = Intent(Intent.ACTION_CALL, uri);
intent.setAction("android.intent.action.VIEW");
activity.startActivity(intent);
--记得添加打电话权限
```

### 安装APK

```lua
import "android.content.*"
import "android.net.*"

intent = Intent(Intent.ACTION_VIEW);
intent.setDataAndType(Uri.parse("file:///sdcard/jc.apk"), "application/vnd.android.package-archive");
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
activity.startActivity(intent);
```

### 振动

```lua
import "android.content.Context"
--导入包
vibrator = activity.getSystemService(Context.VIBRATOR_SERVICE)
vibrator.vibrate( long{100,800} ,-1)
--{等待时间,振动时间,等待时间,振动时间,•••,•••,•••,•••••}
--{0,1000,500,1000,500,1000}
--别忘了申明权限
```

### 获取剪贴板内容

```lua
import"android.content.*"
--导入包
a=activity.getSystemService(Context.CLIPBOARD_SERVICE).getText()
```

### 压缩成ZIP

```text
ZipUtil.zip("文件或文件夹路径","压缩到的路径")
```

### ZIP解压

```lua
ZipUtil.unzip("ZIP路径","解压到的路径")

--另一种Java方法
import "java.io.FileOutputStream"
import "java.util.zip.ZipFile"
import "java.io.File"

zipfile = "/sdcard/压缩包.zip"--压缩文件路径和文件名
sdpath = "/sdcard/文件.lua"--解压后路径和文件名
zipfilepath = "内容.lua"--需要解压的文件名

function unzip(zippath , outfilepath , filename)

local time=os.clock()
  task(function(zippath,outfilepath,filename)
require "import"
import "java.util.zip.*"
import "java.io.*"
local file = File(zippath)
local outFile = File(outfilepath)
local zipFile = ZipFile(file)
local entry = zipFile.getEntry(filename)
local input = zipFile.getInputStream(entry)
local output = FileOutputStream(outFile)
local byte=byte[entry.getSize()]
local temp=input.read(byte)
while temp ~= -1 do
output.write(byte)
temp=input.read(byte)
end
input.close()
output.close()
end,zippath,outfilepath,filename,
function()
print("解压完成，耗时 "..os.clock()-time.." s")
end)

end

unzip(zipfile,sdpath,zipfilepath)
```

### 删除文件夹

```lua
--shell命令的方法
os.execute("rm-r 路径")
```

### 重命名文件夹

```lua
--shell命令的方法
os.execute("mv 路径新路径")
```

### 创建文件夹

```lua
--shell命令的方法
os.execute("mkdir 路径")
```

### 删除文件

```text
os.remove("路径")
```

### 设置标题栏标题

```lua
--标题
activity.setTitle('标题')
--小标题
activity.getActionBar().setSubtitle('小标题')
```

### 获取Lua文件的执行路径

```lua
activity.getLuaDir()
```

### 获取本应用包名

```lua
activity.getPackageName()
```

### 布局设置点击效果

```lua
--5.0或以上可以实现点击水波纹效果
--在布局加入：

style="?android:attr/buttonBarButtonStyle";
```

### 判断某APP是否安装

```lua
if pcall(function() activity.getPackageManager().getPackageInfo("包名",0) end) then
print("安装了")
else
print("没安装")
end
```

### 调用系统下载

```lua
--导入包
import "android.content.Context"
import "android.net.Uri"

downloadManager=activity.getSystemService(Context.DOWNLOAD_SERVICE);
url=Uri.parse("绝对下载链接");
request=DownloadManager.Request(url);
request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_MOBILE|DownloadManager.Request.NETWORK_WIFI);
request.setDestinationInExternalPublicDir("目录名，可以是Download","下载的文件名");
request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
downloadManager.enqueue(request);
```

### 动画结束回调

```lua
--导入包
import "android.view.animation.*"
import "android.view.animation.Animation$AnimationListener"
--控件动画
控件.startAnimation(AlphaAnimation(1,0).setDuration(400).setFillAfter(true).setAnimationListener(AnimationListener{
onAnimationEnd=function()
print"动画结束")
end}))
```

### 关于侧滑

```lua
--侧滑布局是 DrawerLayout;
--关闭侧滑
ID.closeDrawer(3)
--打开侧滑
ID.openDrawer(3)
```

### 关于输入法影响布局的问题

```lua
--使弹出的输入法不影响布局
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);
--使弹出的输入法影响布局
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);
```

### TextView设置字体样式

```lua
--首先要导入包
import "android.graphics.*"
--设置中划线
控件id.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG)
--下划线
控件id.getPaint().setFlags(Paint. UNDERLINE_TEXT_FLAG )
--加粗
控件id.getPaint().setFakeBoldText(true)
--斜体
控件id.getPaint().setTextSkewX(0.2)

--设置TypeFace
import "android.graphics.Typeface"
id.getPaint().setTypeface(字体)
--字体可以为以下
Typeface.DEFAULT --默认字体
Typeface.DEFAULT_BOLD --加粗字体
Typeface.MONOSPACE --monospace字体
Typeface.SANS_SERIF --sans字体
Typeface.SERIF --serif字体
```

### 强制结束自身并清除自身数据

```text
 os.execute("pm clear "..activity.getPackageName())
```

### 递归搜索文件实例

```lua
require "import"

function find(catalog,name)
local n=0
local t=os.clock()
local ret={}
require "import"
import "java.io.File"
import "java.lang.String"
function FindFile(catalog,name)
local name=tostring(name)
local ls=catalog.listFiles() or File{}
for 次数=0,#ls-1 do
--local 目录=tostring(ls[次数])
local f=ls[次数]
if f.isDirectory() then--如果是文件夹则继续匹配
FindFile(f,name)
else--如果是文件则
n=n+1
if n%1000==0 then
--print(n,os.clock()-t)
end
local nm=f.Name
if string.find(nm,name) then
--thread(insert,目录)
table.insert(ret,nm)
print(nm)
end
end
luajava.clear(f)
end
end
FindFile(catalog,name)
print("ok",n,#ret)
end

import "java.io.File"

catalog=File("sdcard/")
name=".j?pn?g"
--task(find,catalog,name,print)
thread(find,catalog,name)
```

### 获取ListView垂直坐标

```lua
function getScrollY()
c = ls.getChildAt(0);
local firstVisiblePosition = ls.getFirstVisiblePosition();
local top = c.getTop();
return -top + firstVisiblePosition * c.getHeight() ;
end
```

### 申请root权限

```lua
--shell命令的方法
os.execute("su")
```

### 传感器

```lua
传感器 = activity.getSystemService(Context.SENSOR_SERVICE)

local 加速度传感器 = 传感器.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
x轴 = event.values[0]
y轴 = event.values[1]
z轴 = event.values[2]
end,nil}), 加速度传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 光线传感器 = 传感器.getDefaultSensor(Sensor.TYPE_LIGHT)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
光线 = event.values[0]
end,nil}), 光线传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 距离传感器 = 传感器.getDefaultSensor(Sensor.TYPE_PROXIMITY)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
距离 = event.values[0]
end,nil}), 距离传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 磁场传感器 = 传感器.getDefaultSensor(Sensor.TYPE_ORIENTATION)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
磁场 = event.values[0]
end,nil}), 磁场传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 温度传感器 = 传感器.getDefaultSensor(Sensor.TYPE_TEMPERATURE)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
温度 = event.values[0]
end,nil}), 温度传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 陀螺仪传感器 = 传感器.getDefaultSensor(Sensor.TYPE_GYROSCOPE)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
陀螺仪 = event.values[0]
end,nil}), 陀螺仪传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 重力传感器 = 传感器.getDefaultSensor(Sensor.TYPE_GRAVITY)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
重力 = event.values[0]
end,nil}), 重力传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 压力传感器 = 传感器.getDefaultSensor(Sensor.TYPE_PRESSURE)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
压力 = event.values[0]
end,nil}), 压力传感器, SensorManager.SENSOR_DELAY_NORMAL)
```

### 获取控件宽高

```lua
--导入包
import "android.content.Context"

function getwh(view)
view.measure(View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED),View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED));
height =view.getMeasuredHeight();
width =view.getMeasuredWidth();
return width,height
end

print(getwh(控件ID))
```

### 播放音频

```lua
--导入包
import "android.media.MediaPlayer"

local 音频播放器=MediaPlayer()
function 播放音频(路径)
音频播放器.reset()
.setDataSource(路径)
.prepare()
.start()
.setOnCompletionListener({
onCompletion=function()
print("播放完毕")
end})
end
```

### 控件旋转

```lua
--Z轴上的旋转角度
View.getRotation()

--X轴上的旋转角度
View.getRotationX()

--Y轴上的旋转角度
View.getRotationY()

--设置Z轴上的旋转角度
View.setRotation(r)

--设置X轴上的旋转角度
View.setRotationX(r)

--设置Y轴上的旋转角度
View.setRotationY(r)

--设置旋转中心点的X坐标
View.setPivotX(p)

--设置旋转中心点的Y坐标
View.setPivotX(p)

--设置摄像机的与旋转目标在Z轴上距离
View.setCameraDistance(d)
```

## 来源：基础代码.txt

### 打印

```lua
print"Hello World！"
print("Hello World")
```

### 注释

```text
单行注释  --
多行注释  --[[]]
```

### 字符串

```lua
a="String"
a=[[String]]
a=[===[String]===]
```

### 赋值

```lua
a="Hello World"

--lua支持多重赋值
a,b="String a","String b"

--交换值
a,b="String a","String b"
a,b=b,a
```

### 类型简介

```lua
Lua 存在的数据类型包括:
1.nil
此类型只有一个值 nil。用于表示“空”值。全局变量默认为 nil，删除一个已经赋值的全局变量只需要将其赋值为 nil（对比 JavaScript，赋值 null 并不能完全删除对象的属性，属性还存在，值为 null）

2.boolean
此类型有两个值 true 和 false。在 Lua 中，false 和 nil 都表示条件假，其他值都表示条件真（区别于 C/C++ 等语言的是，0 是真）


3.number
双精浮点数（IEEE 754 标准），Lua 没有整数类型

4.string
你可以保存任意的二进制数据到字符串中（包括 0）。字符串中的字符是不可以改变的（需要改变时，你只能创建一个新的字符串）。获取字符串的长度，可以使用 # 操作符（长度操作符）。例如：print(#”hello”)。字符串可以使用单引号，也可以使用双引号包裹，对于多行的字符串还可以使用 [[ 和 ]] 包裹。字符串中可以使用转义字符，例如 \n \r 等。使用 [[ 和 ]] 包裹的字符串中的转义字符不会被转义

5.userdata
用于保存任意的 C 数据。userdata 只能支持赋值操作和比较测试

6.function
函数是第一类值（first-class value），我们能够像使用其他变量一样的使用函数（函数能够保存在变量中，可以作为参数传递给函数）

7.thread
区别于我们常常说的系统级线程

8.table
被实现为关联数组（associative arrays），可以通过任何值来进行索引（nil 除外）。和全局变量一样，table 中未赋值的域为 nil，删除一个域只需要将其赋值为 nil（实际上，全局变量就是被放置在一个 table 中）



type 函数用于返回值的类型：
print(type("Hello World")) --> string
print(type(10.4*3))        --> number
print(type(print))         --> function
print(type(type(X)))       --> string
```

### Table(数组)

```lua
table是lua唯一的数据结构。
table是lua中最重要的数据类型。
table类似于 python 中的字典。
table只能通过构造式来创建。其他语言提供的其他数据结构如array、list等等，lua都是通过table来实现的。
table非常实用，可以用在不同的情景下。最常用的方式就是把table当成其他语言的数组。

实例1:
mytable = {}
for index = 1, 100 do
    mytable[index] = math.random(1,1000)
end

说明：
1.数组不必事先定义大小，可动态增长。
2.创建包含100个元素的table，每个元素随机赋1-1000之间的值。
3.可以通过mytable[x]访问任意元素，x表示索引。
4.索引从1开始。

实例2:
tab = { a = 10, b = 20, c = 30, d = 'www.jb51.net' }
print(tab["a"])

说明：
1.table 中的每项要求是 key = value 的形式。
2.key 只能是字符串， 这里的 a, b, c, d 都是字符串，但是不能加上引号。
3.通过 key 来访问 table 的值，这时候， a 必须加上引号。

实例3:
tab = { 10, s = 'abc', 11, 12, 13 }
print(tab[1]) = 10
print(tab[2]) = 11
print(tab[3]) = 12
print(tab[4]) = 13
说明：
1.数标从1开始。
2.省略key，会自动以1开始编号，并跳过设置过的key。
```

### 比较操作符

```lua
--Lua 支持下列比较操作符：

==: 等于
~=: 不等于
<: 小于
>: 大于
<=: 小于等于
>=: 大于等于
这些操作的结果不是 false就是 true。
```

### For循环

```lua
--给定条件进行循环

--输出从1到10
for i=1,10 do
print(i)
end


--输出从10到1
for i=10,1,-1 do
print(i)
end

--打印数组a中所有的值
a={"a","b","c","d"}
for index,content in pairs(a) do
print(content)
end
```

### While循环

```lua
--只要条件为真便会一直循环下去

--输出1到10
a=0
while a~=10 do
a=a+1
print(a)
end

--输出10到1
a=11
while a~=1 do
a=a-1
print(a)
end

--打印数组a中的所有值
shuzu={"a","b","c","d"}
a=0
while a~=#shuzu do
a=a+1
print(shuzu[a])
end
```

### if(判断语句)

```lua
--判断值是否为真
a=true
if a then
print("真")
else
print("假")
end

--比较值是否相同
a=true
b=false
if a==b then
print("真")
else
print("假")
end
```

### function(函数)

```lua
函数有两个用途
1.完成指定功能，函数作为调用语句使用
2.计算并返回值，函数作为赋值语句的表达式使用


实例1:
function 读取文件(路径)
文件内容=io.open(路径):read("*a")
return 文件内容--return用来返回值
end



实例2:
require "import"
import "android.widget.EditText"
import "android.widget.LinearLayout"
function 编辑框()
return EditText(activity)
end
layout={
  LinearLayout;
  id="父布局",
  {编辑框,
    id="edit",
    text="文本",
   },
};
activity.setContentView(loadlayout(layout))
--把这段代码放到调试里面去测试
```

### 基础代码

```lua
activity.setTitle('Title')--设置窗口标题
activity.setContentView(loadlayout(layout))--设置窗口视图
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置主题
activity.getWidth()--获取屏幕宽
activity.getHeight()--获取屏幕高
activity.newActivity("main")--跳转页面
activity.finish()--关闭当前页面
activity.recreate()--重构activity
os.exit()--结束程序
tostring()--转换字符串
tonumber()--转换数字
tointeger()--转换整数
--线程
--thread
thread(function()print"线程"end)
--task
task(function()print"线程"end)
```

## 来源：进阶代码.txt

### 打开微信扫一扫

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.content.Context"
import "android.content.Intent"
import "android.content.ComponentName"
--activity.setContentView(loadlayout("layout"))

activity.finish()
intent=Intent()
intent.setComponent(ComponentName("com.tencent.mm", "com.tencent.mm.ui.LauncherUI"))
intent.putExtra("LauncherUI.From.Scaner.Shortcut", true)
intent.setFlags(335544320)
intent.setAction("android.intent.action.VIEW");
activity.startActivity(intent);
```

### 正则表达式

```lua
--[[.(点): 与任何字符配对
%a: 与任何字母配对
%c: 与任何控制符配对(例如\n)
%d: 与任何数字配对
%l: 与任何小写字母配对
%p: 与任何标点(punctuation)配对
%s: 与空白字符配对
%u: 与任何大写字母配对
%w: 与任何字母/数字配对
%x: 与任何十六进制数配对
%z: 与任何代表0的字符配对
%x(此处x是非字母非数字字符): 与字符x配对. 主要用来处理表达式中有功能的字符(^$()%.[]*+-?)的配对问题, 例如%%与%配对[数个字符类]: 与任何[]中包含的字符类配对. 例如[%w_]与任何字母/数字, 或下划线符号(_)配对[^数个字符类]: 与任何不包含在[]中的字符类配对. 例如[^%s]与任何非空白字符配对

当上述的字符类用大写书写时, 表示与非此字符类的任何字符配对. 例如, %S表示与任何非空白字符配对.

%a  星期简写
%A  星期大写
%b  月份简写
%B  月份大写
%c  日期时间
%d  月份天数
%H  小时[24进制]
%I  小时[12进制]
%j  年中第几天
%m  月份
%M 分钟
%p  am或pm
%S  秒数
%w  星期
%W  第几周
%x  日期
%X  时间
%y  两位数的月份
%Y 完整的月份
%z  时区
%% 百分号



%c - 接受一个数字, 并将其转化为ASCII码表中对应的字符
%d, %i - 接受一个数字并将其转化为有符号的整数格式
%o - 接受一个数字并将其转化为八进制数格式
%u - 接受一个数字并将其转化为无符号整数格式
%x - 接受一个数字并将其转化为十六进制数格式, 使用小写字母
%X - 接受一个数字并将其转化为十六进制数格式, 使用大写字母
%e - 接受一个数字并将其转化为科学记数法格式, 使用小写字母e
%E - 接受一个数字并将其转化为科学记数法格式, 使用大写字母E
%f - 接受一个数字并将其转化为浮点数格式
%g(%G) - 接受一个数字并将其转化为%e(%E, 对应%G)及%f中较短的一种格式
%q - 接受一个字符串并将其转化为可安全被Lua编译器读入的格式
%s - 接受一个字符串并按照给定的参数格式化该字符串
为进一步细化格式, 可以在%号后添加参数. 参数将以如下的顺序读入:

(1) 符号: 一个+号表示其后的数字转义符将让正数显示正号. 默认情况下只有负数显示符号.
(2) 占位符: 一个0, 在后面指定了字串宽度时占位用. 不填时的默认占位符是空格.
(3) 对齐标识: 在指定了字串宽度时, 默认为右对齐, 增加-号可以改为左对齐.
(4) 宽度数值
(5) 小数位数/字串裁切: 在宽度数值后增加的小数部分n, 若后接f(浮点数转义符, 如%6.3f)则设定该浮点数的小数只保留n位, 若后接s(字符串转义符, 如%5.3s)则设定该字符串只显示前n位.

]]
--实例
string1 = "Lua"
string2 = "Tutorial"
number1 = 10
number2 = 20
-- 基本字符串格式化
print(string.format("基本格式化 %s %s",string1,string2))
-- 日期格式化
date = 2; month = 1; year = 2014
print(string.format("日期格式化 %02d/%02d/%03d", date, month, year))
-- 十进制格式化
print(string.format("%.4f",1/3))

--[[
以上代码执行结果为：

基本格式化 Lua Tutorial
日期格式化 02/01/2014
0.3333
]]

--其他例子：

string.format("%c", 83)                 -- 输出S
string.format("%+d", 17.0)              -- 输出+17
string.format("%05d", 17)               -- 输出00017
string.format("%o", 17)                 -- 输出21
string.format("%u", 3.14)               -- 输出3
string.format("%x", 13)                 -- 输出d
string.format("%X", 13)                 -- 输出D
string.format("%e", 1000)               -- 输出1.000000e+03
string.format("%E", 1000)               -- 输出1.000000E+03
string.format("%6.3f", 13)              -- 输出13.000
string.format("%q", "One\nTwo")         -- 输出"One\
                                        -- 　　Two"
string.format("%s", "monkey")           -- 输出monkey
string.format("%10s", "monkey")         -- 输出    monkey
string.format("%5.3s", "monkey")        -- 输出  mon
```

### 获取充电状态(电量)

```lua
import "android.content.Intent"
import "android.os.BatteryManager"
import "android.content.IntentFilter"
filter =IntentFilter(Intent.ACTION_BATTERY_CHANGED)
intent =this.getContext().registerReceiver(nil,filter)
当前电量= intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
充电状态 = intent.getIntExtra(BatteryManager.EXTRA_STATUS, BatteryManager.BATTERY_STATUS_UNKNOWN)
--充电状态值=(1，未知状态)(2，充电中)(3，放电中)(4，未充电)(5，充满了)

--java搬得代码(淡紫色)

print(当前电量,充电状态)
```

### 半圆时钟带刻度

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout={
  LinearLayout;
  layout_width="fill";
  layout_height="fill";
  orientation="vertical";
  {
    LinearLayout;
    id="view";
    layout_width="fill";
    layout_height="fill";
    orientation="vertical";
  };
};

activity.setTheme(android.R.style.Theme_Material_Light)
activity.setContentView(loadlayout(layout))

--activity.ActionBar.hide()


import "android.media.AudioManager"
import "android.media.SoundPool"
local ogg = {"s_00.ogg","s_01.ogg","s_02.ogg","s_03.ogg","s_04.ogg","s_05.ogg",
  "s_06.ogg","s_07.ogg","s_08.ogg","s_09.ogg","s_10.ogg","s_11.ogg",
  "s_12.ogg","s_13.ogg","s_14.ogg","s_15.ogg","s_16.ogg","s_17.ogg",
  "s_18.ogg","s_19.ogg","s_20.ogg","s_21.ogg","s_22.ogg","s_23.ogg",}

local soundPool = SoundPool(1,AudioManager.STREAM_MUSIC,0)
local object = Object[#ogg+1]
for i=1,#ogg do
  object[i] = false
  soundPool.load(activity.getLuaDir("voidce".."/"..ogg[i]),0)
end
soundPool.setOnLoadCompleteListener(SoundPool.OnLoadCompleteListener{
  onLoadComplete=function(s,i,b)
    object[i] = true
  end})




require "import"

import "android.icu.text.SimpleDateFormat"
import "android.animation.ValueAnimator"
SunsetClock = {}
local activity = activity or service
local color = 0xffff0000
local xcolor = 0xffffffff
local drawcolor = 0xff000000
local size = nil
local width = nil --activity.getWidth()
local height = nil --activity.getHeight()
local poortime = 0
local Clock_h,Clock_m,Clock_s = 0,0,0
local invaliSelf = true
local sd = SimpleDateFormat("yyyy-MM-dd")
local sds = SimpleDateFormat("E")
local function getSimp(data)
  local eer = "周日"
  if !pcall(function()
      eer = sds.format(sd.parse(data))
    end) then
    eer = "周日"
  end
  switch(eer)
   case "周一"
    return 1,eer
   case "周二"
    return 2,eer
   case "周三"
    return 3,eer
   case "周四"
    return 4,eer
   case "周五"
    return 5,err
   case "周六"
    return 6,eer
   case "周日"
    return 7,eer
  end
  return 7,eer
end

local 月 = {"一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"}

local 日 = {"1号", "2号", "3号", "4号", "5号", "6号", "7号", "8号", "9号", "10号", "11号", "12号", "13号", "14号", "15号", "16号", "17号", "18号", "19号", "20号", "21号", "22号", "23号", "24号", "25号", "26号", "27号", "28号", "29号", "30号", "31号"}

local 周 = {"星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日"}

local 时 = {"00小时", "01小时", "02小时", "03小时", "04小时", "05小时", "06小时", "07小时", "08小时", "09小时", "10小时", "11小时", "12小时", "13小时", "14小时", "15小时", "16小时", "17小时", "18小时", "19小时", "20小时", "21小时", "22小时", "23小时"}

local 分 = {"00分钟","01分钟", "02分钟", "03分钟", "04分钟", "05分钟", "06分钟", "07分钟", "08分钟", "09分钟", "10分钟",
  "11分钟", "12分钟", "13分钟", "14分钟", "15分钟", "16分钟", "17分钟", "18分钟", "19分钟", "20分钟",
  "21分钟", "22分钟", "23分钟", "24分钟", "25分钟", "26分钟", "27分钟", "28分钟", "29分钟", "30分钟",
  "31分钟", "32分钟", "33分钟", "34分钟", "35分钟", "36分钟", "37分钟", "38分钟", "39分钟", "40分钟",
  "41分钟", "42分钟", "43分钟", "44分钟", "45分钟", "46分钟", "47分钟", "48分钟", "49分钟", "50分钟",
  "51分钟", "52分钟", "53分钟", "54分钟", "55分钟", "56分钟", "57分钟", "58分钟", "59分钟", "60分钟"}

local 秒 = {"01秒", "02秒", "03秒", "04秒", "05秒", "06秒", "07秒", "08秒", "09秒", "10秒",
  "11秒", "12秒", "13秒", "14秒", "15秒", "16秒", "17秒", "18秒", "19秒", "20秒",
  "21秒", "22秒", "23秒", "24秒", "25秒", "26秒", "27秒", "28秒", "29秒", "30秒",
  "31秒", "32秒", "33秒", "34秒", "35秒", "36秒", "37秒", "38秒", "39秒", "40秒",
  "41秒", "42秒", "43秒", "44秒", "45秒", "46秒", "47秒", "48秒", "49秒", "50秒",
  "51秒", "52秒", "53秒", "54秒", "55秒", "56秒", "57秒", "58秒", "59秒", "60秒"}

local simp,_simpc = getSimp(os.date("%Y-%m-%d"))
local h,m,s,e,d,r,y = tonumber(os.date("%H")),tonumber(os.date("%M")),tonumber(os.date("%S")),simp,tonumber(os.date("%d")),tonumber(os.date("%m")),tonumber(os.date("%Y"))
local cs,cm,ch,ce,cd,cr,cy,simpc = (360/#秒)*s,(360/#分)*m,(360/#时)*h,(360/#周)*e,(360/#日)*d,(360/#月)*r,y,_simpc
local cv1,cv2,cv3,cv4,cv5,cv6 = 0,0,0,0,0,0
local ofInt0 = ValueAnimator.ofFloat({0,-90})--整个的动画
.addUpdateListener{
  onAnimationUpdate=function(e)
    local nve = e.getAnimatedValue()
    cv1,cv2,cv3,cv4,cv5,cv6 = nve,nve,nve,nve,nve,nve
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(1000)

local ofInt1 = ValueAnimator.ofFloat({0,360/#秒})
.addUpdateListener{
  onAnimationUpdate=function(e)
    local nve = e.getAnimatedValue()
    cv1 = nve+cs
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(300)
local ofInt2 = ValueAnimator.ofFloat({0,360/#分})
.addUpdateListener{
  onAnimationUpdate=function(e)
    local nve = e.getAnimatedValue()
    cv2 = nve+cm
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(300)
local ofInt3 = ValueAnimator.ofFloat({0,360/#时})
.addUpdateListener{
  onAnimationUpdate=function(e)
    local nve = e.getAnimatedValue()
    cv3 = nve+ch
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(300)
local ofInt4 = ValueAnimator.ofFloat({0,360/#周})
.addUpdateListener{
  onAnimationUpdate=function(ee)
    local nve = ee.getAnimatedValue()
    cv4 = nve+ce-360/#周
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(300)
local ofInt5 = ValueAnimator.ofFloat({0,360/#日})
.addUpdateListener{
  onAnimationUpdate=function(e)
    local nve = e.getAnimatedValue()
    cv5 = nve+cd-360/#日
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(300)
local ofInt6 = ValueAnimator.ofFloat({0,360/#月})
.addUpdateListener{
  onAnimationUpdate=function(e)
    local nve = e.getAnimatedValue()
    cv6 = nve+cr-360/#月
    if _view~=nil and invaliSelf then
      _view.invalidate()
    end
  end
}
.setDuration(300)
function _updateAdd()
  _cs,_cm,_ch,_ce,_cd,_cr,_cy = cs,cm,ch,ce,cd,cr,cy
  local simp,_simpc = getSimp(os.date("%Y-%m-%d"))
  local h,m,s,e,d,r,y = tonumber(os.date("%H")),tonumber(os.date("%M")),tonumber(os.date("%S")),simp,tonumber(os.date("%d")),tonumber(os.date("%m")),tonumber(os.date("%Y"))
  cs,cm,ch,ce,cd,cr,cy,simpc = (360/#秒)*s,(360/#分)*m,(360/#时)*h,(360/#周)*e,(360/#日)*d,(360/#月)*r,y,_simpc
  if ofInt1~=nil and cs~=_cs then
    ofInt1.start()
    if m==59 and ((s==(60-poortime) or s>(60-poortime)) and _function0~=nil) then
      if _function0~=nil then
        _function0(s)
      end
    end
    if h==Clock_h and m==Clock_m and s==Clock_s-1 then
      if _function2~=nil then
        _function2(h,m,s)
      end
    end
  end
  if ofInt2~=nil and cm~=_cm then
    ofInt2.start()
  end
  if ofInt3~=nil and ch~=_ch then
    ofInt3.start()
    if _function1~=nil then
      _function1(h)
    end
  end
  if ofInt4~=nil and ce~=_ce then
    ofInt4.start()
  end
  if ofInt5~=nil and cd~=_cd then
    ofInt5.start()
  end
  if ofInt6~=nil and cr~=_cr then
    ofInt6.start()
  end
end

local luaRunnable = function()
  local pca1,pca2 = pcall(function()
    _updateAdd()
  end)
  if !pca1 then
    print("定时任务异常！"..pca2)
  end
end

local scheduledExecutor10 = Ticker()
scheduledExecutor10.Period=250
scheduledExecutor10.onTick=luaRunnable
local function updateAddOrSubtract(functio)
  scheduledExecutor10.start()
end
local function pauseAddOrSubtract(boo)
  scheduledExecutor10.setEnabled(boo)
end
local function stopAddOrSubtract()
  if scheduledExecutor10~=nil then
    scheduledExecutor10.stop()
    scheduledExecutor10 = nil
  end
end

function SunsetClock:getDrawable()
  return LuaDrawable(function(c,p,d)
    local b = d.bounds
    local w1 = b.right
    local h1 = b.bottom
    local w = width or w1
    local h = height or h1
    if h<w then
      w = h
    end
    p.setColor(color)
    p.setTextSize(size or h/66)
    c.drawColor(drawcolor)--背景
    p.setAntiAlias(true)
    p.setStrokeWidth(3)
    p.setStyle(p.Style.FILL)

    ---画秒钟
    local a1,z1,x1,y1 = (w/8),p.measureText(秒[#秒-1]),view.getX(),-(p.ascent() + p.descent())
    c.save()
    c.translate(y1+10,10)
    c.save()
    c.rotate((cv1 or 0)%-360,0,h/2)
    c.save()
    for i=#秒,1,-1 do
      c.drawText(秒[i],(h/2)-z1,h/2+(y1/2),p)
      c.rotate(360/#秒,0,h/2)
    end
    c.restore()
    c.restore()

    ---画分钟
    local z2 = p.measureText(分[#分-1])+a1
    c.save()
    c.rotate((cv2 or 0)%-360,0,h/2)
    c.save()
    for i=#分,1,-1 do
      c.drawText(分[i],(h/2)-z2,h/2+(y1/2),p)
      c.rotate(360/#分,0,h/2)
    end
    c.restore()
    c.restore()

    ---画小时
    local z3 = p.measureText(时[#时-1])+a1*2
    c.save()
    c.rotate((cv3 or 0)%-360,0,h/2)
    c.save()
    for i=#时,1,-1 do
      c.drawText(时[i],(h/2)-z3,h/2+(y1/2),p)
      c.rotate(360/#时,0,h/2)
    end
    c.restore()
    c.restore()

    ---画星期
    local z4 = p.measureText(周[#周-1])+a1*3
    c.save()
    c.rotate((cv4 or 0)%-360,0,h/2)
    c.save()
    for i=#周,1,-1 do
      c.drawText(周[i],(h/2)-z4,h/2+(y1/2),p)
      c.rotate(360/#周,0,h/2)
    end
    c.restore()
    c.restore()

    ---画日期
    local z5 = p.measureText(日[#日-1])+a1*4
    c.save()
    c.rotate((cv5 or 0)%-360,0,h/2)
    c.save()
    for i=#日,1,-1 do
      c.drawText(日[i],(h/2)-z5,h/2+(y1/2),p)
      c.rotate(360/#日,0,h/2)
    end
    c.restore()
    c.restore()

    ---画月份
    local z6 = p.measureText(月[#月-1])+a1*5
    c.save()
    c.rotate((cv6 or 0)%-360,0,h/2)
    c.save()
    for i=#月,1,-1 do
      c.drawText(月[i],(h/2)-z6,h/2+(y1/2),p)
      c.rotate(360/#月,0,h/2)
    end
    c.restore()
    c.restore()


    ---画年份
    local yr = tostring(cy)
    local z7 = p.measureText(yr)+a1*6.5
    c.drawText(yr,(h/2)-z7,h/2+(y1/2),p)

    p.setColor(xcolor)

    ---画文字星期
    p.setTextSize(size and size*2.08 or h/33)
    local _yr = tostring(simpc)
    local _z8 = p.measureText(yr)
    local _c8 = -(p.ascent() + p.descent())
    c.drawText(_yr,w-(_z8*2),_c8*3,p)

    ---选中框
    p.setStyle(p.Style.STROKE)
    --星期
    --c.drawRoundRect(w-(_z8*2)-10,_c8*1.5-10,w-(_z8),_c8*3+30,10,15,p)
    --时间
    c.drawRoundRect(-(y1-8)*2,(h/2)-y1-5,w-y1-15,(h/2)+y1+5,10,15,p)

    c.restore()


  end)
end
function SunsetClock:start()
  if _view~=nil then
    _cs,_cm,_ch,_ce,_cd,_cr,_cy = 0,0,0,0,0,0,0
    ofInt0.start()--初始动画
    task(1500,function()---等待启动动画结束
      ofInt1.start()
      ofInt2.start()
      ofInt3.start()
      ofInt4.start()
      ofInt5.start()
      ofInt6.start()
    end)
    updateAddOrSubtract()
  end
end
function SunsetClock:setEnabled(boo)
  pauseAddOrSubtract(boo)
  _cs,_cm,_ch,_ce,_cd,_cr,_cy = 0,0,0,0,0,0,0
  if boo then
    ofInt1.start()
    ofInt2.start()
    ofInt3.start()
    ofInt4.start()
    ofInt5.start()
    ofInt6.start()
  end
end
function SunsetClock:stop()
  stopAddOrSubtract()
end
function SunsetClock:new(view)
  if view~=nil then
    _view = view
   else
    error("\n\nview不能为空\n\n")
  end
  return self
end
function SunsetClock:setColor(cor,cor0)
  if type(cor)=="number" then
    color = cor
   elseif type(cor)=="string" then
    local cor = cor:match("#(.+)")
    if cor~=nil then
      color = tonumber("0x"..cor)
    end
  end
  if type(cor0)=="number" then
    xcolor = cor0
   elseif type(cor0)=="string" then
    local cor1 = cor0:match("#(.+)")
    if cor1~=nil then
      xcolor = tonumber("0x"..cor1)
    end
  end
end
function SunsetClock:setTextSize(t)
  size = t
end
function SunsetClock:setHeight(t)
  height = t
end
function SunsetClock:setWidth(t)
  width = t
end
function SunsetClock:invalidateSelf(boolean)
  invaliSelf = boolean
end
function SunsetClock:setBackgroundColor(cor)
  if type(cor)=="number" then
    drawcolor = cor
   elseif type(cor)=="string" then
    local cor = cor:match("#(.+)")
    if cor~=nil then
      drawcolor = tonumber("0x"..cor)
    end
  end
end
function SunsetClock:setPoortime(nn,fun)
  if (type(nn)=="number" and nn<60 and nn>0) and type(fun)=="function" then
    poortime,_function0 = nn,fun
   else
    poortime = 0
  end
end
function SunsetClock:setOnthehour(fun)
  if type(fun)=="function" then
    _function1 = fun
  end
end
function SunsetClock:setAlarmClock(h,m,s,fun)
  if type(h)=="number" then
    Clock_h = h
   else
    Clock_h = 0
  end
  if type(m)=="number" then
    Clock_m = m
   else
    Clock_m = 0
  end
  if type(s)=="number" then
    Clock_s = s
   else
    Clock_s = 0
  end
  if type(fun)=="function" then
    _function2 = fun
  end
end






--注意不要在使用过程中修改系统时间

local sun = SunsetClock:new(view)--初始化，传入布局id

sun:setColor("#FF00FFFF")--设置文字颜色和设置选中框颜色，两个参数，其中一个可以为空

--sun:setBackgroundColor(0x0)--设置背景色，颜色0x和#都可以

--sun:setTextSize(50)--文字大小,默认字体大小会跟随高度变化

--sun:setWidth(100),sun:setHeight(100)--设置画布宽高，默认跟随view高度改变

--sun:setEnabled(false)--false暂停定时器true恢复

--sun:invalidateSelf(false)false暂停画布刷新true恢复

--[[sun:setPoortime(3,function(e)
  print(e)
end)--整点前3秒,每秒回调]]

sun:setOnthehour(function(e)
  if object[e+1] then
    soundPool.play(e+1,1,1,1,0,1)--播放报时音效:
  end
end)--整点回调

--[[sun:setAlarmClock(7,26,5,function(h,m,s)
  print(h,m,s)
end)--设置闹钟回调，格式小时,分钟,秒钟,回调
]]


view.setBackground(sun:getDrawable())--获取drawable画布，给布局设置背景
view.post(Runnable({--view加载成功的回调
  run=function()
    sun:start()--启动计时器
  end}))





function onResume()--返回程序
  sun:invalidateSelf(true)
end

function onPause()--活动暂停
  sun:invalidateSelf(false)
end

function onDestroy()--活动退出
  sun:stop()--停止计时器
  soundPool.release()--释放
end
```

### Lua面向对象

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
dialog={h=100,w=100}
dialog.show=function()
  print("show")
  end
dialog.hide=function()
  print("hide")
  end
dialog.__index=dialog

function dialog.new(h,w)
  local d={}
  setmetatable(d,dialog)
  d.h=h
  d.w=w
  return d
  end

d=dialog.new(3,7)
d.show()
print(d.h)
```

### 创建快捷方式

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.content.*"
import "android.provider.Settings"
layout={
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="fill",
  {
    Button,
    onClick="but",
    text="创建快捷",
    layout_width="fill",
  },
}

activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
--activity.ActionBar.setTitle('AndroLua+')
activity.setContentView(loadlayout(layout))

function 打开系统设置()
return Intent(Settings.ACTION_SETTINGS)
end

function 启动活动(pack)
local shortIntent = Intent()
shortIntent.setAction("android.intent.action.MAIN")
shortIntent.addCategory("android.intent.category.LAUNCHER")
shortIntent.setClassName(activity.getPackageName(), pack)
return shortIntent
end


--android o不可用
--name  创建快捷的名称
--icon  创建快捷的图标(图标可以是R.drawable.icon,也可以是loadbitmap(图片路径))
--pack  快捷启动的intent
function addShortcut(name, icon, inte)
if Build.VERSION.SDK_INT < 26 then
local intent = Intent()
intent.setAction("com.android.launcher.action.INSTALL_SHORTCUT")
intent.putExtra(Intent.EXTRA_SHORTCUT_NAME, name)
if type(icon) == "number" then
local sr = Intent.ShortcutIconResource.fromContext(activity.getApplicationContext(), icon);
intent.putExtra(Intent.EXTRA_SHORTCUT_ICON_RESOURCE, sr)
else
intent.putExtra(Intent.EXTRA_SHORTCUT_ICON,icon)
end

intent.putExtra(Intent.EXTRA_SHORTCUT_INTENT, inte)
activity.sendBroadcast(intent)
Toast.makeText(this,"已创建快捷方式",0).show()
end
end

    but=function()
   addShortcut("快捷",loadbitmap("icon"),打开系统设置())--启动活动("com.androlua.Welcome")
     end





```

### 数组分割

```lua
--将某个数组分成两个数组，一个存放偶数，一个存放奇数
local t = {4,2,3,4,1,6,5,8,7}
function device(array)
    local oushuArr = {}
    local jishuArr = {}
    for i = 1,#array do
        if t[i] % 2 == 0 then
            oushuArr[#oushuArr + 1] = array[i]
        else
            jishuArr[#jishuArr + 1] = array[i]
        end
    end
    return oushuArr,jishuArr
end

local a,b = device(t)

for k,v in pairs(a) do
    print(v)
end
print("=========")
for k,v in pairs(b) do
    print(v)
end
```

### 数组排序删重复

```lua
--将一个数组从小到大排序
--然后将重复出现的数字全部删除(后续数字往前移)
local t = {4,2,3,4,1,6,5,8,7}
local newArray = {}
table.sort(t)
newArray[1] = t[1]
for i = 2,#t do
    if t[i] ~= t[i - 1] then
        newArray[i] = t[i]
    end
end
for k,v in pairs(newArray) do
    print(v)
end
```

### 使用root申请无障碍权限

```lua
function root申请无障碍(包名)
  return io.popen([[
su -c settings put secure enabled_accessibility_services ]]..(包名 or activity.getPackageName())..[[/com.androlua.LuaAccessibilityService
su -c settings put secure accessibility_enabled 1
]])
end
--root申请无障碍() 默认申请本应用
--root申请无障碍(包名) 申请其他应用(仅支持AndroLua系列开发的软件)
```

### 线程协程集合

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

--java线程
Thread(Runnable({
  run=function()
    print(1)
  end
})).start()


--LuaActivity线程
activity.thread(function(arg1, arg2)
  print(arg1, arg2)
end, {1, 2}).excute()


--Lua线程 (和第二个差不多)
thread(function(arg1, arg2)
  print(arg1, arg2)
end, 1, 2)


--真Lua线程 (和前面似乎都一样)
LuaThread(activity, function(arg1, arg2)
  print(arg1, arg2)
end, {3, 4}).start()


--LuaActivity协程
activity.task(function(arg1, arg2)
  print(arg1, arg2)
  return 1
end, {5, 6}, function(re)
  print(re)
end)
--.execute()  有一个协程了，会报错


--Lua线程 (和上面一样，不过这个有用)
task(function(arg1, arg2)
  print(arg1, arg2)
  return 1
end, 5, 6, function(re)
  print(re)
end)

--真Lua线程
local tasks=LuaAsyncTask(activity, function(arg1, arg2)
  print(arg1, arg2)
  return 00
end, function(re)
print(re)
end)
tasks.execute({11, 22})

--还有LuaRunnable, LuaAsyncTask里的一些其他方法没有写，协程可以定义代码运行延迟，综上
--大部分是重复，就是Thread和AsyncTask
--我觉着task和java线程还ok
```

### 自定义Toast位置

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

activity.setTitle('AndroLua+')
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

function print(nr)
    	context=activity.getApplicationContext();
		duration=Toast.LENGTH_SHORT;
		toast=Toast.makeText(context, nr,duration);
        toast.setGravity(Gravity.TOP,0,100)--| Gravity.TOP, 1, 40);--设置位置
		toast.show();
  end

print("小菜鸟")--用法
```

### 控件封装

```lua
--封装代码部分
import"android.animation.ObjectAnimator"
function InputView(t)
  local lay=loadlayout
    {
    FrameLayout,
    {
    TextView,
      text=t.text or "Hint",
      SingleLine=true,
      textSize="18dp",
      layout_height=-1,
      gravity=80,
      paddingBottom="10dp",
      paddingLeft="5dp",
      textColor=0xff797979},
    {
    EditText,
      id=t.id,
      textSize="18dp",
      gravity=80,
      layout_width=-1,
      layout_height=-1,
      SingleLine=true
      }
      }
  lay.getChildAt(1).setOnFocusChangeListener{onFocusChange=function(v,Focus)
      if v.Text=="" then
        local id,h=lay.getChildAt(0),v.height//2
        if Focus then
          ObjectAnimator()
          .ofFloat(id,"textSize",{18,12}).setDuration(200).start()
          .ofFloat(id,"Y",{0,-h}).setDuration(200).start()
          id.TextColor=0xff009688
         else
          ObjectAnimator()
          .ofFloat(id,"textSize",{12,18}).setDuration(200).start()
          .ofFloat(id,"Y",{-h,0}).setDuration(200).start()
          id.TextColor=0xff797979
        end
      end
    end
    }
  return function() return lay end
end

--结束
require"import"
import"android.widget.*"
--import"InputView"正常使用应单独文件导入使用
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

local layout=
{
  LinearLayout,
  orientation=1,
  layout_height=-1,
  layout_width=-1,
  {
    TextView,
    id="txt",
    textSize="20dp"
  },

  {
    InputView{
      id="a",
      text="控件封装"
    },
    layout_width=-1,
    layout_height="50dp"},

  {
    InputView{
      id="b",
      text="自定义控件"
    },
    layout_width=-1,
    layout_height="50dp"
  },

  {
    InputView{
      id="c",
      text="第一次点击获取焦点，不触发点击事件"
    },
    layout_width=-1,
    layout_height="50dp"
  },
}
activity.setContentView(layout)

local time=0
local function click(v)
  time=time+1
  txt.Text="触发点击："..tostring(time)
end
a.onClick=click
b.onClick=click
c.onClick=click
```

### 加载动画绘制

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout={
  FrameLayout;
  layout_height="fill";
  layout_width="fill";
  background="#ff000000";
  {
    LinearLayout;
    layout_height="30dp";
    layout_gravity="center";
    layout_width="30dp";
    id="ceshi";
    background="#ff000000";
  };
};
activity.setContentView(loadlayout(layout))

function 绘制加载动画二(参数)
  return LuaDrawable(function(c,p,d)
    import "android.graphics.Paint"
    import "android.graphics.Path"
    import "android.graphics.RectF"
    import "android.graphics.LinearGradient"
    import "android.graphics.Shader"
    local quYu=d.bounds
    local width=quYu.right
    local height=quYu.bottom
    p.setAntiAlias(true);
    p.setStrokeWidth(width*.2);
    p.setStyle(Paint.Style.STROKE);
    loadHeightNumOne=0
    loadChangeNumOne=height/100
    loadHeightNumTwo=height*.33
    loadHeightNumThree=height*.66
    return function(c)
      if loadHeightNumOne<height then
        loadHeightNumOne=loadHeightNumOne+loadChangeNumOne
       else
        loadHeightNumOne=0
      end
      if loadHeightNumTwo<height then
        loadHeightNumTwo=loadHeightNumTwo+loadChangeNumOne
       else
        loadHeightNumTwo=0
      end
      if loadHeightNumThree<height then
        loadHeightNumThree=loadHeightNumThree+loadChangeNumOne
       else
        loadHeightNumThree=0
      end
      p.setColor(参数.颜色一)
      c.drawLine(.1*width,loadHeightNumOne,.1*width,height-loadHeightNumOne,p)
      p.setColor(参数.颜色二)
      c.drawLine(.5*width,loadHeightNumTwo,.5*width,height-loadHeightNumTwo,p)
      p.setColor(参数.颜色三)
      c.drawLine(.9*width,loadHeightNumThree,.9*width,height-loadHeightNumThree,p)
      d.invalidateSelf()
    end
  end)
end

ceshi.background=绘制加载动画二({颜色一=0xFF327FFF,颜色二=0xFFF03B76,颜色三=0xFF795DFD})
```

### 拉伸变化尺寸

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
function dp2px(dpValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return dpValue * scale + 0.5
end
paddingCenter=dp2px(16)
layout={
  LinearLayout,
  layout_width="fill",
  layout_height="fill",
  {
    AbsoluteLayout,
    -- Gravity="center",
    --  orientation="vertical",
    x=50,
    y=50,
    layout_width="fill",
      layout_height="fill",
      {


      CardView,
      x=paddingCenter/2,
      y=paddingCenter/2,
      id="card",
      Elevation=0,
        backgroundColor=0xff2196F3,
        layout_width="56dp",
        layout_height="56dp",
        {
        ImageView,
        layout_width="fill",
        scaleType="centerCrop";
      layout_height="fill",
          },
      } ,
      {
      LinearLayout,

      id="kuang",
    Gravity="center",
  orientation="vertical",
layout_width="66dp",
layout_height="66dp",

},
}
}

import"android.animation.ValueAnimator"
import "android.graphics.drawable.GradientDrawable"
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
activity.setContentView(loadlayout(layout))
import "android.content.Context"
vibrator=activity.getSystemService(Context.VIBRATOR_SERVICE)

colors= { 0xff363C4A,0xff1F8792 };
bg = GradientDrawable(GradientDrawable.Orientation.TL_BR, colors);
card.getChildAt(0).setBackgroundDrawable(bg)
单位=activity.getWidth()/4
times=500
radiu=dp2px(6)


--card.getChildAt(0).setImageBitmap(loadbitmap(imgs[math.random(1,#imgs)]))
card.setRadius(radiu)
import "android.graphics.drawable.*"
gd=GradientDrawable()
gd.setColor(0x00ffffff)
gd.setStroke(5,0x10000000)

gd.setCornerRadii({radiu,radiu,radiu,radiu,radiu,radiu,radiu,radiu});
gd.setGradientType(1)
kuang.setBackgroundDrawable(gd)
w单位=1
h单位=1
function tointegers(num)
  if num-tointeger(num)>1.5
    return tointeger(num)+1
   else
    return tointeger(num)
  end
end
function update_w(v)
  w单位=tointegers(v.getLayoutParams().width/单位)
  if w单位<=0
    w单位=1
  end
  if h单位<=0
    h单位=1
  end
  --vibrator.vibrate(20)
  ValueAnimator.ofFloat({card.getLayoutParams().width,w单位*单位}).setDuration(times).addUpdateListener( {

    onAnimationUpdate=function(valueAnimator)

      currentValue = valueAnimator.getAnimatedValue();
      card.getLayoutParams().width = currentValue;
      card.requestLayout();
    end
  }).start();
end
function update_h(v)
  h单位=tointegers(v.getLayoutParams().height/单位)
  if w单位<=0
    w单位=1
  end
  if h单位<=0
    h单位=1
  end

  ValueAnimator.ofFloat({card.getLayoutParams().height,h单位*单位}).setDuration(times).addUpdateListener( {

    onAnimationUpdate=function(valueAnimator)

      currentValue = valueAnimator.getAnimatedValue();
      card.getLayoutParams().height = currentValue;
      card.requestLayout();
    end
  }).start();
end
function update_k(v)
  ValueAnimator.ofFloat({v.getLayoutParams().width,w单位*单位+1*paddingCenter}).setDuration(times*.6).addUpdateListener( {

    onAnimationUpdate=function(valueAnimator)

      currentValue = valueAnimator.getAnimatedValue();
      v.getLayoutParams().width = currentValue;
      v.requestLayout();
    end
  }).start();
  ValueAnimator.ofFloat({v.getLayoutParams().height,h单位*单位+1*paddingCenter}).setDuration(times).addUpdateListener( {

    onAnimationUpdate=function(valueAnimator)

      currentValue = valueAnimator.getAnimatedValue();
      v.getLayoutParams().height = currentValue;
      v.requestLayout();
    end
  }).start();
end
update_k(kuang)
update_w(kuang)
update_h(kuang)
sys,sxs=0,0

kuang.onTouch=function(v,e)
  ti=Ticker()
  ti.Period=500
  ti.onTick=function()

    if math.abs((e.getRawX()-sx)+(e.getRawY()-sy))<1
      if tointegers(v.getLayoutParams().width/单位)~=w单位 and tointegers(v.getLayoutParams().width/单位)>0
        update_w(v)
        vibrator.vibrate(25)
      end
      if tointegers(v.getLayoutParams().height/单位)~=h单位 and tointegers(v.getLayoutParams().height/单位)>0
        update_h(v)
        vibrator.vibrate(25)
      end

    end



  end
  --启动Ticker定时器
  if e.action==0--down
    参数=tonumber(os.time())


    sy=e.getRawY()
    sx=e.getRawX()
    ti.start()

   elseif e.action==1--up

    update_w(v)
    update_h(v)
    update_k(kuang)
    ti.stop()
   elseif e.action==2--move
    v.getLayoutParams().width = v.getLayoutParams().width+(e.getRawX()-sx)
    v.getLayoutParams().height = v.getLayoutParams().height+(e.getRawY()-sy)

    v.requestLayout();
    sy=e.getRawY()
    sx=e.getRawX()
  end
  return true
end
```

### 折线统计图

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.content.pm.ActivityInfo"
import "android.net.Uri"
import "android.content.Intent"

dip2px=function(dpValue)
  scale = this.getResources().getDisplayMetrics().density;
  return dpValue * scale + 0.5
end

getStatusBarHeight=function()
  xpcall(function()
    clazz=Class.forName("com.android.internal.R$dimen")
    object = clazz.newInstance();
    height = Integer.parseInt(tostring(clazz.getField("status_bar_height").get(object)))
    statusBarHeight2 =this.getResources().getDimensionPixelSize(height);
  end,
  function(a)
    xpcall(function()
      resourceId = this.getResources().getIdentifier("status_bar_height", "dimen", "android");
      if resourceId > 0
        statusBarHeight1 =this.getResources().getDimensionPixelSize(resourceId);
      end
      end,function(e)
      Utility_fun.toasts("发现错误：\n"..e)
    end,nil)
  end,nil)
  if statusBarHeight2 ~= nil
    return statusBarHeight2
   elseif statusBarHeight1~=nil
    return statusBarHeight1
   else
    return 0
  end
end

layout={
  LinearLayout;
  orientation="vertical";
  layout_height="fill";
  gravity="center|top";
  layout_width="fill";
  {
    ScrollView,
    layout_width="fill";
    layout_height="fill";
    {
      LinearLayout;
      orientation="vertical";
      gravity="center";
      layout_width="fill";
      layout_height="fill";
      {
        HorizontalScrollView;
        layout_width="fill";
            id="ssp",
        {
          LinearLayout;
          layout_width="fill";
          {
            TextView;
            layout_width="100%h";
            layout_height=activity.getWidth()-getStatusBarHeight()-dip2px(45);
            id="tv";
            layout_marginTop=getStatusBarHeight();
          };
        };
      };
      {SeekBar,
        id="Lux_S",
        padding="40dp";
        layout_width="fill";
        max="40000",
        layout_height="45dp";
      },
        {
          Button;
          id="bt_Ov";
          text="横屏查看",
          layout_width="fill";
        };
        {
          Button;
          id="bt_add";
          text="添加样本",
          layout_width="fill";
        };
      {
        TextView;
        id="Table_View";
        text="n",
        textColor="#000000",
        textSize="10sp",
        layout_weight="1",
      };
    },
  };
}
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
activity.setContentView(loadlayout(layout))

import "android.graphics.Paint"
import "android.graphics.Path"
import "android.graphics.Color"
import "android.graphics.Paint$Style"
import "android.provider.Settings"
import "android.graphics.Rect"
import "android.content.res.TypedArray"
import "android.graphics.PorterDuffXfermode"
import "android.graphics.PorterDuff"
import "android.graphics.PixelFormat"
import "java.math.BigDecimal"


SET_BRIGHTNESS=function(is)
  pcall(function()
    Settings.System.putInt(this.getContentResolver()
    ,Settings.System.SCREEN_BRIGHTNESS_MODE
    ,is);
  end,nil)
end

function onCreate()
  array = activity.getTheme().obtainStyledAttributes({
    android.R.attr.colorBackground,
    android.R.attr.textColorPrimary,
  });
  backgroundColor = array.getColor(0, 0xFF00FF);
  textColor = array.getColor(1, 0xFF00FF);
  array.recycle();

  this.setRequestedOrientation(1)
end



ap=256
sp=40000
ps={}
import "cjson"

adds=function()
  for cm=1,10--随机添加十个点
    apm=math.random (0,ap)
    spm=math.random (0,sp)
    --table.insert(ps,{aps=apm,sps=spm,bbs=cm,Top=true})
    table.insert(ps,{aps=(ap/100)*14,sps=0,bbs=1,Top=true})
    table.insert(ps,{aps=(ap/100)*17,sps=100,bbs=12,Top=true})
    table.insert(ps,{aps=(ap/100)*20,sps=300,bbs=12,Top=true})
    table.insert(ps,{aps=(ap/100)*26,sps=400,bbs=12,Top=true})
    table.insert(ps,{aps=(ap/100)*50,sps=600,bbs=12,Top=true})
    table.insert(ps,{aps=(ap/100)*75,sps=4000,bbs=12,Top=true})
    table.insert(ps,{aps=(ap/100)*100,sps=30000,bbs=12,Top=true})

    table.sort(ps,function(a,b)return a.sps<b.sps end)
  end
  --InTable=sorts_in(ps,4)
  --Table_View.Text=#ps.."~"..#InTable..":"..json.encode(InTable):gsub("},{","},\n{")
end

SET=function(a)
  xpcall(function(nu)
    Settings.System.putInt(this.getContentResolver(),
    Settings.System.SCREEN_BRIGHTNESS,nu)
  end,function(e)print(e)end,a)
end

Bigs=function(str,scale)
  return BigDecimal(str).setScale(scale, BigDecimal.ROUND_HALF_UP).doubleValue();
end

sorts_in=function(kl,Ret)
  if (kcd~=nil and (kcd==kl[1].sps and 0 or 1) or 1)==1--判断是否与上次执行table相同避免重复插值
    Ins=function(kls)
      jnps={}
      for u_,m_ in ipairs(kls)
        table.insert(jnps,{aps=m_.aps,sps=m_.sps,bbs=u_,Top=kls[u_].Top})
        if dap~=nil and dsp~=nil and u_<=table.size(kls) and u_~=1
          table.insert(jnps,{aps=(dap+m_.aps)/2,sps=(dsp+m_.sps)/2,bbs=u_,Top=false})--插入平分点。
        end
        dap,dsp=m_.aps,m_.sps
      end
      table.sort(jnps,function(a,b)return a.sps<b.sps end)
      return jnps
    end
    table.sort(kl,function(a,b)return a.sps<b.sps end)
    InTablm=Ins(kl)
    while math.abs(InTablm[2].aps-InTablm[1].aps)>(ap>256 and ap^0.4 or ap^0.1)
      InTablm=Ins(InTablm)
    end
    return InTablm
   else
    return Ret
  end
  kcd=kl[1].sps
  return Ret
end




--[[mSurfaceHolder = tv.getHolder();
mSurfaceHolder.addCallback({surfaceChanged=function(a)

end})
mSurfaceHolder.setFormat(PixelFormat.TRANSPARENT);
]]

insW=LuaDrawable(function(c,p,d)
  dars=d
  --tv.setLayerType(0,p)
  cH=c.Height-dip2px(75)
  cW=c.Width-dip2px(70)
  le_ri=dip2px(15)--设置绘制边距
  store=dip2px(1)
  p.setStyle(Style.FILL);
  p.setStrokeCap(Paint.Cap.ROUND)
  p.setStrokeWidth(store);
  p.setLetterSpacing(-0.02)
  p.setAntiAlias(true)
  p.setColor(0x00000000)
  p.setFakeBoldText(true)
  --p.setXfermode(PorterDuffXfermode(PorterDuff.Mode.SRC))

  xpcall(function()
    p.setColor(Back_C.background.Color)
    rect=Rect(0,0,c.Width,c.Height);
    c.drawRect(rect,p);
    end,function(e)
    p.setColor(backgroundColor)
    rect=Rect(0,0,c.Width,c.Height);
    c.drawRect(rect,p);
  end)

  p.setColor(0xff6bc36b);
  pts = float[10];
  pts[0] =(cW/sp)+dip2px(34)
  pts[1]=dip2px(30)
  pts[2]=(cW/sp)+dip2px(34)
  pts[3]=cH+dip2px(46)

  pts[4]=pts[2]
  pts[5]=pts[3]
  pts[6]=cW+dip2px(50)
  pts[7]=pts[5]
  c.drawLines(pts, p);
  p.setColor(0xff468bff)
  c.drawCircle(pts[0], pts[1], store*3, p);
  p.setColor(0xffff5d5d)
  c.drawCircle(pts[6], pts[7], store*3, p);
  c.translate(dip2px(35),dip2px(45))
  p.setColor(textColor);
  p.setTextSize(dip2px(10))
  p.setStrokeCap(Paint.Cap.ROUND)
  c.drawText("亮度/F",-dip2px(15),-dip2px(25),p)
  c.drawText("Lux/lx",cW+dip2px(3),cH-dip2px(8),p)
  --绘制分割线与数字



  lp=0
  for e=1,sp
    if e==lp+(sp/20)
      p.setColor(0xff6bc36b)
      c.drawCircle((cW/sp)*e,cH+dip2px(1),store*1.5, p);
      p.setColor(0xffbcbcbc);
      p.setStrokeWidth(1);
      p.setStyle(Style.FILL);
      ptsm={(cW/sp)*e,dip2px(0),(cW/sp)*e,cH-dip2px(0)}
      c.drawLines(ptsm,p)

      p.setColor(textColor);
      p.setTextSize(dip2px(6))
      c.drawText(""..e,(cW/sp)*e-dip2px(8),cH+dip2px(15),p)
      lp=e
    end
  end

  alp=0
  for e=1,ap
    if e==alp+(ap/8) or e==1
      p.setColor(0xff6bc36b)
      c.drawCircle(-dip2px(1),(cH/ap)*e,store*1.5, p);
      p.setColor(0xffbcbcbc);
      p.setStrokeWidth(1);
      p.setStyle(Style.FILL);
      ptsm={dip2px(0),(cH/ap)*e,cW-dip2px(0),(cH/ap)*e}
      c.drawLines(ptsm,p)
      p.setColor(textColor);
      p.setTextSize(dip2px(6))
      c.drawText(tointeger(Bigs(((ap-e)/ap)*100,0)).."%",-dip2px(25),(cH/ap)*e,p)
      alp=e
    end
  end

  c.drawText("0",-dip2px(15),cH+dip2px(15),p)
  p.setColor(0xff6bc36b)
  c.drawCircle(-dip2px(1),cH+dip2px(1),store*2, p);


  p.setStrokeWidth(dip2px(1));
  p.setStyle(Style.FILL);
  p.setTextSize(dip2px(6))
  H_=(cH/ap)
  W_=(cW/sp)

  mp,cs=0,0
  for f_,c_ in pairs(ps)
    if mp==0 and cs==0
      mp,cs=c_.sps*W_,(ap-c_.aps)*H_
      ms,mc=c_.sps,c_.aps
    end
    --[[    if c_.Top
      p.setColor(textColor)
      c.drawText(c_.aps.."F".."  "..c_.sps.."LX",c_.sps*W_,(ap-c_.aps)*H_-dip2px(4) , p)
      p.setColor(0xffff5d5d)
      c.drawCircle(c_.sps*W_,(ap-c_.aps)*H_, dip2px(3), p);
     else
      p.setColor(0xcf000000)
      --c.drawCircle(c_.sps*W_,(ap-c_.aps)*H_, dip2px(1), p);
    end]]
    p.setColor(textColor)
    c.drawText(tointeger(Bigs((c_.aps/ap)*100,0)).."%".."  "..c_.sps.."LX",(c_.sps*W_)+dip2px(4),(ap-c_.aps)*H_ , p)
    p.setColor(0x8fcb2eff)
    ptsm={c_.sps*W_,(ap-c_.aps)*H_,mp,cs}
    c.drawLines(ptsm,p)
    p.setColor(0xffcb2eff)
    c.drawCircle((c_.sps*W_),(ap-c_.aps)*H_, dip2px(2), p);
    p.setStyle(Style.FILL);
    mp,cs=c_.sps*W_,(ap-c_.aps)*H_
    ms,mc=c_.sps,c_.aps
  end

  X_YTouch=function(b1,b2)
    p.setStrokeWidth(5);
    p.setColor(0xffff1734)
    ptsm={b2*W_,(ap-b1)*H_,b2*W_,cH,
      0,(ap-b1)*H_,b2*W_,(ap-b1)*H_}
    c.drawLines(ptsm,p)
    p.setColor(0xffff1734)
    c.drawCircle(b2*W_,(ap-b1)*H_, dip2px(3), p);
    p.setTextSize(dip2px(10))
    c.drawText(tointeger(Bigs((b1/ap)*100,0)).."% "..tointeger(b2).."lx",b2*W_,(ap-b1)*H_-dip2px(8),p)
    if i~=0
      ssp.smoothScrollTo((b2*W_)-dip2px(140),0)
    end
    --SET(b1)
    bt_Ov.Text="Y: "..b1.." X: "..b2
  end

  if Bars~=nil and BarL~=nil
    X_YTouch(Bars,BarL)
   else
    X_YTouch(ps[1].aps,ps[1].sps)
  end

end)


ppnd=function(ps_,taab,LX,pm_)
  Fii={{sps=taab[ps_-1].sps,aps=taab[ps_-1].aps,bbs=pi_,Top=false},{aps=pm_.aps,sps=pm_.sps,bbs=pi_,Top=false}}
  apy=sorts_in(Fii,apy~=nil and apy or Fii)
  if LX<apy[table.size(apy)-1].sps
    for pi_,pis_ in ipairs(apy)
      if pis_.sps >=LX
        if pis_~=nil and pi_>=2
          if LX>=apy[pi_-1].sps
            --print(pis_.aps,pis_.sps,LX)
            Table_View.Text=#apy..cjson.encode(apy):gsub("},{","},\n{")
            ks,kl=pis_.aps,pis_.sps
            return pis_.aps,pis_.sps
          end
        end
      end
    end
  end
  if ks~=nil and kl~=nil
    return ks,kl
  end
end

app_Light=function(LX,taab)--判断值
  if LX>0 and LX<taab[table.size(taab)].sps
    for ps_,pm_ in pairs(taab)
      if pm_.sps >=LX
        if ps_~=nil and ps_>=2
          if LX>=taab[ps_-1].sps
            return ppnd(ps_,taab,LX,pm_)
          end
        end
      end
    end
   else
    if LX~=0
      return taab[table.size(taab)].aps,taab[table.size(taab)].sps
     else
      return taab[1].aps,taab[1].sps
    end
  end
  return taab[1].aps,taab[1].sps
end

adds()
--sorts_in(ps)
tv.background=insW

bt_add.onClick=function()
  adds()
  --mlp=sorts_in(ps)
  dars.invalidateSelf()
end
function onDestroy()
  if 传感器~=nil
    传感器.unregisterListener(S_Linsen)
    SET_BRIGHTNESS(automicBrightness and 1 or 0)
  end
end


import "android.content.Context"
import "android.hardware.Sensor"
import "android.hardware.SensorEventListener"
function onStart()
  isCan=Settings.System.canWrite(this)
  if isCan
    automicBrightness = Settings.System.getInt(this.getContentResolver(),Settings.System.SCREEN_BRIGHTNESS_MODE) == Settings.System.SCREEN_BRIGHTNESS_MODE_AUTOMATIC;
    if not automicBrightness
      SET_BRIGHTNESS(0)
    end
    传感器 = activity.getSystemService(Context.SENSOR_SERVICE)
    光线传感器 = 传感器.getDefaultSensor(Sensor.TYPE_LIGHT)
    S_Linsen=SensorEventListener({
      onSensorChanged=function(event)
        if IsStar==nil or IsStar
          Bars,BarL=app_Light(event.values[0],ps)
          if dars~=nil and Bars~=nil
            SET(Bars)
            bt_Ov.Text=Bars.." "..event.values[0]
            Lux_S.setProgress(event.values[0])
            dars.invalidateSelf()
          end
        end
    end,nil})

    传感器.registerListener(S_Linsen, 光线传感器, 4)
   else
    if intent==nil
      print("没有修改系统设置权限，请开启")
      intent = Intent(Settings.ACTION_MANAGE_WRITE_SETTINGS,
      Uri.parse("package:"..this.getPackageName()))intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
      this.startActivityForResult(intent, 200);
     else
      print("仍然未允许权限，程序将无法运行！")
    end
  end
end

Lux_S.setOnSeekBarChangeListener{
  onProgressChanged=function(a,i)
    if IsStar==false
      Bars,BarL=app_Light(i,ps)
      ii=i
      if isCan
        SET(Bars)
       else
        print("无权限，设置亮度失败！")
      end
      if dars~=nil
        dars.invalidateSelf()
      end
    end
  end,
  onStartTrackingTouch=function()
    IsStar=false
  end,
  onStopTrackingTouch=function()
    IsStar=true
  end
}

bt_Ov.onClick=function()
  Ov_R=this.getRequestedOrientation()
  if Ov_R==1
    this.setRequestedOrientation(0);
   elseif Ov_R==0
    this.setRequestedOrientation(1);
  end
end
```

### 文件管理器

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "java.io.File"
import "android.webkit.MimeTypeMap"
import "android.content.Intent"
import "android.net.Uri"
import "android.graphics.Paint"
import "android.text.format.Formatter"
import "android.text.format.DateUtils"
import "android.text.style.RelativeSizeSpan"
import "android.text.SpannableString"
import "android.text.Spanned"
import "android.view.*"
layout={
  LinearLayout;
  orientation="vertical";
  layout_height="fill";
  layout_width="fill";
  {
    PullingLayout;
    id="pl";
    layout_height=0;
    PullDownEnabled=true;
    layout_weight=1;
    {
      ListView;
      id="lv";
      layout_width="fill";
      layout_height="fill";
    };
  };
  {
    LinearLayout;
    layout_width="fill";
    {
      Button;
      id="CopyBtn";
      layout_width=0;
      layout_weight=1;
      Text="复制";
      textColor="#ffff0000";
      background="ann.png";
    };
    {
      Button;
      id="CutBtn";
      layout_width=0;
      layout_weight=1;
      Text="剪切";
      textColor="#ffff0000";
      background="ann.png";
    };
    {
      Button;
      id="PasteBtn";
      layout_width=0;
      layout_weight=1;
      Text="粘贴";
      textColor="#ffff0000";
      background="ann.png";
    };
    {
      Button;
      id="DeleteBtn";
      layout_width=0;
      layout_weight=1;
      Text="删除";
      textColor="#ffff0000";
      background="ann.png";
    };
    {
      Button;
      id="NewBtn";
      layout_width=0;
      layout_weight=1;
      Text="新建";
      textColor="#ffff0000";
      background="ann.png";
    };
  };
};
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
activity.setContentView(loadlayout(layout))

onCreateOptionsMenu=function(menu)
  menu.add(0,1,1,"排序").setShowAsActionFlags(2)
  CB=CheckBox()
  CB.Text="显示隐藏文件"
  CB.onClick=function(v)
    ShowHidden=v.isChecked()
    ReOpen()
  end
  menu.add(0,2,0,"显示隐藏文件").setActionView(CB).setShowAsActionFlags(2)
end
onOptionsItemSelected=function(item)
  if item.Title=="排序" then
    SortMethodItem={"名称（升序）","名称（降序）","时间（升序）","时间（降序）"}
    SortMethodDlg=AlertDialog.Builder(this)
    .setSingleChoiceItems(SortMethodItem,SortMethod-1,{onClick=function(v,p)
        SortMethod=p+1
        print(SortMethodItem[SortMethod])
        ReOpen()
      end
    })
    SortMethodDlg.show()
  end
end

datalist={}
adapter=ArrayAdapter(this,android.R.layout.simple_list_item_multiple_choice,datalist)
lv.setChoiceMode(ListView.CHOICE_MODE_MULTIPLE_MODAL)

function getSelectedItem()
  SelectedItem={}
  CheckItemIds=luajava.astable(lv.getCheckItemIds())
  for k,v in pairs(CheckItemIds) do
    table.insert(SelectedItem,FileList[v+1])
  end
  CheckedItemCount=lv.getCheckedItemCount()
end

lv.setMultiChoiceModeListener(ListView.MultiChoiceModeListener{
  onItemCheckedStateChanged=function(mode,position,id,checked)
    getSelectedItem()
  end,

  onCreateActionMode=function(mode,menu)
    mActionMode=mode
    mode.setTitle("选择")
    menu.add(0,1,0,"全选").setShowAsActionFlags(2)
    menu.add(0,2,1,"反选").setShowAsActionFlags(2)
    return true
  end,

  onPrepareActionMode=function(mode,menu)
    return true
  end,

  onActionItemClicked=function(mode,item)
    func[item.Title]()
    return true
  end,

  onDestroyActionMode=function(mode)
    SelectedItem={}
  end
})
lv.setAdapter(adapter)
lv.onItemClick=function(l,v,p,i)
  if FileList[p+1].isDirectory() then
    OpenDir(FileList[p+1].toString())
   else
    OpenFile(FileList[p+1].toString())
  end
end
func={
  ["全选"]=function()
    for n=0,lv.getCount()-1 do
      lv.setItemChecked(n,true)
    end
  end,
  ["反选"]=function()
    for n=0,lv.getCount()-1 do
      lv.setItemChecked(n,not(lv.isItemChecked(n)))
    end
  end,
}

function SortFile(mFileList)
  SortFunctions={
    --名称（升序）
    function(a,b)
      return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.Name<b.Name)
    end,
    --"名称（降序）
    function(a,b)
      return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.Name>b.Name)
    end,
    --时间（升序）
    function(a,b)
      return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.lastModified()<b.lastModified())
    end,
    --时间（降序）
    function(a,b)
      return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.lastModified()>b.lastModified())
    end,
  }
  table.sort(mFileList,SortFunctions[SortMethod])
end
SortMethod=1

function OpenFile(path)
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match(".+%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName) or "*/*"
  print("打开",FileName.."\n".."文件类型="..Mime)
  if Mime then
    intent = Intent()
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
    intent.setAction(Intent.ACTION_VIEW);
    intent.setDataAndType(Uri.fromFile(File(path)), Mime);
    activity.startActivity(intent)
    return true
   else
    return false
  end
end

function dp2px(dpValue)
  local scale = this.getResources().getDisplayMetrics().density
  return (dpValue * scale)
end
mPaint=Paint()
mPaint.setTextSize(dp2px(18))

function DealName(v)
  local name=v.Name
  local sizes=Formatter.formatFileSize(activity,v.length())
  local time=v.lastModified()
  time=DateUtils.getRelativeTimeSpanString(time,
  System.currentTimeMillis(),
  DateUtils.MINUTE_IN_MILLIS,
  DateUtils.FORMAT_SHOW_TIME|DateUtils.FORMAT_SHOW_DATE|DateUtils.FORMAT_ABBREV_ALL)
  local l=mPaint.breakText(name,0,utf8.len(name),true,0.75*activity.getWidth(),nil)
  if utf8.len(name)>l then
    name=utf8.sub(name,1,l//2).."..."..utf8.sub(name,-l//2,-1)
  end
  local s1=name.."\n"
  local s2=time
  if v.isFile() then
    s2=s2.."  | "..sizes
  end
  s=s1..s2
  local a,b=utf8.find(s,s2)
  print(a,b,utf8.len(s))
  spannableString=SpannableString(s)
  sizeSpan=RelativeSizeSpan(0.8)
  spannableString.setSpan(sizeSpan,a-1,b,Spanned.SPAN_INCLUSIVE_EXCLUSIVE)
  return spannableString
end

function OpenDir(dir)
  Dir=dir
    adapter.clear()
  FileList=luajava.astable(File(dir).listFiles())
  SortFile(FileList)
  mFileList={}
  for k,v in pairs(FileList) do
    if not(v.isHidden()) or ShowHidden then
      table.insert(mFileList,v)
    end
  end
  FileList=mFileList
  if #FileList==0 then
    print("空")
    CheckedItemCount=0
    SelectedItem={}
  end
  --print(CheckedItemCount)
  for k,v in pairs(FileList) do
    adapter.add(DealName(v))
  end
end

function ReOpen()
  if mActionMode then
    mActionMode.finish()
  end
  OpenDir(Dir)
end

pl.onRefresh=function(v)
  ReOpen()
  v.refreshFinish(0)
end

function onKeyDown(code,event)
  if code==4 then
    if Dir~=SDPath then
      OpenDir(File(Dir).getParent())
     else
      activity.finish()
    end
    return true
  end
end

function CopyTable(table1)
  local table2={}
  for k,v in pairs(table1) do
    table2[k]=v
  end
  return table2
end

TaskFileList={}
function getTaskFileListContent()
  local s=""
  for k,v in pairs(TaskFileList) do
    s=s.."\n"..v.toString()
  end
  return s
end

CopyBtn.onClick=function()
  if #SelectedItem~=0 then
    TaskFileList=CopyTable(SelectedItem)
    mActionMode.finish()
    TaskType="Copy"
    print("选择了"..getTaskFileListContent())
  end
end

CutBtn.onClick=function()
  if #SelectedItem~=0 then
    TaskFileList=CopyTable(SelectedItem)
    mActionMode.finish()
    TaskType="Cut"
    print("选择了"..getTaskFileListContent())
  end
end

PasteBtn.onClick=function()
  if TaskFileList and TaskType then
    if TaskType=="Copy" then
      for k,v in pairs(TaskFileList) do
        LuaUtil.copyDir(v,File(Dir,v.Name))
      end
      print("复制了"..getTaskFileListContent().."\n到"..Dir)
     elseif TaskType=="Cut"
      for k,v in pairs(TaskFileList) do
        os.execute("mv "..v.toString().." "..Dir.."/"..v.Name)
      end
      print("移动了"..getTaskFileListContent().."\n到"..Dir)
    end
    ReOpen()
    TaskFileList=nil
    TaskType=nil
  end
end

DeleteBtn.onClick=function()
  if #SelectedItem~=0 then
    TaskFileList=CopyTable(SelectedItem)
    mActionMode.finish()
    for k,v in pairs(TaskFileList) do
      LuaUtil.rmDir(v)
    end
    print("删除了"..getTaskFileListContent())
    TaskFileList=nil
    ReOpen()
  end
end

NewBtn.onClick=function()
  NewET=EditText(this)
  AlertDialog.Builder(this)
  .setTitle("新建")
  .setView(NewET)
  .setPositiveButton("文件",{onClick=function(v)
      if #(NewET.Text)~=0 then
        File(Dir,NewET.Text).createNewFile()
        print("新建文件",NewET.Text)
        ReOpen()
      end
    end
  })
  .setNeutralButton("文件夹",{onClick=function(v)
      if #(NewET.Text)~=0 then
        File(Dir,NewET.Text).mkdir()
        print("新建文件夹",NewET.Text)
        ReOpen()
      end
    end
  })
  .setNegativeButton("取消",nil)
  .show()
end

ShowHidden=false--true
SDPath=Environment.getExternalStorageDirectory().toString()
OpenDir(SDPath)
```

### Json互转Table

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

--json方法1-->
cjson=require "cjson"

local 手机号=10086;
表={
  ["姓名"]="三水一金",
  ["手机"]=手机号
}
转json=cjson.encode(表)
转table=cjson.decode(转json)

print(转json)
print(dump(转table))

--打印table表-->
for m,y in pairs(转table) do
  print(m,y)
end



print("---分割线---")



--json方法2-->
json=require "cjson"

--将table类型转换成json类型的字符串

tab1={
  "a",
  "b",
  "c",
  "d",
  "e"
}

转json=json.encode(tab1)

print(转json)

--将json类型转换成table类型

转table=json.decode(转json)

for k,v in pairs(转table)
  print(v)
end



print("---分割线---")



--带索引table表
table2={
  ["姓名"]="三水一金",
  ["性别"]="男",
  ["年龄"]="17"
}

转json=json.encode(table2)

print(转json)

--将json类型转换成table类型

转table=json.decode(转json)

for k,v in pairs(转table)
  print(k.." : "..v)
end
```

### 仿MT管理器相对布局加适配器

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout=
{
  LinearLayout,--线性布局
  orientation="vertical",--布局方向
  layout_width="fill",--布局宽度
  layout_height="fill",--布局高度

  {
    RelativeLayout,--相对布局
    layout_width="fill",--布局宽度
    layout_height="fill",--布局高度
    {
      ListView,--列表视图控件
      layout_width="50%w",--布局宽度
      layout_height="fill",--布局高度
      dividerHeight="1",--分割线高度
      verticalScrollBarEnabled=false,--隐藏滑条
      id="liebiao1",--控件ID
      layout_alignParentLeft=true,--重力居左
      layout_centerVertical=true,--将控件置于垂直方向的中心位置
    },
    {
      TextView,--垂直分割线
      layout_width="1px",--布局宽度
      layout_height="fill",--布局高度
      backgroundColor="#bebebe",--背景色
      layout_centerInParent=true,--将控件置于父控件的中心位置
      layout_centerHorizontal=true,--将控件置于水平方向的中心位置
    },
    {
      ListView,--列表视图控件
      layout_width="50%w",--布局宽度
      layout_height="fill",--布局高度
      dividerHeight="1",--分割线高度
      verticalScrollBarEnabled=false,--隐藏滑条
      id="liebiao2",--控件ID
      layout_alignParentRight=true,--重力居右
      layout_centerVertical=true,--将控件置于垂直方向的中心位置
    },
  },--相对布局结束

}--线性布局结束

activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
activity.setTitle("仿MT管理器相对布局加适配器")


activity.setContentView(loadlayout(layout))
适配布局={
  LinearLayout,--线性布局
  orientation="vertical",--布局方向
  layout_width="fill",--布局宽度
  layout_height="wrap",--布局高度
  gravity="left|center",--重力居左｜置中
  padding="10dp",--布局填充
  {
    TextView,--文本框控件
    text="标题",--文本内容
    textSize="15sp",--文本大小
    textColor="#222222",--文本颜色
    id="wenben1",--控件ID
  },
  {
    TextView,--文本框控件
    text="简介",--文本内容
    textSize="12sp",--文本大小
    id="wenben2",--控件ID
  },
}--线性布局结束

--来自James
-- -------第一个列表
构建adp1=LuaAdapter(activity,适配布局)
liebiao1.setAdapter(构建adp1)
--远端或本地数据
总列表a=[[
《aaaa》
【QQ744066461】

《bbb》
【James】

《cccccc》
【tencent】
]]
--开始取值
读取列表a=总列表a:gmatch("《(.-)》\n【(.-)】")--截取格式
for a,b in 读取列表a do--循环取值
  构建adp1.add{--构建视图控件
    wenben1=a,--标题
    wenben2=b,--简介
  }
end
--单击适配项目
liebiao1.onItemClick=function(a,b)
  print(b.Tag.wenben1.Text)--标题
  print(b.Tag.wenben2.Text)--简介
  return true--返回
end


-- -------第二个列表
构建adp2=LuaAdapter(activity,适配布局)
liebiao2.setAdapter(构建adp2)
--远端或本地数据
总列表b=[[
《James》
【QQ：744066461】

《andlua》
【nb】

《可以多加几行测试》
【tencent】

《噜啦噜啦嘞》
【tencent】
]]
--开始取值
读取列表b=总列表b:gmatch("《(.-)》\n【(.-)】")--截取格式
for a,b in 读取列表b do--循环取值
  构建adp2.add{--构建视图控件
    wenben1=a,--标题
    wenben2=b,--简介
  }
end
--单击适配项目
liebiao2.onItemClick=function(a,b)
  print(b.Tag.wenben1.Text)--标题
  print(b.Tag.wenben2.Text)--简介
  return true--返回
end
```

### PullingLayout自定义下拉布局

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout={
  LinearLayout;
  layout_width="fill";
  orientation="vertical";
  layout_height="fill";
  {
    PullingLayout;
    id="pl";
    PullDownEnabled="true";
    PullUpEnabled="true";
    layout_height="fill";
    layout_width="fill";
    {
      LinearLayout;
       {
        TextView;
        textSize="14sp",
        textColor=0xff000000,
        gravity="center";
        text="下拉刷新";
      };
    };
  };
};

plly={
  LinearLayout;
  layout_width="match_parent";
  gravity="center";
  orientation="vertical";
  {
    ProgressBar;
  };
};

activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

activity.setContentView(loadlayout(layout))
activity.setTitle("自定义下拉刷新")

function PullingLayout自定义上拉布局(pl,lay)
  pl.getChildAt(0).getChildAt(0).removeView(pl.getChildAt(0).getChildAt(0).getChildAt(0))
  pl.getChildAt(0).getChildAt(0).addView(loadlayout(lay))
end
PullingLayout自定义上拉布局(pl,plly)
function PullingLayout自定义下拉布局(pl,lay)
  pl.getChildAt(2).getChildAt(0).removeView(pl.getChildAt(2).getChildAt(0).getChildAt(0))
  pl.getChildAt(2).getChildAt(0).addView(loadlayout(lay))
end
PullingLayout自定义下拉布局(pl,plly)
```

### 仿TabLayout

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout={
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="fill",
  backgroundColor=0xFFF5F5F5,
  {
    LinearLayout,
    layout_height="56dp",
    id="parent",
    backgroundColor=0xFFF4A7B9,
    layout_width="fill",
    {
      HorizontalScrollView,
      layout_height="fill",
      layout_width="fill",
      layout_marginTop="12dp",
      {
        FrameLayout,
        layout_height="fill",
        layout_width="fill",
        {
          LinearLayout,
          layout_height="fill",
          layout_width="fill",
          id="choose",
        },
        {
          LinearLayout,
          layout_height="fill",
          layout_width="fill",
          id="bar",
        },

      }
    }
  },
  {
    PageView,
    id="pg",
    layout_width="fill",
    pages={},
  }
}
--dingyi
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
activity.setContentView(loadlayout(layout))

local function addTab(t)
  return {
    CardView;
    cardBackgroundColor=0x00000000,
    elevation="0";
    radius="38dp";
    id="___",
    layout_marginLeft="12dp",
    onClick=function(v)
      for i=0,v.parent.getChildCount()-1 do
        if v.parent.getChildAt(i).id==v.id then
          pg.setCurrentItem(i)
          return
        end
      end
    end,
    {
      LinearLayout;
      layout_width="-2";
      layout_height="-2";
      padding="5dp",
      paddingLeft="14dp",
      paddingRight="14dp",
      orientation="vertical";
      {
        TextView;
        textSize="14sp",
        textColor=0xFFFFD6E2,
        gravity="center";
        text=t;
      };
    };
  };
end

local function setWidth(a,b)

  local q=a.layoutParams
  q.width=b
  a.layoutParams=q
end

function dp2px(dpValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return dpValue * scale + 0.5
end

bar.addView(loadlayout(addTab("用户应用"),nil,bar.class))
bar.addView(loadlayout(addTab("系统应用"),nil,bar.class))
bar.addView(loadlayout(addTab("已冻结"),nil,bar.class))
bar.addView(loadlayout(addTab("已冻结2222"),nil,bar.class))
bar.addView(loadlayout(addTab("已冻结2222"),nil,bar.class))

bar.getChildAt(0).getChildAt(0).getChildAt(0).textColor=0xffffffff

choose.addView(loadlayout({
  CardView;
  cardBackgroundColor=0xFFF2BECA,
  elevation="0";
  radius="38dp";
  layout_height="28dp",
  layout_marginLeft="12dp",
},nil,choose.class))

bar.getChildAt(0).post {
  run=function()
    setWidth(choose.getChildAt(0),bar.getChildAt(0).width)
  end
}


for i=1,4 do
  pg.adapter.add(loadlayout{
    LinearLayout,
    layout_width="fill",
    layout_height="fill",
    {
      TextView,
      text=i..""
    }
  })
end

local data={
  scrollData={},

}



pg.setOnPageChangeListener{
  onPageSelected=function(t)

    for i=0,bar.getChildCount()-1 do
      bar.getChildAt(i).getChildAt(0).getChildAt(0).textColor=0xFFFFD6E2
    end
    bar.getChildAt(t).getChildAt(0).getChildAt(0).textColor=0xffffffff
    setWidth(choose.getChildAt(0),bar.getChildAt(t).width)
    choose.getChildAt(0).x=bar.getChildAt(t).x
  end,
  onPageScrollStateChanged=function(i)
    data.scrollData.scroll=i>0
  end,
  onPageScrolled=function(a,b,c)
    local nowView=bar.getChildAt(a)

    local nextView=bar.getChildAt(a==bar.getChildCount() and bar.getChildCount() or a+1)

    if data.scrollData.scroll and b~=0 then
      if data.scrollData.last and data.scrollData.last<b then
        if nextView.width<nowView.width then
          setWidth(choose.getChildAt(0),nowView.width-((nowView.width-nextView.width)*b))
         else
          setWidth(choose.getChildAt(0),nowView.width+((nextView.width-nowView.width)*b))
        end
        choose.getChildAt(0).x=nextView.x-((nextView.x-nowView.x)*(1-b))
       else

        local lastView=bar.getChildAt(a)
        local nowView=bar.getChildAt(a+1)

        if lastView.width>nowView.width then
          setWidth(choose.getChildAt(0),nowView.width+((lastView.width-nowView.width)*(1-b)))
         else
          setWidth(choose.getChildAt(0),nowView.width-((nowView.width-lastView.width)*(1-b)))
        end

        choose.getChildAt(0).x=lastView.x+((nowView.x-lastView.x)*b)
      end

    end

    if b==0 or c==0 then
      for i=0,bar.getChildCount()-1 do
        bar.getChildAt(i).getChildAt(0).getChildAt(0).textColor=0xFFFFD6E2
      end
      bar.getChildAt(pg.getCurrentItem()).getChildAt(0).getChildAt(0).textColor=0xffffffff
    end

    data.scrollData.page=a
    data.scrollData.last=b
  end,
}
```

### ExpandableListView的使用

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout={
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="fill",
  {
    ExpandableListView,
    id="Expandable",
    layout_width="fill",
  },
}
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题activity.setContentView(loadlayout(layout))
activity.setContentView(loadlayout(layout))--显示

headLinears={
  LinearLayout;
  layout_width="fill";
  layout_height="50dp";
  orientation="vertical";
  gravity="center|left";
  background="#00ffff",
  {
    TextView;
    layout_marginLeft="40dp";
    id="App_name";
    gravity="center";
  };
};
headLinears2={
  LinearLayout;
  layout_width="fill";
  layout_height="50dp";
  orientation="vertical";
  gravity="center|left";
  background="#ff00ff",
  {
    TextView;
    layout_marginLeft="45dp";
    id="App_name";
    gravity="center";
  };
};

fg={}
fs={}
ase={}

ns={"Check view","Adapter view","Advanced Widget","Layout","Advanced Layout",
}

wds={
  {"CheckBox","RadioButton","ToggleButton","Switch"},
  {"ListView","ExpandableListView","Spinner"},
  {"SeekBar","ProgressBar","RatingBar",
    "DatePicker","TimePicker","NumberPicker"},
  {"LinearLayout","AbsoluteLayout","FrameLayout"},
  {"RadioGroup","GridLayout",
    "ScrollView","HorizontalScrollView"},
}


mAdapter=LuaExpandableListAdapter(activity,fg,fs,headLinears,headLinears2)

for k,v in ipairs(ns) do
  table.insert(fg,{App_name={Text=v}})
  for ks,kv in ipairs(wds[k]) do
    ase[ks]={App_name={Text=wds[k][ks]}}
  end
  table.insert(fs,ase)
  ase={}
end


mAdapter.notifyDataSetChanged()
Expandable.setAdapter(mAdapter)

Expandable.onChildClick=function(l,v,g,c)
  print(":ChildClick")
end

Expandable.onGroupClick=function(l,v,p,s)
  print(":GroupClick")
end
```

### HSV配色版

```lua
import "android.text.method.DigitsKeyListener"
import "android.graphics.Color"
import "android.graphics.Shader$TileMode"
import "android.graphics.*"
import "android.graphics.drawable.shapes.*"
import "android.graphics.drawable.*"
import "android.graphics.Paint$Cap"

function rgb2hsv(color,hsv)
  $hsv0=hsv
  if not hsv
    hsv=float[3]
  end
  if type(color)=="string"
    color=Color.parseColor(color)
  end
  Color.colorToHSV(color, hsv)
  if hsv0
    return hsv
   else
    return luajava.astable(hsv)
  end
end
function hsv2rgb(hsv1,hsv2)
  local hsv,alpha
  if hsv2
    alpha=hsv1
    hsv=hsv2
   else
    alpha=255
    hsv=hsv1
  end
  hsv[1]=hsv[1]%360
  if type(hsv)=="table"
    hsv=float(hsv)
  end
  $color=Color.HSVToColor(alpha,hsv)
  return color
end

function HSV配色盘(color0,fun)
  if type(color0)=="number"
    color0=string.format("%0x",color0)
  end
  HSV_dialog=AlertDialog.Builder(this)
  .setTitle("HSV配色盘")
  if fun
    HSV_dialog.setPositiveButton("确定",{onClick=lambda s:fun("#"..HSV_input.text,s)})
   else
    HSV_dialog.setPositiveButton("复制",{onClick=function(v)
        activity.getSystemService(Context.CLIPBOARD_SERVICE).setText("#"..HSV_input.text)
        Toast.makeText(activity, "复制成功",Toast.LENGTH_SHORT).show()
      end})
  end
  HSV_dialog.setNegativeButton("取消",nil)
  .setCancelable(false)
  .setView(loadlayout{
    LinearLayout;
    padding="10dp";
    orientation="vertical";
    gravity="center";
    {
      CardView;
      CardElevation="5dp";
      layout_width="80dp";
      layout_height="80dp";
      backgroundColor=4294967295;
      layout_marginTop="8dp";
      radius="40dp";
      id="HSV_card";
    };
    {
      LinearLayout;
      gravity="center";
      layout_width="fill";
      padding="5dp";
      {
        TextView;
        textSize="18dp";
        text="#";
        backgroundColor=0;
        textColor=0xff000000;
      };
      {
        EditText;
        id="HSV_input";
        singleLine=true;
        padding=0;
        backgroundColor=0;
        hint="FFFFFF";
        textSize="18dp";
        keyListener=DigitsKeyListener.getInstance("1234567890abcdefABCDEF")
      };
    };
    {
      SeekBar;
      id="拖动H";
      layout_width="280dp";
      Max=360*2;
      layout_margin="5dp";
    };
    {
      SeekBar;
      id="拖动S";
      layout_width="280dp";
      layout_margin="5dp";
      Max=255;
    };
    {
      SeekBar;
      id="拖动V";
      layout_width="280dp";
      layout_margin="5dp";
      Max=255
    };
    {
      TextView;
      id="HSV_value";
      gravity="center";
      layout_width="fill";
      padding="5dp";
      text="H:360.0 S:255 V:255";
    };
  }

  )
  .show()

  local H,S,V=0,0,1
  HSV_input.addTextChangedListener{
    onTextChanged=function(s)
      $s=tostring(s)
      if #s==6
        $n=tonumber("0xff"..s)
        if HSV_lock
          HSV_lock=nil
         else
          HSV_lock1=1
          $hsv=rgb2hsv(n)
          拖动H.Progress=tointeger(hsv[1]*2)
          拖动S.Progress=tointeger(hsv[2]*255)
          拖动V.Progress=tointeger(hsv[3]*255)
          H,S,V= hsv[1],hsv[2],hsv[3]
          刷新S()
          刷新V()
        end
        HSV_card.backgroundColor=n
        HSV_value.text="H:"..(tointeger(H*10)/10).." S:"..tointeger(S*255).." V:"..tointeger(V*255)
       elseif #s>6
        HSV_lock1=1
        HSV_lock=nil
        HSV_input.text=s:sub(-6,-1)
      end
    end
  }
  拖动H.post(function()
    local w,h,tab,hsv,hk,hkdx=拖动H.width,拖动H.height,{},{0,1,1},{},拖动H.Thumb
    for i=0,360*2
      hsv[1]=i/2
      tab[i]=hsv2rgb(hsv)
      hk[i]=PorterDuffColorFilter(tab[i],PorterDuff.Mode.SRC_ATOP)
    end

    $draw=LuaDrawable(function(c,p,s)
      local w,h,pos=c.width,c.height,拖动H.Progress
      c.drawLine(0,h/2,w-(h*2)+(h/6),h/2,p)
      hkdx.setColorFilter(hk[pos])
    end)
    拖动H.setProgressDrawable(draw);

    draw.paint
    .setDither(true)
    .setAntiAlias(true)
    .setStrokeCap(Cap.ROUND)
    .setStrokeWidth(拖动H.height/8)
    .setShadowLayer(8,0,0,0xff000000)
    .setShader(LinearGradient(0,h/2,w-(h*2)+(h/6),h/2,tab,nil,Shader.TileMode.MIRROR))

    拖动H.setOnSeekBarChangeListener{
      onProgressChanged=function()
        H=拖动H.Progress/2
        if HSV_lock1
          HSV_lock1=nil
         else
          HSV_lock=1
          HSV_input.text=string.format("%0x",hsv2rgb({H,S,V})):sub(-6,-1):upper()
        end
      end,
      onStopTrackingTouch=function()
        刷新S()
        刷新V()
      end,
    }
  end)


  function 刷新S()
    拖动S.post(function()
      $hkdx=拖动S.Thumb
      local w,h,tab,hsv,hk=拖动S.width,拖动S.height,{},{H,1,1},{}

      for i=0,255
        hsv[2]=i*(1/255)
        tab[i]=hsv2rgb(hsv)
        hk[i]=PorterDuffColorFilter(tab[i],PorterDuff.Mode.SRC_ATOP)
      end

      $draw=LuaDrawable(function(c,p,s)
        local w,h,pos=c.width,c.height,拖动S.Progress
        c.drawLine(0,h/2,w-(h*2)+(h/6),h/2,p)
        hkdx.setColorFilter(hk[pos])
      end)
      拖动S.setProgressDrawable(draw);

      draw.paint
      .setDither(true)
      .setAntiAlias(true)
      .setStrokeCap(Cap.ROUND)
      .setStrokeWidth(拖动S.height/8)
      .setShadowLayer(8,0,0,0xff000000)
      .setShader(LinearGradient(0,h/2,w-(h*2)+(h/6),h/2,tab,nil,Shader.TileMode.MIRROR))

      拖动S.setOnSeekBarChangeListener{
        onProgressChanged=function()
          S=拖动S.Progress*(1/255)
          if HSV_lock1
            HSV_lock1=nil
           else
            HSV_lock=1
            HSV_input.text=string.format("%0x",hsv2rgb({H,S,V})):sub(-6,-1):upper()
          end
        end}
    end)
  end
 -- 刷新S()

  function 刷新V()
    拖动V.post(function()
      $hkdx=拖动V.Thumb
      local w,h,tab,hsv,hk=拖动V.width,拖动V.height,{},{H,1,1},{}

      for i=0,255
        hsv[3]=i*(1/255)
        tab[i]=hsv2rgb(hsv)
        hk[i]=PorterDuffColorFilter(tab[i],PorterDuff.Mode.SRC_ATOP)
      end

      $draw=LuaDrawable(function(c,p,s)
        local w,h,pos=c.width,c.height,拖动V.Progress
        c.drawLine(0,h/2,w-(h*2)+(h/6),h/2,p)
        hkdx.setColorFilter(hk[pos])
      end)
      拖动V.setProgressDrawable(draw);

      draw.paint
      .setDither(true)
      .setAntiAlias(true)
      .setStrokeCap(Cap.ROUND)
      .setStrokeWidth(拖动V.height/8)
      .setShadowLayer(8,0,0,0xff000000)
      .setShader(LinearGradient(0,h/2,w-(h*2)+(h/6),h/2,tab,nil,Shader.TileMode.MIRROR))

      拖动V.setOnSeekBarChangeListener{
        onProgressChanged=function()
          V=拖动V.Progress*(1/255)
          if HSV_lock1
            HSV_lock1=nil
           else
            HSV_lock=1
            HSV_input.text=string.format("%0x",hsv2rgb({H,S,V})):sub(-6,-1):upper()
          end
        end}
    end)
  end
--  刷新V()

  HSV_input.text=(color0 or "FFFFFF"):upper():sub(-6,-1)
end





activity.setTheme(android.R.style.Theme_Material_Light_DarkActionBar)

--HSV配色盘()--弹出配色盘，默认白色，点击复制
--HSV配色盘("#00ff00")--弹出对话框，颜色代码，默认点击复制
HSV配色盘(0xFFA774FF,function(颜色,对话框)--弹出对话框，颜色代码，设置点击事件
  print("你选择了",颜色,"颜色")
end)
```

### 计算公式

```lua
--Lua-四舍五入(常用)
function Round(num, i)
    local mult = 10^(i or 0)
    local mult10 = mult * 10
    return math.floor((num * mult10 + 5)/10)/ mult
end

--Lua-四舍五入(奇进偶舍)
function Round2(num, i)
    local tmp = math.abs(num)*(10^(i+1))
    local cal = math.abs(num)*(10^(i+1))/(10^(i+1))
    local result = 0
    if(math.floor(tmp)-math.floor(tmp/10)*10==5) then
        if(tmp-math.floor(tmp)==0) then
            local numInt1,numInt2 = math.modf(math.floor(tmp/10))
            if(numInt1 % 2 == 0) then
                result = math.floor(tmp/10)/(10^i)
            end
        end
    end
	if(result==0) then
        result = math.floor((cal * ((10^(i or 0)) * 10) + 5)/10)/ (10^(i or 0))
	end
    return ( num>0 and result ) or (0-result)
end


--相乘，判断了是否有null值
function Multiply(num1,num2)
    if(num1==nil) then
        return 0
    end
    if(num2==nil) then
        return 0
    end

    local temp=tostring(num1*num2)
    temp=tonumber(temp)
    return temp
end

--相除
function Divide(denominator,numerator)
    if(numerator==nil) then
        return 0
    end
    if(denominator==nil) then
        return 0
    end
    if(numerator==0) then
        return 0
    end
    return denominator/numerator
end

--取整
function Ceil(num)
    if(num==nil) then
        return 0
    end

    if (num <= 0) then
        return math.ceil(num)
    end

    if (math.ceil(num) == num) then
        return math.ceil(num)
    else
        return math.ceil(num) - 1
    end
end

--取整
function Ceil2(num)
    if(num==nil) then
        return 0
    end

    local t1,t2 = math.modf(num)
    return t1
end

```

### 绘制纸感标签

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
--分辨率转换函数
function 转分辨率(sdp)
  --导入所需类
  import "android.util.TypedValue"

  local dm=this.getResources().getDisplayMetrics()

  local types={px=0,dp=1,sp=2,pt=3,["in"]=4,mm=5}

  local n,ty=sdp:match("^(%-?[%.%d]+)(%a%a)$")

  return TypedValue.applyDimension(types[ty],tonumber(n),dm)

end


function drawTag()

  import "android.graphics.RectF"

  import "android.graphics.Point"

  import "android.graphics.Paint"

  import "android.graphics.Path"

  import "android.graphics.Color"

  return LuaDrawable(function(c,p,d)

    c.drawColor(0x00000000)

    p.setAntiAlias(true);

    p.setStrokeWidth(20);

    p.setStyle(Paint.Style.FILL);

    local quYu=d.bounds

    local width=quYu.right

    local height=quYu.bottom

    local radius = 转分辨率("12dp")

    p.setColor(0xffffffff)

    p.setShadowLayer(25,0,0,0x22000000)

    local path=Path();

    path.moveTo(0,radius/2);

    path.arcTo(RectF(0,0,radius,radius),180,90);

    path.lineTo(width-radius,0);

    path.arcTo(RectF(width-radius,0,width,radius),270,90);

    path.lineTo(width,height-radius/2);

    path.arcTo(RectF(width-radius,height-radius,width,height),0,90);

    path.lineTo(width*.06,height);

    path.lineTo(0,height-width*.06)

    c.drawPath(path,p);

    p.setShadowLayer(15,1,-1,0x22000000)

    local path2 = Path()

    path2.moveTo(0,height-width*.06)

    path2.lineTo(width*.06-radius/2,height-width*.06);

    path2.arcTo(RectF(width*.06-radius,height-width*.06,width*.06,height-width*.06+radius),270,90);

    path2.lineTo(width*.06,height)

    c.drawPath(path2,p)

  end)

end





function setTxt(view,view2)
  local txt={"不好意思，我把你弄丢了。",
    "长风破浪会有时，直挂云帆济沧海。",
    "你是无意穿堂风，偏偏孤倨引山洪。",
    "愿你天黑有灯，下雨有伞，未来的路有良人相伴。",
    "我有一个梦，也许有一天，灿烂的阳光能照进黑暗森林。",
    "自古美人如名将，不许人间见白头。",
    "什么都无法舍弃的人，什么都无法改变。",
    "天不生我李淳罡，剑道万古长如夜。",
    "斑竹枝，斑竹枝，点点泪痕寄相思。",
    "一切都会变好,超级好,爆好,无敌好。",
    "把喜欢的一切留在身边，这便是努力的意义。",
    "悲喜自渡，他人难悟易误。",
    "且以深情共白首，愿无岁月可回头"
    }
  import "java.io.File"
  import "android.graphics.Typeface"
 -- local bf=File(activity.getLuaDir().."/hkhbt.ttf");
 -- local tf=Typeface.createFromFile(bf)
  view.setTypeface(tf).setLineSpacing(1.6,1.6).setLetterSpacing(0.13);
  view2.setTypeface(tf).setLineSpacing(1.6,1.6).setLetterSpacing(0.13);
  Http.get("https://v1.hitokoto.cn/?encode=json","utf8",function(code,content,cookie,header)
    if code==200 then
      local cjson=import "cjson"
      local json=cjson.decode(content)
      view.setText(json.hitokoto)
      view2.setText("——"..(json.from or "未知作者"))
     else
      view.setText(txt[math.random(0,12)])
      view2.setText("——没有网络")
    end
  end)
end





layout={
  FrameLayout;
  layout_height="fill";
  layout_width="fill";
  clipChildren=false;
  background="#ffffffff";
  {
    LinearLayout;
    layout_height="30%w";
    layout_width="90%w";
    layout_gravity="center";
    backgroundDrawable=drawTag();
    orientation="vertical";
    {
      FrameLayout;
      layout_height="fill";
      layout_width="fill";
      layout_weight="1";
      {
        TextView;
        id="yiyan_txt";
        textSize="13dp";
        textColor="#ff333333";
        layout_height="fill";
        layout_width="fill";
        layout_margin="15dp";
      }
    },
    {
      TextView;
      id="yiyan_wri";
      textSize="13dp";
      textColor="#ff333333";
      layout_height="wrap";
      layout_width="fill";
      layout_marginTop="0dp";
      layout_marginRight="15dp";
      layout_marginLeft="25dp";
      layout_marginBottom="10dp";
      gravity="center|right";
    }
  }
}
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

activity.setContentView(loadlayout(layout))

setTxt(yiyan_txt,yiyan_wri)
```

### 选色器

```lua
require "import"
import "android.widget.*"
import "android.view.*"
import "android.graphics.PorterDuffColorFilter"
import "android.graphics.PorterDuff"
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
取色器=
{
  LinearLayout;
  orientation="vertical";
  layout_width="fill";
  layout_height="fill";
  gravity="center";
  {
    CardView;
    id="卡片图";
    layout_margin="10dp";
    radius="40dp",
    elevation="0dp",
    layout_width="20%w";
    layout_height="20%w";
  };
  {
    TextView;
    layout_margin="0dp";
    textSize="12sp";
    id="颜色文本";
    textColor=左侧栏项目色;
  };
  {
    SeekBar;
    id="拖动一";
    layout_margin="15dp";
    layout_width="match";
    layout_height="wrap";
  };
  {
    SeekBar;
    id="拖动二";
    layout_margin="15dp";
    layout_width="match";
    layout_height="wrap";
  };
  {
    SeekBar;
    id="拖动三";
    layout_margin="15dp";
    layout_width="match";
    layout_height="wrap";
  };
  {
    SeekBar;
    id="拖动四";
    layout_margin="15dp";
    layout_width="match";
    layout_height="wrap";
  };
};
--对话框View
local 取色器=loadlayout(取色器)
拖动一.setMax(255)
拖动二.setMax(255)
拖动三.setMax(255)
拖动四.setMax(255)
拖动一.setProgress(0xff)
拖动二.setProgress(0x1e)
拖动三.setProgress(0x8a)
拖动四.setProgress(0xe8)
--监听
拖动一.setOnSeekBarChangeListener{
  onProgressChanged=function(view, i)
    updateArgb()
  end
}

拖动二.setOnSeekBarChangeListener{
  onProgressChanged=function(view, i)
    updateArgb()
  end
}

拖动三.setOnSeekBarChangeListener{
  onProgressChanged=function(view, i)
    updateArgb()
  end
}

拖动四.setOnSeekBarChangeListener{
  onProgressChanged=function(view, i)
    updateArgb()
  end
}
--更新颜色
function updateArgb()
  local a=拖动一.getProgress()
  local r=拖动二.getProgress()
  local g=拖动三.getProgress()
  local b=拖动四.getProgress()
  local argb_hex=(a<<24|r<<16|g<<8|b)
  颜色文本.Text=string.format("%#x", argb_hex)
  卡片图.setCardBackgroundColor(argb_hex)
end
--翻译进度
argbBuild=AlertDialog.Builder(activity)
argbBuild.setView(取色器)
argbBuild.setTitle("选色器")
argbBuild.setPositiveButton("复制", {
  onClick=function(view)
    local a=拖动一.getProgress()
    local r=拖动二.getProgress()
    local g=拖动三.getProgress()
    local b=拖动四.getProgress()
    local argb_hex=(a<<24|r<<16|g<<8|b)
    local argb_str=string.format("%#x", argb_hex)
    activity.getSystemService(Context.CLIPBOARD_SERVICE).setText(argb_str)
    print("已复制到剪贴板")
  end
})
argbBuild.setNeutralButton("取消",{onClick=function()

  end})--设置否认按钮
--实例化对话框
argbDialog=argbBuild.create()
argbDialog.setCanceledOnTouchOutside(false)
function showArgbDialog()
  --展示对话框
  argbDialog.show()
  --更新颜色
  updateArgb()
end
showArgbDialog()
```

### 电视直播列表+适配器

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
layout={
  LinearLayout;
  orientation="horizontal";
  layout_height="fill";
  layout_width="fill";
  {
    ListView;
    id="home_list";
    layout_height="fill";
    layout_width="34%w";
  },
  {
    LinearLayout;
    layout_height="fill";
    layout_width="1";
    background="#ff252525";
  },
  {
    FrameLayout;
    layout_height="fill";
    layout_width="fill";
    {
      ListView;
      id="cctv_list";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="wstv_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="mhjy_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="shpd_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="zztj_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="yksp_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="mgsp_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
    {
      ListView;
      id="hsys_list";
      Visibility="gone";
      layout_height="fill";
      layout_width="fill";
    },
  }
}
activity.setContentView(loadlayout((layout)))

item={
  LinearLayout;
  layout_height="50dp";
  layout_width="fill";
  {
    TextView;
    id="zhibo_txt";
    layout_height="fill";
    layout_width="fill";
    gravity="center";
  },
  {
    TextView;
    id="zhibo_url";
    layout_height="0";
    layout_width="0";
  }
}
adapter=LuaAdapter(activity,item)
home_list.Adapter=adapter
adapter2=LuaAdapter(activity,item)
cctv_list.Adapter=adapter2
adapter3=LuaAdapter(activity,item)
wstv_list.Adapter=adapter3
adapter4=LuaAdapter(activity,item)
mhjy_list.Adapter=adapter4
adapter5=LuaAdapter(activity,item)
shpd_list.Adapter=adapter5
adapter6=LuaAdapter(activity,item)
zztj_list.Adapter=adapter6
adapter7=LuaAdapter(activity,item)
yksp_list.Adapter=adapter7
adapter8=LuaAdapter(activity,item)
mgsp_list.Adapter=adapter8
adapter9=LuaAdapter(activity,item)
hsys_list.Adapter=adapter9





function 获取直播()
  Http.get("http://jrys.wy2sf.com/yingshi/getAllTV","utf8",function(code,content,cookie,header)
    if code==200 then
      local cjson=import "cjson"
      local json=cjson.decode(content)
      print(json.msg)
      if json.msg=="成功！" then
        for k,v in ipairs(json.data) do
          adapter.add{zhibo_txt=json.data[k].tvType.t_name}
        end
        for k,v in ipairs(json.data[1].tvBeans) do
          adapter2.add{zhibo_txt=json.data[1].tvBeans[k].d_name,zhibo_url=json.data[1].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[2].tvBeans) do
          adapter3.add{zhibo_txt=json.data[2].tvBeans[k].d_name,zhibo_url=json.data[2].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[3].tvBeans) do
          adapter4.add{zhibo_txt=json.data[3].tvBeans[k].d_name,zhibo_url=json.data[3].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[4].tvBeans) do
          adapter5.add{zhibo_txt=json.data[4].tvBeans[k].d_name,zhibo_url=json.data[4].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[5].tvBeans) do
          adapter6.add{zhibo_txt=json.data[5].tvBeans[k].d_name,zhibo_url=json.data[5].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[6].tvBeans) do
          adapter7.add{zhibo_txt=json.data[6].tvBeans[k].d_name,zhibo_url=json.data[6].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[7].tvBeans) do
          adapter8.add{zhibo_txt=json.data[7].tvBeans[k].d_name,zhibo_url=json.data[7].tvBeans[k].d_url}
        end
        for k,v in ipairs(json.data[8].tvBeans) do
          adapter9.add{zhibo_txt=json.data[8].tvBeans[k].d_name,zhibo_url=json.data[8].tvBeans[k].d_url}
        end
       else
        print("获取数据失败")
      end
     else
      print("链接服务器失败")
    end
  end)
end


获取直播()





home_list.onItemClick=function(l,v,p,q)
  if q==1 then
    cctv_list.setVisibility(View.VISIBLE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.GONE)
   elseif q==2 then
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.VISIBLE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.GONE)
   elseif q==3 then
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.VISIBLE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.GONE)
   elseif q==4 then
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.VISIBLE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.GONE)
   elseif q==5 then
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.VISIBLE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.GONE)
   elseif q==6 then
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.VISIBLE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.GONE)
   elseif q==7 then
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.VISIBLE)
    hsys_list.setVisibility(View.GONE)
   else
    cctv_list.setVisibility(View.GONE)
    wstv_list.setVisibility(View.GONE)
    mhjy_list.setVisibility(View.GONE)
    shpd_list.setVisibility(View.GONE)
    zztj_list.setVisibility(View.GONE)
    yksp_list.setVisibility(View.GONE)
    mgsp_list.setVisibility(View.GONE)
    hsys_list.setVisibility(View.VISIBLE)
  end
end



cctv_list.onItemClick=function(l,v,p,q)
  print(v.tag.zhibo_url.text)
end
```

### TextView使用HTML标签

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.text.method.*"
import "android.text.Html"
import "android.widget.*"
import "android.view.*"
layout={--主布局
  LinearLayout;
  gravity="center";
  orientation="vertical";
  {
    TextView;
    layout_width="wrap_content";
    textSize="18sp";
    text="hello AndroLua+";
    id="t1";
  };
};

activity.setTitle('AndroLua+')
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题

activity.setContentView(loadlayout(layout))


nr1=[[<a href='http://www.baidu.com' target="_blank" >百度一下</a> ]]
t1.setText(Html.fromHtml(nr1))
t1.setMovementMethod(LinkMovementMethod.getInstance())
```

### 手绘蝴蝶线动画

```lua
require "import"
import "android.os.*"
import "android.app.*"
import "android.view.*"
import "android.widget.*"
import "android.graphics.*"
import "android.animation.*"
--[[
怎么学都学不会的何同学制作
]]
local layout =
{
  LinearLayout, --线性布局
  orientation = 'vertical', --方向
  layout_width = 'fill', --宽度
  layout_height = 'fill', --高度
  {
    SurfaceView;
    layout_width = 'fill', --宽度
    layout_height = 'fill', --高度
    id = "surface",
  };
};
activity.setContentView(loadlayout(layout))
--定义值动画
local animation = ValueAnimator.ofFloat({ 0, 8*math.pi })
animation.setDuration(10000)
animation.setRepeatCount(-1)
animation.setRepeatMode(2)
animation.start()

--suface是你的SurfaceView的id
local holder = surface.getHolder()
holder.addCallback(SurfaceHolder.Callback {
  --视图改变
  surfaceChanged = function(holder, format, width, height)
  end,
  --视图创建，一般绘制就在这
  surfaceCreated = function(holder)
    animation.addUpdateListener(ValueAnimator.AnimatorUpdateListener {
      onAnimationUpdate = function(animate)
        local k = animate.getAnimatedValue()

        --拿到画布canvas
        local canvas = holder.lockCanvas()
        if canvas ~= nil then
          --这里绘制
          canvas.drawColor(0xffffffff) --绘制背景为白色
          --定义画笔Paint
          local paint = Paint()
          paint.setColor(0xff6493c1) --画笔颜色
          paint.setStyle(Paint.Style.STROKE) --画笔样式：描边
          paint.setStrokeWidth(10) --画笔宽度
          paint.setStrokeCap(Paint.Cap.ROUND) --画笔笔头：圆滑
          --定义绘制变量
          local cx = canvas.getWidth() / 2 --中心x点
          local cy = canvas.getHeight() / 2 --中心y点
          local zoom = 150
          --这里进行详细绘制操作
          local path = Path()
          path.moveTo(cx,cy)
          for angle = 0, k, 0.01 do
            local r = (math.exp(math.cos(angle))-2*math.cos(4*angle)+(math.sin(angle/15))^2)*zoom
            local x = r*math.cos(angle)+cx
            local y = r*math.sin(angle)+cy
            path.lineTo(x,y)
          end
          canvas.drawPath(path,paint)
        end
        --提交画布canvas
        holder.unlockCanvasAndPost(canvas)
      end
    })
  end,
  --视图销毁，主要是动态绘制时的销毁线程
  surfaceDestroyed = function(holder)
    animation.removeAllUpdateListeners()
    animation.cancel()
  end
})
```

### 绘制三角函数图像

```lua
import "android.graphics.RectF"
import "android.graphics.Path"
import "android.graphics.Paint"
import "android.graphics.Color"


layout={
  LinearLayout;
  layout_height="fill";
  layout_width="fill";
  orientation="vertical";
  {
    SurfaceView;
    layout_width='fill',
    layout_height='fill',
    id="surface",
  };
};

activity.setContentView(loadlayout(layout))
--作者:帕帝天秀
--QQ:3373587110
surface.setBackground(
LuaDrawable(function(canvas,paint)
  canvas.drawColor(0xffffffff)
  mPaintLine = Paint();
  mPaintLine.setAntiAlias(true)
  mPaintLine.setStrokeWidth(5);
  mPaintLine.setStyle(Paint.Style.STROKE);
  mPaintLine.setColor(Color.BLACK);
  mPaintLine.setFlags(Paint.ANTI_ALIAS_FLAG);

  mCirclePaint = Paint();
  mCirclePaint.setColor(Color.RED);
  mCirclePaint.setFlags(Paint.ANTI_ALIAS_FLAG);

  drawXLine(canvas)
  drawYLine(canvas)
  drawXArrow(canvas)
  drawYArrow(canvas)
  drawCenterPoint(canvas);
  drawPathRight(canvas)
  drawPathLeft(canvas)

end)
)
width=activity.width
height=activity.height

function drawCenterPoint(canvas)
  canvas.drawCircle(width/2,height/2,10,mCirclePaint);
end

function ponitX(i)
  return -math.sin(math.pi/180*i)
end

function drawPathLeft(canvas)
  local path=Path();
  path.moveTo(width/2,height/2);
  for i=0,480 do
    x = width/6/180*-i+width/2;
    y = (ponitX(-i)*height/8)+height/2;
    path.lineTo(x,y);
  end
  canvas.drawPath(path,mPaintLine);
end

function drawPathRight(canvas)
  local path=Path();
  path.moveTo(width/2,height/2);
  i1=0
  for i1=0,480 do
    x = width/6/180*i1+width/2;
    y = (ponitX(i1)*height/8)+height/2;
    path.lineTo(x,y);
  end
  canvas.drawPath(path,mPaintLine);
end


function drawXArrow(canvas)
  local path = Path();
  path.moveTo(width-20,height/2-20);
  path.lineTo(width,height/2);
  path.lineTo(width-20,height/2+20);
  canvas.drawPath(path,mPaintLine);
end

function drawYArrow(canvas)
  local path = Path();
  path.moveTo(width/2-20,20);
  path.lineTo(width/2,0);
  path.lineTo(width/2+20,20);
  canvas.drawPath(path,mPaintLine);
end

function drawXLine(canvas)
  local startX = 0;
  local startY = height/2;
  local stopX = width;
  local stopY = startY;
  canvas.drawLine(startX,startY,stopX,stopY,mPaintLine);
end

function drawYLine(canvas)
  local startX = width/2;
  local startY = 0;
  local stopX = startX;
  local stopY = height;
  canvas.drawLine(startX,startY,stopX,stopY,mPaintLine);
end
```

### 二次函数绘制

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

local layout={
  LinearLayout;
  layout_height="fill";
  layout_width="fill";
  orientation="vertical";
  BackgroundColor=0xffffffff;
  {
    LinearLayout;
    layout_width="fill";
    orientation="horizontal";
    gravity="center";
    {
      EditText;
      hint="a=";
      id="aTiet";
      layout_weight=1;
      layout_width="fill";
    };
    {
      EditText;
      hint="b=";
      layout_margin="8dp",
      id="bTiet";
      layout_width="fill";
      layout_weight=1;
    };
    {
      EditText;
      hint="c=";
      id="cTiet";
      layout_marginTop="8dp",
      layout_marginLeft="8dp",
      layout_marginBottom="8dp",
      layout_width="fill";
      layout_weight=1;
    };
  };
  {
    LinearLayout;
    layout_width="fill";
    orientation="horizontal";
    {
      Button;
      text="绘制函数";
      layout_weight=1;
      layout_marginLeft="8dp";
      layout_marginRight="8dp";
      id="Drawing";
    };
    {
      Button;
      text="还原";
      layout_weight=1;
      layout_marginLeft="8dp";
      layout_marginRight="8dp";
      id="Reduction";
    };
    {
      Button;
      text="画笔颜色";
      layout_weight=1;
      layout_marginLeft="8dp";
      layout_marginRight="8dp";
      id="selectColor";
    };
  };
  {
    TextView;
    text="画笔大小:50";
    id="paintSizes";
    layout_marginLeft="8dp";
    layout_marginRight="8dp";
  };
  {
    SeekBar;
    Min=20;
    Max=100;
    layout_width="fill";
    id="paintSizeSeekBar";

    --layout_margin="15dp";
  };
  {
    ImageView;
    id="surface";
    layout_height="fill";
    layout_width="fill";
  };
};


activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置md主题
activity.setTitle("二次函数绘制")
activity.setContentView(loadlayout(layout))

paintSizeSeekBar.setProgress(50)


paintSizeSeekBar.setOnSeekBarChangeListener{
  onStartTrackingTouch=function()

  end,
  onStopTrackingTouch=function()

  end,
  onProgressChanged=function(view,value)
    paintSizes.setText("画笔大小:"..tostring(value))
  end
}

function drawAPlaneRightAngleCoordinateSystem()
  import "android.graphics.Bitmap"
  import "android.graphics.Canvas"
  baseBitmap=Bitmap.createBitmap(1000, 1000, Bitmap.Config.ARGB_8888);
  width=baseBitmap.getWidth();
  height=baseBitmap.getHeight();
  canvas=Canvas(baseBitmap)
  import "android.graphics.Paint"
  spaint=Paint();
  spaint.setStrokeWidth(1);
  spaint.setColor(-16777216);
  spaint.setAntiAlias(true);

  paint=Paint();
  paint.setStrokeWidth(1);
  paint.setColor(-65536);
  paint.setAntiAlias(true);

  Textpaint=Paint();
  Textpaint.setTextSize(40);
  Textpaint.setColor(-16776961);
  canvas.drawColor(-1);
  canvas.drawLine(0,(width/2),width,(width/2),paint);
  canvas.drawLine((width/2),0,(width/2),width,paint);
  canvas.drawLine((width/2),0,((width/2)-(width/50)),(width/(width/20)),paint);
  canvas.drawLine((width/2),0,((width/2)+(width/50)),(width/(width/20)),paint);
  canvas.drawText("y",((width/2)+(width/50)),(width/20),Textpaint);
  canvas.drawLine(width,(width/2),(width-(width/(width/20))),((width/2)-(width/50)),paint);
  canvas.drawLine(width,(width/2),(width-(width/(width/20))),((width/2)+(width/50)),paint);
  canvas.drawText("x",(width-(width/40)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine((((width/5)*4)+(width/10)),(width/2),(((width/5)*4)+(width/10)),((width/2)-(width/50)),spaint);
  canvas.drawText("4",(((width/5)*4)+(width/10)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine(((width/5)*4),(width/2),((width/5)*4),((width/2)-(width/50)),spaint);
  canvas.drawText("3",((width/5)*4),((width/2)+(width/20)),Textpaint);
  canvas.drawLine((((width/5)*4)-(width/10)),(width/2),(((width/5)*4)-(width/10)),((width/2)-(width/50)),spaint);
  canvas.drawText("2",(((width/5)*4)-(width/10)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine((((width/5)*4)-(width/5)),(width/2),(((width/5)*4)-(width/5)),((width/2)-(width/50)),spaint);
  canvas.drawText("1",(((width/5)*4)-(width/5)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine((((width/2)/5)*4),(width/2),(((width/2)/5)*4),((width/2)-(width/50)),spaint);
  canvas.drawText("-1",(((width/2)/5)*4),((width/2)+(width/20)),Textpaint);
  canvas.drawLine(((((width/2)/5)*4)-(width/10)),(width/2),((((width/2)/5)*4)-(width/10)),((width/2)-(width/50)),spaint);
  canvas.drawText("-2",((((width/2)/5)*4)-(width/10)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine(((((width/2)/5)*4)-(width/5)),(width/2),((((width/2)/5)*4)-(width/5)),((width/2)-(width/50)),spaint);
  canvas.drawText("-3",((((width/2)/5)*4)-(width/5)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine(((((width/2)/5)*4)-((width/10)*3)),(width/2),((((width/2)/5)*4)-((width/10)*3)),((width/2)-(width/50)),spaint);
  canvas.drawText("-4",((((width/2)/5)*4)-((width/10)*3)),((width/2)+(width/20)),Textpaint);
  canvas.drawText("0",((width/2)+(width/50)),((width/2)+(width/20)),Textpaint);
  canvas.drawLine((width/2),(((width/2)/5)*4),((width/2)+(width/50)),(((width/2)/5)*4),spaint);
  canvas.drawText("1",((width/2)-(width/20)),(((width/2)/5)*4),Textpaint);
  canvas.drawLine((width/2),((((width/2)/5)*4)-(width/10)),((width/2)+(width/50)),((((width/2)/5)*4)-(width/10)),spaint);
  canvas.drawText("2",((width/2)-(width/20)),((((width/2)/5)*4)-(width/10)),Textpaint);
  canvas.drawLine((width/2),((((width/2)/5)*4)-(width/5)),((width/2)+(width/50)),((((width/2)/5)*4)-(width/5)),spaint);
  canvas.drawText("3",((width/2)-(width/20)),((((width/2)/5)*4)-(width/5)),Textpaint);
  canvas.drawLine((width/2),((((width/2)/5)*4)-((width/10)*3)),((width/2)+(width/50)),((((width/2)/5)*4)-((width/10)*3)),spaint);
  canvas.drawText("4",((width/2)-(width/20)),((((width/2)/5)*4)-((width/10)*3)),Textpaint);
  canvas.drawLine((width/2),((width/5)*4),((width/2)+(width/50)),((width/5)*4),spaint);
  canvas.drawText("-1",((width/2)-(width/20)),(((width/5)*4)-(width/5)),Textpaint);
  canvas.drawLine((width/2),(((width/5)*4)-(width/10)),((width/2)+(width/50)),(((width/5)*4)-(width/10)),spaint);
  canvas.drawText("-2",((width/2)-(width/20)),(((width/5)*4)-(width/10)),Textpaint);
  canvas.drawLine((width/2),(((width/5)*4)-(width/5)),((width/2)+(width/50)),(((width/5)*4)-(width/5)),spaint);
  canvas.drawText("-3",((width/2)-(width/20)),((width/5)*4),Textpaint);
  canvas.drawLine((width/2),((((width/2)/5)*4)+(width/2)),((width/2)+(width/50)),((((width/2)/5)*4)+(width/2)),spaint);
  canvas.drawText("-4",((width/2)-(width/20)),((((width/2)/5)*4)+(width/2)),Textpaint);
  surface.setImageBitmap(baseBitmap)
end

drawAPlaneRightAngleCoordinateSystem()

paintColor=0xff000000

function Drawing.onClick()
  import "android.graphics.Paint"
  import "android.graphics.Color"
  mPaintLine = Paint();
  mPaintLine.setAntiAlias(true)
  mPaintLine.setStrokeWidth(paintSizeSeekBar.getProgress()/10);
  mPaintLine.setStyle(Paint.Style.STROKE);
  mPaintLine.setColor(paintColor);
  mPaintLine.setFlags(Paint.ANTI_ALIAS_FLAG);
  import "android.graphics.Path"
  local path=Path();
  if aTiet.Text=="" then
    a=0
   else
    a=aTiet.Text
  end
  if bTiet.Text=="" then
    b=0
   else
    b=bTiet.Text
  end
  if cTiet.Text=="" then
    c=0
   else
    c=cTiet.Text
  end
  for x=-1000,1000 do
    y=tointeger((-a/100*(x^2))+(-b*x)+-c*100);
    path.lineTo(width/2+x,height/2+y);
  end
  canvas.drawPath(path,mPaintLine);
  surface.setImageBitmap(baseBitmap)
end


function Reduction.onClick()
  drawAPlaneRightAngleCoordinateSystem()
end

function selectColor.onClick()
  local colorTitle={"黑色","蓝色","青色","灰色","绿色","红色","黄色"}
  local colorValue={
    0xFF000000,
    0xff0c0beb,
    0xff02e4fc,
    0xff7e9398,
    0xff26e330,
    0xffc92a18,
    0xffeee226,
  }

  AlertDialog.Builder(this)
  .setTitle("画笔颜色")
  .setItems(colorTitle,{onClick=function(l,v)
      paintColor=colorValue[v+1]
    end
  })
  .show()
end

function onCreateOptionsMenu(menu)
  menu.add("联系作者").onMenuItemClick=function(a)
    import "android.net.Uri"
    import "android.content.Intent"
    url="mqqapi://card/show_pslcard?src_type=internal&source=sharecard&version=1&uin=3373587110"
    activity.startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))
  end
  menu.add("羚").onMenuItemClick=function(a)

  end
end
```

### QQ变音

```lua
--未适配Android11
function QQ变音(qq,FromPath)
  import "java.io.File"
  local SDC=Environment.getExternalStorageDirectory().toString()
  local time=os.date("%Y%m")
  local time2=tonumber(os.date("%d"))
  local RootDirectory=SDC.."/Android/data/com.tencent.mobileqq/Tencent/MobileQQ/"..qq.."/ptt/"..time.."/"..time2.."/"
  local FileTable=luajava.astable(File(RootDirectory).listFiles())
  TimeTable={}
  for i=1,#FileTable do
    local path=FileTable[i]
    local FileName=File(path.toString()).getName()
    if FileName~=nil then
      local FileNameTime=tonumber(FileName:match("_(.-).slk"))
      table.insert(TimeTable,FileNameTime)
    end
  end
  table.sort(TimeTable)
  local FileNameTime=TimeTable[#TimeTable]
  if FileNameTime~=nil then
    local FilePath=RootDirectory.."stream_"..FileNameTime..".slk"
    LuaUtil.copyDir(FromPath,FilePath)
  end
end

QQ变音("3373587110","/storage/emulated/0/music/DJ Ubur Ubur x Paket Phoenix IndiHome_Seven Nation.mp3")
```

### 遍历设置字体

```lua
--遍历设置字体
import "android.graphics.Typeface"
font=Typeface.create("宋体",Typeface.BOLD)
function setFont(view)
if luajava.instanceof(view,TextView) then
  view.setTypeface(font)
  elseif luajava.instanceof(view,ViewGroup) then
  for i=0,view.getChildCount()-1 do
    setFont(view.getChildAt(i))
    end
  end
--来源：柯南
end
setFont(activity.getDecorView())
```

### 仿QQ顶部提示动画

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
--import "layout"

layout={
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="fill",
  {
    LinearLayout,
    layout_width="fill",
    layout_height="56dp";
    background="#ffffff";
    elevation="2dp";
    Visibility="invisible";
    id="linear";
    {
      TextView;
      text="仿QQ提示";
      textColor="#009688";
      layout_gravity="center";
      textSize="14dp";
      layout_marginLeft="16dp";
      };
  },
{
  Button;
  text="显示";
  id="button";
  layout_gravity="center";
  layout_marginTop="100dp";
  };
};


activity.setContentView(loadlayout(layout))

button. onClick=function()
  linear.setVisibility(View.VISIBLE)
import "android.view.animation.*"
linear.startAnimation(TranslateAnimation(0,0,-linear.height,0).setDuration(200))
--来源：Androlua官方二群
--作者：Paixs
task(1000,function()
  linear.setVisibility(View.INVISIBLE)
--import "android.view.animation.*"
linear.startAnimation(TranslateAnimation(0,0,0,-linear.height).setDuration(200))
  end)

end
```

### Page加列表显示实例

```lua
require "import"
import "android.widget.LinearLayout"
import "android.widget.TextView"
import "android.widget.PageView"
import "android.widget.Button"
import "android.webkit.WebSettings$TextSize"
import "android.graphics.ColorFilter"
import "android.widget.ImageView"
import "android.widget.FrameLayout"
import "android.widget.ListView"
import "com.androlua.LuaAdapter"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
--import "layout"
--import "nihao"
--import "liebiao"
--activity.setTitle('AndroLua+')
--activity.setTheme(android.R.style.Theme_Holo_Light)



layout={--第一个界面
  LinearLayout;
  orientation="vertical";
{
  LinearLayout;
  layout_width="fill";
  orientation="vertical";
  layout_height="fill";
  {
    LinearLayout;
    layout_width="fill";
    orientation="horizontal";
    layout_height="3dp";
    layout_marginTop="1dp";
    {
      TextView;
      layout_width="50%w";
      background="#000000";
      layout_height="fill";
      id="hg1";
    };
  };
  {
    PageView;
    layout_width="fill";
    id="page";
    --orientation="vertical";
    layout_height="fill";
    pages={
      {
        LinearLayout;
        layout_width="fill";
        orientation="vertical";
        layout_height="fill";
        --重点来了
         {
    ListView;
    id="id";
    layout_width="fill";
  };
  {
    ListView;
    id="id2";
    layout_width="fill";
  };
  {
    ListView;
    id="id3";
    layout_width="fill";
  };
       -- 重点结束


      };
      {
        LinearLayout;
        layout_width="fill";
        orientation="vertical";
        layout_height="fill";

        --右边的按钮
        {
          Button;
          layout_width="fill";
        };
      --结束
      };
    };
  };
}
};


nihao={--第二个界面
  FrameLayout;
  id="fu";
  layout_width="fill";
   {
    LinearLayout;
    orientation="vertical";
    layout_margin="6dp";
    layout_width="fill";
    {
      TextView;
      layout_margin="7";
      layout_width="fill";
      TextSize="5sp";
      id="标题";
      textColor="#FF2A2A2A";
      text="hello AndroLua+";
    };
    {
      TextView;
      textColor="#FFA1A1A1";
      layout_width="fill";
      id="提示";
    };
  };
  {
    LinearLayout;
    orientation="vertical";
    layout_width="fill";
    layout_gravity="center";
    {
      ImageView;
      layout_marginRight="4dp";
      layout_height="15dp";
      layout_width="15dp";
      id="icon";
      ColorFilter="#FF000000";
      layout_gravity="right";
    };
  };
};




activity.setContentView(loadlayout(layout))
--下面这一窜是控制滑动的东西
--我也不知道是控制是什么的
page.addOnPageChangeListener{
onPageScrolled=function(p,pO,pp)
if pO~=0 then
hg1.setX(activity.getWidth()/2*pO)
end
end,
}
--结束

----来源：Androlua官方二群
----作者：ywcz


--配置列表参数__item布局表文件____开始
图片="Right.png"    --1楼开始
adp=LuaAdapter(activity,nihao)
adp.add{标题="LCD密度",提示="界面清爽不臃肿",icon=图片}
id.setAdapter(adp)--1楼结尾

adp=LuaAdapter(activity,nihao)    --二楼开始
adp.add{标题="Host优化",提示="优化Host提高访问网络效率"}
adp.add{标题="WiFi密码查看",提示="查看您连接的WiFi的密码",icon=图片}
adp.add{标题="本机信息修改",提示="厂家、型号、内核等",icon=图片}
adp.add{标题="虚拟内存设置",提示="提高程序运行速度",icon=图片}
adp.add{标题="设置多点触控",提示="十指切水果",icon=图片}
adp.add{标题="触摸与滑动设置",提示="触摸灵敏度等",icon=图片}
adp.add{标题="Build.prop编辑器",提示="编辑Build.prop配置文件",icon=图片}
id2.setAdapter(adp)--2楼结尾

--[[adp=LuaAdapter(activity,liebiao)   --3楼开始
adp.add{标题="你好",提示="晚上来搞笑的",icon=图片}
id3.setAdapter(adp)]]
--3楼结束
--配置列表参数__item布局表文件____结束
id.onItemClick=function(l,v,p,i)
  print("点击了"..v.Tag.标题.Text)
end

id2.onItemClick=function(l,v,p,i)
  print("点击了"..v.Tag.标题.Text)
end
```

### 自绘制开关

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.graphics.drawable.GradientDrawable"

--activity.setTitle('AndroLua+')
--activity.setTheme(android.R.style.Theme_Holo_Light)

activity.setContentView(loadlayout({
  LinearLayout;
  gravity="center";
  layout_height="fill";
  layout_width="fill";
  orientation="vertical";
  --BackgroundColor="#ffffff00",
  id="fhhk";
  {
    Switch;
    id="Grt";
  };
  {
    Switch;
    id="Grt1";
  };
  {
    Switch;
    id="Grt2";
  };
  {
    Switch;
    id="Grt3";
  };
  {
    Switch;
  };
}))

--转换像素单位
dip={toPx=function(context, dpValue)
    scale = context.getResources().getDisplayMetrics().density;
    return dpValue * scale + 0.5
  end}

function CircleBack_SeekDra(InsideColor,Ad_Size,ble,Colorse3)
  local colors = InsideColor
  local Sizes=dip.toPx(this,Ad_Size)--设置开关大小
  local Stroke=dip.toPx(this,7)--设置拖块边距
  local Track_Stroke=dip.toPx(this,0)--设置背景边距
  local drawable = GradientDrawable(GradientDrawable.Orientation.TOP_BOTTOM,{});
  drawable.setCornerRadius(Sizes/2);
  drawable.setColor(InsideColor)
  if ble
    drawable.setStroke(Stroke, 0x00ffffff)
    drawable.setSize(Sizes,Sizes)
   else
    drawable.setAlpha(60)
    drawable.setStroke(Track_Stroke, 0x00ffffff)
  end
  drawable.setGradientType(GradientDrawable.RECTANGLE);
  return drawable
end

Switch_x=function(view,Colors,Colors2,Colors3,Ad_Size)
  pcall(function()
    if view.isChecked()
      Colorse=Colors
      Colorse2=Colors
     else
      Colorse=Colors2
      Colorse2=Colors3
    end
    local padd_W=dip.toPx(this,Ad_Size/2.5)
    view.setThumbDrawable(CircleBack_SeekDra(Colorse,Ad_Size,true))
    .setTrackDrawable(CircleBack_SeekDra(Colorse2,Ad_Size,false))
    .setPadding(padd_W,padd_W,padd_W,padd_W)
  end)
end

Grt.setOnCheckedChangeListener({
  onCheckedChanged=function(buttonView, isChecked)
    Switch_x(Grt,Ad_Color,Ad_Color2,Ad_Color3,Ad_Size,Ad_Padding)--更新状态
  end})
Grt1.setOnCheckedChangeListener({
  onCheckedChanged=function(buttonView, isChecked)
    Switch_x(Grt1,0xffff6e17,0xFFECECEC,0xff000000,35,7)--预加载
  end})
Grt2.setOnCheckedChangeListener({
  onCheckedChanged=function(buttonView, isChecked)
    Switch_x(Grt2,0xff2ecbff,0xFFECECEC,0xff000000,45,7)--预加载
  end})
Grt3.setOnCheckedChangeListener({
  onCheckedChanged=function(buttonView, isChecked)
    Switch_x(Grt3,0xffae5dff,0xFFECECEC,0xff000000,55,7)--预加载
  end})

Ad_Color=0xffff0000--边框颜色与拖块选中颜色
Ad_Color3=0xff000000--设置底部背景未选中颜色
Ad_Color2=0xFFECECEC
Ad_Size=25--按钮的大小dip
Ad_Padding=7--设置拖块边距范围dip
Switch_x(Grt,Ad_Color,Ad_Color2,Ad_Color3,Ad_Size,Ad_Padding)--预加载
Switch_x(Grt1,0xffff6e17,0xFFECECEC,0xff000000,35,7)--预加载
Switch_x(Grt2,0xff2ecbff,0xFFECECEC,0xff000000,45,7)--预加载
Switch_x(Grt3,0xffae5dff,0xFFECECEC,0xff000000,55,7)--预加载
```

### 绘制有行数的编辑框

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"


activity.setTitle('AndroLua+')
--by小菜鸟
--需要配合ScrollView使用
--2017/09/05/13:00
--activity.setTheme(android.R.style.Theme_Holo_Light)
activity.setContentView(loadlayout({
  LinearLayout;
  layout_height="fill";
  orientation="vertical";
  layout_width="fill";
  {
    ScrollView;
    layout_height="fill";
    DescendantFocusability=131072;
    id="s";
    layout_width="fill";
    {
      EditText;
      id="v";
      text=" ";
      layout_height="fill";
      layout_width="fill";
    };
  };
}))
function getTextWidth(paint,str)--计算文字宽度
  local iRet = 0;
  if str ~= nil and #str > 0 then
    local len =#str;
    local widths =float[len];
    paint.getTextWidths(str, widths);
    for j = 0,len-1 do
      iRet=iRet+Math.ceil(widths[j]);
    end
  end
  return iRet;
end
import "android.graphics.Color"
import "android.graphics.Paint"
v.setFocusable(true)
v.background=LuaDrawable(function(canvas,paint,d)
  paint.setColor(Color.GRAY);--颜色
  paint.setAntiAlias(true);
  paint.setStrokeWidth(2);--宽度
  if(#v.Text~=0)then
    paint.setTextSize(v.getTextSize()-5);--行号文字大小
    paint.setStyle(Paint.Style.FILL);
    for l=0,v.getLineCount() do
      Y=((l+1)*v.getLineHeight())-(v.getLineHeight()/10);
      canvas.drawText(tostring(l+1),0,Y,paint);--行号
      canvas.save();
      v.setPadding(getTextWidth(paint,tostring(l+1))+15,0,0,0)--设置左边距
    end
   else
    Y=((1)*v.getLineHeight())-(v.getLineHeight()/10);
    canvas.drawText("1",0,Y,paint);
    canvas.save();
    v.setPadding(getTextWidth(paint,"1")+15,0,0,0)--设置左边距
  end
  y=v.getLineCount()*v.getLineHeight();
  canvas.drawLine(v.getCompoundPaddingLeft()-5,0,v.getCompoundPaddingLeft()-5,v.getHeight()+y,paint);--左边的线条
  canvas.save();
  canvas.restore();
end)
s.onTouch=function(v,event)
  v.requestFocusFromTouch();
  return false;
end
```

### 自绘制气泡拖动条

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

--activity.setTitle('AndroLua+')
--activity.setTheme(android.R.style.Theme_Holo_Light)
activity.setContentView(loadlayout({
  ScrollView;
  layout_height="fill";
  layout_width="fill";
  {
    LinearLayout;
    layout_width="fill";
    gravity="top";
    layout_height="fill";
    orientation="vertical";
    {
      TextView;
      layout_width="fill";
      padding="15dp";
      id="t1";
    };
    {
      View;
      layout_width="fill";
      layout_height="35dp";
      id="animseekBar";
    };
    {
      TextView;
      layout_width="fill";
      padding="15dp";
      id="t2";
    };
  };
}
))

import "android.graphics.*"
import "android.graphics.Paint$Style"
import "android.view.animation.*"
import "android.animation.ObjectAnimator"
import "android.animation.PropertyValuesHolder"

dip2px=function(dpValue)
  local scale = activity.getResources().getDisplayMetrics().density;
  return dpValue * scale + 0.5
end

sp2px=function(spValues)
  local fontScale = activity.getResources().getDisplayMetrics().scaledDensity;
  return spValues * fontScale+ 0.5
end

getStatusBarHeight=function()
  _,statusBarHeight2=xpcall(function()
    local clazz=Class.forName("com.android.internal.R$dimen")
    local object = clazz.newInstance();
    local height = Integer.parseInt(tostring(clazz.getField("status_bar_height").get(object)))
    return this.getResources().getDimensionPixelSize(height);
  end,

  function(a)
    _,statusBarHeight1=pcall(function()
      local resourceId = this.getResources().getIdentifier("status_bar_height", "dimen", "android");
      if resourceId > 0
        return this.getResources().getDimensionPixelSize(resourceId);
      end
    end,
    function(e)
    end,nil)

  end,
  nil)
  if type(statusBarHeight2):lower()=="number"
    return statusBarHeight2
   elseif type(statusBarHeight1):lower()=="number"
    return statusBarHeight1
   else
    return 0
  end
end

BgColor=0xFFBDBDBD--背景色
TopColor=0xFF9C27B0--进度条颜色
TopHandleColor=0xFF9C27B0--滑块颜色
BubblesColor=0xffffffff--气泡字体颜色
BgHeight=dip2px(3)--背景高度
TopHeight=dip2px(4)--进度条高度
TopHandle=dip2px(8)--滑块半径
Max=100--最大值
Dragprogress=0--当前进度
CanvasMargin=dip2px(48)--边距
bubblesTextSize=sp2px(12)--气泡内文字大小
Seekbarids=animseekBar

import "android.content.Context"
mWindowManager = activity.getSystemService(Context.WINDOW_SERVICE);

IndicatorWindow=function(getH,getW)
  local mLayoutParams = WindowManager.LayoutParams();
  mLayoutParams.gravity = Gravity.START | Gravity.BOTTOM;
  mLayoutParams.width =getW;
  mLayoutParams.height = getH;
  mLayoutParams.format = PixelFormat.TRANSLUCENT;
  mLayoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
  | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
  | WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED;
  --MIUI禁止了开发者使用TYPE_TOAST，Android 7.1.1 对TYPE_TOAST的使用更严格
  if Build.MANUFACTURER:lower()~="xiaomi" or Build.VERSION.SDK_INT >= 25
    mLayoutParams.type = WindowManager.LayoutParams.TYPE_APPLICATION;
   else
    mLayoutParams.type = WindowManager.LayoutParams.TYPE_TOAST;
  end
  return mLayoutParams
end


ShowBubbles=function(ids)
  local mBubbleView=
  {
    LinearLayout;
    layout_width="fill";
    layout_height="fill";
    id="Mcft_Main",
    {
      View,
      id=ids,
      layout_width="54dp";
      layout_height="55dp";
      Visibility=View.GONE,
    }
  }
  mWindowManager.addView(loadlayout(mBubbleView), IndicatorWindow(0,0) );
end

PaintBubbles=function(progess)
  newPaint=Paint()
  newPaint.setTextSize(bubblesTextSize)
  TextMin=newPaint.measureText(tostring(progess))
  Indicator=sp2px(24)
  BubblesSize=bubblesTextSize+sp2px(3)
  local Paintd=LuaDrawable(function(canvas,paint)
    local cW=canvas.Width
    local cH=canvas.Height
    paint.setAntiAlias(true)
    paint.setStyle(Style.FILL);

    paint.setFakeBoldText(false)
    paint.setTextSize(bubblesTextSize)

    paint.setColor(TopHandleColor)
    local path =Path()
    path.moveTo(Indicator/2,(cH/1.75))
    path.lineTo(cW-(Indicator/2),cH/1.75)
    path.lineTo(cW/2,cH)
    path.close()
    canvas.drawPath(path, paint)

    canvas.drawCircle(cW/2,cH/2,BubblesSize, paint);

    paint.setColor(BubblesColor)
    canvas.drawText(tostring(progess),(cW/2)-(TextMin/2),(cH/2)+(bubblesTextSize/2.5),paint)
  end)
  return {draw=Paintd,Circlesize=(BubblesSize*2)+Indicator}
end






locationUpdate=function(views,getX,getY,progess)
  local outRect1 = Rect();
  activity.getWindow().getDecorView().getWindowVisibleDisplayFrame(outRect1);
  local WidowHeight=outRect1.height()
  -Seekbarids.getParent().getHeight()
  local clazz=Class.forName("com.android.internal.R$dimen")
  local object = clazz.newInstance();
  local height = Integer.parseInt(tostring(clazz.getField("status_bar_height").get(object)))
  local status_bar_height = this.getResources().getDimensionPixelSize(height);

  local BuDraw=PaintBubbles(progess)
  views.setBackground(BuDraw.draw)
  views.setX(getX+((CanvasMargin/2)-(views.getWidth()/2)))
  views.setY((getY+status_bar_height)-(views.getHeight()/2))
  --views.setImageDrawable(BuDraw.draw)
  local newParams=IndicatorWindow(outRect1.height(),Seekbarids.getParent().getWidth())
  mWindowManager.updateViewLayout(views.getParent(), newParams);

end

LogoutBubble=function(view)
  mWindowManager.removeViewImmediate(view.getParent());
end


PropertyS=function(a,b,view,isType)
  local Attribute_definition = PropertyValuesHolder.ofFloat("method",{ a,b})
  local mValueAnimator = ObjectAnimator.ofPropertyValuesHolder({Attribute_definition})
  .setStartDelay(0)
  .setInterpolator(AnticipateOvershootInterpolator())
  .setDuration(500)
  mValueAnimator.addUpdateListener({
    onAnimationUpdate=function(animation)
      local Animationaprogress=animation.getAnimatedValue()
      if isType
        view.setVisibility(View.VISIBLE)
      end
      view.setAlpha(Animationaprogress)
      view.setPivotY(view.getHeight()*1.4)
      .setPivotX(view.getWidth()/2)
      .setScaleX(Animationaprogress)
      .setScaleY(Animationaprogress)
      --.setRotationX(-90+(Animationaprogress*90))
      AnimFolatN=Animationaprogress
      SeekbarAmin.invalidateSelf()
    end,
  });
  mValueAnimator.addListener({
    onAnimationEnd=function()
      if not isType
        view.setVisibility(View.GONE)
        LogoutBubble(view)
      end
    end
  }).start()
end


--mBubbleView.setVisibility(GONE); --防闪烁

--mLayoutParams.x = (mBubbleCenterRawX + 0.5f);
--mWindowManager.updateViewLayout(mBubbleView, mLayoutParams);


getWidthPercentage=function(w,max,proges)
  local WidthDivision=(w/max)*proges
  return {TowX=WidthDivision,Proges=proges}
end

AnimFolatN=0
SeekbarAmin=LuaDrawable(function(canvas,piant,d)
  canvasWidth=canvas.getWidth()-CanvasMargin
  canvasHeight=canvas.getHeight()

  canvas.translate(CanvasMargin/2,0)

  piant.setStrokeCap(Paint.Cap.ROUND)
  piant.setAntiAlias(true)

  --画背景进度条
  piant.setColor(BgColor);
  piant.setStrokeWidth(BgHeight);
  canvas.drawLines({0,canvasHeight/2,canvasWidth,canvasHeight/2},piant)

  --画顶部进度条
  piant.setColor(TopColor);
  piant.setStrokeWidth(TopHeight+((TopHeight/4)*AnimFolatN));
  canvas.drawLines({0,canvasHeight/2,getWidthPercentage(canvasWidth,Max,Dragprogress).TowX,canvasHeight/2},piant)

  --画滑块
  piant.setColor(0x23000000);
  canvas.drawCircle(getWidthPercentage(canvasWidth,Max,Dragprogress).TowX,canvasHeight/2,((TopHandle*1.7)*AnimFolatN), piant);
  piant.setColor(TopHandleColor);
  canvas.drawCircle(getWidthPercentage(canvasWidth,Max,Dragprogress).TowX,canvasHeight/2,TopHandle-((TopHandle)*AnimFolatN), piant);
end)

Seekbarids.background=SeekbarAmin

getPView=function(a)
  a=a.getParent()
  while a~=nil and not tostring(a):find("ScrollView")
    a=a.getParent()
  end
  return a==nil and 0 or a.getScrollY()
end

Seekbarids.onTouch=function(v,event)
  local Maction=event.getAction()
  if Maction==MotionEvent.ACTION_DOWN
    if event.getX()>CanvasMargin/2 and event.getX()<v.getWidth()-CanvasMargin/2
      and event.getY()>(v.getHeight()/2)-TopHandle and event.getY()<(v.getHeight()/2)+TopHandle
      WithinRange=true
      ShowBubbles("bubbles")
      PropertyS(0,1,bubbles,true)
      v.getParent().requestDisallowInterceptTouchEvent(true);
     else
      WithinRange=false
    end
   elseif Maction==MotionEvent.ACTION_MOVE
    if WithinRange
      v.getParent().requestDisallowInterceptTouchEvent(true);
      local getXTo=event.getX()
      local ranger=(event.getX()-CanvasMargin/2)/(v.getWidth()-CanvasMargin)
      --计算百分比
      Dragprogress=tointeger((ranger<=0 and 0 or (ranger>=1 and 1 or ranger))*Max)
      locationUpdate(bubbles,getWidthPercentage(Seekbarids.getWidth()-CanvasMargin,Max,Dragprogress).TowX
      ,Seekbarids.getY()-getPView(v),Dragprogress)
      SeekbarAmin.invalidateSelf()
    end
   elseif Maction==MotionEvent.ACTION_UP
    v.getParent().requestDisallowInterceptTouchEvent(false);
    pcall(function()
      if bubbles and WithinRange
        PropertyS(1,0,bubbles,false)
      end
    end)
  end
  return true;
end

t1.text=[[
Android一词的本义指“机器人”，同时也是Google于2007年11月5日宣布的基于Linux平台的开源手机操作系统的名称，该平台由操作系统、中间件、用户界面和应用软件组成。
]]
t2.text=[[
2003年10月，Andy Rubin等人创建Android公司，并组建Android团队。
2005年8月17日，Google低调收购了成立仅22个月的高科技企业Android及其团队。安迪鲁宾成为Google公司工程部副总裁，继续负责Android项目。
2007年11月5日，谷歌公司正式向外界展示了这款名为Android的操作系统，并且在这天谷歌宣布建立一个全球性的联盟组织，该组织由34家手机制造商、软件开发商、电信运营商以及芯片制造商共同组成，并与84家硬件制造商、软件开发商及电信营运商组成开放手持设备联盟（Open Handset Alliance）来共同研发改良Android系统，这一联盟将支持谷歌发布的手机操作系统以及应用软件，Google以Apache免费开源许可证的授权方式，发布了Android的源代码。[3] [4]
2008年，在GoogleI/O大会上，谷歌提出了AndroidHAL架构图，在同年8月18号，Android获得了美国联邦通信委员会（FCC）的批准，在2008年9月，谷歌正式发布了Android 1.0系统，这也是Android系统最早的版本。
2009年4月，谷歌正式推出了Android 1.5这款手机，从Android 1.5版本开始，谷歌开始将Android的版本以甜品的名字命名，Android 1.5命名为Cupcake（纸杯蛋糕）。该系统与Android 1.0相比有了很大的改进。
2009年9月，谷歌发布了Android 1.6的正式版，并且推出了搭载Android 1.6正式版的手机HTC Hero（G3），凭借着出色的外观设计以及全新的Android 1.6操作系统，HTC Hero（G3）成为当时全球最受欢迎的手机。Android 1.6也有一个有趣的甜品名称，它被称为Donut（甜甜圈）。
2010年2月，Linux内核开发者Greg Kroah-Hartman将Android的驱动程序从Linux内核“状态树”（“staging tree”）上除去，从此，Android与Linux开发主流将分道扬镳。在同年5月份，谷歌正式发布了Android 2.2操作系统。谷歌将Android 2.2操作系统命名为Froyo，翻译完名为冻酸奶。
2010年10月，谷歌宣布Android系统达到了第一个里程碑，即电子市场上获得官方数字认证的Android应用数量已经达到了10万个，Android系统的应用增长非常迅速。在2010年12月，谷歌正式发布了Android 2.3操作系统Gingerbread （姜饼）。
2011年1月，谷歌称每日的Android设备新用户数量达到了30万部，到2011年7月，这个数字增长到55万部，而Android系统设备的用户总数达到了1.35亿，Android系统已经成为智能手机领域占有量最高的系统。
2011年8月2日，Android手机已占据全球智能机市场48%的份额，并在亚太地区市场占据统治地位，终结了Symbian（塞班系统）的霸主地位，跃居全球第一。
2011年9月，Android系统的应用数目已经达到了48万，而在智能手机市场，Android系统的占有率已经达到了43%。继续在排在移动操作系统首位。谷歌将会发布全新的Android 4.0操作系统，这款系统被谷歌命名为Ice Cream Sandwich（冰激凌三明治）。
2012年1月6日，谷歌Android Market已有10万开发者推出超过40万活跃的应用，大多数的应用程序为免费。Android Market应用程序商店目录在新年首周周末突破40万基准，距离突破30万应用仅4个月。在2011年早些时候，Android Market从20万增加到30万应用也花了四个月。[5]
2013年11月1日，Android4.4正式发布，从具体功能上讲，Android4.4提供了各种实用小功能，新的Android系统更智能，添加更多的Emoji表情图案，UI的改进也更现代，如全新的HelloiOS7半透明效果。
2015年，网络安全公司Zimperium研究人员警告，安卓(Android)存在“致命”安全漏洞,黑客发送一封彩信便能在用户毫不知情的情况下完全控制手机。[6]
2018年10月，谷歌表示，将于2018年12月6日停止Android系统中的Nearby Notifications（附近通知）服务，因为Android用户收到太多的附近商家推销信息的垃圾邮件。[7]
2020年3月，谷歌的Android安全公告中提到，新更新已经提供了CVE-2020-0069补丁来解决针对联发科芯片的一个严重安全漏洞。[8]
]]
```

### 计算平均数,中位数,极差,标准差

```lua
local numericalArray={10,12,18,20,22,23,23,27,31,32,34,34,38,42,43,48}
local number=0
local number2=0

table.sort(numericalArray)
for index=1,#numericalArray do
  number=number+numericalArray[index]
  if index==#numericalArray then
    local average=number/#numericalArray
    for index2=1,#numericalArray do
      number2=number2+(numericalArray[index2]-average)^2
      if index2==#numericalArray then
        local variance=number2/#numericalArray
        if #numericalArray%2==1then
          mediumOrder=numericalArray[math.floor(#numericalArray/2)+1]
         else
          mediumOrder=(numericalArray[math.floor(#numericalArray/2)]+numericalArray[math.floor(#numericalArray/2)+1])/2
        end
        print("中位数->"..mediumOrder.."\n平均数->"..average.."\n方差->"..variance.."\n标准差->"..math.sqrt(variance))
      end
    end
  end
end
```

## 来源：实用代码.txt

### 获取设备标识码

```lua
import "android.provider.Settings$Secure"
android_id = Secure.getString(activity.getContentResolver(), Secure.ANDROID_ID)
```

### 获取IMEI

```lua
import "android.content.Context"
imei=activity.getSystemService(Context.TELEPHONY_SERVICE).getDeviceId()
```

### 控件背景渐变动画

```lua
view=控件id
color1 = 0xffFF8080;
color2 = 0xff8080FF;
color3 = 0xff80ffff;
color4 = 0xff80ff80;
import "android.animation.ObjectAnimator"
import "android.animation.ArgbEvaluator"
import "android.animation.ValueAnimator"
import "android.graphics.Color"
colorAnim = ObjectAnimator.ofInt(view,"backgroundColor",{color1, color2, color3,color4})
colorAnim.setDuration(3000)
colorAnim.setEvaluator(ArgbEvaluator())
colorAnim.setRepeatCount(ValueAnimator.INFINITE)
colorAnim.setRepeatMode(ValueAnimator.REVERSE)
colorAnim.start()
```

### 精准获取屏幕尺寸

```lua
function getScreenPhysicalSize(ctx)
  import "android.util.DisplayMetrics"
  dm = DisplayMetrics();
  ctx.getWindowManager().getDefaultDisplay().getMetrics(dm);
  diagonalPixels = Math.sqrt(Math.pow(dm.widthPixels, 2) + Math.pow(dm.heightPixels, 2));
  return diagonalPixels / (160 * dm.density);
end
print(getScreenPhysicalSize(activity))
```

### 发送邮件

```lua
import "android.content.Intent"
i = Intent(Intent.ACTION_SEND)
i.setType("message/rfc822")
i.putExtra(Intent.EXTRA_EMAIL, {"2113075983@.com"})
i.putExtra(Intent.EXTRA_SUBJECT,"Feedback")
i.putExtra(Intent.EXTRA_TEXT,"Content")
activity.startActivity(Intent.createChooser(i, "Choice"))
```

### 自定义默认弹窗标题,消息,按钮的颜色

```lua
dialog=AlertDialog.Builder(this)
.setTitle("标题")
.setMessage("消息")
.setPositiveButton("积极",{onClick=function(v) print"点击了积极按钮"end})
.setNeutralButton("中立",nil)
.setNegativeButton("否认",nil)
.show()
dialog.create()


--更改消息颜色
message=dialog.findViewById(android.R.id.message)
message.setTextColor(0xff1DA6DD)

--更改Button颜色
import "android.graphics.Color"
dialog.getButton(dialog.BUTTON_POSITIVE).setTextColor(0xff1DA6DD)
dialog.getButton(dialog.BUTTON_NEGATIVE).setTextColor(0xff1DA6DD)
dialog.getButton(dialog.BUTTON_NEUTRAL).setTextColor(0xff1DA6DD)

--更改Title颜色
import "android.text.SpannableString"
import "android.text.style.ForegroundColorSpan"
import "android.text.Spannable"
sp = SpannableString("标题")
sp.setSpan(ForegroundColorSpan(0xff1DA6DD),0,#sp,Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
dialog.setTitle(sp)
```

### 获取手机存储空间

```lua
--获取手机内置剩余存储空间
 function GetSurplusSpace()
 fs =  StatFs(Environment.getDataDirectory().getPath())
 return Formatter.formatFileSize(activity, (fs.getAvailableBytes()))
 end

 --获取手机内置存储总空间
 function GetTotalSpace()
 path = Environment.getExternalStorageDirectory()
 stat = StatFs(path.getPath())
 blockSize = stat.getBlockSize()
 totalBlocks = stat.getBlockCount()
 return Formatter.formatFileSize(activity, blockSize * totalBlocks)
 end
```

### 获取视频第一帧

```lua
function GetVideoFrame(path)
  import "android.media.MediaMetadataRetriever"
  media = MediaMetadataRetriever()
  media.setDataSource(tostring(path))
  return media.getFrameAtTime()
end
```

### 选择文件模块

```lua
import "android.widget.ArrayAdapter"
import "android.widget.LinearLayout"
import "android.widget.TextView"
import "java.io.File"
import "android.widget.ListView"
import "android.app.AlertDialog"
function ChoiceFile(StartPath,callback)
  --创建ListView作为文件列表
  lv=ListView(activity).setFastScrollEnabled(true)
  --创建路径标签
  cp=TextView(activity)
  lay=LinearLayout(activity).setOrientation(1).addView(cp).addView(lv)
  ChoiceFile_dialog=AlertDialog.Builder(activity)--创建对话框
  .setTitle("选择文件")
  .setView(lay)
  .show()
  adp=ArrayAdapter(activity,android.R.layout.simple_list_item_1)
  lv.setAdapter(adp)
  function SetItem(path)
    path=tostring(path)
    adp.clear()--清空适配器
    cp.Text=tostring(path)--设置当前路径
    if path~="/" then--不是根目录则加上../
      adp.add("../")
    end
    ls=File(path).listFiles()
    if ls~=nil then
      ls=luajava.astable(File(path).listFiles()) --全局文件列表变量
      table.sort(ls,function(a,b)
        return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.Name<b.Name)
      end)
    else
      ls={}
    end
    for index,c in ipairs(ls) do
      if c.isDirectory() then--如果是文件夹则
        adp.add(c.Name.."/")
      else--如果是文件则
        adp.add(c.Name)
      end
    end
  end
  lv.onItemClick=function(l,v,p,s)--列表点击事件
    项目=tostring(v.Text)
    if tostring(cp.Text)=="/" then
      路径=ls[p+1]
    else
      路径=ls[p]
    end

    if 项目=="../" then
      SetItem(File(cp.Text).getParentFile())
    elseif 路径.isDirectory() then
      SetItem(路径)
    elseif 路径.isFile() then
      callback(tostring(路径))
      ChoiceFile_dialog.hide()
    end

  end

  SetItem(StartPath)
end

--ChoiceFile(StartPath,callback)
--第一个参数为初始化路径,第二个为回调函数
--原创
```

### 选择路径模块

```lua
require "import"
import "android.widget.ArrayAdapter"
import "android.widget.LinearLayout"
import "android.widget.TextView"
import "java.io.File"
import "android.widget.ListView"
import "android.app.AlertDialog"
function ChoicePath(StartPath,callback)
  --创建ListView作为文件列表
  lv=ListView(activity).setFastScrollEnabled(true)
  --创建路径标签
  cp=TextView(activity)
  lay=LinearLayout(activity).setOrientation(1).addView(cp).addView(lv)
  ChoiceFile_dialog=AlertDialog.Builder(activity)--创建对话框
  .setTitle("选择路径")
  .setPositiveButton("OK",{
  onClick=function()
  callback(tostring(cp.Text))
  end})
.setNegativeButton("Canel",nil)
  .setView(lay)
  .show()
  adp=ArrayAdapter(activity,android.R.layout.simple_list_item_1)
  lv.setAdapter(adp)
  function SetItem(path)
    path=tostring(path)
    adp.clear()--清空适配器
    cp.Text=tostring(path)--设置当前路径
    if path~="/" then--不是根目录则加上../
      adp.add("../")
    end
    ls=File(path).listFiles()
    if ls~=nil then
      ls=luajava.astable(File(path).listFiles()) --全局文件列表变量
      table.sort(ls,function(a,b)
        return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.Name<b.Name)
      end)
    else
      ls={}
    end
    for index,c in ipairs(ls) do
      if c.isDirectory() then--如果是文件夹则
        adp.add(c.Name.."/")
      end
    end
  end
  lv.onItemClick=function(l,v,p,s)--列表点击事件
    项目=tostring(v.Text)
    if tostring(cp.Text)=="/" then
      路径=ls[p+1]
    else
      路径=ls[p]
    end

    if 项目=="../" then
      SetItem(File(cp.Text).getParentFile())
    elseif 路径.isDirectory() then
      SetItem(路径)
    elseif 路径.isFile() then
      callback(tostring(路径))
      ChoiceFile_dialog.hide()
    end

  end

  SetItem(StartPath)
end


import "android.os.*"
ChoicePath(Environment.getExternalStorageDirectory().toString(),
function(path)
print(path)
end)

--第一个参数为初始化路径,第二个为回调函数
--原创
```

### 获取视图中的所有文本

```lua
function GetAllText(view)
textTable={}
function GetText(Parent)
local number=Parent.getChildCount()
for i=0,number do
local view=Parent.getChildAt(i)
if pcall(function()view.addView(TextView(activity))end) then
GetText(view)
elseif pcall(function()view.getText()end) then
table.insert(textTable,tostring(view.Text))
end
end
end
GetText(view)
return textTable
end

print(table.unpack(GetAllText(Parent)))
```

### 控件圆角

```lua
function CircleButton(view,InsideColor,radiu)
  import "android.graphics.drawable.GradientDrawable"
  drawable = GradientDrawable()
  drawable.setShape(GradientDrawable.RECTANGLE)
  drawable.setColor(InsideColor)
  drawable.setCornerRadii({radiu,radiu,radiu,radiu,radiu,radiu,radiu,radiu});
  view.setBackgroundDrawable(drawable)
end
角度=50
控件id=ed
控件颜色=0xFF09639C
CircleButton(控件id,控件颜色,角度)
```

### 匹配汉字

```lua
function filter_spec_chars(s)
	local ss = {}
	for k = 1, #s do
		local c = string.byte(s,k)
		if not c then break end
		if (c>=48 and c<=57) or (c>= 65 and c<=90) or (c>=97 and c<=122) then
			if not string.char(c):find("%w") then
   table.insert(ss, string.char(c))
	end
 	elseif c>=228 and c<=233 then
			local c1 = string.byte(s,k+1)
			local c2 = string.byte(s,k+2)
			if c1 and c2 then
				local a1,a2,a3,a4 = 128,191,128,191
				if c == 228 then a1 = 184
				elseif c == 233 then a2,a4 = 190,c1 ~= 190 and 191 or 165
				end
				if c1>=a1 and c1<=a2 and c2>=a3 and c2<=a4 then
					k = k + 2
					table.insert(ss, string.char(c,c1,c2))
				end
			end
		end
	end
	return table.concat(ss)
end
print(filter_spec_chars("A1B2汉C3D4字E5F6,,,"))
--来源网络,加了个if过滤掉英文与数字,使其只捕获中文
```

### 播放音乐与视频

```lua
import "android.media.MediaPlayer"
mediaPlayer =  MediaPlayer()

--初始化参数
mediaPlayer.reset()

--设置播放资源
mediaPlayer.setDataSource("storage/sdcard0/a.mp3")

--开始缓冲资源
mediaPlayer.prepare()

--是否循环播放该资源
mediaPlayer.setLooping(true)

--缓冲完成的监听
mediaPlayer.setOnPreparedListener(MediaPlayer.OnPreparedListener() {
    onPrepared=function(mediaPlayer
        mediaPlayer.start()
   end});

--是否在播放
mediaPlayer.isPlaying()

--暂停播放
mediaPlayer.pause()

--从30位置开始播放
mediaPlayer.seekTo(30)

--停止播放
mediaPlayer.stop()






--播放视频
--视频的播放与音乐播放过程一样：

--先创建一个媒体对象
import "android.media.MediaPlayer"
mediaPlayer =  MediaPlayer()
--初始化参数
mediaPlayer.reset()

--设置播放资源
mediaPlayer.setDataSource("storage/sdcard0/a.mp4")

--拿到显示的SurfaceView
sh = surfaceView.getHolder()
sh.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS)

--设置显示SurfaceView
mediaPlayer.setDisplay(sh)

--设置音频流格式
mediaPlayer.setAudioStreamType(AudioManager.Stream_Music)

--开始缓冲资源
mediaPlayer.prepare()

--缓冲完成的监听
mediaPlayer.setOnPreparedListener(MediaPlayer.OnPreparedListener{
   onPrepared=function(mediaPlayer)
		--开始播放
        mediaPlayer.start()
   end
});

--释放播放器
mediaPlayer.release()


--非原创
```

### 获取系统SDK，Android版本及设备型号

```lua
device_model = Build.MODEL --设备型号

version_sdk = Build.VERSION.SDK --设备SDK版本

version_release = Build.VERSION.RELEASE --设备的系统版本
```

### 控件颜色修改

```lua
import "android.graphics.PorterDuffColorFilter"
import "android.graphics.PorterDuff"

--修改按钮颜色
button.getBackground().setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))

--修改编辑框颜色
edittext.getBackground().setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP));

--修改Switch颜色
switch.ThumbDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP));
switch.TrackDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))

--修改ProgressBar颜色
progressbar.IndeterminateDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))

--修改SeekBar滑条颜色
seekbar.ProgressDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))
--修改SeekBar滑块颜色
seekbar.Thumb.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))
```

### 修改对话框按钮颜色

```lua
function DialogButtonFilter(dialog,button,WidgetColor)
if Build.VERSION.SDK_INT >= 21 then
import "android.graphics.PorterDuffColorFilter"
import "android.graphics.PorterDuff"
if button==1 then
dialog.getButton(dialog.BUTTON_POSITIVE).setTextColor(WidgetColor)
elseif button==2 then
dialog.getButton(dialog.BUTTON_NEGATIVE).setTextColor(WidgetColor)
elseif button==3 then
dialog.getButton(dialog.BUTTON_NEUTRAL).setTextColor(WidgetColor)
end
end
end
--第一个参数为对话框的变量
--第二个参数为1时，则修改POSITIVE按钮颜色,为二则修改NEGATIVE按钮颜色,为三则修改NEUTRAL按钮颜色
--第三个参数为要修改成的颜色
```

### 查询本地所有视频

```lua
function QueryAllVideo()
import "android.provider.MediaStore"
cursor = activity.ContentResolver
mImageUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
mCursor = cursor.query(mImageUri,nil,nil,nil,MediaStore.Video.Media.DATE_TAKEN)
mCursor.moveToLast()
VideoTable={}
while mCursor.moveToPrevious() do
   path = mCursor.getString(mCursor.getColumnIndex(MediaStore.Video.Media.DATA))
   table.insert(VideoTable,tostring(path))
end
mCursor.close()
return VideoTable
end
--返回一个表
```

### 查询本地所有图片

```lua
function QueryAllImage()
import "android.provider.MediaStore"
cursor = activity.ContentResolver
mImageUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
mCursor = cursor.query(mImageUri,nil,nil,nil,MediaStore.Images.Media.DATE_TAKEN)
mCursor.moveToLast()
imageTable={}
while mCursor.moveToPrevious() do
   path = mCursor.getString(mCursor.getColumnIndex(MediaStore.Images.Media.DATA))
   table.insert(imageTable,tostring(path))
end
mCursor.close()
return imageTable
end
--返回一个表
```

### 递归查找文件

```lua
function outPath(ret)
for i,p in pairs(luajava.astable(ret)) do
print(p)
end
end
function find(catalog,name)
 local n=0
 local t=os.clock()
 local ret={}
 require "import"
 import "java.io.File"
 import "java.lang.String"
 function FindFile(catalog,name)
   local name=tostring(name)
   local ls=catalog.listFiles() or File{}
   for 次数=0,#ls-1 do
     --local 目录=tostring(ls[次数])
     local f=ls[次数]
     if f.isDirectory() then--如果是文件夹则继续匹配
       FindFile(f,name)
     else--如果是文件则
       n=n+1
       if n%1000==0 then
         print(n,os.clock()-t)
       end
      local nm=f.Name
       if string.find(nm,name) then
         --thread(insert,目录)
         table.insert(ret,tostring(f))
       end
     end
   luajava.clear(f)
   end
 end
 FindFile(catalog,name)
 call("outPath",ret)
end

import "java.io.File"

catalog=File("/sdcard/AndroLua")
name=".j?pn?g"
thread(find,catalog,name)
```

### 获取手机内置存储路径

```text
Environment.getExternalStorageDirectory().toString()
```

### 获取已安装程序的包名、版本号、最后更新时间、图标、应用名称

```lua
function GetAppInfo(包名)
  import "android.content.pm.PackageManager"
  local pm = activity.getPackageManager();
  local 图标 = pm.getApplicationInfo(tostring(包名),0)
  local 图标 = 图标.loadIcon(pm);
  local pkg = activity.getPackageManager().getPackageInfo(包名, 0);
  local 应用名称 = pkg.applicationInfo.loadLabel(activity.getPackageManager())
  local 版本号 = activity.getPackageManager().getPackageInfo(包名, 0).versionName
  local 最后更新时间 = activity.getPackageManager().getPackageInfo(包名, 0).lastUpdateTime
  local cal = Calendar.getInstance();
  cal.setTimeInMillis(最后更新时间);
  local 最后更新时间 = cal.getTime().toLocaleString()
  return 包名,版本号,最后更新时间,图标,应用名称
end
```

### 获取指定安装包的包名,图标,应用名

```lua
import "android.content.pm.PackageManager"
import "android.content.pm.ApplicationInfo"
function GetApkInfo(archiveFilePath)
pm = activity.getPackageManager()
info = pm.getPackageArchiveInfo(archiveFilePath, PackageManager.GET_ACTIVITIES);
if info ~= nil then
  appInfo = info.applicationInfo;
 appName = tostring(pm.getApplicationLabel(appInfo))
  packageName = appInfo.packageName; --安装包名称
  version=info.versionName; --版本信息
   icon = pm.getApplicationIcon(appInfo);--图标
end
return packageName,version,icon
end
```

### 获取某程序是否安装

```lua
if pcall(function() activity.getPackageManager().getPackageInfo("包名",0) end) then
  print("安装了")
else
  print("没安装")
end
```

### 设置TextView字体风格

```lua
import "android.graphics.Paint"
--设置中划线
id.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG)
--设置下划线
id.getPaint().setFlags(Paint. UNDERLINE_TEXT_FLAG )
--设置加粗
id.getPaint().setFakeBoldText(true)
--设置斜体
id.getPaint().setTextSkewX(0.2)

--设置TypeFace
import "android.graphics.Typeface"
id.getPaint().setTypeface()
--参数列表
Typeface.DEFAULT 默认字体
Typeface.DEFAULT_BOLD 加粗字体
Typeface.MONOSPACE monospace字体
Typeface.SANS_SERIF sans字体
Typeface.SERIF serif字体
```

### 缩放图片

```lua
function rotateToFit(bm,degrees)
    import "android.graphics.Matrix"
    import "android.graphics.Bitmap"
    width = bm.getWidth()
    height = bm.getHeight()
    matrix =  Matrix()
    matrix.postRotate(degrees)
    bmResult = Bitmap.createBitmap(bm, 0, 0, width, height, matrix, true)
    return bmResult
  end
bm=loadbitmap(图片路径)
缩放级别=2
rotateToFit(bm,degrees)
--非原创
```

### 获取运营商名称

```lua
import "android.content.Context"
运营商名称 = this.getSystemService(Context.TELEPHONY_SERVICE).getNetworkOperatorName()
print(运营商名称)
--添加权限   READ_PHONE_STATE
```

### Drawable着色

```lua
function ToColor(path,color)
 local  aa=BitmapDrawable(loadbitmap(tostring(path)))
   aa.setColorFilter(PorterDuffColorFilter(color,PorterDuff.Mode.SRC_ATOP))
return aa
end
```

### 保存图片到本地

```lua
function SavePicture(name,bm)
if  bm then
import "java.io.FileOutputStream"
import "java.io.File"
import "android.graphics.Bitmap"
name=tostring(name)
f = File(name)
out = FileOutputStream(f)
bm.compress(Bitmap.CompressFormat.PNG,90, out)
out.flush()
out.close()
return true
else
return false
end
end
```

### 调用应用商店搜索应用

```lua
import "android.content.Intent"
import "android.net.Uri"
intent = Intent("android.intent.action.VIEW")
intent .setData(Uri.parse( "market://details?id="..activity.getPackageName()))
this.startActivity(intent)
```

### 分享

```lua
--分享文件
function Sharing(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  intent = Intent();
  intent.setAction(Intent.ACTION_SEND);
  intent.setType(Mime);
  file = File(path);
  uri = Uri.fromFile(file);
  intent.putExtra(Intent.EXTRA_STREAM,uri);
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
  activity.startActivity(Intent.createChooser(intent, "分享到:"));
  end

--分享文字
text="分享的内容"
intent=Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_SUBJECT, "分享");
intent.putExtra(Intent.EXTRA_TEXT, text);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
activity.startActivity(Intent.createChooser(intent,"分享到:"));
```

### 调用其它程序打开文件

```lua
function OpenFile(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  if Mime then
    intent = Intent();
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    intent.setAction(Intent.ACTION_VIEW);
    intent.setDataAndType(Uri.fromFile(File(path)), Mime);
    activity.startActivity(intent);
  else
    Toastc("找不到可以用来打开此文件的程序")
  end
end
```

### 图片圆角

```lua
function GetRoundedCornerBitmap(bitmap,roundPx)
  import "android.graphics.PorterDuffXfermode"
  import "android.graphics.Paint"
  import "android.graphics.RectF"
  import "android.graphics.Bitmap"
  import "android.graphics.PorterDuff$Mode"
  import "android.graphics.Rect"
  import "android.graphics.Canvas"
  import "android.util.Config"
  width = bitmap.getWidth()
  output = Bitmap.createBitmap(width, width,Bitmap.Config.ARGB_8888)
  canvas = Canvas(output);
  color = 0xff424242;
  paint = Paint()
  rect = Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
  rectF = RectF(rect);
  paint.setAntiAlias(true);
  canvas.drawARGB(0, 0, 0, 0);
  paint.setColor(color);
  canvas.drawRoundRect(rectF, roundPx, roundPx, paint);
  paint.setXfermode(PorterDuffXfermode(Mode.SRC_IN));
  canvas.drawBitmap(bitmap, rect, rect, paint);
  return output;
end
import "android.graphics.drawable.BitmapDrawable"
圆角弧度=50
bitmap=loadbitmap(picturePath)
RoundPic=GetRoundedCornerBitmap(bitmap)
```

### 一键加群与QQ聊天

```lua
import "android.net.Uri"
import "android.content.Intent"
--加群
url="mqqapi://card/show_pslcard?src_type=internal&version=1&uin=383792635&card_type=group&source=qrcode"
activity.startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))

--QQ聊天
url="mqqwpa://im/chat?chat_type=wpa&uin=2113075983"
activity.startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))
```

### 发送短信

```lua
--后台发送短信
 require "import"
 import "android.telephony.*"
 SmsManager.getDefault().sendTextMessage(tostring(号码), nil, tostring(内容), nil, nil)

--调用系统发送短信
import "android.content.Intent"
import "android.net.Uri"
uri = Uri.parse("smsto:"..号码)
intent = Intent(Intent.ACTION_SENDTO, uri)
intent.putExtra("sms_body",内容)
intent.setAction("android.intent.action.VIEW")
activity.startActivity(intent)
```

### 判断数组中是否存在某个值

```lua
function Table_exists(tables,value)
for index,content in pairs(tables) do
if content:find(value) then
return true
end
end
end
```

### 字符串操作

```lua
strings="左中右"

--取字符串左边
左=strings:match("(.+)中")


--取字符串中间
中=strings:match("左(.-)右")


--取字符串右边
右=strings:match("(.+)右")

--替换
string.gsub(原字符串,替换的字符串,替换成的字符串)

--匹配子串位置
起始位置,结束位置=string.find(字符串,子串)


--按位置捕获字符串
string.sub(字符串,子串起始位置,子串结束位置)
```

### 剪切板操作

```lua
import "android.content.Context"
--导入类

a=activity.getSystemService(Context.CLIPBOARD_SERVICE).getText()
--获取剪贴板

activity.getSystemService(Context.CLIPBOARD_SERVICE).setText(edit.Text)
--写入剪贴板
```

### 各种事件

```lua
function main(...)
  --...是newActivity传递过来的参数。
  print("入口函数",...)
end

function onCreate()
  print("窗口创建")
end

function onStart()
  print("活动开始")
end

function onResume()
  print("返回程序")
end

function onPause()
  print("活动暂停")
end

function onStop()
  print("活动停止")
end

function onDestroy()
  print("程序已退出")
end

function onResult(name,...)
  --name：返回的活动名称
  --...：返回的参数
  print("返回活动",name,...)
end

function onCreateOptionsMenu(menu)
  --menu：选项菜单。
  menu.add("菜单")
end

function onOptionsItemSelected(item)
  --item：选中的菜单项
  print(item.Title)
end

function onConfigurationChanged(config)
  --config：配置信息
  print("屏幕方向关闭")
end

function onKeyDown(keycode,event)
  --keycode：键值
  --event：事件
  print("按键按下",keycode)
end

function onKeyUp(keycode,event)
  --keycode：键值
  --event：事件
  print("按键抬起",keycode)
end

function onKeyLongPress(keycode,event)
  --keycode：键值
  --event：事件
  print("按键长按",keycode)
end

function onTouchEvent(event)
  --event：事件
  print("触摸事件",event)
end

function onKeyDown(c,e)
  if c==4 then
--返回键事件
end
end


id.onClick=function()
--控件被单击
end

id.onLongClick=function()
--控件被长按
end


id.onItemClick=function(p,v,i,s)
--列表项目被单击
项目=v.Text
return true
end

id.onItemLongClick=function(p,v,i,s)
--列表项目被长按
项目=v.Text
return true
end


id.onItemLongClick=function(p,v,i,s)
--列表项目被长按
项目=v.Text
return true
end

--Spinner的项目单击事件
id.onItemSelected=function(l,v,p,i)
项目=v.Text
end

--ExpandableListView的父项目与子项目单击事件
id.onGroupClick=function(l,v,p,s)
  print(v.Text..":GroupClick")
end

id.onChildClick=function(l,v,g,c)
  print(v.Text..":ChildClick")
end
```

### Shell执行

```lua
function exec(cmd)
local p=io.popen(string.format('%s',cmd))
local s=p:read("*a")
p:close()
return s
end

print(exec("echo  ...."))

部分常用命令:
--删除文件或文件夹
rm -r /路径

--复制文件或文件夹
cp -r inpath outpath

--移动文件或文件夹
mv -r inpath outpath

--挂载系统目录
mount -o remount,rw path

--修改系统文件权限
chmod 755 /system/build.prop

--重启
reboot 

--关机
reboot -p

--重启至recovery
reboot recovery
```

## 来源：网络操作.txt

### 自带Http模块

```lua
获取内容 get函数
Http.get(url,cookie,charset,header,callback)
url 网络请求的链接网址
cookie 使用的cookie，也就是服务器的身份识别信息
charset 内容编码
header 请求头
callback 请求完成后执行的函数

除了url和callback其他参数都不是必须的

回调函数接受四个参数值分别是
code 响应代码，2xx表示成功，4xx表示请求错误，5xx表示服务器错误，-1表示出错
content 内容，如果code是-1，则为出错信息
cookie 服务器返回的用户身份识别信息
header 服务器返回的头信息

向服务器发送数据 post函数
Http.post(url,data,cookie,charset,header,callback)
除了增加了一个data外，其他参数和get完全相同
data 向服务器发送的数据

下载文件 download函数
Http.download(url,path,cookie,header,callback)
参数中没有编码参数，其他同get，
path 文件保存路径

需要特别注意一点，只支持同时有127个网络请求，否则会出错


Http其实是对Http.HttpTask的封装，Http.HttpTask使用的更加通用和灵活的形式
参数格式如下
Http.HttpTask( url, String method, cookie, charset, header,  callback)
所有参数都是必选，没有则传入nil

url 请求的网址
method 请求方法可以是get，post，put，delete等
cookie 身份验证信息
charset 内容编码
header 请求头
callback 回调函数

该函数返回的是一个HttpTask对象，
需要调用execute方法才可以执行，
t=Http.HttpTask(xxx)
t.execute{data}

注意调用的括号是花括号，内容可以是字符串或者byte数组，
使用这个形式可以自己封装异步上传函数
```

### TrafficStats类

```lua
import "android.net.TrafficStats"
getMobileRxBytes()  --获取通过Mobile连接收到的字节总数，不包含WiFi
getMobileRxPackets()  --获取Mobile连接收到的数据包总数
getMobileTxBytes()  --Mobile发送的总字节数
getMobileTxPackets()  --Mobile发送的总数据包数
getTotalRxBytes()  --获取总的接受字节数，包含Mobile和WiFi等
getTotalRxPackets()  --总的接受数据包数，包含Mobile和WiFi等
getTotalTxBytes()  --总的发送字节数，包含Mobile和WiFi等
getTotalTxPackets()  --发送的总数据包数，包含Mobile和WiFi等
getUidRxBytes(int uid)  --获取某个网络UID的接受字节数
getUidTxBytes(int uid) --获取某个网络UID的发送字节数
--例:TrafficStats.getTotalRxBytes()
```

### 开启关闭WiFi

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
wifi.setWifiEnabled(true)--关闭则false
```

### 断开网络

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
wifi.disconnect()
```

### WiFi是否打开

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
wi = wifi.isWifiEnabled()
```

### WiFi是否连接

```lua
connManager = activity.getSystemService(Context.CONNECTIVITY_SERVICE)
    mWifi = connManager.getNetworkInfo(ConnectivityManager.TYPE_WIFI);
    if tostring(mWifi):find("none)")  then
    --未连接
    else
    --连接
    end
```

### 数据网络是否连接

```lua
manager = activity.getSystemService(Context.CONNECTIVITY_SERVICE);
gprs = manager.getNetworkInfo(ConnectivityManager.TYPE_MOBILE).getState();
if tostring(gprs)== "CONNECTED" then
print"当前数据网络"
end
```

### 获取WiFi信息

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
 wifi.getConfiguredNetworks()
```

### 获取WiFi状态

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
print(wifi.getWifiState())
```

### IP地址

```lua
--查看某网站IP地址
address=InetAddress.getByName("www.10010.com");

--查看本机IP地址
address=InetAddress.getLocalHost();

--查看IP地址
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getDhcpInfo()
string.match(tostring(wifi),"ipaddr(.-)gate")
```

### 获取Dns

```lua
import "android.content.Context"

--获取Dns1
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getDhcpInfo()
 print(string.match(tostring(wifi),"dns1 (.-) dns2"))

--获取Dns2
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getDhcpInfo()
 dns2 = string.match(tostring(wifi),"dns2 (.-) D")
```

### 获取网络名称

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print(wifiInfo.getSSID())
```

### 获取WiFi加密类型

```lua
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getConfiguredNetworks()
print(string.match(tostring(wifi),[[KeyMgmt: (.-) P]]))
```

### 获取网络信号强度

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print(wifiInfo.getRssi())
```

### 获取SSID是否被隐藏

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print(wifiInfo.getHiddenSSID())
```

### 获取Mac地址

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print( wifiInfo.getMacAddress())
```

## 来源：文件操作.txt

### 创建新文件

```lua
--使用File类
import "java.io.File"--导入File类
File(文件路径).createNewFile()

--使用io库
io.open("/sdcard/aaaa", 'w')
```

### 创建新文件夹

```lua
--使用File类
import "java.io.File"--导入File类
File(文件夹路径).mkdir()

--创建多级文件夹
File(文件夹路径).mkdirs()

--shell
os.execute('mkdir '..文件夹路径)
```

### 重命名与移动文件

```lua
--Shell
os.execute("mv "..oldname.." "..newname)

--os
os.rename (oldname, newname)

--File
import "java.io.File"--导入File类
File(旧).renameTo(File(新))
```

### 追加更新文件

```text
io.open(文件路径,"a+"):write("更新的内容"):close()
```

### 更新文件

```text
io.open(文件路径,"w+"):write("更新的内容"):close()
```

### 写入文件

```text
io.open(文件路径,"w"):write("内容"):close()
```

### 写入文件(自动创建父文件夹)

```lua
function 写入文件(路径,内容)
  import "java.io.File"
  f=File(tostring(File(tostring(路径)).getParentFile())).mkdirs()
  io.open(tostring(路径),"w"):write(tostring(内容)):close()
end
```

### 读取文件

```text
io.open(文件路径):read("*a")
```

### 按行读取文件

```lua
for c in io.lines(文件路径) do
print(c)
end
```

### 删除文件或文件夹

```lua
--使用File类
import "java.io.File"--导入File类
File(文件路径).delete()
--使用os方法
os.remove (filename)
```

### 复制文件

```text
LuaUtil.copyDir(from,to)
```

### 递归删除文件夹或文件

```lua
--使用LuaUtil辅助库
LuaUtil.rmDir(路径)

--使用Shell
os.execute("rm -r "..路径)
```

### 替换文件内字符串

```lua
function 替换文件字符串(路径,要替换的字符串,替换成的字符串)
if 路径 then
  路径=tostring(路径)
  内容=io.open(路径):read("*a")
  io.open(路径,"w+"):write(tostring(内容:gsub(要替换的字符串,替换成的字符串))):close()
else
return false
end
end
```

### 获取文件列表

```text
import("java.io.File")
luajava.astable(File(文件夹路径).listFiles())
```

### 获取文件名称

```lua
import "java.io.File"--导入File类
File(路径).getName()
```

### 获取文件大小

```lua
function GetFileSize(path)
  import "java.io.File"
  import "android.text.format.Formatter"
  size=File(tostring(path)).length()
  Sizes=Formatter.formatFileSize(activity, size)
  return Sizes
end
```

### 获取文件或文件夹最后修改时间

```lua
function GetFilelastTime(path)
  f = File(path);
  cal = Calendar.getInstance();
  time = f.lastModified()
  cal.setTimeInMillis(time);
  return cal.getTime().toLocaleString()
end
```

### 获取文件字节

```lua
import "java.io.File"--导入File类
File(路径).length()
```

### 获取文件父文件夹路径

```lua
import "java.io.File"--导入File类
File(path).getParentFile()
```

### 获取文件Mime类型

```lua
function GetFileMime(name)
import "android.webkit.MimeTypeMap"
ExtensionName=tostring(name):match("%.(.+)")
Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
return tostring(Mime)
end
print(GetFileMime("/sdcard/a.png"))
```

### 判断路径是不是文件夹

```lua
import "java.io.File"--导入File类
File(路径).isDirectory()
--也可用来判断文件夹存不存在
```

### 判断路径是不是文件

```lua
import "java.io.File"--导入File类
File(路径).isFile()
--也可用来判断文件存不存在
```

### 判断文件或文件夹存不存在

```lua
import "java.io.File"--导入File类
File(路径).exists()

--使用io
function file_exists(path)
local f=io.open(path,'r')
if f~=nil then io.close(f) return true else return false end
end
```

### 判断是不是系统隐藏文件

```lua
import "java.io.File"--导入File类
File(路径).isHidden()
```

## 来源：用户界面.txt

### 标题栏(ActionBar)

```lua
--部分常用API
show:显示
hide:隐藏
Elevation:设置阴影
BgroundDrawable:设置背景
DisplayHomeAsUpEnabled(boolean):设置是否显示返回图标



--设置标题
activity.ActionBar.setTitle('大标题')
activity.ActionBar.setSubTitle("小标题")

--设置ActionBar背景颜色
import "android.graphics.drawable.ColorDrawable"
activity.ActionBar.setBackgroundDrawable(ColorDrawable(Color))

--自定义ActionBar标题颜色
import "android.text.SpannableString"
import "android.text.style.ForegroundColorSpan"
import "android.text.Spannable"
sp = SpannableString("标题")
sp.setSpan(ForegroundColorSpan(0xff1DA6DD),0,#sp,Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
activity.ActionBar.setTitle(sp)

--自定义ActionBar布局
DisplayShowCustomEnabled(true)
CustomView(loadlayout(layout))


--ActionBar返回按钮
activity.ActionBar.setDisplayHomeAsUpEnabled(true)
--自定义返回按钮图标
activity.ActionBar.setHomeAsUpIndicator(drawable)


--菜单
function onCreateOptionsMenu(menu)
  menu.add("菜单1")
  menu.add("菜单2")
  menu.add("菜单3")
end
function onOptionsItemSelected(item)
  print("你选择了:"..item.Title)
end





--Tab导航使用
import "android.app.ActionBar$TabListener"
actionBar=activity.ActionBar
actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
tab = actionBar.newTab().setText("Tab1").setTabListener(TabListener({
  onTabSelected=function()
    print"Tab1"
  end}))
tab2=actionBar.newTab().setText("Tab2").setTabListener(TabListener({
  onTabSelected=function()
    print"Tab2"
  end}))
actionBar.addTab(tab)
actionBar.addTab(tab2)
```

### 五大布局

```lua
--Android中常用的5大布局方式有以下几种：
--线性布局（LinearLayout）：按照垂直或者水平方向布局的组件。
--帧布局（FrameLayout）：组件从屏幕左上方布局组件。
--表格布局（TableLayout）：按照行列方式布局组件。
--相对布局（RelativeLayout）：相对其它组件的布局方式。
--绝对布局（AbsoluteLayout）：按照绝对坐标来布局组件。


1.线性布局(LinearLayout)
线性布局是Android开发中最常见的一种布局方式，它是按照垂直或者水平方向来布局，通过orientation属性可以设置线性布局的方向。属性值有垂直（vertical）和水平（horizontal）两种。
常用的属性：
orientation：可以设置布局的方向
gravity:用来控制组件的对齐方式
layout_weight控制各个控件在布局中的相对大小,layout_weight的属性是一个非负整数值。
线性布局会根据该控件layout_weight值与其所处布局中所有控件layout_weight值之和的比值为该控件分配占用的区域
--[[例如，在水平布局的LinearLayout中有两个Button，这两个Button的layout_weight属性值都为1,那么这两个按钮都会被拉伸到整个屏幕宽度的一半。如果layout_weight指为0，控件会按原大小显示，不会被拉伸.
对于其余layout_weight属性值大于0的控件，系统将会减去layout_weight属性值为0的控件的宽度或者高度,再用剩余的宽度或高度按相应的比例来分配每一个控件显示的宽度或高度]]


2.帧布局(FrameLayout)
帧布局是从屏幕的左上角（0,0）坐标开始布局，多个组件层叠排列，第一个添加的组件放到最底层，最后添加到框架中的视图显示在最上面。上一层的会覆盖下一层的控件。


3.表格布局（TableLayout）
表格布局是一个ViewGroup以表格显示它的子视图（view）元素，即行和列标识一个视图的位置。
表格布局常用的属性如下：
collapseColumns：隐藏指定的列
shrinkColumns：收缩指定的列以适合屏幕，不会挤出屏幕
stretchColumns：尽量把指定的列填充空白部分
layout_column:控件放在指定的列
layout_span:该控件所跨越的列数


4.相对布局（RelativeLayout）
相对布局是按照组件之间的相对位置来布局，比如在某个组件的左边，右边，上面和下面等。


5.绝对布局(AbsoluteLayout)
采用坐标轴的方式定位组件，左上角是（0，0）点，往右x轴递增，往下Y轴递增,组件定位属性为layout_x 和layout_y来确定坐标。
```

### Widget(普通控件)

```lua
--Button(按钮控件)、TextView(文本控件)、EditText(编辑框控件)

常用API:
id.setText("文本")--设置控件文本
id.getText()--获取控件文本
id.setWidth(300)--设置控件宽度
id.setHeight(300)--设置控件高度


--点击事件
id.onClick=function()
print"你触发了点击事件"
end

--长按事件
id.onLongClick=function()
print"你触发了长按事件"
end



--图片控件(ImageView与ImageButton)
--设置图片
--布局表中用src属性就可以，如:src=图片路径,

--动态设置
id.setImageBitmap(loadbitmap(图片路径))
--设置Drawable对象
import "android.graphics.drawable.BitmapDrawable"
id.setImageDrawable(BitmapDrawable(loadbitmap(图片路径)))

--缩放，scaleType
--字段
CENTER    --按原来size居中显示，长/宽超过View的长/宽，截取图片的居中部分显示 
CENTER_CROP    --按比例扩大图片的size居中显示，使图片长(宽)等于或大于View的长(宽) 
CENTER_INSIDE  --完整居中显示，按比例缩小使图片长/宽等于或小于View的长/宽 
FIT_CENTER     --按比例扩大/缩小到View的宽度，居中显示 
FIT_END        --按比例扩大/缩小到View的宽度，显示在View的下部分位置 
FIT_START      --按比例扩大/缩小到View的宽度，显示在View的上部分位置 
FIT_XY         --不按比例扩大/缩小到View的大小显示 
MATRIX         --用矩阵来绘制，动态缩小放大图片来显示。 


--点击与长按事件同上
```

### Check View(检查控件)

```lua
--CheckBox(复选框),Switch(开关控件),ToggleButton(切换按钮)
--直接判断是否选中然后执行相应事件即可
--判断API
check.isSelected()--返回布尔值


--RadioButton(单选按钮)与RadioGroup
--将RadioButton的父布局设定为RadioGroup然后绑定下面的监听即可
rp.setOnCheckedChangeListener{
  onCheckedChanged=function(g,c)
  l=g.findViewById(c)
  print(l.Text)
  end}
```

### SeekBar(拖动条)

```lua
--绑定监听
seekbar.setOnSeekBarChangeListener{
onStartTrackingTouch=function()
--开始拖动
end,
onStopTrackingTouch=function()
--停止拖动
end,
onProgressChanged=function()
--状态改变
end}

--部分API
Progress--当前进度
Max--最大进度
```

### ProgressBar(进度条)

```lua
--超大号圆形风格
style="?android:attr/progressBarStyleLarge"
--小号风格
style="?android:attr/progressBarStyleSmall"
--标题型风格
style="?android:attr/progressBarStyleSmallTitle"
--长形进度条
style="?android:attr/progressBarStyleHorizontal"

--部分API
max --最大进度值
progress --设置进度值
secondaryProgress="70" --初始化的底层第二个进度值

id.incrementProgressBy(5)
--ProgressBar进度值增加5
id.incrementProgressBy(-5)
--ProgressBar进度值减少5
id.incrementSecondaryProgressBy(5)
--ProgressBar背后的第二个进度条 进度值增加5
id.incrementSecondaryProgressBy(-5)
--ProgressBar背后的第二个进度条 进度值减少5
```

### Adapter View(适配器控件)

```lua
--适配器控件主要包括(ListView,GridView,Spinner,ExpandableList等)

--想要动态为此类控件添加项目就必须得要依靠适配器！
--适配器使用
--AarrayAdapter(简单适配器)
--创建项目数组
数据={}
--添加项目数组
for i=1,100 do
table.insert(数据,tostring(i))
end
--创建适配器
array_adp=ArrayAdapter(activity,android.R.layout.simple_list_item_1,String(数据))
--设置适配器
lv.setAdapter(array_adp)


--LuaAdapter(Lua适配器)
--创建自定义项目视图
item={
  LinearLayout,
  orientation="vertical",
    layout_width="fill",
   {
    TextView,
    id="text",
    layout_margin="15dp",
    layout_width="fill"
  },
}
--创建项目数组
data={}
--创建适配器
adp=LuaAdapter(activity,data,item)
--添加数据
for n=1,100 do
  table.insert(data,{
    text={
      Text=tostring(n),
    },
  })
end
--设置适配器
lv.Adapter=adp


--以上的适配器ListView、Spinner与GridView等控件通用

--那么ExpandableListView(折叠列表)怎么办呢？
--别怕，安卓系统还提供了一个ArrayExpandableListAdapter来给我们使用，可以简单的适配ExpandableListView，下面给出实例

ns={
  "Widget","Check view","Adapter view","Advanced Widget","Layout","Advanced Layout",
}

wds={
  {"Button","EditText","TextView",
    "ImageButton","ImageView"},
  {"CheckBox","RadioButton","ToggleButton","Switch"},
  {"ListView","ExpandableListView","Spinner"},
  {"SeekBar","ProgressBar","RatingBar",
    "DatePicker","TimePicker","NumberPicker"},
  {"LinearLayout","AbsoluteLayout","FrameLayout"},
  {"RadioGroup","GridLayout",
    "ScrollView","HorizontalScrollView"},
}


mAdapter=ArrayExpandableListAdapter(activity)
for k,v in ipairs(ns) do
  mAdapter.add(v,wds[k])
end
el.setAdapter(mAdapter)
--这样就实现ExpandableListView项目的适配了




--当然AdapterView的事件响应也是与普通控件不同的。

--ListView与GridView的单击与长按事件
--项目被单击
id.onItemClick=function(l,v,p,i)
print(v.Text)
return true
end
--项目被长按
id.onItemLongClick=function(l,v,p,i)
print(v.Text)
return true
end


--Spinner的项目单击事件
id.onItemSelected=function(l,v,p,i)
print(v.Text)
end

--ExpandableListView的父项目与子项目单击事件
id.onGroupClick=function(l,v,p,s)
print(v.Text..":GroupClick")
end

id.onChildClick=function(l,v,g,c)
print(v.Text..":ChildClick")
end
```

### LuaWebView(浏览器控件)

```lua
--常用API
id.loadUrl("http://www.androlua.cn")--加载网页
id.loadUrl("file:///storage/sdcard0/index.html")--加载本地文件
id.getTitle()--获取网页标题
id.getUrl()--获取当前Url
id.requestFocusFromTouch()--设置支持获取手势焦点
id.getSettings().setJavaScriptEnabled(true)--设置支持JS
id.setPluginsEnabled(true)--支持插件
id.setUseWideViewPort(false)--调整图片自适应
id.getSettings().setSupportZoom(true)--支持缩放
id.getSettings().setLayoutAlgorithm(LayoutAlgorithm.SINGLE_COLUMN)--支持重新布局
id.supportMultipleWindows()--设置多窗口
id.stopLoading()--停止加载网页


--状态监听
id.setWebViewClient{
shouldOverrideUrlLoading=function(view,url)
--Url即将跳转
 end,
onPageStarted=function(view,url,favicon)
--网页加载
end,
onPageFinished=function(view,url)
--网页加载完成
end}
```

### AutoCompleteTextView(自动补全文本框)

```lua
--适配数据
arr={"Rain","Rain1","Rain2"};
arrayAdapter=LuaArrayAdapter(activity,{TextView,padding="10dp",textSize="18sp",layout_width="fill",textColor="#ff000000"}, String(arr))
actw.setAdapter(arrayAdapter)

Threshold=1--设置输入几个字符后才能出现提示
```

### TimePicker(时间选择器)

```lua
--时间改变监听器
import "android.widget.TimePicker$OnTimeChangedListener"
id.setOnTimeChangedListener{
  onTimeChanged=function(view,时,分)
    print(时,分)
  end}

--部分API
时=id.getCurrentHour()--获取小时
分=id.getCurrentMinute()--获取分钟
id.setIs24HourView(Boolean(true))--设置24小时制
```

### DatePicker(日期选择器)

```lua
id=dp
日=id.getDayOfMonth()--获取选择的天数
月=id.getMonth ()--获取选择的月份
年=id.getYear()--获取选择的年份
id.updateDate(2016,1,1)--更新日期
print(年,月,日)
```

### NnumberPicker(数值选择器)

```text
setMinValue(0)--设置最小值
setMaxValue(100)--设置最大值
setValue(50)--设置当前值
getValue()--获取选择的值
OnValueChangedListener--数值改变监听器
```

### AlertDialog(对话框)

```lua
--常用API
.setTitle("标题")--设置标题
.setMessage("设置消息")--设置消息
.setView(loadlayout(layout))--设置自定义视图
.setPositiveButton("积极",{onClick=function() end})--设置积极按钮
.setNeutralButton("中立",nil)--设置中立按钮
.setNegativeButton("否认",nil)--设置否认按钮




--普通对话框
AlertDialog.Builder(this)
.setTitle("标题")
.setMessage("消息")
.setPositiveButton("积极",{onClick=function(v) print"点击了积极按钮"end})
.setNeutralButton("中立",nil)
.setNegativeButton("否认",nil)
.show()




--输入对话框
InputLayout={
  LinearLayout;
  orientation="vertical";
  Focusable=true,
  FocusableInTouchMode=true,
  {
    TextView;
    id="Prompt",
    textSize="15sp",
    layout_marginTop="10dp";
    layout_marginLeft="3dp",
    layout_width="80%w";
    layout_gravity="center",
    text="输入:";
  };
  {
    EditText;
    hint="输入";
    layout_marginTop="5dp";
    layout_width="80%w";
    layout_gravity="center",
    id="edit";
  };
};

AlertDialog.Builder(this)
.setTitle("标题")
.setView(loadlayout(InputLayout))
.setPositiveButton("确定",{onClick=function(v) print(edit.Text)end})
.setNegativeButton("取消",nil)
.show()
import "android.view.View$OnFocusChangeListener"
edit.setOnFocusChangeListener(OnFocusChangeListener{
 onFocusChange=function(v,hasFocus)
if hasFocus then
Prompt.setTextColor(0xFD009688)
end
end})



--下载文件对话框
Download_layout={
  LinearLayout;
  orientation="vertical";
  id="Download_father_layout",
  {
    TextView;
    id="linkhint",
    layout_marginTop="10dp";
    text="下载链接",
    layout_width="80%w";
    textColor=WidgetColors,
    layout_gravity="center";
  };
  {
    EditText;
    id="linkedit",
    layout_width="80%w";
    layout_gravity="center";
  };
  {
    TextView;
    id="pathhint",
    text="下载路径",
    layout_width="80%w";
    textColor=WidgetColors,
    layout_marginTop="10dp";
    layout_gravity="center";
  };
  {
    EditText;
    id="pathedit",
    layout_width="80%w";
    layout_gravity="center";
  };
};

AlertDialog.Builder(this)
.setTitle("下载文件")
.setView(loadlayout(Download_layout))
.setPositiveButton("下载",{onClick=function(v)
  end})
.setNegativeButton("取消",nil)
.show()







--列表对话框
items={}
for i=1,5 do
table.insert(items,"项目"..tostring(i))
end
AlertDialog.Builder(this)
.setTitle("列表对话框")
.setItems(items,{onClick=function(l,v) print(items[v+1])end})
.show()


--单选对话框
单选列表={}
for i=1,5 do
table.insert(单选列表,"单选项目"..tostring(i))
end
local 单选对话框=AlertDialog.Builder(this)
.setTitle("列表对话框")
.setSingleChoiceItems(单选列表,-1,{onClick=function(v,p)print(单选列表[p+1])end})
单选对话框.show();



--多选对话框
items={}
for i=1,5 do
table.insert(items,"多选项目"..tostring(i))
end
多选对话框=AlertDialog.Builder(this)
.setTitle("多选框")
.setMultiChoiceItems(items, nil,{ onClick=function(v,p)print(items[p+1])end})
多选对话框.show();
```

### ProgressDialog(进度对话框)

```lua
--ProgressDialog__进度条对话框

dialog = ProgressDialog.show(this, "提示", "正在登陆中").hide()
--最简单便捷的方式

dialog2 = ProgressDialog.show(this, "提示", "正在登陆中", false).hide()
--最后一个boolean设置是否是不明确的状态

dialog3 = ProgressDialog.show(this, "提示", "正在登陆中",false, true).hide()
--最后一个boolean设置可以不可以点击取消

dialog4 = ProgressDialog.show(this, "提示", "正在登陆中",false, true, DialogInterface.OnCancelListener{
  onCancel=function()
    print("对话框取消")
  end
}).hide()

--最后一个参数监听对话框取消，并执行事件





--圆形旋转样式
dialog5= ProgressDialog(this)
dialog5.setProgressStyle(ProgressDialog.STYLE_SPINNER)
dialog5.setTitle("Loading...")
--设置进度条的形式为圆形转动的进度条
dialog5.setMessage("ProgressDialog")
dialog5.setCancelable(true)--设置是否可以通过点击Back键取消
dialog5.setCanceledOnTouchOutside(false)--设置在点击Dialog外是否取消Dialog进度条
dialog5.setOnCancelListener{
  onCancel=function(l)
    print("取消Dialog5")
  end}
--取消对话框监听事件
dialog5.show().hide()





--水平样式
dialog6= ProgressDialog(this)
dialog6.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
--设置进度条的形式为水平进度条
dialog6.setTitle("ProgressDialog_HORIZONTAL")
dialog6.setCancelable(true)--设置是否可以通过点击Back键取消
dialog6.setCanceledOnTouchOutside(false)--设置在点击Dialog外是否取消Dialog进度条
dialog6.setOnCancelListener{
  onCancel=function(l)
    print("取消Dialog6")
  end}
--取消对话框监听事件
dialog6.setMax(100)
--设置最大进度值
dialog6.show().hide()

function 增加(i)
  dialog6.incrementProgressBy(10)
  dialog6.incrementSecondaryProgressBy(10)
  if i=="10" then
    dialog6.dismiss()
    print("加载完成")
  end
  --当进度走完时销毁对话框
end
function 加载()
  require "import"
  for i=1,10 do
    Thread.sleep(300)
    call("增加",tostring(i))
  end
end
--thread(加载)
```

### InputMethodManager(输入法管理器)

```lua
在Android的开发中，有时候会遇到软键盘弹出时挡住输入框的情况。
这时候可以设置下软键盘的模式就可以了。
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE|WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN)
有时候需要软键盘不要把我们的布局整体推上去，这时候可以这样：
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN)

模式常量：

软输入区域是否可见。
SOFT_INPUT_MASK_STATE = 0x0f

未指定状态。
SOFT_INPUT_STATE_UNSPECIFIED = 0

不要修改软输入法区域的状态
SOFT_INPUT_STATE_UNCHANGED = 1

隐藏输入法区域（当用户进入窗口时
SOFT_INPUT_STATE_HIDDEN = 2

当窗口获得焦点时，隐藏输入法区域
SOFT_INPUT_STATE_ALWAYS_HIDDEN = 3

显示输入法区域（当用户进入窗口时）
SOFT_INPUT_STATE_VISIBLE = 4

当窗口获得焦点时，显示输入法区域
SOFT_INPUT_STATE_ALWAYS_VISIBLE = 5

窗口应当主动调整，以适应软输入窗口。
SOFT_INPUT_MASK_ADJUST = 0

窗口应当主动调整，以适应软输入窗口。
SOFT_INPUT_MASK_ADJUST = 0xf0

未指定状态，系统将根据窗口内容尝试选择一个输入法样式。
SOFT_INPUT_ADJUST_UNSPECIFIED = 0x00

当输入法显示时，允许窗口重新计算尺寸，使内容不被输入法所覆盖。
不可与SOFT_INPUT_ADJUSP_PAN混合使用；如果两个都没有设置，系统将根据窗口内容自动设置一个选项。
SOFT_INPUT_ADJUST_RESIZE = 0x10

输入法显示时平移窗口。它不需要处理尺寸变化，框架能够移动窗口以确保输入焦点可见。
不可与SOFT_INPUT_ADJUST_RESIZE混合使用；如果两个都没有设置，系统将根据窗口内容自动设置一个选项。
SOFT_INPUT_ADJUST_PAN = 0x20

当用户转至此窗口时，由系统自动设置，所以你不要设置它。
当窗口显示之后该标志自动清除。
SOFT_INPUT_IS_FORWARD_NAVIGATION = 0x100


其它Api参考:
import "android.view.inputmethod.InputMethodManager"


调用显示系统默认的输入法
imm =  activity.getSystemService(Context.INPUT_METHOD_SERVICE)
imm.showSoftInput(m_receiverView(接受软键盘输入的视图(View)),InputMethodManager.SHOW_FORCED(提供当前操作的标记，SHOW_FORCED表示强制显示))



如果输入法关闭则打开，如果输入法打开则关闭
imm = activity.getSystemService(Context.INPUT_METHOD_SERVICE)
imm.toggleSoftInput(0,InputMethodManager.HIDE_NOT_ALWAYS)


获取软键盘是否打开
imm = activity.getSystemService(Context.INPUT_METHOD_SERVICE)
isOpen=imm.isActive()
--返回一个布尔值


隐藏软键盘
activity.getSystemService(INPUT_METHOD_SERVICE)).hideSoftInputFromWindow(WidgetSearchActivity.this.getCurrentFocus().getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS)

显示软键盘
activity.getSystemService(INPUT_METHOD_SERVICE)).showSoftInput(控件ID, 0)
```

### PopMenu(弹出式菜单)

```lua
pop=PopupMenu(activity,view)
menu=pop.Menu
menu.add("项目1").onMenuItemClick=function(a)

end
menu.add("项目2").onMenuItemClick=function(a)

end
pop.show()--显示
```

### PopWindow(弹出式窗口)

```lua
pop=PopWindow(activity)--创建PopWindow
pop.setContentView(loadlayout(布局))--设置布局
pop.setWidth(activity.Width*0.3)--设置宽度
pop.setHeight(activity.Width*0.3)--设置高度
pop.setFocusable(true)--设置可获得焦点
window.setTouchable(true)--设置可触摸
--设置点击外部区域是否可以消失
pop.setOutsideTouchable(false)
--显示
pop.showAtLocation(view,0,0,0)
```

### Toast(提示)

```lua
--默认Toast
Toast.makeText(activity, "Toast",Toast.LENGTH_SHORT).show()

--自定义位置Toast
Toast.makeText(activity,"自定义位置Toast", Toast.LENGTH_LONG).setGravity(Gravity.CENTER, 0, 0).show()

--带图片Toast
图片=loadbitmap("/sdcard/a.png")
toast = Toast.makeText(activity,"带图片的Toast", Toast.LENGTH_LONG)
toastView = toast.getView()
imageCodeProject = ImageView(activity)
imageCodeProject.setImageBitmap(图片)
toastView.addView(imageCodeProject, 0)
toast.show()

--自定义布局Toast
布局=loadlayout(layout)
local toast=Toast.makeText(activity,"提示",Toast.LENGTH_SHORT).setView(布局).show()
```

### 控件常用属性

```lua
--EditText(输入框)
singleLine=true--设置单行输入
Error="错误的输入"--设置用户输入了错误的信息时的提醒
MaxLines=5--设置最大输入行数
MaxEms=5--设置每行最大宽度为五个字符的宽度
InputType="number"--设置只可输入数字
Hint="请输入"--设置编辑框为空时的提示文字


--ImageView(图片视图)
src="a.png"--设置控件图片资源
scaleType="fitXY"--设置图片缩放显示
ColorFilter=Color.BLUE--设置图片着色



--ListView(列表视图)
Items={"item1","item2","item3"}--设置列表项目,但只能在布局表设置,动态添加项目请看Adapter View详解。
DividerHeight=0--设置无隔断线
fastScrollEnabled=true--设置是否显示快速滑块



layout_marginBottom--离某元素底边缘的距离
layout_marginLeft--离某元素左边缘的距离
layout_marginRight--离某元素右边缘的距离
layout_marginTop--离某元素上边缘的距离
gravity--属性是对该view 内容的限定．比如一个button 上面的text. 你可以设置该text 在view的靠左，靠右等位置．以button为例，gravity="right"则button上面的文字靠右
layout_gravity--是用来设置该view相对与起父view 的位置．比如一个button 在linearlayout里，你想把该button放在靠左、靠右等位置就可以通过该属性设置．以button为例，layout_gravity="right"则button靠右
scaleType
--[[是控制图片如何resized/moved来匹对ImageView的size。ImageView.ScaleType / scaleType值的意义区别：
CENTER /center 按图片的原来size居中显示，当图片长/宽超过View的长/宽，则截取图片的居中部分显示
CENTER_CROP / centerCrop 按比例扩大图片的size居中显示，使得图片长(宽)等于或大于View的长(宽)
CENTER_INSIDE / centerInside 将图片的内容完整居中显示，通过按比例缩小或原来的size使得图片长/宽等于或小于View的长/宽
FIT_CENTER / fitCenter 把图片按比例扩大/缩小到View的宽度，居中显示
FIT_END / fitEnd 把图片按比例扩大/缩小到View的宽度，显示在View的下部分位置
FIT_START / fitStart 把图片按比例扩大/缩小到View的宽度，显示在View的上部分位置
FIT_XY / fitXY 把图片不按比例扩大/缩小到View的大小显示
MATRIX / matrix 用矩阵来绘制，动态缩小放大图片来显示。
]]
id--为控件指定相应的ID
text--指定控件当中显示的文字
textSize--指定控件当中字体的大小
background--指定该控件所使用的背景色
width--指定控件的宽度
height--指定控件的高度
layout_width--指定Container组件的宽度
layout_height--指定Container组件的高度
layout_weight--View中很重要的属性，按比例划分空间
padding--指定控件的内边距，也就是说控件当中的内容
sigleLine--如果设置为真的话，则控件的内容在同一行中进行显示
```

### Animation(动画)

```lua
--动画主要包括以下几种
Alpha:渐变透明度动画效果
Scale:渐变尺寸伸缩动画效果
Translate:画面转换位置移动动画效果
Rotate:画面转换位置移动动画效果

--共有的属性有
Duration --属性为动画持续时间 时间以毫秒为单位
fillAfter --当设置为true,该动画转化在动画结束后被应用
fillBefore --当设置为true,该动画转化在动画开始前被应用
repeatCount--动画的重复次数
repeatMode --定义重复的行为
startOffset --动画之间的时间间隔，从上次动画停多少时间开始执行下个动画
id.startAnimation(Animation)--设置控件开始应用这个动画



--动画状态监听
import "android.view.animation.Animation$AnimationListener"
动画.setAnimationListener(AnimationListener{
  onAnimationStart=function()
    print"动画开始"
  end,
onAnimationEnd=function()
  print"动画结束"
  end,
onAnimationRepeat=function()
  print"动画重复"
  end})


--实例
--控件向右旋转180度
Rotate_right=RotateAnimation(180, 0,
Animation.RELATIVE_TO_SELF, 0.5,
Animation.RELATIVE_TO_SELF, 0.5)
Rotate_right.setDuration(440)
Rotate_right.setFillAfter(true)

--控件向左旋转180度
Rotate_left=RotateAnimation(0, 180,
Animation.RELATIVE_TO_SELF, 0.5,
Animation.RELATIVE_TO_SELF, 0.5)
Rotate_left.setDuration(440)
Rotate_left.setFillAfter(true)



--动画设置___从上往下平移动画
Translate_up_down=TranslateAnimation(0, 0, 55, 0)
Translate_up_down.setDuration(800)
Translate_up_down.setFillAfter(true)



--动画设置___透明动画
Alpha=AlphaAnimation(0,1)
Alpha.setDuration(800)


--动画参数值
--AlphaAnimation(透明动画)
AlphaAnimation(float fromStart,float fromEnd)
float fromStart 动画起始透明值
float fromEnd 动画结束透明值

--ScaleAnimation(缩放动画)
ScaleAnimation(float fromX, float toX, float fromY, float toY,int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
float fromX 动画起始时 X坐标上的伸缩尺寸
float toX 动画结束时 X坐标上的伸缩尺寸
float fromY 动画起始时Y坐标上的伸缩尺寸
float toY 动画结束时Y坐标上的伸缩尺寸
int pivotXType 动画在X轴相对于物件位置类型
float pivotXValue 动画相对于物件的X坐标的开始位置
int pivotYType 动画在Y轴相对于物件位置类型
float pivotYValue 动画相对于物件的Y坐标的开始位置


--TranslateAnimation(位移动画)
TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)
float fromXDelta 动画开始的点离当前View X坐标上的差值
float toXDelta 动画结束的点离当前View X坐标上的差值
float fromYDelta 动画开始的点离当前View Y坐标上的差值
float toYDelta 动画结束的点离当前View Y坐标上的差值

--RotateAnimation(旋转动画)
RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
float fromDegrees：旋转的开始角度.
float toDegrees：旋转的结束角度.
int pivotXType：X轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT.
float pivotXValue：X坐标的伸缩值
int pivotYType：Y轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT.
float pivotYValue：Y坐标的伸缩值.
```

### LayoutAnimationController(布局动画控制器)

```lua
--LayoutAnimationController可以控制一组控件按照规定显示

--导入类
import "android.view.animation.AnimationUtils"
import "android.view.animation.LayoutAnimationController"


--创建一个Animation对象
animation = AnimationUtils.loadAnimation(activity,android.R.anim.slide_in_left)

--得到对象
lac = LayoutAnimationController(animation)

--设置控件显示的顺序
lac.setOrder(LayoutAnimationController.ORDER_NORMAL)
--LayoutAnimationController.ORDER_NORMAL   顺序显示
--LayoutAnimationController.ORDER_REVERSE 反显示
--LayoutAnimationController.ORDER_RANDOM 随机显示

--设置控件显示间隔时间
lac.setDelay(time)

--设置组件应用
view.setLayoutAnimation(lac)
```

### ObjectAnimator(属性动画)

```lua
ObjectAnimator(对象动画)
--属性动画概念：
所谓属性动画：
改变一切能改变的对象的属性值，不同于补间动画
只能改变 alpha，scale，rotate，translate
听着有点抽象，举例子说明。


补间动画能实现的:
1.alpha(透明)
--第一个参数为 view对象,第二个参数为 动画改变的类型,第三,第四个参数依次是开始透明度和结束透明度。
alpha = ObjectAnimator.ofFloat(text, "alpha", 0, 1)
alpha.setDuration(2000)--设置动画时间
alpha.setInterpolator(DecelerateInterpolator())--设置动画插入器，减速
alpha.setRepeatCount(-1)--设置动画重复次数，这里-1代表无限
alpha.setRepeatMode(Animation.REVERSE)--设置动画循环模式。
alpha.start()--启动动画。

2.scale(缩放)
animatorSet =  AnimatorSet()--组合动画
scaleX = ObjectAnimator.ofFloat(text, "scaleX", 1, 0)
scaleY = ObjectAnimator.ofFloat(text, "scaleY", 1, 0)
animatorSet.setDuration(2000)
animatorSet.setInterpolator(DecelerateInterpolator());
animatorSet.play(scaleX).with(scaleY)--两个动画同时开始
animatorSet.start();

3.translate(平移)
translationUp = ObjectAnimator.ofFloat(button, "Y",button.getY(), 0)
translationUp.setInterpolator(DecelerateInterpolator())
translationUp.setDuration(1500)
translationUp.start()

4. rotate(旋转)
set =  AnimatorSet()
anim = ObjectAnimator .ofFloat(phone, "rotationX", 0, 180)
anim.setDuration(2000)
anim2 = ObjectAnimator .ofFloat(phone, "rotationX", 180, 0)
anim2.setDuration(2000)
anim3 = ObjectAnimator .ofFloat(phone, "rotationY", 0, 180)
anim3.setDuration(2000)
anim4 = ObjectAnimator .ofFloat(phone, "rotationY", 180, 0)
anim4.setDuration(2000)
set.play(anim).before(anim2)--先执行anim动画之后在执行anim2
set.play(anim3).before(anim4)
set.start()


补间动画不能实现的:
5.android 改变背景颜色的动画实现如下
translationUp = ObjectAnimator.ofInt(button,"backgroundColor",{Color.RED, Color.BLUE, Color.GRAY,Color.GREEN})
translationUp.setInterpolator(DecelerateInterpolator())
translationUp.setDuration(1500)
translationUp.setRepeatCount(-1)
translationUp.setRepeatMode(Animation.REVERSE)
translationUp.setEvaluator(ArgbEvaluator())
translationUp.start()
--[[
ArgbEvaluator：这种评估者可以用来执行类型之间的插值整数值代表ARGB颜色。
FloatEvaluator：这种评估者可以用来执行浮点值之间的插值。
IntEvaluator：这种评估者可以用来执行类型int值之间的插值。
RectEvaluator：这种评估者可以用来执行类型之间的插值矩形值。

由于本例是改变View的backgroundColor属性的背景颜色所以此处使用ArgbEvaluator
]]
```

### overridePendingTransition(设置窗口动画)

```lua
activity.overridePendingTransition(android.R.anim.fade_in,android.R.anim.fade_out)
```

### 配色参考

```lua
--靛蓝配粉色
靛蓝色=0xFF3F51B5
粉色=0xFFE91E63

--蓝色配青绿色
蓝色=0xFF2196F3
青绿色=0xFF009688

--其它:
暗橙色=0xFFFF5722
酸橙色=0xFFCDDC39
深紫色=0xFF673AB7
青色=0xFF0097A7
红色=0xFFF44336
亮蓝=0xFF03A9F4
```

## 来源：整合代码.txt

### 统计字符数

```lua
function utfstrlen(str)
  local len = #str;
  local left = len;
  local cnt = 0;
  local arr={0,0xc0,0xe0,0xf0,0xf8,0xfc};
  while left ~= 0 do
    local tmp=string.byte(str,-left);
    local i=#arr;
    while arr[i] do
      if tmp>=arr[i] then left=left-i;break
      end
      i=i-1;
    end
    cnt=cnt+1;
  end
  return cnt;
end
```

### 设置控件字体

```lua
import "android.graphics.Typeface"
import "java.io.File"

--布局表中调用
function 字体(t)
  return Typeface.createFromFile(File(activity.getLuaDir().."/res/"..t..".ttf"))
end

--栗子:Typeface=字体("Product")

--代码中调用
function 字体设置(id,t)
  id.setTypeface(Typeface.createFromFile(File(activity.getLuaDir().."/res/"..t..".ttf")))
end

--栗子:字体设置(tv,"Product")

--字体需要放在res文件夹下

--QQ 773772682 提供
```

### 正则取文件名

```lua
function 取文件名(path)
  return path:match(".+/(.+)$")
end

function 取文件名无后缀(path)
  return path:match(".+/(.+)%..+$")
end

print(取文件名("/com/mukapp/top/muk.lua"))
print(取文件名无后缀("/com/mukapp/top/muk.lua"))
```

### 自定义Callback

```lua
function SentText(text,callback)
  xpcall(function()
    Toast.makeText(activity,text,Toast.LENGTH_SHORT).show()
    callback(true)
  end,function()
    callback(false)
  end)
end

SentText("emmmm",function(is)
  print(is)
end)
```

### 通知图库更新图片

```lua
--方法1 通过广播
activity.sendBroadcast(Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE,Uri.parse("file://"..图片路径)))

--方法2 MediaScannerConnection
import "android.media.MediaScannerConnection"
MediaScannerConnection.scanFile(activity, {File(图片路径).getAbsolutePath()}, nil, nil)
```

### 遍历汉字字符串

```lua
a="啊♂啊啊♂"

for i=1 , utf8.len(a) do
  print(utf8.sub(a,i,i))
end
```

### 模拟按键

```lua
function sendKeyCode(keyCode)
  xpcall(function()
    Runtime.getRuntime().exec("input keyevent " .. keyCode)
  end,function(e)
    print(e)
  end)
end

sendKeyCode(KeyEvent.KEYCODE_VOLUME_DOWN)--音量-

--[[
KEYCODE_0      '0' key.   7
KEYCODE_1      '1' key.   8
KEYCODE_2      '2' key.   9
KEYCODE_3      '3' key.   10
KEYCODE_4      '4' key.   11
KEYCODE_5      '5' key.   12
KEYCODE_6      '6' key.   13
KEYCODE_7      '7' key.   14
KEYCODE_8      '8' key.   15
KEYCODE_9      '9' key.   16

KEYCODE_A      'A' key.   29
KEYCODE_B      'B' key.   30
KEYCODE_C      'C' key.   31
KEYCODE_D      'D' key.   32
KEYCODE_E      'E' key.   33
KEYCODE_F      'F' key.   34
KEYCODE_G      'G' key.   35
KEYCODE_H      'H' key.   36
KEYCODE_I      'I' key.   37
KEYCODE_J      'J' key.   38
KEYCODE_K      'K' key.   39
KEYCODE_L      'L' key.   40
KEYCODE_M      'M' key.   41
KEYCODE_N      'N' key.   42
KEYCODE_O      'O' key.   43
KEYCODE_P      'P' key.   44
KEYCODE_Q      'Q' key.   45
KEYCODE_R      'R' key.   46
KEYCODE_S      'S' key.   47
KEYCODE_T      'T' key.   48
KEYCODE_U      'U' key.   49
KEYCODE_V      'V' key.   50
KEYCODE_W      'W' key.   51
KEYCODE_X      'X' key.   52
KEYCODE_Y      'Y' key.   53
KEYCODE_Z      'Z' key.   54

META_ALT_LEFT_ON   This mask is used to check whether the left ALT meta key is pressed.            16
META_ALT_MASK      This mask is a combination of META_ALT_ON, META_ALT_LEFT_ON and META_ALT_RIGHT_ON.      50
META_ALT_ON      This mask is used to check whether one of the ALT meta keys is pressed.            2
META_ALT_RIGHT_ON   This mask is used to check whether the right the ALT meta key is pressed.         32
META_CAPS_LOCK_ON   This mask is used to check whether the CAPS LOCK meta key is on.            1048576
META_CTRL_LEFT_ON   This mask is used to check whether the left CTRL meta key is pressed.            8192
META_CTRL_MASK      This mask is a combination of META_CTRL_ON, META_CTRL_LEFT_ON and META_CTRL_RIGHT_ON.      28672
META_CTRL_ON      This mask is used to check whether one of the CTRL meta keys is pressed.         4096
META_CTRL_RIGHT_ON   This mask is used to check whether the right CTRL meta key is pressed.            16384
META_FUNCTION_ON   This mask is used to check whether the FUNCTION meta key is pressed.            8
META_META_LEFT_ON   This mask is used to check whether the left META meta key is pressed.            131072
META_META_MASK      This mask is a combination of META_META_ON, META_META_LEFT_ON and META_META_RIGHT_ON.      458752
META_META_ON      This mask is used to check whether one of the META meta keys is pressed.         65536
META_META_RIGHT_ON   This mask is used to check whether the right META meta key is pressed.            262144
META_NUM_LOCK_ON   This mask is used to check whether the NUM LOCK meta key is on.               2097152
META_SCROLL_LOCK_ON   This mask is used to check whether the SCROLL LOCK meta key is on.            4194304
META_SHIFT_LEFT_ON   This mask is used to check whether the left SHIFT meta key is pressed.            64
META_SHIFT_MASK      This mask is a combination of META_SHIFT_ON, META_SHIFT_LEFT_ON and META_SHIFT_RIGHT_ON.   193
META_SHIFT_ON      This mask is used to check whether one of the SHIFT meta keys is pressed.         1
META_SHIFT_RIGHT_ON   This mask is used to check whether the right SHIFT meta key is pressed.            128
META_SYM_ON      This mask is used to check whether the SYM meta key is pressed.               4

KEYCODE_APOSTROPHE   ''' key.   75
KEYCODE_AT      '@' key.   77
KEYCODE_BACKSLASH   '\' key.   73
KEYCODE_COMMA      ',' key.   55
KEYCODE_EQUALS      '=' key.   70
KEYCODE_GRAVE      '`' key.   68
KEYCODE_LEFT_BRACKET   '[' key.   71
KEYCODE_MINUS      '-' key.   69
KEYCODE_PERIOD      '.' key.   56
KEYCODE_PLUS      '+' key.   81
KEYCODE_POUND      '#' key.   18
KEYCODE_RIGHT_BRACKET   ']' key.   72
KEYCODE_SEMICOLON   ';' key.   74
KEYCODE_SLASH      '/' key.   76
KEYCODE_STAR      '*' key.   17
KEYCODE_SPACE      Space key.   62
KEYCODE_TAB      Tab key.   61

KEYCODE_ENTER      Enter key.      66
KEYCODE_ESCAPE      Escape key.      111
KEYCODE_CAPS_LOCK   Caps Lock key.      115
KEYCODE_CLEAR      Clear key.      28
KEYCODE_PAGE_DOWN   Page Down key.      93
KEYCODE_PAGE_UP      Page Up key.      92
KEYCODE_SCROLL_LOCK   Scroll Lock key.   116
KEYCODE_MOVE_END   End.         123
KEYCODE_MOVE_HOME   Home.         122
KEYCODE_INSERT      Insert key.      124
KEYCODE_SHIFT_LEFT   Left Shift.      59
KEYCODE_SHIFT_RIGHT   Right Shift.      60

KEYCODE_F1   F1 key.      131
KEYCODE_F2   F2 key.      132
KEYCODE_F3   F3 key.      133
KEYCODE_F4   F4 key.      134
KEYCODE_F5   F5 key.      135
KEYCODE_F6   F6 key.      136
KEYCODE_F7   F7 key.      137
KEYCODE_F8   F8 key.      138
KEYCODE_F9   F9 key.      139
KEYCODE_F10   F10 key.   140
KEYCODE_F11   F11 key.   141
KEYCODE_F12   F12 key.   142

KEYCODE_BACK      Back key.      4
KEYCODE_CALL      Call key.      5
KEYCODE_ENDCALL      End Call key.      6
KEYCODE_CAMERA      Camera key.      27
KEYCODE_FOCUS      Camera Focus key.   80
KEYCODE_VOLUME_UP   Volume Up key.      24
KEYCODE_VOLUME_DOWN   Volume Down key.   25
KEYCODE_VOLUME_MUTE   Volume Mute key.   164
KEYCODE_MENU      Menu key.      82
KEYCODE_HOME      Home key.      3
KEYCODE_POWER      Power key.      26
KEYCODE_SEARCH      Search key.      84
KEYCODE_NOTIFICATION   Notification key.   83
KEYCODE_NUM      Number modifier key.   78
KEYCODE_SYM      Symbol modifier key.   63
KEYCODE_SETTINGS   Settings key.      176

KEYCODE_DEL      Backspace key. Deletes characters before the insertion point, unlike KEYCODE_FORWARD_DEL.   67
KEYCODE_FORWARD_DEL   Forward Delete key. Deletes characters ahead of the insertion point, unlike KEYCODE_DEL.   112

KEYCODE_NUMPAD_0      Numeric keypad '0' key.      144
KEYCODE_NUMPAD_1      Numeric keypad '1' key.      145
KEYCODE_NUMPAD_2      Numeric keypad '2' key.      146
KEYCODE_NUMPAD_3      Numeric keypad '3' key.      147
KEYCODE_NUMPAD_4      Numeric keypad '4' key.      148
KEYCODE_NUMPAD_5      Numeric keypad '5' key.      149
KEYCODE_NUMPAD_6      Numeric keypad '6' key.      150
KEYCODE_NUMPAD_7      Numeric keypad '7' key.      151
KEYCODE_NUMPAD_8      Numeric keypad '8' key.      152
KEYCODE_NUMPAD_9      Numeric keypad '9' key.      153
KEYCODE_NUMPAD_ADD      Numeric keypad '+' key       157
KEYCODE_NUMPAD_COMMA      Numeric keypad ',' key       159
KEYCODE_NUMPAD_DIVIDE      Numeric keypad '/' key       154
KEYCODE_NUMPAD_DOT      Numeric keypad '.' key       158
KEYCODE_NUMPAD_EQUALS      Numeric keypad '=' key.      161
KEYCODE_NUMPAD_LEFT_PAREN   Numeric keypad '(' key.      162
KEYCODE_NUMPAD_MULTIPLY      Numeric keypad '*' key      155
KEYCODE_NUMPAD_RIGHT_PAREN   Numeric keypad ')' key.      163
KEYCODE_NUMPAD_SUBTRACT      Numeric keypad '-' key      156
KEYCODE_NUMPAD_ENTER      Numeric keypad Enter key.   160
KEYCODE_NUM_LOCK      Numeric keypad Num Lock key.   143


KEYCODE_MEDIA_FAST_FORWARD   Fast Forward media key.      90
KEYCODE_MEDIA_NEXT      Play Next media key.      87
KEYCODE_MEDIA_PAUSE      Pause media key.      127
KEYCODE_MEDIA_PLAY      Play media key.         126
KEYCODE_MEDIA_PLAY_PAUSE   Play/Pause media key.      85
KEYCODE_MEDIA_PREVIOUS      Play Previous media key.   88
KEYCODE_MEDIA_RECORD      Record media key.      130
KEYCODE_MEDIA_REWIND      Rewind media key.      89
KEYCODE_MEDIA_STOP      Stop media key.         86
]]
```

### 返回桌面

```lua
import "android.content.Intent"
home=Intent(Intent.ACTION_MAIN);
home.addCategory(Intent.CATEGORY_HOME);
activity.startActivity(home);
```

### 关于textSize与TextSize的单位问题

```lua
--[[

教程向代码
建议点击下方“运行”箭头运行代码观看

]]

require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

activity.ActionBar.hide()

layout={
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="fill",
  background="#FFFFFFFF";
  {
    TextView,
    text="关于textSize与TextSize的单位问题",
    layout_width="-1",
    layout_height="56dp",
    textSize="20sp";
    gravity="left|center";
    paddingLeft="16dp";
    background="#FFFFFFFF";
    textColor="#212121";
    maxLines=1;
  },
  {
    ScrollView,
    layout_width="fill",
    layout_height="fill",
    {
      LinearLayout,
      orientation="vertical",
      layout_width="fill",
      layout_height="fill",
      padding="8dp";

      {
        TextView,
        text="textSize 14dp MLua手册qwq",
        layout_width="-1",
        layout_height="-2",
        textSize="14dp";
      },
      {
        TextView,
        text="textSize 14sp MLua手册qwq",
        layout_width="-1",
        layout_height="-2",
        textSize="14sp";
      },
      {
        TextView,
        text="textSize 14 MLua手册qwq",
        layout_width="-1",
        layout_height="-2",
        textSize=14;
      },


      {
        TextView,
        text="TextSize 14dp MLua手册qwq",
        layout_width="-1",
        layout_height="-2",
        TextSize="14dp";
      },
      {
        TextView,
        text="TextSize 14sp MLua手册qwq",
        layout_width="-1",
        layout_height="-2",
        TextSize="14sp";
      },
      {
        TextView,
        text="TextSize 14 MLua手册qwq",
        layout_width="-1",
        layout_height="-2",
        TextSize=14;
      },

      {
        TextView,
        text="\n通过上面的对比我们可以看到，当使用textSize时,dp、sp和无单位的显示效果是相同的；当使用TextSize时,dp和sp会变成特大，而不带单位则和上面的textSize显示效果相同。\n\n来自MUK的建议：\n在我们的工程中最好使用textSize+sp，这样在大部分机型都不会出现布局错位及字体特大的情况。",
        layout_width="-1",
        layout_height="-2",
        textSize=14;
      },
    },
  };
}

activity.setContentView(loadlayout(layout))

--Coolapk：MLua手册
```

### when的用法

```lua
--when就相当于简化过的if，对于一些简单的判断使用when会爽不少
--例如下面这个栗子

--以前判断要这样写
if 5>1 then
  print("OK")
end
--在androlua+4.3.3更新后可以这样写
when 5>1 print("OK")

--以前判断要这样写
if 1>5 then
  print("emmmm")
 else
  print("OK")
end
--在androlua+4.3.3更新后可以这样写
when 1>5 print("emmmm") else print("OK")
```

### defer关键字

```lua
--[[
延时执行
还有自动回收
可以在error时执行
    --@ninenr语录
]]

function test()
  print(1)
  print(2)
  print(3)
  print(4)
end
test()--运行这个函数，可以看到由上至下打印出1234

function test()
  defer print(1)
  defer print(2)
  print(3)
  defer print(4)
end
test()--运行这个函数，可以看到打印出了3421
--说明代码运行顺序是先运行无defer的，然后有defer的从下往上运行
```

### LuaWebView自定义进度条

```lua
--需要web.dex，稍微有点麻烦，详见视频教程

--删除进度条
web.removeView(web.getChildAt(0))
--导包
import "com.lua.*"
--进度改变事件
web.setWebChromeClient(LuaWebChrome(LuaWebChrome.IWebChrine{
  onProgressChanged=function(view, newProgress)
    --事件
  end,
}));
```

### 字符串保留URL

```lua
string.gkeepUrl=function(str)
  local strurltab={}
  for i,v in string.gfind(str,"https?://[-A-Za-z0-9+&@#/%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|]") do
    strurltab[#strurltab+1]=string.sub(str,i,v)
  end
  return strurltab
end
--返回的是table，该方式支持保留多个链接

--调用示例
str="MLuaForum https://www.mukapp.top/ Lua优化性能小结 https://www.mukapp.top/?thread-19.htm"

print(dump(str:gkeepUrl()))

local str1=""
for i,v in ipairs(string.gkeepUrl(str)) do
  str1=str1..v.." "
end
print(str1)
```

### 字符串保留与过滤中文

```lua
--有中文符号会乱码
string.filterChinese=function(str)return string.gsub(str,"[\u4e00-\u9fa5]+","")end
string.keepChinese=function(str)return string.gsub(str,"[^\u4e00-\u9fa5]+","")end

--调用示例
str="MLua手册是一个全新的Androlua+的手册"
--过滤中文
print(string.filterChinese(str))
print(str:filterChinese())
--保留中文
print(string.keepChinese(str))
print(str:keepChinese())
```

### LuaWebView设置UA

```lua
import "android.webkit.WebSettings"

local webSettings = LuaWebViewID.getSettings();
local newUserAgent = "UA字符串";
webSettings.setUserAgentString(newUserAgent);
```

### 获取与设置cookie

```lua
import "android.webkit.CookieSyncManager"
import "android.webkit.CookieManager"

function 设置Cookie(context,url,content)
  CookieSyncManager.createInstance(context)
  local cookieManager = CookieManager.getInstance()
  cookieManager.setAcceptCookie(true)
  cookieManager.removeSessionCookie()
  cookieManager.removeAllCookie()
  cookieManager.setCookie(url, content)
  CookieSyncManager.getInstance().sync()
end

function 获取Cookie(url)
  local cookieManager = CookieManager.getInstance();
  return cookieManager.getCookie(url);
end

--示例
--获取https://www.baidu.com的cookie并打印
print(获取Cookie("https://www.baidu.com"))
--设置https://www.baidu.com的cookie为This is cookie
设置Cookie(activity,"https://www.baidu.com","This is cookie")
--获取https://www.baidu.com的cookie并打印
print(获取Cookie("https://www.baidu.com"))
```

### 获取控件图片

```lua
import "android.graphics.Bitmap"

function 获取控件图片(view)
  local linearParams = view.getLayoutParams()
  local vw=linearParams.width
  local linearParams = view.getLayoutParams()
  local vh=linearParams.height
  view.setDrawingCacheEnabled(true)
  view.layout(0,0,vw,vh)
  return Bitmap.createBitmap(view.getDrawingCache())
end

--调用示例
--布局
layout={
  LinearLayout,
  layout_width="-1",
  layout_height="-1",
  orientation="vertical";
  {
    TextView,
    textSize="14sp";
    layout_width="56dp",
    layout_height="42dp",
    text="TextView1",
    textColor="#FFFFFFFF";
    background="#212121",
    id="tv";
  },
  {
    ImageView,
    layout_width="-1",
    layout_height="-1",
    id="img";
  },
}
--设置布局
activity.setContentView(loadlayout(layout))
--获取tv的图片设置给img
img.setImageBitmap(获取控件图片(tv))
```

### Json解析示例

```lua
--导入
JSON=import "json"

--json字符串
json_str=[==[
[
    {
		"title": "第一本书",
		"bookId": "book_1"
	},
	{
		"title": "第二本书",
		"bookId": "book_2"
	}
]
]==]

--解析json
json_o=JSON.decode(json_str)

--打印table
print(dump(json_o))

--遍历打印table
for i,v in ipairs(json_o) do
  print(v.title,v.bookId)
end
```

### 使用系统TTS播报语音

```lua
import "android.speech.tts.*"

mTextSpeech = TextToSpeech(activity, TextToSpeech.OnInitListener{
  onInit=function(status)
    --如果装载TTS成功
    if (status == TextToSpeech.SUCCESS)
      result = mTextSpeech.setLanguage(Locale.CHINESE);
      --[[LANG_MISSING_DATA-->语言的数据丢失
          LANG_NOT_SUPPORTED-->语言不支持]]
      if (result == TextToSpeech.LANG_MISSING_DATA or result == TextToSpeech.LANG_NOT_SUPPORTED)
        --不支持中文
        print("您的手机不支持中文语音播报功能。");
        result = mTextSpeech.setLanguage(Locale.ENGLISH);
        if (result == TextToSpeech.LANG_MISSING_DATA or result == TextToSpeech.LANG_NOT_SUPPORTED)
          --不支持中文和英文
          print("您的手机不支持语音播报功能。");
         else
          --不支持中文但支持英文
          --语调,1.0默认
          mTextSpeech.setPitch(1);
          --语速,1.0默认
          mTextSpeech.setSpeechRate(1);
          mTextSpeech.speak("hello,MLua Manual.Hello,World!", TextToSpeech.QUEUE_FLUSH, nil);
        end
       else
        --支持中文
        --语调,1.0默认
        mTextSpeech.setPitch(1);
        --语速,1.0默认
        mTextSpeech.setSpeechRate(1);
        mTextSpeech.speak("你好，MLua手册。你好，世界！", TextToSpeech.QUEUE_FLUSH, nil);
      end
    end
  end
});
```

### MLua手册代码页底栏贝塞尔曲线

```lua
import "com.androlua.LuaDrawable"
import "android.graphics.Path"
import "android.graphics.Paint"

function dp2px(dpValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return dpValue * scale + 0.5
end

function px2dp(pxValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return pxValue / scale + 0.5
end

function px2sp(pxValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity;
  return pxValue / scale + 0.5
end

function sp2px(spValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return spValue * scale + 0.5
end

layout={
  RelativeLayout;
  layout_width="-1";
  background=backgroundc;
  layout_height="-1";
  {
    RelativeLayout,
    layout_width="-1",
    layout_height="-1",
    id="llb",
    gravity="bottom";
    {
      RelativeLayout,
      layout_width="fill",
      layout_height="56dp",
      clickable="true",
      id="ll",
      {
        LinearLayout;
        layout_width="-1";
        layout_height="-1";
        gravity="left|center";
        paddingLeft="8dp";
        paddingRight="8dp";
        {
          LinearLayout;
          layout_height="-1";
          layout_width="-2";
          layout_weight="1";
        };
        {
          TextView;
          layout_height="-1";
          layout_width="56dp";
          layout_marginRight="20dp";
          layout_marginLeft="8dp";
          --background=grayc;
        };
      };
    };
  },
  {
    LinearLayout;
    layout_width="-1";
    layout_height="-1";
    orientation="vertical";
    gravity="right|bottom";
    {
      CardView;
      layout_width="56dp",
      layout_height="56dp",
      radius="28dp";
      layout_margin="28dp";
      CardBackgroundColor="0xffffffff";
      elevation="6dp";
      alpha=1;
    };
  };

}

activity.setContentView(loadlayout(layout))

myLuaDrawable=LuaDrawable(function(mCanvas,mPaint,mDrawable)

  mPaint.setColor(0xffffffff)
  mPaint.setAntiAlias(true)
  mPaint.setStrokeWidth(20)
  mPaint.setStyle(Paint.Style.FILL)
  mPaint.setStrokeCap(Paint.Cap.ROUND)

  w=mDrawable.getBounds().right
  h=mDrawable.getBounds().bottom

  mPath=Path()

  mPath.moveTo(w, h);
  mPath.lineTo(0, h);
  mPath.lineTo(0, h-dp2px(56));

  mPath.lineTo(w-dp2px(56+16+16+8), h-dp2px(56));
  mPath.rQuadTo(dp2px(8), dp2px(0),dp2px(8+1), dp2px(8))
  mPath.rCubicTo(dp2px(8-1), dp2px(28+4),dp2px(56-1), dp2px(28+4),dp2px(56+8-2), dp2px(0))
  mPath.rQuadTo(dp2px(1), dp2px(-8),dp2px(8+1), dp2px(-8))
  mPath.rLineTo(w, 0);

  mCanvas.drawColor(0x00000000)
  mCanvas.drawPath(mPath, mPaint);

  mPath.close();
end)

ll.background=myLuaDrawable

myLuaDrawable=LuaDrawable(function(mCanvas,mPaint,mDrawable)

  mPaint.setColor(0x21000000)
  mPaint.setAntiAlias(true)
  mPaint.setStrokeWidth(dp2px(4))
  mPaint.setStyle(Paint.Style.FILL)
  mPaint.setStrokeCap(Paint.Cap.ROUND)

  w=mDrawable.getBounds().right
  h=mDrawable.getBounds().bottom

  mPath=Path()

  mPath.moveTo(w, h);
  mPath.lineTo(0, h);
  mPath.lineTo(0, h-dp2px(56));

  mPath.lineTo(w-dp2px(56+16+16+8), h-dp2px(56));
  mPath.rQuadTo(dp2px(8), dp2px(0),dp2px(8+1), dp2px(8))
  mPath.rCubicTo(dp2px(8-1), dp2px(28+4),dp2px(56-1), dp2px(28+4),dp2px(56+8-2), dp2px(0))
  mPath.rQuadTo(dp2px(1), dp2px(-8),dp2px(8+1), dp2px(-8))
  mPath.rLineTo(w, 0);

  mCanvas.drawColor(0x00000000)
  mPaint.setShadowLayer(dp2px(1), 0, dp2px(-1), 0x70FFFFFF);

  mCanvas.drawPath(mPath, mPaint);

  mPath.close();
end)

llb.background=myLuaDrawable
```

### 适配异形屏的全屏

```lua
function 全屏()
  window = activity.getWindow();
  window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_FULLSCREEN|View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
  window.addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN)
  xpcall(function()
    lp = window.getAttributes();
    lp.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES;
    window.setAttributes(lp);
  end,
  function(e)
  end)
end
--使用该代码可能需要隐藏ActionBar

--调用示例
全屏()
```

### 圆形图片

```lua
function 圆形图片(bitmap)
  import "android.graphics.PorterDuffXfermode"
  import "android.graphics.Paint"
  import "android.graphics.RectF"
  import "android.graphics.Bitmap"
  import "android.graphics.PorterDuff$Mode"
  import "android.graphics.Rect"
  import "android.graphics.Canvas"
  import "android.util.Config"
  width = bitmap.getWidth()
  output = Bitmap.createBitmap(width, bitmap.getHeight(),Bitmap.Config.ARGB_8888)
  canvas = Canvas(output);
  color = 0xff424242;
  paint = Paint()
  rect = Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
  rectF = RectF(rect);
  paint.setAntiAlias(true);
  canvas.drawARGB(0, 0, 0, 0);
  paint.setColor(color);
  canvas.drawRoundRect(rectF, width/2, bitmap.getHeight()/2, paint);
  paint.setXfermode(PorterDuffXfermode(Mode.SRC_IN));
  canvas.drawBitmap(bitmap, rect, rect, paint);
  return output;
end

--调用示例
img=ImageView(activity)
activity.setContentView(img)
img.setImageBitmap(圆形图片(loadbitmap("https://image.uisdc.com/wp-content/uploads/2019/06/uisdc-banner-20190614-2.jpg")))
```

### 字符串MD5

```lua
function MD5(str)
  local HexTable = {"0","1","2","3","4","5","6","7","8","9","A","B","C","D","E","F"}
  local A = 0x67452301
  local B = 0xefcdab89
  local C = 0x98badcfe
  local D = 0x10325476

  local S11 = 7
  local S12 = 12
  local S13 = 17
  local S14 = 22
  local S21 = 5
  local S22 = 9
  local S23 = 14
  local S24 = 20
  local S31 = 4
  local S32 = 11
  local S33 = 16
  local S34 = 23
  local S41 = 6
  local S42 = 10
  local S43 = 15
  local S44 = 21

  local function F(x,y,z)
    return (x & y) | ((~x) & z)
  end
  local function G(x,y,z)
    return (x & z) | (y & (~z))
  end
  local function H(x,y,z)
    return x ~ y ~ z
  end
  local function I(x,y,z)
    return y ~ (x | (~z))
  end
  local function FF(a,b,c,d,x,s,ac)
    a = a + F(b,c,d) + x + ac
    a = (((a & 0xffffffff) << s) | ((a & 0xffffffff) >> 32 - s)) + b
    return a & 0xffffffff
  end
  local function GG(a,b,c,d,x,s,ac)
    a = a + G(b,c,d) + x + ac
    a = (((a & 0xffffffff) << s) | ((a & 0xffffffff) >> 32 - s)) + b
    return a & 0xffffffff
  end
  local function HH(a,b,c,d,x,s,ac)
    a = a + H(b,c,d) + x + ac
    a = (((a & 0xffffffff) << s) | ((a & 0xffffffff) >> 32 - s)) + b
    return a & 0xffffffff
  end
  local function II(a,b,c,d,x,s,ac)
    a = a + I(b,c,d) + x + ac
    a = (((a & 0xffffffff) << s) | ((a & 0xffffffff) >> 32 - s)) + b
    return a & 0xffffffff
  end



  local function MD5StringFill(s)
    local len = s:len()
    local mod512 = len * 8 % 512
    --需要填充的字节数
    local fillSize = (448 - mod512) // 8
    if mod512 > 448 then
      fillSize = (960 - mod512) // 8
    end

    local rTab = {}

    --记录当前byte在4个字节的偏移
    local byteIndex = 1
    for i = 1,len do
      local index = (i - 1) // 4 + 1
      rTab[index] = rTab[index] or 0
      rTab[index] = rTab[index] | (s:byte(i) << (byteIndex - 1) * 8)
      byteIndex = byteIndex + 1
      if byteIndex == 5 then
        byteIndex = 1
      end
    end
    --先将最后一个字节组成4字节一组
    --表示0x80是否已插入
    local b0x80 = false
    local tLen = #rTab
    if byteIndex ~= 1 then
      rTab[tLen] = rTab[tLen] | 0x80 << (byteIndex - 1) * 8
      b0x80 = true
    end

    --将余下的字节补齐
    for i = 1,fillSize // 4 do
      if not b0x80 and i == 1 then
        rTab[tLen + i] = 0x80
       else
        rTab[tLen + i] = 0x0
      end
    end

    --后面加原始数据bit长度
    local bitLen = math.floor(len * 8)
    tLen = #rTab
    rTab[tLen + 1] = bitLen & 0xffffffff
    rTab[tLen + 2] = bitLen >> 32

    return rTab
  end

  --	Func:	计算MD5
  --	Param:	string
  --	Return:	string
  ---------------------------------------------

  function string.md5(s)
    --填充
    local fillTab = MD5StringFill(s)
    local result = {A,B,C,D}

    for i = 1,#fillTab // 16 do
      local a = result[1]
      local b = result[2]
      local c = result[3]
      local d = result[4]
      local offset = (i - 1) * 16 + 1
      --第一轮
      a = FF(a, b, c, d, fillTab[offset + 0], S11, 0xd76aa478)
      d = FF(d, a, b, c, fillTab[offset + 1], S12, 0xe8c7b756)
      c = FF(c, d, a, b, fillTab[offset + 2], S13, 0x242070db)
      b = FF(b, c, d, a, fillTab[offset + 3], S14, 0xc1bdceee)
      a = FF(a, b, c, d, fillTab[offset + 4], S11, 0xf57c0faf)
      d = FF(d, a, b, c, fillTab[offset + 5], S12, 0x4787c62a)
      c = FF(c, d, a, b, fillTab[offset + 6], S13, 0xa8304613)
      b = FF(b, c, d, a, fillTab[offset + 7], S14, 0xfd469501)
      a = FF(a, b, c, d, fillTab[offset + 8], S11, 0x698098d8)
      d = FF(d, a, b, c, fillTab[offset + 9], S12, 0x8b44f7af)
      c = FF(c, d, a, b, fillTab[offset + 10], S13, 0xffff5bb1)
      b = FF(b, c, d, a, fillTab[offset + 11], S14, 0x895cd7be)
      a = FF(a, b, c, d, fillTab[offset + 12], S11, 0x6b901122)
      d = FF(d, a, b, c, fillTab[offset + 13], S12, 0xfd987193)
      c = FF(c, d, a, b, fillTab[offset + 14], S13, 0xa679438e)
      b = FF(b, c, d, a, fillTab[offset + 15], S14, 0x49b40821)

      --第二轮
      a = GG(a, b, c, d, fillTab[offset + 1], S21, 0xf61e2562)
      d = GG(d, a, b, c, fillTab[offset + 6], S22, 0xc040b340)
      c = GG(c, d, a, b, fillTab[offset + 11], S23, 0x265e5a51)
      b = GG(b, c, d, a, fillTab[offset + 0], S24, 0xe9b6c7aa)
      a = GG(a, b, c, d, fillTab[offset + 5], S21, 0xd62f105d)
      d = GG(d, a, b, c, fillTab[offset + 10], S22, 0x2441453)
      c = GG(c, d, a, b, fillTab[offset + 15], S23, 0xd8a1e681)
      b = GG(b, c, d, a, fillTab[offset + 4], S24, 0xe7d3fbc8)
      a = GG(a, b, c, d, fillTab[offset + 9], S21, 0x21e1cde6)
      d = GG(d, a, b, c, fillTab[offset + 14], S22, 0xc33707d6)
      c = GG(c, d, a, b, fillTab[offset + 3], S23, 0xf4d50d87)
      b = GG(b, c, d, a, fillTab[offset + 8], S24, 0x455a14ed)
      a = GG(a, b, c, d, fillTab[offset + 13], S21, 0xa9e3e905)
      d = GG(d, a, b, c, fillTab[offset + 2], S22, 0xfcefa3f8)
      c = GG(c, d, a, b, fillTab[offset + 7], S23, 0x676f02d9)
      b = GG(b, c, d, a, fillTab[offset + 12], S24, 0x8d2a4c8a)

      --第三轮
      a = HH(a, b, c, d, fillTab[offset + 5], S31, 0xfffa3942)
      d = HH(d, a, b, c, fillTab[offset + 8], S32, 0x8771f681)
      c = HH(c, d, a, b, fillTab[offset + 11], S33, 0x6d9d6122)
      b = HH(b, c, d, a, fillTab[offset + 14], S34, 0xfde5380c)
      a = HH(a, b, c, d, fillTab[offset + 1], S31, 0xa4beea44)
      d = HH(d, a, b, c, fillTab[offset + 4], S32, 0x4bdecfa9)
      c = HH(c, d, a, b, fillTab[offset + 7], S33, 0xf6bb4b60)
      b = HH(b, c, d, a, fillTab[offset + 10], S34, 0xbebfbc70)
      a = HH(a, b, c, d, fillTab[offset + 13], S31, 0x289b7ec6)
      d = HH(d, a, b, c, fillTab[offset + 0], S32, 0xeaa127fa)
      c = HH(c, d, a, b, fillTab[offset + 3], S33, 0xd4ef3085)
      b = HH(b, c, d, a, fillTab[offset + 6], S34, 0x4881d05)
      a = HH(a, b, c, d, fillTab[offset + 9], S31, 0xd9d4d039)
      d = HH(d, a, b, c, fillTab[offset + 12], S32, 0xe6db99e5)
      c = HH(c, d, a, b, fillTab[offset + 15], S33, 0x1fa27cf8)
      b = HH(b, c, d, a, fillTab[offset + 2], S34, 0xc4ac5665)

      --第四轮
      a = II(a, b, c, d, fillTab[offset + 0], S41, 0xf4292244)
      d = II(d, a, b, c, fillTab[offset + 7], S42, 0x432aff97)
      c = II(c, d, a, b, fillTab[offset + 14], S43, 0xab9423a7)
      b = II(b, c, d, a, fillTab[offset + 5], S44, 0xfc93a039)
      a = II(a, b, c, d, fillTab[offset + 12], S41, 0x655b59c3)
      d = II(d, a, b, c, fillTab[offset + 3], S42, 0x8f0ccc92)
      c = II(c, d, a, b, fillTab[offset + 10], S43, 0xffeff47d)
      b = II(b, c, d, a, fillTab[offset + 1], S44, 0x85845dd1)
      a = II(a, b, c, d, fillTab[offset + 8], S41, 0x6fa87e4f)
      d = II(d, a, b, c, fillTab[offset + 15], S42, 0xfe2ce6e0)
      c = II(c, d, a, b, fillTab[offset + 6], S43, 0xa3014314)
      b = II(b, c, d, a, fillTab[offset + 13], S44, 0x4e0811a1)
      a = II(a, b, c, d, fillTab[offset + 4], S41, 0xf7537e82)
      d = II(d, a, b, c, fillTab[offset + 11], S42, 0xbd3af235)
      c = II(c, d, a, b, fillTab[offset + 2], S43, 0x2ad7d2bb)
      b = II(b, c, d, a, fillTab[offset + 9], S44, 0xeb86d391)

      --加入到之前计算的结果当中
      result[1] = result[1] + a
      result[2] = result[2] + b
      result[3] = result[3] + c
      result[4] = result[4] + d
      result[1] = result[1] & 0xffffffff
      result[2] = result[2] & 0xffffffff
      result[3] = result[3] & 0xffffffff
      result[4] = result[4] & 0xffffffff
    end

    --将Hash值转换成十六进制的字符串
    local retStr = ""
    for i = 1,4 do
      for _ = 1,4 do
        local temp = result[i] & 0x0F
        local str = HexTable[temp + 1]
        result[i] = result[i] >> 4
        temp = result[i] & 0x0F
        retStr = retStr .. HexTable[temp + 1] .. str
        result[i] = result[i] >> 4
      end
    end

    return retStr
  end

  return string.md5(str)
end

--调用示例
print(MD5("MLua手册"))
```

### 带颜色的字体

```lua
import "android.text.SpannableString"
import "android.text.style.ForegroundColorSpan"
import "android.text.Spannable"
function 颜色字体(t,c)
  local sp = SpannableString(t)
  sp.setSpan(ForegroundColorSpan(c),0,#sp,Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
  return sp
end

--调用示例
activity.setTitle(颜色字体("MLua手册 qwq",0xff2196f3))--设置标题为0xff2196f3颜色的 MLua手册 qwq
```

### 判断有无悬浮窗权限

```lua
import "android.provider.Settings"
function 判断悬浮窗权限()
  if (Build.VERSION.SDK_INT >= 23 and not Settings.canDrawOverlays(this)) then
    return false
   elseif Build.VERSION.SDK_INT < 23 then
    return nil
   else
    return true
  end
end
--[[
Build.VERSION.SDK_INT >= 23 是因为安卓6.0以下没有统一判断悬浮窗权限的方法
当安卓版本大于6.0且有悬浮窗权限时返回true
当安卓版本大于6.0且无悬浮窗权限时返回false
当安卓版本小于6.0时无法判断返回nil
]]

--调用例子
print(判断悬浮窗权限())
```

### 获取悬浮窗权限

```lua
import "android.net.Uri"
import "android.content.Intent"
import "android.provider.Settings"
function 获取悬浮窗权限()
  intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
  intent.setData(Uri.parse("package:" .. activity.getPackageName()));
  activity.startActivityForResult(intent, 100);
end

--调用示例
获取悬浮窗权限()
```

### 高斯模糊

```lua
import "android.renderscript.Element"
import "android.renderscript.Allocation"
import "android.renderscript.RenderScript"
import "android.graphics.Bitmap"
import "android.renderscript.ScriptIntrinsicBlur"
import "android.graphics.Matrix"

function 高斯模糊(id,tp,radius1,radius2)
  function blur( context, bitmap, blurRadius)
    renderScript = RenderScript.create(context);
    blurScript = ScriptIntrinsicBlur.create(renderScript, Element.U8_4(renderScript));
    inAllocation = Allocation.createFromBitmap(renderScript, bitmap);
    outputBitmap = bitmap;
    outAllocation = Allocation.createTyped(renderScript, inAllocation.getType());
    blurScript.setRadius(blurRadius);
    blurScript.setInput(inAllocation);
    blurScript.forEach(outAllocation);
    outAllocation.copyTo(outputBitmap);
    inAllocation.destroy();
    outAllocation.destroy();
    renderScript.destroy();
    blurScript.destroy();
    return outputBitmap;
  end
  function zoomBitmap(bitmap,scale)
    w = bitmap.getWidth();
    h = bitmap.getHeight();
    matrix = Matrix();
    matrix.postScale(scale, scale);
    bitmap = Bitmap.createBitmap(bitmap, 0, 0, w, h, matrix, true);
    return bitmap;
  end
  function blurAndZoom(context,bitmap,blurRadius,scale)
    return zoomBitmap(blur(context,zoomBitmap(bitmap, 1 / scale), blurRadius), scale);
  end
  id.setImageBitmap(blurAndZoom(activity,tp,radius1,radius2))
end
--[[
高斯模糊(id,tp,radius1,radius2)
radius1 范围：1-25
radius2 范围：1-？(据图片而定太大报错)
]]

--调用例子
img=ImageView(activity)
activity.setContentView(img)

高斯模糊(img,loadbitmap("https://image.uisdc.com/wp-content/uploads/2019/06/uisdc-banner-20190614-2.jpg"),4,2)
```

### 打开QQ名片/QQ群名片

```lua
import "android.net.Uri"
import "android.content.Intent"

function QQ群(h)
  url="mqqapi://card/show_pslcard?src_type=internal&version=1&uin="..h.."&card_type=group&source=qrcode"
  activity.startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))
end

function QQ(h)
  url="mqqapi://card/show_pslcard?src_type=internal&source=sharecard&version=1&uin="..h
  activity.startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))
end

--调用
QQ群("686976850")
QQ("1773798610")
```

### 控件设置点击波纹背景

```lua
import "android.content.res.ColorStateList"
function 波纹(id,lx,color)
  xpcall(function()
    ripple = activity.obtainStyledAttributes({android.R.attr.selectableItemBackgroundBorderless}).getResourceId(0,0)
    ripples = activity.obtainStyledAttributes({android.R.attr.selectableItemBackground}).getResourceId(0,0)
    for index,content in ipairs(id) do
      if lx=="圆" then
      content.setBackgroundDrawable(activity.Resources.getDrawable(ripple).setColor(ColorStateList(int[0].class{int{}},int{color})))
      end
      if lx=="方" then
        content.setBackgroundDrawable(activity.Resources.getDrawable(ripples).setColor(ColorStateList(int[0].class{int{}},int{color})))
      end
    end
  end,function(e)end)
end
--[[
波纹(id,lx,color)
id：控件id,table
lx：波纹类型,圆或方,string
color 波纹颜色,number

安卓5及以上可用。
该代码需要MD主题。
]]

--调用例子
layout={
  LinearLayout;
  onClick=function()print("MLua手册")end;
  layout_width="-1";
  layout_height="-1";
  id="lay";
}
activity.setContentView(loadlayout(layout))
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)
波纹({lay},"圆",0x21000000)
```

### 设置activity背景颜色

```lua
function activity背景颜色(color)
  _window = activity.getWindow();
  _window.setBackgroundDrawable(ColorDrawable(color));
  _wlp = _window.getAttributes();
  _wlp.gravity = Gravity.BOTTOM;
  _wlp.width = WindowManager.LayoutParams.MATCH_PARENT;
  _wlp.height = WindowManager.LayoutParams.MATCH_PARENT;--WRAP_CONTENT
  _window.setAttributes(_wlp);
end
--该函数需设置布局后使用

--调用例子
activity.setContentView(LinearLayout(activity))
activity背景颜色(0xff424242)
```

### dp、px、sp之间的转换

```lua
function dp2px(dpValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return dpValue * scale + 0.5
end

function px2dp(pxValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return pxValue / scale + 0.5
end

function px2sp(pxValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity;
  return pxValue / scale + 0.5
end

function sp2px(spValue)
  local scale = activity.getResources().getDisplayMetrics().scaledDensity
  return spValue * scale + 0.5
end

--调用例子
print(dp2px(64))
```

### 调用浏览器搜索关键字

```lua
import "android.content.Intent"
import "android.app.SearchManager"
intent =  Intent()
intent.setAction(Intent.ACTION_WEB_SEARCH)
intent.putExtra(SearchManager.QUERY,"Alua开发手册")
activity.startActivity(intent)
```

### 调用浏览器打开网页

```lua
import "android.content.Intent"
import "android.net.Uri"
url="http://www.androlua.cn"
viewIntent =  Intent("android.intent.action.VIEW",Uri.parse(url))
activity.startActivity(viewIntent)
```

### 打开其它程序

```lua
packageName=程序包名
import "android.content.Intent"
import "android.content.pm.PackageManager"
manager = activity.getPackageManager()
open = manager.getLaunchIntentForPackage(packageName)
this.startActivity(open)
```

### 安装其它程序

```lua
import "android.content.Intent"
import "android.net.Uri"
intent = Intent(Intent.ACTION_VIEW)
安装包路径="/sdcard/a.apk"
intent.setDataAndType(Uri.parse("file://"..安装包路径), "application/vnd.android.package-archive")
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
activity.startActivity(intent)
```

### 卸载其它程序

```lua
import "android.net.Uri"
import "android.content.Intent"
包名="com.huluxia.gametools"
uri = Uri.parse("package:"..包名)
intent =  Intent(Intent.ACTION_DELETE,uri)
activity.startActivity(intent)
```

### 播放Mp4

```lua
import "android.content.Intent"
import "android.net.Uri"
intent =  Intent(Intent.ACTION_VIEW)
uri = Uri.parse("file:///sdcard/a.mp4")
intent.setDataAndType(uri, "video/mp4")
activity.startActivity(intent)
```

### 播放Mp3

```lua
import "android.content.Intent"
import "android.net.Uri"
intent =  Intent(Intent.ACTION_VIEW)
uri = Uri.parse("file:///sdcard/song.mp3")
intent.setDataAndType(uri, "audio/mp3")
this.startActivity(intent)
```

### 搜索应用

```lua
import "android.content.Intent"
import "android.net.Uri"
intent = Intent("android.intent.action.VIEW")
intent .setData(Uri.parse( "market://details?id="..activity.getPackageName()))
this.startActivity(intent)
```

### 调用系统设置

```lua
import "android.content.Intent"
import "android.provider.Settings"
intent = Intent(Settings.ACTION_BLUETOOTH_SETTINGS)
this.startActivity(intent)
--[[
原代码：
intent = Intent(android.provider.Settings.ACTION_SETTINGS)
this.startActivity(intent)
19-10-03修正错误]]

字段列表:
ACTION_SETTINGS	系统设置
CTION_APN_SETTINGS APN设置
ACTION_LOCATION_SOURCE_SETTINGS 位置和访问信息
ACTION_WIRELESS_SETTINGS 网络设置
ACTION_AIRPLANE_MODE_SETTINGS 无线和网络热点设置
ACTION_SECURITY_SETTINGS 位置和安全设置
ACTION_WIFI_SETTINGS 无线网WIFI设置
ACTION_WIFI_IP_SETTINGS 无线网IP设置
ACTION_BLUETOOTH_SETTINGS 蓝牙设置
ACTION_DATE_SETTINGS 时间和日期设置
ACTION_SOUND_SETTINGS 声音设置
ACTION_DISPLAY_SETTINGS 显示设置——字体大小等
ACTION_LOCALE_SETTINGS 语言设置
ACTION_INPUT_METHOD_SETTINGS 输入法设置
ACTION_USER_DICTIONARY_SETTINGS 用户词典
ACTION_APPLICATION_SETTINGS 应用程序设置
ACTION_APPLICATION_DEVELOPMENT_SETTINGS 应用程序设置
ACTION_QUICK_LAUNCH_SETTINGS 快速启动设置
ACTION_MANAGE_APPLICATIONS_SETTINGS 已下载（安装）软件列表
ACTION_SYNC_SETTINGS 应用程序数据同步设置
ACTION_NETWORK_OPERATOR_SETTINGS 可用网络搜索
ACTION_DATA_ROAMING_SETTINGS 移动网络设置
ACTION_INTERNAL_STORAGE_SETTINGS 手机存储设置
```

### 调用系统打开文件

```lua
function OpenFile(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  if Mime then
    intent = Intent()
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
    intent.setAction(Intent.ACTION_VIEW);
    intent.setDataAndType(Uri.fromFile(File(path)), Mime);
    activity.startActivity(intent)
return true
  else
    return false
  end
end
OpenFile(文件路径)
```

### 调用图库选择图片

```lua
import "android.content.Intent"
  local intent= Intent(Intent.ACTION_PICK)
  intent.setType("image/*")
  this.startActivityForResult(intent, 1)
-------

--回调
function onActivityResult(requestCode,resultCode,intent)
  if intent then
    local cursor =this.getContentResolver ().query(intent.getData(), nil, nil, nil, nil)
    cursor.moveToFirst()
import "android.provider.MediaStore"
    local idx = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA)
    fileSrc = cursor.getString(idx)
    bit=nil
    --fileSrc回调路径路径
import "android.graphics.BitmapFactory"
    bit =BitmapFactory.decodeFile(fileSrc)
  --  iv.setImageBitmap(bit)
  end
end--nirenr
```

### 调用文件管理器选择文件

```lua
function ChooseFile()
import "android.content.Intent"
import "android.net.Uri"
import "java.net.URLDecoder"
import "java.io.File"
intent = Intent(Intent.ACTION_GET_CONTENT)
intent.setType("*/");
intent.addCategory(Intent.CATEGORY_OPENABLE)
activity.startActivityForResult(intent,1);
function onActivityResult(requestCode,resultCode,data)
  if resultCode == Activity.RESULT_OK then
  local str = data.getData().toString()
  local decodeStr = URLDecoder.decode(str,"UTF-8")
  print(decodeStr)
  end
end
end

ChooseFile()
```

### 分享文件

```lua
function Sharing(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  intent = Intent()
  intent.setAction(Intent.ACTION_SEND)
  intent.setType(Mime)
  file = File(path)
  uri = Uri.fromFile(file)
  intent.putExtra(Intent.EXTRA_STREAM,uri)
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
  activity.startActivity(Intent.createChooser(intent, "分享到:"))
end

Sharing(文件路径)
```

### 发送短信

```lua
import "android.net.Uri"
import "android.content.Intent"
uri = Uri.parse("smsto:10010")
intent = Intent(Intent.ACTION_SENDTO, uri)
intent.putExtra("sms_body","cxll")
intent.setAction("android.intent.action.VIEW")
activity.startActivity(intent)
```

### 发送彩信

```lua
import "android.net.Uri"
import "android.content.Intent"
uri=Uri.parse("file:///sdcard/a.png") --图片路径
intent= Intent();
intent.setAction(Intent.ACTION_SEND);
intent.putExtra("address",mobile) --邮件地址
intent.putExtra("sms_body",content) --邮件内容
intent.putExtra(Intent.EXTRA_STREAM,uri)
intent.setType("image/png") --设置类型
this.startActivity(intent)
```

### 拨打电话

```lua
import "android.net.Uri"
import "android.content.Intent"
uri = Uri.parse("tel:10010")
intent = Intent(Intent.ACTION_CALL, uri)
intent.setAction("android.intent.action.VIEW")
activity.startActivity(intent)
```

### 打印

```lua
print"Hello World！"
print("Hello World")
```

### 注释

```text
单行注释  --
多行注释  --[[]]
```

### 字符串

```lua
a="String"
a=[[String]]
a=[===[String]===]
```

### 赋值

```lua
a="Hello World"

--lua支持多重赋值
a,b="String a","String b"

--交换值
a,b="String a","String b"
a,b=b,a
```

### For循环

```lua
--给定条件进行循环

--输出从1到10
for i=1,10 do
print(i)
end

--输出从10到1
for i=10,1,-1 do
print(i)
end

--打印数组a中所有的值
a={"a","b","c","d"}
for index,content in pairs(a) do
print(content)
end
```

### While循环

```lua
--只要条件为真便会一直循环下去

--输出1到10
a=0
while a~=10 do
a=a+1
print(a)
end

--输出10到1
a=11
while a~=1 do
a=a-1
print(a)
end

--打印数组a中的所有值
shuzu={"a","b","c","d"}
a=0
while a~=#shuzu do
a=a+1
print(shuzu[a])
end
```

### if(判断语句)

```lua
--判断值是否为真
a=true
if a then
print("真")
else
print("假")
end

--比较值是否相同
a=true
b=false
if a==b then
print("真")
else
print("假")
end
```

### function(函数)

```lua
函数有两个用途
1.完成指定功能，函数作为调用语句使用
2.计算并返回值，函数作为赋值语句的表达式使用

实例1:
function 读取文件(路径)
文件内容=io.open(路径):read("*a")
return 文件内容--return用来返回值
end

实例2:
require "import"
import "android.widget.EditText"
import "android.widget.LinearLayout"
function 编辑框()
return EditText(activity)
end
layout={
  LinearLayout;
  id="父布局",
  {编辑框,
    id="edit",
    text="文本",
   },
};
activity.setContentView(loadlayout(layout))
--把这段代码放到调试里面去测试
```

### 基础代码

```lua
activity.setTitle('Title')--设置窗口标题
activity.setContentView(loadlayout(layout))--设置窗口视图
activity.setTheme(android.R.style.Theme_DeviceDefault_Light)--设置主题
activity.getWidth()--获取屏幕宽
activity.getHeight()--获取屏幕高
activity.newActivity("main")--跳转页面
activity.finish()--关闭当前页面
activity.recreate()--重构activity
os.exit()--结束程序
tostring()--转换字符串
tonumber()--转换数字
tointeger()--转换整数
--线程
--thread
thread(function()print"线程"end)
--task
task(function()print"线程"end)
```

### 获取设备标识码

```lua
import "android.provider.Settings$Secure"
android_id = Secure.getString(activity.getContentResolver(), Secure.ANDROID_ID)
```

### 获取IMEI

```lua
import "android.content.Context"
imei=activity.getSystemService(Context.TELEPHONY_SERVICE).getDeviceId()
```

### 控件背景渐变动画

```lua
view=控件id
color1 = 0xffFF8080;
color2 = 0xff8080FF;
color3 = 0xff80ffff;
color4 = 0xff80ff80;
import "android.animation.ObjectAnimator"
import "android.animation.ArgbEvaluator"
import "android.animation.ValueAnimator"
import "android.graphics.Color"
colorAnim = ObjectAnimator.ofInt(view,"backgroundColor",{color1, color2, color3,color4})
colorAnim.setDuration(3000)
colorAnim.setEvaluator(ArgbEvaluator())
colorAnim.setRepeatCount(ValueAnimator.INFINITE)
colorAnim.setRepeatMode(ValueAnimator.REVERSE)
colorAnim.start()
```

### 精准获取屏幕尺寸

```lua
function getScreenPhysicalSize(ctx)
  import "android.util.DisplayMetrics"
  dm = DisplayMetrics();
  ctx.getWindowManager().getDefaultDisplay().getMetrics(dm);
  diagonalPixels = Math.sqrt(Math.pow(dm.widthPixels, 2) + Math.pow(dm.heightPixels, 2));
  return diagonalPixels / (160 * dm.density);
end
print(getScreenPhysicalSize(activity))
```

### 发送邮件

```lua
import "android.content.Intent"
i = Intent(Intent.ACTION_SEND)
i.setType("message/rfc822")
i.putExtra(Intent.EXTRA_EMAIL, {"2113075983@.com"})
i.putExtra(Intent.EXTRA_SUBJECT,"Feedback")
i.putExtra(Intent.EXTRA_TEXT,"Content")
activity.startActivity(Intent.createChooser(i, "Choice"))
```

### 自定义默认弹窗标题,消息,按钮的颜色

```lua
dialog=AlertDialog.Builder(this)
.setTitle("标题")
.setMessage("消息")
.setPositiveButton("积极",{onClick=function(v) print"点击了积极按钮"end})
.setNeutralButton("中立",nil)
.setNegativeButton("否认",nil)
.show()
dialog.create()

--更改消息颜色
message=dialog.findViewById(android.R.id.message)
message.setTextColor(0xff1DA6DD)

--更改Button颜色
import "android.graphics.Color"
dialog.getButton(dialog.BUTTON_POSITIVE).setTextColor(0xff1DA6DD)
dialog.getButton(dialog.BUTTON_NEGATIVE).setTextColor(0xff1DA6DD)
dialog.getButton(dialog.BUTTON_NEUTRAL).setTextColor(0xff1DA6DD)

--更改Title颜色
import "android.text.SpannableString"
import "android.text.style.ForegroundColorSpan"
import "android.text.Spannable"
sp = SpannableString("标题")
sp.setSpan(ForegroundColorSpan(0xff1DA6DD),0,#sp,Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
dialog.setTitle(sp)
```

### 获取手机存储空间

```lua
--获取手机内置剩余存储空间
 function GetSurplusSpace()
 fs =  StatFs(Environment.getDataDirectory().getPath())
 return Formatter.formatFileSize(activity, (fs.getAvailableBytes()))
 end

 --获取手机内置存储总空间
 function GetTotalSpace()
 path = Environment.getExternalStorageDirectory()
 stat = StatFs(path.getPath())
 blockSize = stat.getBlockSize()
 totalBlocks = stat.getBlockCount()
 return Formatter.formatFileSize(activity, blockSize * totalBlocks)
 end
```

### 获取视频第一帧

```lua
function GetVideoFrame(path)
  import "android.media.MediaMetadataRetriever"
  media = MediaMetadataRetriever()
  media.setDataSource(tostring(path))
  return media.getFrameAtTime()
end
```

### 选择文件模块

```lua
import "android.widget.ArrayAdapter"
import "android.widget.LinearLayout"
import "android.widget.TextView"
import "java.io.File"
import "android.widget.ListView"
import "android.app.AlertDialog"
function ChoiceFile(StartPath,callback)
  --创建ListView作为文件列表
  lv=ListView(activity).setFastScrollEnabled(true)
  --创建路径标签
  cp=TextView(activity)
  lay=LinearLayout(activity).setOrientation(1).addView(cp).addView(lv)
  ChoiceFile_dialog=AlertDialog.Builder(activity)--创建对话框
  .setTitle("选择文件")
  .setView(lay)
  .show()
  adp=ArrayAdapter(activity,android.R.layout.simple_list_item_1)
  lv.setAdapter(adp)
  function SetItem(path)
    path=tostring(path)
    adp.clear()--清空适配器
    cp.Text=tostring(path)--设置当前路径
    if path~="/" then--不是根目录则加上../
      adp.add("../")
    end
    ls=File(path).listFiles()
    if ls~=nil then
      ls=luajava.astable(File(path).listFiles()) --全局文件列表变量
      table.sort(ls,function(a,b)
        return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.Name<b.Name)
      end)
    else
      ls={}
    end
    for index,c in ipairs(ls) do
      if c.isDirectory() then--如果是文件夹则
        adp.add(c.Name.."/")
      else--如果是文件则
        adp.add(c.Name)
      end
    end
  end
  lv.onItemClick=function(l,v,p,s)--列表点击事件
    项目=tostring(v.Text)
    if tostring(cp.Text)=="/" then
      路径=ls[p+1]
    else
      路径=ls[p]
    end

    if 项目=="../" then
      SetItem(File(cp.Text).getParentFile())
    elseif 路径.isDirectory() then
      SetItem(路径)
    elseif 路径.isFile() then
      callback(tostring(路径))
      ChoiceFile_dialog.hide()
    end

  end

  SetItem(StartPath)
end

--ChoiceFile(StartPath,callback)
--第一个参数为初始化路径,第二个为回调函数
--原创
```

### 选择路径模块

```lua
require "import"
import "android.widget.ArrayAdapter"
import "android.widget.LinearLayout"
import "android.widget.TextView"
import "java.io.File"
import "android.widget.ListView"
import "android.app.AlertDialog"
function ChoicePath(StartPath,callback)
  --创建ListView作为文件列表
  lv=ListView(activity).setFastScrollEnabled(true)
  --创建路径标签
  cp=TextView(activity)
  lay=LinearLayout(activity).setOrientation(1).addView(cp).addView(lv)
  ChoiceFile_dialog=AlertDialog.Builder(activity)--创建对话框
  .setTitle("选择路径")
  .setPositiveButton("OK",{
  onClick=function()
  callback(tostring(cp.Text))
  end})
.setNegativeButton("Canel",nil)
  .setView(lay)
  .show()
  adp=ArrayAdapter(activity,android.R.layout.simple_list_item_1)
  lv.setAdapter(adp)
  function SetItem(path)
    path=tostring(path)
    adp.clear()--清空适配器
    cp.Text=tostring(path)--设置当前路径
    if path~="/" then--不是根目录则加上../
      adp.add("../")
    end
    ls=File(path).listFiles()
    if ls~=nil then
      ls=luajava.astable(File(path).listFiles()) --全局文件列表变量
      table.sort(ls,function(a,b)
        return (a.isDirectory()~=b.isDirectory() and a.isDirectory()) or ((a.isDirectory()==b.isDirectory()) and a.Name<b.Name)
      end)
    else
      ls={}
    end
    for index,c in ipairs(ls) do
      if c.isDirectory() then--如果是文件夹则
        adp.add(c.Name.."/")
      end
    end
  end
  lv.onItemClick=function(l,v,p,s)--列表点击事件
    项目=tostring(v.Text)
    if tostring(cp.Text)=="/" then
      路径=ls[p+1]
    else
      路径=ls[p]
    end

    if 项目=="../" then
      SetItem(File(cp.Text).getParentFile())
    elseif 路径.isDirectory() then
      SetItem(路径)
    elseif 路径.isFile() then
      callback(tostring(路径))
      ChoiceFile_dialog.hide()
    end

  end

  SetItem(StartPath)
end

import "android.os.*"
ChoicePath(Environment.getExternalStorageDirectory().toString(),
function(path)
print(path)
end)

--第一个参数为初始化路径,第二个为回调函数
--原创
```

### 获取视图中的所有文本

```lua
function GetAllText(view)
textTable={}
function GetText(Parent)
local number=Parent.getChildCount()
for i=0,number do
local view=Parent.getChildAt(i)
if pcall(function()view.addView(TextView(activity))end) then
GetText(view)
elseif pcall(function()view.getText()end) then
table.insert(textTable,tostring(view.Text))
end
end
end
GetText(view)
return textTable
end

print(table.unpack(GetAllText(Parent)))
```

### 控件圆角

```lua
function CircleButton(view,InsideColor,radiu)
  import "android.graphics.drawable.GradientDrawable"
  drawable = GradientDrawable()
  drawable.setShape(GradientDrawable.RECTANGLE)
  drawable.setColor(InsideColor)
  drawable.setCornerRadii({radiu,radiu,radiu,radiu,radiu,radiu,radiu,radiu});
  view.setBackgroundDrawable(drawable)
end
角度=50
控件id=ed
控件颜色=0xFF09639C
CircleButton(控件id,控件颜色,角度)
```

### 匹配汉字

```lua
function filter_spec_chars(s)
	local ss = {}
	for k = 1, #s do
		local c = string.byte(s,k)
		if not c then break end
		if (c>=48 and c<=57) or (c>= 65 and c<=90) or (c>=97 and c<=122) then
			if not string.char(c):find("%w") then
   table.insert(ss, string.char(c))
	end
 	elseif c>=228 and c<=233 then
			local c1 = string.byte(s,k+1)
			local c2 = string.byte(s,k+2)
			if c1 and c2 then
				local a1,a2,a3,a4 = 128,191,128,191
				if c == 228 then a1 = 184
				elseif c == 233 then a2,a4 = 190,c1 ~= 190 and 191 or 165
				end
				if c1>=a1 and c1<=a2 and c2>=a3 and c2<=a4 then
					k = k + 2
					table.insert(ss, string.char(c,c1,c2))
				end
			end
		end
	end
	return table.concat(ss)
end
print(filter_spec_chars("A1B2汉C3D4字E5F6,,,"))
--来源网络,加了个if过滤掉英文与数字,使其只捕获中文
```

### 播放音乐与视频

```lua
import "android.media.MediaPlayer"
mediaPlayer =  MediaPlayer()

--初始化参数
mediaPlayer.reset()

--设置播放资源
mediaPlayer.setDataSource("storage/sdcard0/a.mp3")

--开始缓冲资源
mediaPlayer.prepare()

--是否循环播放该资源
mediaPlayer.setLooping(true)

--缓冲完成的监听
mediaPlayer.setOnPreparedListener(MediaPlayer.OnPreparedListener() {
    onPrepared=function(mediaPlayer
        mediaPlayer.start()
   end});

--是否在播放
mediaPlayer.isPlaying()

--暂停播放
mediaPlayer.pause()

--从30位置开始播放
mediaPlayer.seekTo(30)

--停止播放
mediaPlayer.stop()


--播放视频
--视频的播放与音乐播放过程一样：

--先创建一个媒体对象
import "android.media.MediaPlayer"
mediaPlayer =  MediaPlayer()
--初始化参数
mediaPlayer.reset()

--设置播放资源
mediaPlayer.setDataSource("storage/sdcard0/a.mp4")

--拿到显示的SurfaceView
sh = surfaceView.getHolder()
sh.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS)

--设置显示SurfaceView
mediaPlayer.setDisplay(sh)

--设置音频流格式
mediaPlayer.setAudioStreamType(AudioManager.Stream_Music)

--开始缓冲资源
mediaPlayer.prepare()

--缓冲完成的监听
mediaPlayer.setOnPreparedListener(MediaPlayer.OnPreparedListener{
   onPrepared=function(mediaPlayer)
		--开始播放
        mediaPlayer.start()
   end
});

--释放播放器
mediaPlayer.release()

--非原创
```

### 获取系统SDK，Android版本及设备型号

```lua
device_model = Build.MODEL --设备型号

version_sdk = Build.VERSION.SDK --设备SDK版本

version_release = Build.VERSION.RELEASE --设备的系统版本
```

### 控件颜色修改

```lua
import "android.graphics.PorterDuffColorFilter"
import "android.graphics.PorterDuff"

--修改按钮颜色
button.getBackground().setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))

--修改编辑框颜色
edittext.getBackground().setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP));

--修改Switch颜色
switch.ThumbDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP));
switch.TrackDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))

--修改ProgressBar颜色
progressbar.IndeterminateDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))

--修改SeekBar滑条颜色
seekbar.ProgressDrawable.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))
--修改SeekBar滑块颜色
seekbar.Thumb.setColorFilter(PorterDuffColorFilter(0xFFFB7299,PorterDuff.Mode.SRC_ATOP))
```

### 修改对话框按钮颜色

```lua
function DialogButtonFilter(dialog,button,WidgetColor)
if Build.VERSION.SDK_INT >= 21 then
import "android.graphics.PorterDuffColorFilter"
import "android.graphics.PorterDuff"
if button==1 then
dialog.getButton(dialog.BUTTON_POSITIVE).setTextColor(WidgetColor)
elseif button==2 then
dialog.getButton(dialog.BUTTON_NEGATIVE).setTextColor(WidgetColor)
elseif button==3 then
dialog.getButton(dialog.BUTTON_NEUTRAL).setTextColor(WidgetColor)
end
end
end
--第一个参数为对话框的变量
--第二个参数为1时，则修改POSITIVE按钮颜色,为二则修改NEGATIVE按钮颜色,为三则修改NEUTRAL按钮颜色
--第三个参数为要修改成的颜色
```

### 查询本地所有视频

```lua
function QueryAllVideo()
import "android.provider.MediaStore"
cursor = activity.ContentResolver
mImageUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
mCursor = cursor.query(mImageUri,nil,nil,nil,MediaStore.Video.Media.DATE_TAKEN)
mCursor.moveToLast()
VideoTable={}
while mCursor.moveToPrevious() do
   path = mCursor.getString(mCursor.getColumnIndex(MediaStore.Video.Media.DATA))
   table.insert(VideoTable,tostring(path))
end
mCursor.close()
return VideoTable
end
--返回一个表
```

### 查询本地所有图片

```lua
function QueryAllImage()
import "android.provider.MediaStore"
cursor = activity.ContentResolver
mImageUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
mCursor = cursor.query(mImageUri,nil,nil,nil,MediaStore.Images.Media.DATE_TAKEN)
mCursor.moveToLast()
imageTable={}
while mCursor.moveToPrevious() do
   path = mCursor.getString(mCursor.getColumnIndex(MediaStore.Images.Media.DATA))
   table.insert(imageTable,tostring(path))
end
mCursor.close()
return imageTable
end
--返回一个表
```

### 递归查找文件

```lua
function outPath(ret)
for i,p in pairs(luajava.astable(ret)) do
print(p)
end
end
function find(catalog,name)
 local n=0
 local t=os.clock()
 local ret={}
 require "import"
 import "java.io.File"
 import "java.lang.String"
 function FindFile(catalog,name)
   local name=tostring(name)
   local ls=catalog.listFiles() or File{}
   for 次数=0,#ls-1 do
     --local 目录=tostring(ls[次数])
     local f=ls[次数]
     if f.isDirectory() then--如果是文件夹则继续匹配
       FindFile(f,name)
     else--如果是文件则
       n=n+1
       if n%1000==0 then
         print(n,os.clock()-t)
       end
      local nm=f.Name
       if string.find(nm,name) then
         --thread(insert,目录)
         table.insert(ret,tostring(f))
       end
     end
   luajava.clear(f)
   end
 end
 FindFile(catalog,name)
 call("outPath",ret)
end

import "java.io.File"

catalog=File("/sdcard/AndroLua")
name=".j?pn?g"
thread(find,catalog,name)
```

### 获取手机内置存储路径

```text
Environment.getExternalStorageDirectory().toString()
```

### 获取已安装程序的包名、版本号、最后更新时间、图标、应用名称

```lua
function GetAppInfo(包名)
  import "android.content.pm.PackageManager"
  local pm = activity.getPackageManager();
  local 图标 = pm.getApplicationInfo(tostring(包名),0)
  local 图标 = 图标.loadIcon(pm);
  local pkg = activity.getPackageManager().getPackageInfo(包名, 0);
  local 应用名称 = pkg.applicationInfo.loadLabel(activity.getPackageManager())
  local 版本号 = activity.getPackageManager().getPackageInfo(包名, 0).versionName
  local 最后更新时间 = activity.getPackageManager().getPackageInfo(包名, 0).lastUpdateTime
  local cal = Calendar.getInstance();
  cal.setTimeInMillis(最后更新时间);
  local 最后更新时间 = cal.getTime().toLocaleString()
  return 包名,版本号,最后更新时间,图标,应用名称
end
```

### 获取指定安装包的包名,图标,应用名

```lua
import "android.content.pm.PackageManager"
import "android.content.pm.ApplicationInfo"
function GetApkInfo(archiveFilePath)
pm = activity.getPackageManager()
info = pm.getPackageArchiveInfo(archiveFilePath, PackageManager.GET_ACTIVITIES);
if info ~= nil then
  appInfo = info.applicationInfo;
 appName = tostring(pm.getApplicationLabel(appInfo))
  packageName = appInfo.packageName; --安装包名称
  version=info.versionName; --版本信息
   icon = pm.getApplicationIcon(appInfo);--图标
end
return packageName,version,icon
end
```

### 获取某程序是否安装

```lua
if pcall(function() activity.getPackageManager().getPackageInfo("包名",0) end) then
  print("已安装")
else
  print("未安装")
end
```

### 设置TextView字体风格

```lua
import "android.graphics.Paint"
--设置中划线
id.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG)
--设置下划线
id.getPaint().setFlags(Paint. UNDERLINE_TEXT_FLAG )
--设置加粗
id.getPaint().setFakeBoldText(true)
--设置斜体
id.getPaint().setTextSkewX(0.2)

--设置TypeFace
import "android.graphics.Typeface"
id.getPaint().setTypeface()
--参数列表
Typeface.DEFAULT 默认字体
Typeface.DEFAULT_BOLD 加粗字体
Typeface.MONOSPACE monospace字体
Typeface.SANS_SERIF sans字体
Typeface.SERIF serif字体
```

### 缩放图片

```lua
function rotateToFit(bm,degrees)
    import "android.graphics.Matrix"
    import "android.graphics.Bitmap"
    width = bm.getWidth()
    height = bm.getHeight()
    matrix =  Matrix()
    matrix.postRotate(degrees)
    bmResult = Bitmap.createBitmap(bm, 0, 0, width, height, matrix, true)
    return bmResult
  end
bm=loadbitmap(图片路径)
缩放级别=2
rotateToFit(bm,degrees)
--非原创
```

### 获取运营商名称

```lua
import "android.content.Context"
运营商名称 = this.getSystemService(Context.TELEPHONY_SERVICE).getNetworkOperatorName()
print(运营商名称)
--添加权限   READ_PHONE_STATE
```

### Drawable着色

```lua
function ToColor(path,color)
 local  aa=BitmapDrawable(loadbitmap(tostring(path)))
   aa.setColorFilter(PorterDuffColorFilter(color,PorterDuff.Mode.SRC_ATOP))
return aa
end
```

### 保存图片到本地

```lua
function SavePicture(name,bm)
if  bm then
import "java.io.FileOutputStream"
import "java.io.File"
import "android.graphics.Bitmap"
name=tostring(name)
f = File(name)
out = FileOutputStream(f)
bm.compress(Bitmap.CompressFormat.PNG,90, out)
out.flush()
out.close()
return true
else
return false
end
end
```

### 调用应用商店搜索应用

```lua
import "android.content.Intent"
import "android.net.Uri"
intent = Intent("android.intent.action.VIEW")
intent .setData(Uri.parse( "market://details?id="..activity.getPackageName()))
this.startActivity(intent)
```

### 分享

```lua
--分享文件
function Sharing(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  intent = Intent();
  intent.setAction(Intent.ACTION_SEND);
  intent.setType(Mime);
  file = File(path);
  uri = Uri.fromFile(file);
  intent.putExtra(Intent.EXTRA_STREAM,uri);
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
  activity.startActivity(Intent.createChooser(intent, "分享到:"));
  end

--分享文字
text="分享的内容"
intent=Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_SUBJECT, "分享");
intent.putExtra(Intent.EXTRA_TEXT, text);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
activity.startActivity(Intent.createChooser(intent,"分享到:"));
```

### 调用其它程序打开文件

```lua
function OpenFile(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  if Mime then
    intent = Intent();
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    intent.setAction(Intent.ACTION_VIEW);
    intent.setDataAndType(Uri.fromFile(File(path)), Mime);
    activity.startActivity(intent);
  else
    Toastc("找不到可以用来打开此文件的程序")
  end
end
```

### 图片圆角

```lua
function GetRoundedCornerBitmap(bitmap,roundPx)
  import "android.graphics.PorterDuffXfermode"
  import "android.graphics.Paint"
  import "android.graphics.RectF"
  import "android.graphics.Bitmap"
  import "android.graphics.PorterDuff$Mode"
  import "android.graphics.Rect"
  import "android.graphics.Canvas"
  import "android.util.Config"
  width = bitmap.getWidth()
  output = Bitmap.createBitmap(width, width,Bitmap.Config.ARGB_8888)
  canvas = Canvas(output);
  color = 0xff424242;
  paint = Paint()
  rect = Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
  rectF = RectF(rect);
  paint.setAntiAlias(true);
  canvas.drawARGB(0, 0, 0, 0);
  paint.setColor(color);
  canvas.drawRoundRect(rectF, roundPx, roundPx, paint);
  paint.setXfermode(PorterDuffXfermode(Mode.SRC_IN));
  canvas.drawBitmap(bitmap, rect, rect, paint);
  return output;
end
import "android.graphics.drawable.BitmapDrawable"
圆角弧度=50
bitmap=loadbitmap(picturePath)
RoundPic=GetRoundedCornerBitmap(bitmap)
```

### 发送短信

```lua
--后台发送短信
 require "import"
 import "android.telephony.*"
 SmsManager.getDefault().sendTextMessage(tostring(号码), nil, tostring(内容), nil, nil)

--调用系统发送短信
import "android.content.Intent"
import "android.net.Uri"
uri = Uri.parse("smsto:"..号码)
intent = Intent(Intent.ACTION_SENDTO, uri)
intent.putExtra("sms_body",内容)
intent.setAction("android.intent.action.VIEW")
activity.startActivity(intent)
```

### 判断数组中是否存在某个值

```lua
function Table_exists(tables,value)
for index,content in pairs(tables) do
if content:find(value) then
return true
end
end
end
```

### 字符串操作

```lua
strings="左中右"

--取字符串左边
左=strings:match("(.+)中")

--取字符串中间
中=strings:match("左(.-)右")

--取字符串右边
右=strings:match("(.+)右")

--替换
string.gsub(原字符串,替换的字符串,替换成的字符串)

--匹配子串位置
起始位置,结束位置=string.find(字符串,子串)

--按位置捕获字符串
string.sub(字符串,子串起始位置,子串结束位置)
```

### 剪切板操作

```lua
import "android.content.Context"
--导入类

a=activity.getSystemService(Context.CLIPBOARD_SERVICE).getText()
--获取剪贴板

activity.getSystemService(Context.CLIPBOARD_SERVICE).setText(edit.Text)
--写入剪贴板
```

### 各种事件

```lua
function main(...)
  --...是newActivity传递过来的参数。
  print("入口函数",...)
end

function onCreate()
  print("窗口创建")
end

function onStart()
  print("活动开始")
end

function onResume()
  print("返回程序")
end

function onPause()
  print("活动暂停")
end

function onStop()
  print("活动停止")
end

function onDestroy()
  print("程序已退出")
end

function onResult(name,...)
  --name：返回的活动名称
  --...：返回的参数
  print("返回活动",name,...)
end

function onCreateOptionsMenu(menu)
  --menu：选项菜单。
  menu.add("菜单")
end

function onOptionsItemSelected(item)
  --item：选中的菜单项
  print(item.Title)
end

function onConfigurationChanged(config)
  --config：配置信息
  print("屏幕方向关闭")
end

function onKeyDown(keycode,event)
  --keycode：键值
  --event：事件
  print("按键按下",keycode)
end

function onKeyUp(keycode,event)
  --keycode：键值
  --event：事件
  print("按键抬起",keycode)
end

function onKeyLongPress(keycode,event)
  --keycode：键值
  --event：事件
  print("按键长按",keycode)
end

function onTouchEvent(event)
  --event：事件
  print("触摸事件",event)
end

function onKeyDown(c,e)
  if c==4 then
--返回键事件
end
end

id.onClick=function()
--控件被单击
end

id.onLongClick=function()
--控件被长按
end

id.onItemClick=function(p,v,i,s)
--列表项目被单击
项目=v.Text
return true
end

id.onItemLongClick=function(p,v,i,s)
--列表项目被长按
项目=v.Text
return true
end

id.onItemLongClick=function(p,v,i,s)
--列表项目被长按
项目=v.Text
return true
end

--Spinner的项目单击事件
id.onItemSelected=function(l,v,p,i)
项目=v.Text
end

--ExpandableListView的父项目与子项目单击事件
id.onGroupClick=function(l,v,p,s)
  print(v.Text..":GroupClick")
end

id.onChildClick=function(l,v,g,c)
  print(v.Text..":ChildClick")
end
```

### Shell执行

```lua
function exec(cmd)
local p=io.popen(string.format('%s',cmd))
local s=p:read("*a")
p:close()
return s
end

print(exec("echo  ...."))

部分常用命令:
--删除文件或文件夹
rm -r /路径

--复制文件或文件夹
cp -r inpath outpath

--移动文件或文件夹
mv -r inpath outpath

--挂载系统目录
mount -o remount,rw path

--修改系统文件权限
chmod 755 /system/build.prop

--重启
reboot�

--关机
reboot -p

--重启至recovery
reboot recovery
```

### 创建新文件

```lua
--使用File类
import "java.io.File"--导入File类
File(文件路径).createNewFile()

--使用io库
io.open("/sdcard/aaaa", 'w')
```

### 创建新文件夹

```lua
--使用File类
import "java.io.File"--导入File类
File(文件夹路径).mkdir()

--创建多级文件夹
File(文件夹路径).mkdirs()

--shell
os.execute('mkdir '..文件夹路径)
```

### 重命名与移动文件

```lua
--Shell
os.execute("mv "..oldname.." "..newname)

--os
os.rename (oldname, newname)

--File
import "java.io.File"--导入File类
File(旧).renameTo(File(新))
```

### 追加更新文件

```text
io.open(文件路径,"a+"):write("更新的内容"):close()
```

### 更新文件

```text
io.open(文件路径,"w+"):write("更新的内容"):close()
```

### 写入文件

```text
io.open(文件路径,"w"):write("内容"):close()
```

### 写入文件(自动创建父文件夹)

```lua
function 写入文件(路径,内容)
  import "java.io.File"
  f=File(tostring(File(tostring(路径)).getParentFile())).mkdirs()
  io.open(tostring(路径),"w"):write(tostring(内容)):close()
end
```

### 读取文件

```text
io.open(文件路径):read("*a")
```

### 按行读取文件

```lua
for c in io.lines(文件路径) do
print(c)
end
```

### 删除文件或文件夹

```lua
--使用File类
import "java.io.File"--导入File类
File(文件路径).delete()
--使用os方法
os.remove (filename)
```

### 复制文件

```text
LuaUtil.copyDir(from,to)
```

### 递归删除文件夹或文件

```lua
--使用LuaUtil辅助库
LuaUtil.rmDir(路径)

--使用Shell
os.execute("rm -r "..路径)
```

### 替换文件内字符串

```lua
function 替换文件字符串(路径,要替换的字符串,替换成的字符串)
if 路径 then
  路径=tostring(路径)
  内容=io.open(路径):read("*a")
  io.open(路径,"w+"):write(tostring(内容:gsub(要替换的字符串,替换成的字符串))):close()
else
return false
end
end
```

### 获取文件列表

```text
import("java.io.File")
luajava.astable(File(文件夹路径).listFiles())
```

### 获取文件名称

```lua
import "java.io.File"--导入File类
File(路径).getName()
```

### 获取文件大小

```lua
function GetFileSize(path)
  import "java.io.File"
  import "android.text.format.Formatter"
  size=File(tostring(path)).length()
  Sizes=Formatter.formatFileSize(activity, size)
  return Sizes
end
```

### 获取文件或文件夹最后修改时间

```lua
function GetFilelastTime(path)
  f = File(path);
  cal = Calendar.getInstance();
  time = f.lastModified()
  cal.setTimeInMillis(time);
  return cal.getTime().toLocaleString()
end
```

### 获取文件字节

```lua
import "java.io.File"--导入File类
File(路径).length()
```

### 获取文件父文件夹路径

```lua
import "java.io.File"--导入File类
File(path).getParentFile()
```

### 获取文件Mime类型

```lua
function GetFileMime(name)
import "android.webkit.MimeTypeMap"
ExtensionName=tostring(name):match("%.(.+)")
Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
return tostring(Mime)
end
print(GetFileMime("/sdcard/a.png"))
```

### 判断路径是不是文件夹

```lua
import "java.io.File"--导入File类
File(路径).isDirectory()
--也可用来判断文件夹存不存在
```

### 判断路径是不是文件

```lua
import "java.io.File"--导入File类
File(路径).isFile()
--也可用来判断文件存不存在
```

### 判断文件或文件夹存不存在

```lua
import "java.io.File"--导入File类
File(路径).exists()

--使用io
function file_exists(path)
local f=io.open(path,'r')
if f~=nil then io.close(f) return true else return false end
end
```

### 判断是不是系统隐藏文件

```lua
import "java.io.File"--导入File类
File(路径).isHidden()
```

### 字符串操作

```lua
--字符串转大写
string.upper(字符串)
--字符串转小写
string.lower(字符串)
--字符串替换
string.gsub(字符串,被替换的字符,替换的字符,替换次数)
```

### 设置控件大小

```lua
--设置宽度
linearParams = 控件ID.getLayoutParams()
linearParams.width =宽度
控件ID.setLayoutParams(linearParams)
--同理设置高度
linearParams = 控件ID.getLayoutParams()
linearParams.height =高度
控件ID.setLayoutParams(linearParams)
```

### 载入窗口传参

```lua
activity.newActivity("窗口名",{参数})

--渐变动画效果的，中间是安卓跳转动画代码
activity.newActivity("窗口名",android.R.anim.fade_in,android.R.anim.fade_out,{参数})
```

### EditText只能输数字

```lua
import "android.text.InputType"
import "android.text.method.DigitsKeyListener"
控件ID.setInputType(InputType.TYPE_CLASS_NUMBER)
控件ID.setKeyListener(DigitsKeyListener.getInstance("0123456789"))
```

### 窗口全屏

```lua
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN)
```

### 关闭当前窗口

```lua
activity.finish()
```

### 按两次返回键退出

```lua
参数=0
function onKeyDown(code,event)
if string.find(tostring(event),"KEYCODE_BACK") ~= nil then
if 参数+2 > tonumber(os.time()) then
activity.finish()
else
 Toast.makeText(activity,"再按一次返回键退出" , Toast.LENGTH_SHORT )
.show()
参数=tonumber(os.time())
end
return true
end
end
```

### 取字符串中间

```text
string.match("左测试测试右","左(.-)右")
```

### 判断文件是否存在

```lua
--先导入io包
import "java.io.*"
file,err=io.open("路径")
print(err)
if err==nil then
print("存在")
else
print("不存在")
end
```

### 判断文件夹是否存在

```lua
--先导入io包
import "java.io.*"
if File(文件夹路径).isDirectory()then
print("存在")
else
print("不存在")
end
```

### 窗口回调事件

```lua
function onActivityResult()
--事件
end
```

### 隐藏标题栏

```lua
activity.ActionBar.hide()
```

### 自定义布局对话框

```lua
local dl=AlertDialog.Builder(activity)
.setTitle("自定义布局对话框")
.setView(loadlayout(layout))
dl.show()
```

### 列表下滑到最底事件

```lua
list.setOnScrollListener{
onScrollStateChanged=function(l,s)
if list.getLastVisiblePosition()==list.getCount()-1 then
--事件
end
end}
```

### 标题栏返回按钮

```lua
activity.getActionBar().setDisplayHomeAsUpEnabled(true)
```

### 列表长按事件

```lua
ID.setOnItemLongClickListener(AdapterView.OnItemLongClickListener{
onItemLongClick=function(parent, v, pos,id)
--事件
end
})
```

### 列表点击事件

```lua
ID.setOnItemClickListener(AdapterView.OnItemClickListener{
onItemClick=function(parent, v, pos,id)
--事件
end
})
```

### 关于V4的圆形下拉刷新

```lua
--设置下拉刷新监听事件
swipeRefreshLayout.setOnRefreshListener(this);
--设置进度条的颜色
swipeRefreshLayout.setColorSchemeColors(Color.RED, Color.BLUE, Color.GREEN);
--设置圆形进度条大小
swipeRefreshLayout.setSize(SwipeRefreshLayout.LARGE);
--设置进度条背景颜色
swipeRefreshLayout.setProgressBackgroundColorSchemeColor(Color.DKGRAY);
--设置下拉多少距离之后开始刷新数据
swipeRefreshLayout.setDistanceToTriggerSync(50);
```

### 对话框Dialog

```lua
--简单对话框
AlertDialog.Builder(this).setTitle("标题")
.setMessage("简单消息框")
.setPositiveButton("确定",nil)
.show();

--带有三个按钮的对话框
AlertDialog.Builder(this)
.setTitle("确认")
.setMessage("确定吗？")
.setPositiveButton("是",nil)
.setNegativeButton("否",nil)
.setNeutralButton("不知道",nil)
.show();

--带输入框的
AlertDialog.Builder(this)
.setTitle("请输入")
.setIcon(android.R.drawable.ic_dialog_info)
.setView(EditText(this))
.setPositiveButton("确定", nil)
.setNegativeButton("取消", nil)
.show();

--单选的
AlertDialog.Builder(this)
.setTitle("请选择")
.setIcon(android.R.drawable.ic_dialog_info)
.setSingleChoiceItems({"选项1","选项2","选项3","选项4"}, 0,
DialogInterface.OnClickListener() {
 onClick(dialog,which) {
dialog.dismiss();
 }
}
)
.setNegativeButton("取消", null)
.show();

--多选的
AlertDialog.Builder(this)
.setTitle("多选框")
.setMultiChoiceItems({"选项1","选项2","选项3","选项4"}, null, null)
.setPositiveButton("确定", null)
.setNegativeButton("取消", null)
.show();

--列表的
AlertDialog.Builder(this)
.setTitle("列表框")
.setItems({"列表项1","列表项2","列表项3"},nil)
.setNegativeButton("确定",nil)
.show();

--图片的
img = ImageView(this);
img.setImageResource(R.drawable.icon);
AlertDialog.Builder(this)
.setTitle("图片框")
.setView(img)
.setPositiveButton("确定",nil)
.show();
```

### 删除ListView中某项

```text
adp.remove(pos)
```

### 打开某APP

```lua
--导入包
import "android.content.*"

intent = Intent();
componentName = ComponentName("com.androlua","com.androlua.Welcome");
intent.setComponent(componentName);
activity.startActivity(intent);
```

### 设置横屏竖屏

```lua
--横屏
activity.setRequestedOrientation(0);
--竖屏
activity.setRequestedOrientation(1);
```

### 设置控件图片

```lua
--设置的图片也可以输入路径
ID.setImageBitmap(loadbitmap("图片.png"))
```

### 禁用编辑框

```lua
--代码中设置
editText.setFocusable(false);
--布局表中设置
Focusable=false;
```

### 隐藏滚动条

```lua
--横向
horizontalScrollBarEnabled=false;
--竖向
VerticalScrollBarEnabled=false;
```

### 图片着色

```lua
--代码中设置
ID.setColorFilter(0xffff0000)
--布局表中设置
ColorFilter="#ffff0000"；
```

### 获取IMEI号

```lua
import "android.content.*"
--导入包

imei=activity.getSystemService(Context.TELEPHONY_SERVICE).getDeviceId();
print(imei)

--别忘了添加权限"READ_PHONE_STATE"
```

### 分享文字

```lua
import "android.content.*"

text="分享的内容"
intent=Intent(Intent.ACTION_SEND);
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_SUBJECT, "分享");
intent.putExtra(Intent.EXTRA_TEXT, text);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
activity.startActivity(Intent.createChooser(intent,"分享到:"));
```

### 发送短信

```lua
--导入包
import "android.content.*"
import "android.net.*"

uri = Uri.parse("smsto:15800001234");
intent = Intent(Intent.ACTION_SENDTO, uri);
intent.putExtra("sms_body","你好")
intent.setAction("android.intent.action.VIEW");
activity.startActivity(intent);
```

### 拔号

```lua
import "android.content.*"
import "android.net.*"
--导入包
uri = Uri.parse("tel:15800001234");
intent = Intent(Intent.ACTION_CALL, uri);
intent.setAction("android.intent.action.VIEW");
activity.startActivity(intent);
--记得添加打电话权限
```

### 安装APK

```lua
import "android.content.*"
import "android.net.*"

intent = Intent(Intent.ACTION_VIEW);
intent.setDataAndType(Uri.parse("file:///sdcard/jc.apk"), "application/vnd.android.package-archive");
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
activity.startActivity(intent);
```

### 震动

```lua
import "android.content.Context"
--导入包
vibrator = activity.getSystemService(Context.VIBRATOR_SERVICE)
vibrator.vibrate( long{100,800} ,-1)
--{等待时间,振动时间,等待时间,振动时间,…}
--{0,1000,500,1000,500,1000}
--别忘了申明权限
```

### 获取剪贴板内容

```lua
import"android.content.*"
--导入包
a=activity.getSystemService(Context.CLIPBOARD_SERVICE).getText()
```

### 压缩成ZIP

```text
ZipUtil.zip("文件或文件夹路径","压缩到的路径")
```

### ZIP解压

```lua
ZipUtil.unzip("ZIP路径","解压到的路径")

--另一种Java方法
import "java.io.FileOutputStream"
import "java.util.zip.ZipFile"
import "java.io.File"

zipfile = "/sdcard/压缩包.zip"--压缩文件路径和文件名
sdpath = "/sdcard/文件.lua"--解压后路径和文件名
zipfilepath = "内容.lua"--需要解压的文件名

function unzip(zippath , outfilepath , filename)

local time=os.clock()
  task(function(zippath,outfilepath,filename)
require "import"
import "java.util.zip.*"
import "java.io.*"
local file = File(zippath)
local outFile = File(outfilepath)
local zipFile = ZipFile(file)
local entry = zipFile.getEntry(filename)
local input = zipFile.getInputStream(entry)
local output = FileOutputStream(outFile)
local byte=byte[entry.getSize()]
local temp=input.read(byte)
while temp ~= -1 do
output.write(byte)
temp=input.read(byte)
end
input.close()
output.close()
end,zippath,outfilepath,filename,
function()
print("解压完成，耗时 "..os.clock()-time.." s")
end)

end

unzip(zipfile,sdpath,zipfilepath)
```

### 删除文件夹

```lua
--shell命令的方法
os.execute("rm-r 路径")
```

### 重命名文件夹

```lua
--shell命令的方法
os.execute("mv 路径新路径")
```

### 创建文件夹

```lua
--shell命令的方法
os.execute("mkdir 路径")
```

### 删除文件

```text
os.remove("路径")
```

### 设置标题栏标题

```lua
--标题
activity.setTitle('标题')
--小标题
activity.getActionBar().setSubtitle('小标题')
```

### 获取Lua文件的执行路径

```lua
activity.getLuaDir()
```

### 获取本应用包名

```lua
activity.getPackageName()
```

### 布局设置点击效果

```lua
--5.0或以上可以实现点击水波纹效果
--在布局加入：

style="?android:attr/buttonBarButtonStyle";
```

### 判断某APP是否安装

```lua
if pcall(function() activity.getPackageManager().getPackageInfo("包名",0) end) then
print("安装了")
else
print("没安装")
end
```

### 调用系统下载

```lua
--导入包
import "android.content.Context"
import "android.net.Uri"

downloadManager=activity.getSystemService(Context.DOWNLOAD_SERVICE);
url=Uri.parse("绝对下载链接");
request=DownloadManager.Request(url);
request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_MOBILE|DownloadManager.Request.NETWORK_WIFI);
request.setDestinationInExternalPublicDir("目录名，可以是Download","下载的文件名");
request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
downloadManager.enqueue(request);
```

### 动画结束回调

```lua
--导入包
import "android.view.animation.*"
import "android.view.animation.Animation$AnimationListener"
--控件动画
控件.startAnimation(AlphaAnimation(1,0).setDuration(400).setFillAfter(true).setAnimationListener(AnimationListener{
onAnimationEnd=function()
print"动画结束")
end}))
```

### 关于侧滑

```lua
--侧滑布局是 DrawerLayout;
--关闭侧滑
ID.closeDrawer(3)
--打开侧滑
ID.openDrawer(3)
```

### 关于输入法影响布局的问题

```lua
--使弹出的输入法不影响布局
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);
--使弹出的输入法影响布局
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);
```

### TextView设置字体样式

```lua
--首先要导入包
import "android.graphics.*"
--设置中划线
控件id.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG)
--下划线
控件id.getPaint().setFlags(Paint. UNDERLINE_TEXT_FLAG )
--加粗
控件id.getPaint().setFakeBoldText(true)
--斜体
控件id.getPaint().setTextSkewX(0.2)

--设置TypeFace
import "android.graphics.Typeface"
id.getPaint().setTypeface(字体)
--字体可以为以下
Typeface.DEFAULT --默认字体
Typeface.DEFAULT_BOLD --加粗字体
Typeface.MONOSPACE --monospace字体
Typeface.SANS_SERIF --sans字体
Typeface.SERIF --serif字体
```

### 强制结束自身并清除自身数据

```text
 os.execute("pm clear "..activity.getPackageName())
```

### 递归搜索文件实例

```lua
require "import"

function find(catalog,name)
local n=0
local t=os.clock()
local ret={}
require "import"
import "java.io.File"
import "java.lang.String"
function FindFile(catalog,name)
local name=tostring(name)
local ls=catalog.listFiles() or File{}
for 次数=0,#ls-1 do
--local 目录=tostring(ls[次数])
local f=ls[次数]
if f.isDirectory() then--如果是文件夹则继续匹配
FindFile(f,name)
else--如果是文件则
n=n+1
if n%1000==0 then
--print(n,os.clock()-t)
end
local nm=f.Name
if string.find(nm,name) then
--thread(insert,目录)
table.insert(ret,nm)
print(nm)
end
end
luajava.clear(f)
end
end
FindFile(catalog,name)
print("ok",n,#ret)
end

import "java.io.File"

catalog=File("sdcard/")
name=".j?pn?g"
--task(find,catalog,name,print)
thread(find,catalog,name)
```

### 获取ListView垂直坐标

```lua
function getScrollY()
c = ls.getChildAt(0);
local firstVisiblePosition = ls.getFirstVisiblePosition();
local top = c.getTop();
return -top + firstVisiblePosition * c.getHeight() ;
end
```

### 申请root权限

```lua
--shell命令的方法
os.execute("su")
```

### 传感器

```lua
传感器 = activity.getSystemService(Context.SENSOR_SERVICE)

local 加速度传感器 = 传感器.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
x轴 = event.values[0]
y轴 = event.values[1]
z轴 = event.values[2]
end,nil}), 加速度传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 光线传感器 = 传感器.getDefaultSensor(Sensor.TYPE_LIGHT)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
光线 = event.values[0]
end,nil}), 光线传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 距离传感器 = 传感器.getDefaultSensor(Sensor.TYPE_PROXIMITY)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
距离 = event.values[0]
end,nil}), 距离传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 磁场传感器 = 传感器.getDefaultSensor(Sensor.TYPE_ORIENTATION)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
磁场 = event.values[0]
end,nil}), 磁场传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 温度传感器 = 传感器.getDefaultSensor(Sensor.TYPE_TEMPERATURE)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
温度 = event.values[0]
end,nil}), 温度传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 陀螺仪传感器 = 传感器.getDefaultSensor(Sensor.TYPE_GYROSCOPE)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
陀螺仪 = event.values[0]
end,nil}), 陀螺仪传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 重力传感器 = 传感器.getDefaultSensor(Sensor.TYPE_GRAVITY)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
重力 = event.values[0]
end,nil}), 重力传感器, SensorManager.SENSOR_DELAY_NORMAL)

local 压力传感器 = 传感器.getDefaultSensor(Sensor.TYPE_PRESSURE)
传感器.registerListener(SensorEventListener({
onSensorChanged=function(event)
压力 = event.values[0]
end,nil}), 压力传感器, SensorManager.SENSOR_DELAY_NORMAL)
```

### 获取控件宽高

```lua
--导入包
import "android.content.Context"

function getwh(view)
view.measure(View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED),View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED));
height =view.getMeasuredHeight();
width =view.getMeasuredWidth();
return width,height
end

print(getwh(控件ID))
```

### 播放音频

```lua
--导入包
import "android.media.MediaPlayer"

local 音频播放器=MediaPlayer()
function 播放音频(路径)
音频播放器.reset()
.setDataSource(路径)
.prepare()
.start()
.setOnCompletionListener({
onCompletion=function()
print("播放完毕")
end})
end
```

### 控件旋转

```lua
--Z轴上的旋转角度
View.getRotation()

--X轴上的旋转角度
View.getRotationX()

--Y轴上的旋转角度
View.getRotationY()

--设置Z轴上的旋转角度
View.setRotation(r)

--设置X轴上的旋转角度
View.setRotationX(r)

--设置Y轴上的旋转角度
View.setRotationY(r)

--设置旋转中心点的X坐标
View.setPivotX(p)

--设置旋转中心点的Y坐标
View.setPivotX(p)

--设置摄像机的与旋转目标在Z轴上距离
View.setCameraDistance(d)
```

### 标题栏(ActionBar)

```lua
--部分常用API
show:显示
hide:隐藏
Elevation:设置阴影
BgroundDrawable:设置背景
DisplayHomeAsUpEnabled(boolean):设置是否显示返回图标

--设置标题
activity.ActionBar.setTitle('大标题')
activity.ActionBar.setSubTitle("小标题")

--设置ActionBar背景颜色
import "android.graphics.drawable.ColorDrawable"
activity.ActionBar.setBackgroundDrawable(ColorDrawable(Color))

--自定义ActionBar标题颜色
import "android.text.SpannableString"
import "android.text.style.ForegroundColorSpan"
import "android.text.Spannable"
sp = SpannableString("标题")
sp.setSpan(ForegroundColorSpan(0xff1DA6DD),0,#sp,Spannable.SPAN_EXCLUSIVE_INCLUSIVE)
activity.ActionBar.setTitle(sp)

--自定义ActionBar布局
DisplayShowCustomEnabled(true)
CustomView(loadlayout(layout))

--ActionBar返回按钮
activity.ActionBar.setDisplayHomeAsUpEnabled(true)
--自定义返回按钮图标
activity.ActionBar.setHomeAsUpIndicator(drawable)

--菜单
function onCreateOptionsMenu(menu)
  menu.add("菜单1")
  menu.add("菜单2")
  menu.add("菜单3")
end
function onOptionsItemSelected(item)
  print("你选择了:"..item.Title)
end

--Tab导航使用
import "android.app.ActionBar$TabListener"
actionBar=activity.ActionBar
actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);
tab = actionBar.newTab().setText("Tab1").setTabListener(TabListener({
  onTabSelected=function()
    print"Tab1"
  end}))
tab2=actionBar.newTab().setText("Tab2").setTabListener(TabListener({
  onTabSelected=function()
    print"Tab2"
  end}))
actionBar.addTab(tab)
actionBar.addTab(tab2)
```

### 五大布局

```lua
--Android中常用的5大布局方式有以下几种：
--线性布局（LinearLayout）：按照垂直或者水平方向布局的组件。
--帧布局（FrameLayout）：组件从屏幕左上方布局组件。
--表格布局（TableLayout）：按照行列方式布局组件。
--相对布局（RelativeLayout）：相对其它组件的布局方式。
--绝对布局（AbsoluteLayout）：按照绝对坐标来布局组件。

1.线性布局(LinearLayout)
线性布局是Android开发中最常见的一种布局方式，它是按照垂直或者水平方向来布局，通过orientation属性可以设置线性布局的方向。属性值有垂直（vertical）和水平（horizontal）两种。
常用的属性：
orientation：可以设置布局的方向
gravity:用来控制组件的对齐方式
layout_weight控制各个控件在布局中的相对大小,layout_weight的属性是一个非负整数值。
线性布局会根据该控件layout_weight值与其所处布局中所有控件layout_weight值之和的比值为该控件分配占用的区域
--[[例如，在水平布局的LinearLayout中有两个Button，这两个Button的layout_weight属性值都为1,那么这两个按钮都会被拉伸到整个屏幕宽度的一半。如果layout_weight指为0，控件会按原大小显示，不会被拉伸.
对于其余layout_weight属性值大于0的控件，系统将会减去layout_weight属性值为0的控件的宽度或者高度,再用剩余的宽度或高度按相应的比例来分配每一个控件显示的宽度或高度]]

2.帧布局(FrameLayout)
帧布局是从屏幕的左上角（0,0）坐标开始布局，多个组件层叠排列，第一个添加的组件放到最底层，最后添加到框架中的视图显示在最上面。上一层的会覆盖下一层的控件。

3.表格布局（TableLayout）
表格布局是一个ViewGroup以表格显示它的子视图（view）元素，即行和列标识一个视图的位置。
表格布局常用的属性如下：
collapseColumns：隐藏指定的列
shrinkColumns：收缩指定的列以适合屏幕，不会挤出屏幕
stretchColumns：尽量把指定的列填充空白部分
layout_column:控件放在指定的列
layout_span:该控件所跨越的列数

4.相对布局（RelativeLayout）
相对布局是按照组件之间的相对位置来布局，比如在某个组件的左边，右边，上面和下面等。

5.绝对布局(AbsoluteLayout)
采用坐标轴的方式定位组件，左上角是（0，0）点，往右x轴递增，往下Y轴递增,组件定位属性为layout_x 和layout_y来确定坐标。
```

### Widget(普通控件)

```lua
--Button(按钮控件)、TextView(文本控件)、EditText(编辑框控件)

常用API:
id.setText("文本")--设置控件文本
id.getText()--获取控件文本
id.setWidth(300)--设置控件宽度
id.setHeight(300)--设置控件高度

--点击事件
id.onClick=function()
print"你触发了点击事件"
end

--长按事件
id.onLongClick=function()
print"你触发了长按事件"
end

--图片控件(ImageView与ImageButton)
--设置图片
--布局表中用src属性就可以，如:src=图片路径,

--动态设置
id.setImageBitmap(loadbitmap(图片路径))
--设置Drawable对象
import "android.graphics.drawable.BitmapDrawable"
id.setImageDrawable(BitmapDrawable(loadbitmap(图片路径)))

--缩放，scaleType
--字段
CENTER�   --按原来size居中显示，长/宽超过View的长/宽，截取图片的居中部分显示�
CENTER_CROP    --按比例扩大图片的size居中显示，使图片长(宽)等于或大于View的长(宽)�
CENTER_INSIDE  --完整居中显示，按比例缩小使图片长/宽等于或小于View的长/宽�
FIT_CENTER     --按比例扩大/缩小到View的宽度，居中显示�
FIT_END        --按比例扩大/缩小到View的宽度，显示在View的下部分位置�
FIT_START      --按比例扩大/缩小到View的宽度，显示在View的上部分位置�
FIT_XY         --不按比例扩大/缩小到View的大小显示�
MATRIX         --用矩阵来绘制，动态缩小放大图片来显示。�

--点击与长按事件同上
```

### Check View(检查控件)

```lua
--CheckBox(复选框),Switch(开关控件),ToggleButton(切换按钮)
--直接判断是否选中然后执行相应事件即可
--判断API
check.isChecked()--返回是否勾选、布尔值
check.isSelected()--返回是否选中、布尔值

--RadioButton(单选按钮)与RadioGroup
--将RadioButton的父布局设定为RadioGroup然后绑定下面的监听即可
rp.setOnCheckedChangeListener{
  onCheckedChanged=function(g,c)
  l=g.findViewById(c)
  print(l.Text)
  end}
```

### SeekBar(拖动条)

```lua
--绑定监听
seekbar.setOnSeekBarChangeListener{
onStartTrackingTouch=function()
--开始拖动
end,
onStopTrackingTouch=function()
--停止拖动
end,
onProgressChanged=function()
--状态改变
end}

--部分API
Progress--当前进度
Max--最大进度
```

### ProgressBar(进度条)

```lua
--超大号圆形风格
style="?android:attr/progressBarStyleLarge"
--小号风格
style="?android:attr/progressBarStyleSmall"
--标题型风格
style="?android:attr/progressBarStyleSmallTitle"
--长形进度条
style="?android:attr/progressBarStyleHorizontal"

--部分API
max --最大进度值
progress --设置进度值
secondaryProgress="70" --初始化的底层第二个进度值

id.incrementProgressBy(5)
--ProgressBar进度值增加5
id.incrementProgressBy(-5)
--ProgressBar进度值减少5
id.incrementSecondaryProgressBy(5)
--ProgressBar背后的第二个进度条 进度值增加5
id.incrementSecondaryProgressBy(-5)
--ProgressBar背后的第二个进度条 进度值减少5
```

### Adapter View(适配器控件)

```lua
--适配器控件主要包括(ListView,GridView,Spinner,ExpandableList等)

--想要动态为此类控件添加项目就必须得要依靠适配器！
--适配器使用
--AarrayAdapter(简单适配器)
--创建项目数组
数据={}
--添加项目数组
for i=1,100 do
table.insert(数据,tostring(i))
end
--创建适配器
array_adp=ArrayAdapter(activity,android.R.layout.simple_list_item_1,String(数据))
--设置适配器
lv.setAdapter(array_adp)

--LuaAdapter(Lua适配器)
--创建自定义项目视图
item={
  LinearLayout,
  orientation="vertical",
    layout_width="fill",
   {
    TextView,
    id="text",
    layout_margin="15dp",
    layout_width="fill"
  },
}
--创建项目数组
data={}
--创建适配器
adp=LuaAdapter(activity,data,item)
--添加数据
for n=1,100 do
  table.insert(data,{
    text={
      Text=tostring(n),
    },
  })
end
--设置适配器
lv.Adapter=adp

--以上的适配器ListView、Spinner与GridView等控件通用

--那么ExpandableListView(折叠列表)怎么办呢？
--别怕，安卓系统还提供了一个ArrayExpandableListAdapter来给我们使用，可以简单的适配ExpandableListView，下面给出实例

ns={
  "Widget","Check view","Adapter view","Advanced Widget","Layout","Advanced Layout",
}

wds={
  {"Button","EditText","TextView",
    "ImageButton","ImageView"},
  {"CheckBox","RadioButton","ToggleButton","Switch"},
  {"ListView","ExpandableListView","Spinner"},
  {"SeekBar","ProgressBar","RatingBar",
    "DatePicker","TimePicker","NumberPicker"},
  {"LinearLayout","AbsoluteLayout","FrameLayout"},
  {"RadioGroup","GridLayout",
    "ScrollView","HorizontalScrollView"},
}

mAdapter=ArrayExpandableListAdapter(activity)
for k,v in ipairs(ns) do
  mAdapter.add(v,wds[k])
end
el.setAdapter(mAdapter)
--这样就实现ExpandableListView项目的适配了

--当然AdapterView的事件响应也是与普通控件不同的。

--ListView与GridView的单击与长按事件
--项目被单击
id.onItemClick=function(l,v,p,i)
print(v.Text)
return true
end
--项目被长按
id.onItemLongClick=function(l,v,p,i)
print(v.Text)
return true
end

--Spinner的项目单击事件
id.onItemSelected=function(l,v,p,i)
print(v.Text)
end

--ExpandableListView的父项目与子项目单击事件
id.onGroupClick=function(l,v,p,s)
print(v.Text..":GroupClick")
end

id.onChildClick=function(l,v,g,c)
print(v.Text..":ChildClick")
end
```

### LuaWebView(浏览器控件)

```lua
--常用API
id.loadUrl("http://www.androlua.cn")--加载网页
id.loadUrl("file:///storage/sdcard0/index.html")--加载本地文件
id.getTitle()--获取网页标题
id.getUrl()--获取当前Url
id.requestFocusFromTouch()--设置支持获取手势焦点
id.getSettings().setJavaScriptEnabled(true)--设置支持JS
id.setPluginsEnabled(true)--支持插件
id.setUseWideViewPort(false)--调整图片自适应
id.getSettings().setSupportZoom(true)--支持缩放
id.getSettings().setLayoutAlgorithm(LayoutAlgorithm.SINGLE_COLUMN)--支持重新布局
id.supportMultipleWindows()--设置多窗口
id.stopLoading()--停止加载网页

--状态监听
id.setWebViewClient{
shouldOverrideUrlLoading=function(view,url)
--Url即将跳转
 end,
onPageStarted=function(view,url,favicon)
--网页加载
end,
onPageFinished=function(view,url)
--网页加载完成
end}
```

### AutoCompleteTextView(自动补全文本框)

```lua
--适配数据
arr={"Rain","Rain1","Rain2"};
arrayAdapter=LuaArrayAdapter(activity,{TextView,padding="10dp",textSize="18sp",layout_width="fill",textColor="#ff000000"}, String(arr))
actw.setAdapter(arrayAdapter)

Threshold=1--设置输入几个字符后才能出现提示
```

### TimePicker(时间选择器)

```lua
--时间改变监听器
import "android.widget.TimePicker$OnTimeChangedListener"
id.setOnTimeChangedListener{
  onTimeChanged=function(view,时,分)
    print(时,分)
  end}

--部分API
时=id.getCurrentHour()--获取小时
分=id.getCurrentMinute()--获取分钟
id.setIs24HourView(Boolean(true))--设置24小时制
```

### DatePicker(日期选择器)

```lua
id=dp
日=id.getDayOfMonth()--获取选择的天数
月=id.getMonth ()--获取选择的月份
年=id.getYear()--获取选择的年份
id.updateDate(2016,1,1)--更新日期
print(年,月,日)
```

### NnumberPicker(数值选择器)

```text
setMinValue(0)--设置最小值
setMaxValue(100)--设置最大值
setValue(50)--设置当前值
getValue()--获取选择的值
OnValueChangedListener--数值改变监听器
```

### AlertDialog(对话框)

```lua
--常用API
.setTitle("标题")--设置标题
.setMessage("设置消息")--设置消息
.setView(loadlayout(layout))--设置自定义视图
.setPositiveButton("积极",{onClick=function() end})--设置积极按钮
.setNeutralButton("中立",nil)--设置中立按钮
.setNegativeButton("否认",nil)--设置否认按钮

--普通对话框
AlertDialog.Builder(this)
.setTitle("标题")
.setMessage("消息")
.setPositiveButton("积极",{onClick=function(v) print"点击了积极按钮"end})
.setNeutralButton("中立",nil)
.setNegativeButton("否认",nil)
.show()

--输入对话框
InputLayout={
  LinearLayout;
  orientation="vertical";
  Focusable=true,
  FocusableInTouchMode=true,
  {
    TextView;
    id="Prompt",
    textSize="15sp",
    layout_marginTop="10dp";
    layout_marginLeft="3dp",
    layout_width="80%w";
    layout_gravity="center",
    text="输入:";
  };
  {
    EditText;
    hint="输入";
    layout_marginTop="5dp";
    layout_width="80%w";
    layout_gravity="center",
    id="edit";
  };
};

AlertDialog.Builder(this)
.setTitle("标题")
.setView(loadlayout(InputLayout))
.setPositiveButton("确定",{onClick=function(v) print(edit.Text)end})
.setNegativeButton("取消",nil)
.show()
import "android.view.View$OnFocusChangeListener"
edit.setOnFocusChangeListener(OnFocusChangeListener{
 onFocusChange=function(v,hasFocus)
if hasFocus then
Prompt.setTextColor(0xFD009688)
end
end})

--下载文件对话框
Download_layout={
  LinearLayout;
  orientation="vertical";
  id="Download_father_layout",
  {
    TextView;
    id="linkhint",
    layout_marginTop="10dp";
    text="下载链接",
    layout_width="80%w";
    textColor=WidgetColors,
    layout_gravity="center";
  };
  {
    EditText;
    id="linkedit",
    layout_width="80%w";
    layout_gravity="center";
  };
  {
    TextView;
    id="pathhint",
    text="下载路径",
    layout_width="80%w";
    textColor=WidgetColors,
    layout_marginTop="10dp";
    layout_gravity="center";
  };
  {
    EditText;
    id="pathedit",
    layout_width="80%w";
    layout_gravity="center";
  };
};

AlertDialog.Builder(this)
.setTitle("下载文件")
.setView(loadlayout(Download_layout))
.setPositiveButton("下载",{onClick=function(v)
  end})
.setNegativeButton("取消",nil)
.show()

--列表对话框
items={}
for i=1,5 do
table.insert(items,"项目"..tostring(i))
end
AlertDialog.Builder(this)
.setTitle("列表对话框")
.setItems(items,{onClick=function(l,v) print(items[v+1])end})
.show()

--单选对话框
单选列表={}
for i=1,5 do
table.insert(单选列表,"单选项目"..tostring(i))
end
local 单选对话框=AlertDialog.Builder(this)
.setTitle("列表对话框")
.setSingleChoiceItems(单选列表,-1,{onClick=function(v,p)print(单选列表[p+1])end})
单选对话框.show();

--多选对话框
items={}
for i=1,5 do
table.insert(items,"多选项目"..tostring(i))
end
多选对话框=AlertDialog.Builder(this)
.setTitle("多选框")
.setMultiChoiceItems(items, nil,{ onClick=function(v,p)print(items[p+1])end})
多选对话框.show();
```

### ProgressDialog(进度对话框)

```lua
--ProgressDialog__进度条对话框

dialog = ProgressDialog.show(this, "提示", "正在登陆中").hide()
--最简单便捷的方式

dialog2 = ProgressDialog.show(this, "提示", "正在登陆中", false).hide()
--最后一个boolean设置是否是不明确的状态

dialog3 = ProgressDialog.show(this, "提示", "正在登陆中",false, true).hide()
--最后一个boolean设置可以不可以点击取消

dialog4 = ProgressDialog.show(this, "提示", "正在登陆中",false, true, DialogInterface.OnCancelListener{
  onCancel=function()
    print("对话框取消")
  end
}).hide()

--最后一个参数监听对话框取消，并执行事件

--圆形旋转样式
dialog5= ProgressDialog(this)
dialog5.setProgressStyle(ProgressDialog.STYLE_SPINNER)
dialog5.setTitle("Loading...")
--设置进度条的形式为圆形转动的进度条
dialog5.setMessage("ProgressDialog")
dialog5.setCancelable(true)--设置是否可以通过点击Back键取消
dialog5.setCanceledOnTouchOutside(false)--设置在点击Dialog外是否取消Dialog进度条
dialog5.setOnCancelListener{
  onCancel=function(l)
    print("取消Dialog5")
  end}
--取消对话框监听事件
dialog5.show().hide()

--水平样式
dialog6= ProgressDialog(this)
dialog6.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
--设置进度条的形式为水平进度条
dialog6.setTitle("ProgressDialog_HORIZONTAL")
dialog6.setCancelable(true)--设置是否可以通过点击Back键取消
dialog6.setCanceledOnTouchOutside(false)--设置在点击Dialog外是否取消Dialog进度条
dialog6.setOnCancelListener{
  onCancel=function(l)
    print("取消Dialog6")
  end}
--取消对话框监听事件
dialog6.setMax(100)
--设置最大进度值
dialog6.show().hide()

function 增加(i)
  dialog6.incrementProgressBy(10)
  dialog6.incrementSecondaryProgressBy(10)
  if i=="10" then
    dialog6.dismiss()
    print("加载完成")
  end
  --当进度走完时销毁对话框
end
function 加载()
  require "import"
  for i=1,10 do
    Thread.sleep(300)
    call("增加",tostring(i))
  end
end
--thread(加载)
```

### InputMethodManager(输入法管理器)

```lua
在Android的开发中，有时候会遇到软键盘弹出时挡住输入框的情况。
这时候可以设置下软键盘的模式就可以了。
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE|WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN)
有时候需要软键盘不要把我们的布局整体推上去，这时候可以这样：
activity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN)

模式常量：

软输入区域是否可见。
SOFT_INPUT_MASK_STATE = 0x0f

未指定状态。
SOFT_INPUT_STATE_UNSPECIFIED = 0

不要修改软输入法区域的状态
SOFT_INPUT_STATE_UNCHANGED = 1

隐藏输入法区域（当用户进入窗口时
SOFT_INPUT_STATE_HIDDEN = 2

当窗口获得焦点时，隐藏输入法区域
SOFT_INPUT_STATE_ALWAYS_HIDDEN = 3

显示输入法区域（当用户进入窗口时）
SOFT_INPUT_STATE_VISIBLE = 4

当窗口获得焦点时，显示输入法区域
SOFT_INPUT_STATE_ALWAYS_VISIBLE = 5

窗口应当主动调整，以适应软输入窗口。
SOFT_INPUT_MASK_ADJUST = 0

窗口应当主动调整，以适应软输入窗口。
SOFT_INPUT_MASK_ADJUST = 0xf0

未指定状态，系统将根据窗口内容尝试选择一个输入法样式。
SOFT_INPUT_ADJUST_UNSPECIFIED = 0x00

当输入法显示时，允许窗口重新计算尺寸，使内容不被输入法所覆盖。
不可与SOFT_INPUT_ADJUSP_PAN混合使用；如果两个都没有设置，系统将根据窗口内容自动设置一个选项。
SOFT_INPUT_ADJUST_RESIZE = 0x10

输入法显示时平移窗口。它不需要处理尺寸变化，框架能够移动窗口以确保输入焦点可见。
不可与SOFT_INPUT_ADJUST_RESIZE混合使用；如果两个都没有设置，系统将根据窗口内容自动设置一个选项。
SOFT_INPUT_ADJUST_PAN = 0x20

当用户转至此窗口时，由系统自动设置，所以你不要设置它。
当窗口显示之后该标志自动清除。
SOFT_INPUT_IS_FORWARD_NAVIGATION = 0x100

其它Api参考:
import "android.view.inputmethod.InputMethodManager"


调用显示系统默认的输入法
imm =  activity.getSystemService(Context.INPUT_METHOD_SERVICE)
imm.showSoftInput(m_receiverView(接受软键盘输入的视图(View)),InputMethodManager.SHOW_FORCED(提供当前操作的标记，SHOW_FORCED表示强制显示))

如果输入法关闭则打开，如果输入法打开则关闭
imm = activity.getSystemService(Context.INPUT_METHOD_SERVICE)
imm.toggleSoftInput(0,InputMethodManager.HIDE_NOT_ALWAYS)


获取软键盘是否打开
imm = activity.getSystemService(Context.INPUT_METHOD_SERVICE)
isOpen=imm.isActive()
--返回一个布尔值

隐藏软键盘
activity.getSystemService(INPUT_METHOD_SERVICE)).hideSoftInputFromWindow(WidgetSearchActivity.this.getCurrentFocus().getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS)

显示软键盘
activity.getSystemService(INPUT_METHOD_SERVICE)).showSoftInput(控件ID, 0)
```

### PopMenu(弹出式菜单)

```lua
pop=PopupMenu(activity,view)
menu=pop.Menu
menu.add("项目1").onMenuItemClick=function(a)

end
menu.add("项目2").onMenuItemClick=function(a)

end
pop.show()--显示
```

### PopWindow(弹出式窗口)

```lua
pop=PopWindow(activity)--创建PopWindow
pop.setContentView(loadlayout(布局))--设置布局
pop.setWidth(activity.Width*0.3)--设置宽度
pop.setHeight(activity.Width*0.3)--设置高度
pop.setFocusable(true)--设置可获得焦点
window.setTouchable(true)--设置可触摸
--设置点击外部区域是否可以消失
pop.setOutsideTouchable(false)
--显示
pop.showAtLocation(view,0,0,0)
```

### Toast(提示)

```lua
--默认Toast
Toast.makeText(activity, "Toast",Toast.LENGTH_SHORT).show()

--自定义位置Toast
Toast.makeText(activity,"自定义位置Toast", Toast.LENGTH_LONG).setGravity(Gravity.CENTER, 0, 0).show()

--带图片Toast
图片=loadbitmap("/sdcard/a.png")
toast = Toast.makeText(activity,"带图片的Toast", Toast.LENGTH_LONG)
toastView = toast.getView()
imageCodeProject = ImageView(activity)
imageCodeProject.setImageBitmap(图片)
toastView.addView(imageCodeProject, 0)
toast.show()

--自定义布局Toast
布局=loadlayout(layout)
local toast=Toast.makeText(activity,"提示",Toast.LENGTH_SHORT).setView(布局).show()
```

### 控件常用属性

```lua
--EditText(输入框)
singleLine=true--设置单行输入
Error="错误的输入"--设置用户输入了错误的信息时的提醒
MaxLines=5--设置最大输入行数
MaxEms=5--设置每行最大宽度为五个字符的宽度
InputType="number"--设置只可输入数字
Hint="请输入"--设置编辑框为空时的提示文字

--ImageView(图片视图)
src="a.png"--设置控件图片资源
scaleType="fitXY"--设置图片缩放显示
ColorFilter=Color.BLUE--设置图片着色

--ListView(列表视图)
Items={"item1","item2","item3"}--设置列表项目,但只能在布局表设置,动态添加项目请看Adapter View详解。
DividerHeight=0--设置无隔断线
fastScrollEnabled=true--设置是否显示快速滑块

layout_marginBottom--离某元素底边缘的距离
layout_marginLeft--离某元素左边缘的距离
layout_marginRight--离某元素右边缘的距离
layout_marginTop--离某元素上边缘的距离
gravity--属性是对该view 内容的限定．比如一个button 上面的text. 你可以设置该text 在view的靠左，靠右等位置．以button为例，gravity="right"则button上面的文字靠右
layout_gravity--是用来设置该view相对与起父view 的位置．比如一个button 在linearlayout里，你想把该button放在靠左、靠右等位置就可以通过该属性设置．以button为例，layout_gravity="right"则button靠右
scaleType
--[[是控制图片如何resized/moved来匹对ImageView的size。ImageView.ScaleType / scaleType值的意义区别：
CENTER /center 按图片的原来size居中显示，当图片长/宽超过View的长/宽，则截取图片的居中部分显示
CENTER_CROP / centerCrop 按比例扩大图片的size居中显示，使得图片长(宽)等于或大于View的长(宽)
CENTER_INSIDE / centerInside 将图片的内容完整居中显示，通过按比例缩小或原来的size使得图片长/宽等于或小于View的长/宽
FIT_CENTER / fitCenter 把图片按比例扩大/缩小到View的宽度，居中显示
FIT_END / fitEnd 把图片按比例扩大/缩小到View的宽度，显示在View的下部分位置
FIT_START / fitStart 把图片按比例扩大/缩小到View的宽度，显示在View的上部分位置
FIT_XY / fitXY 把图片不按比例扩大/缩小到View的大小显示
MATRIX / matrix 用矩阵来绘制，动态缩小放大图片来显示。
]]
id--为控件指定相应的ID
text--指定控件当中显示的文字
textSize--指定控件当中字体的大小
background--指定该控件所使用的背景色
width--指定控件的宽度
height--指定控件的高度
layout_width--指定Container组件的宽度
layout_height--指定Container组件的高度
layout_weight--View中很重要的属性，按比例划分空间
padding--指定控件的内边距，也就是说控件当中的内容
sigleLine--如果设置为真的话，则控件的内容在同一行中进行显示
```

### Animation(动画)

```lua
--动画主要包括以下几种
Alpha:渐变透明度动画效果
Scale:渐变尺寸伸缩动画效果
Translate:画面转换位置移动动画效果
Rotate:画面转换位置移动动画效果

--共有的属性有
Duration --属性为动画持续时间 时间以毫秒为单位
fillAfter --当设置为true,该动画转化在动画结束后被应用
fillBefore --当设置为true,该动画转化在动画开始前被应用
repeatCount--动画的重复次数
repeatMode --定义重复的行为
startOffset --动画之间的时间间隔，从上次动画停多少时间开始执行下个动画
id.startAnimation(Animation)--设置控件开始应用这个动画

--动画状态监听
import "android.view.animation.Animation$AnimationListener"
动画.setAnimationListener(AnimationListener{
  onAnimationStart=function()
    print"动画开始"
  end,
onAnimationEnd=function()
  print"动画结束"
  end,
onAnimationRepeat=function()
  print"动画重复"
  end})

--实例
--控件向右旋转180度
Rotate_right=RotateAnimation(180, 0,
Animation.RELATIVE_TO_SELF, 0.5,
Animation.RELATIVE_TO_SELF, 0.5)
Rotate_right.setDuration(440)
Rotate_right.setFillAfter(true)

--控件向左旋转180度
Rotate_left=RotateAnimation(0, 180,
Animation.RELATIVE_TO_SELF, 0.5,
Animation.RELATIVE_TO_SELF, 0.5)
Rotate_left.setDuration(440)
Rotate_left.setFillAfter(true)

--动画设置___从上往下平移动画
Translate_up_down=TranslateAnimation(0, 0, 55, 0)
Translate_up_down.setDuration(800)
Translate_up_down.setFillAfter(true)

--动画设置___透明动画
Alpha=AlphaAnimation(0,1)
Alpha.setDuration(800)

--动画参数值
--AlphaAnimation(透明动画)
AlphaAnimation(float fromStart,float fromEnd)
float fromStart 动画起始透明值
float fromEnd 动画结束透明值

--ScaleAnimation(缩放动画)
ScaleAnimation(float fromX, float toX, float fromY, float toY,int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
float fromX 动画起始时 X坐标上的伸缩尺寸
float toX 动画结束时 X坐标上的伸缩尺寸
float fromY 动画起始时Y坐标上的伸缩尺寸
float toY 动画结束时Y坐标上的伸缩尺寸
int pivotXType 动画在X轴相对于物件位置类型
float pivotXValue 动画相对于物件的X坐标的开始位置
int pivotYType 动画在Y轴相对于物件位置类型
float pivotYValue 动画相对于物件的Y坐标的开始位置

--TranslateAnimation(位移动画)
TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)
float fromXDelta 动画开始的点离当前View X坐标上的差值
float toXDelta 动画结束的点离当前View X坐标上的差值
float fromYDelta 动画开始的点离当前View Y坐标上的差值
float toYDelta 动画结束的点离当前View Y坐标上的差值

--RotateAnimation(旋转动画)
RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
float fromDegrees：旋转的开始角度.
float toDegrees：旋转的结束角度.
int pivotXType：X轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT.
float pivotXValue：X坐标的伸缩值
int pivotYType：Y轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT.
float pivotYValue：Y坐标的伸缩值.
```

### LayoutAnimationController(布局动画控制器)

```lua
--LayoutAnimationController可以控制一组控件按照规定显示

--导入类
import "android.view.animation.AnimationUtils"
import "android.view.animation.LayoutAnimationController"

--创建一个Animation对象
animation = AnimationUtils.loadAnimation(activity,android.R.anim.slide_in_left)

--得到对象
lac = LayoutAnimationController(animation)

--设置控件显示的顺序
lac.setOrder(LayoutAnimationController.ORDER_NORMAL)
--LayoutAnimationController.ORDER_NORMAL   顺序显示
--LayoutAnimationController.ORDER_REVERSE 反显示
--LayoutAnimationController.ORDER_RANDOM 随机显示

--设置控件显示间隔时间
lac.setDelay(time)

--设置组件应用
view.setLayoutAnimation(lac)
```

### ObjectAnimator(属性动画)

```lua
ObjectAnimator(对象动画)
--属性动画概念：
所谓属性动画：
改变一切能改变的对象的属性值，不同于补间动画
只能改变 alpha，scale，rotate，translate
听着有点抽象，举例子说明。

补间动画能实现的:
1.alpha(透明)
--第一个参数为 view对象,第二个参数为 动画改变的类型,第三,第四个参数依次是开始透明度和结束透明度。
alpha = ObjectAnimator.ofFloat(text, "alpha", 0, 1)
alpha.setDuration(2000)--设置动画时间
alpha.setInterpolator(DecelerateInterpolator())--设置动画插入器，减速
alpha.setRepeatCount(-1)--设置动画重复次数，这里-1代表无限
alpha.setRepeatMode(Animation.REVERSE)--设置动画循环模式。
alpha.start()--启动动画。

2.scale(缩放)
animatorSet =  AnimatorSet()--组合动画
scaleX = ObjectAnimator.ofFloat(text, "scaleX", 1, 0)
scaleY = ObjectAnimator.ofFloat(text, "scaleY", 1, 0)
animatorSet.setDuration(2000)
animatorSet.setInterpolator(DecelerateInterpolator());
animatorSet.play(scaleX).with(scaleY)--两个动画同时开始
animatorSet.start();

3.translate(平移)
translationUp = ObjectAnimator.ofFloat(button, "Y",button.getY(), 0)
translationUp.setInterpolator(DecelerateInterpolator())
translationUp.setDuration(1500)
translationUp.start()

4. rotate(旋转)
set =  AnimatorSet()
anim = ObjectAnimator .ofFloat(phone, "rotationX", 0, 180)
anim.setDuration(2000)
anim2 = ObjectAnimator .ofFloat(phone, "rotationX", 180, 0)
anim2.setDuration(2000)
anim3 = ObjectAnimator .ofFloat(phone, "rotationY", 0, 180)
anim3.setDuration(2000)
anim4 = ObjectAnimator .ofFloat(phone, "rotationY", 180, 0)
anim4.setDuration(2000)
set.play(anim).before(anim2)--先执行anim动画之后在执行anim2
set.play(anim3).before(anim4)
set.start()

补间动画不能实现的:
5.android 改变背景颜色的动画实现如下
translationUp = ObjectAnimator.ofInt(button,"backgroundColor",{Color.RED, Color.BLUE, Color.GRAY,Color.GREEN})
translationUp.setInterpolator(DecelerateInterpolator())
translationUp.setDuration(1500)
translationUp.setRepeatCount(-1)
translationUp.setRepeatMode(Animation.REVERSE)
translationUp.setEvaluator(ArgbEvaluator())
translationUp.start()
--[[
ArgbEvaluator：这种评估者可以用来执行类型之间的插值整数值代表ARGB颜色。
FloatEvaluator：这种评估者可以用来执行浮点值之间的插值。
IntEvaluator：这种评估者可以用来执行类型int值之间的插值。
RectEvaluator：这种评估者可以用来执行类型之间的插值矩形值。

由于本例是改变View的backgroundColor属性的背景颜色所以此处使用ArgbEvaluator
]]
```

### overridePendingTransition(设置窗口动画)

```lua
activity.overridePendingTransition(android.R.anim.fade_in,android.R.anim.fade_out)
```

### 自带Http模块

```lua
获取内容 get函数
Http.get(url,cookie,charset,header,callback)
url 网络请求的链接网址
cookie 使用的cookie，也就是服务器的身份识别信息
charset 内容编码
header 请求头
callback 请求完成后执行的函数

除了url和callback其他参数都不是必须的

回调函数接受四个参数值分别是
code 响应代码，2xx表示成功，4xx表示请求错误，5xx表示服务器错误，-1表示出错
content 内容，如果code是-1，则为出错信息
cookie 服务器返回的用户身份识别信息
header 服务器返回的头信息

向服务器发送数据 post函数
Http.post(url,data,cookie,charset,header,callback)
除了增加了一个data外，其他参数和get完全相同
data 向服务器发送的数据

下载文件 download函数
Http.download(url,path,cookie,header,callback)
参数中没有编码参数，其他同get，
path 文件保存路径

需要特别注意一点，只支持同时有127个网络请求，否则会出错

Http其实是对Http.HttpTask的封装，Http.HttpTask使用的更加通用和灵活的形式
参数格式如下
Http.HttpTask( url, String method, cookie, charset, header,  callback)
所有参数都是必选，没有则传入nil

url 请求的网址
method 请求方法可以是get，post，put，delete等
cookie 身份验证信息
charset 内容编码
header 请求头
callback 回调函数

该函数返回的是一个HttpTask对象，
需要调用execute方法才可以执行，
t=Http.HttpTask(xxx)
t.execute{data}

注意调用的括号是花括号，内容可以是字符串或者byte数组，
使用这个形式可以自己封装异步上传函数
```

### TrafficStats类

```lua
import "android.net.TrafficStats"
getMobileRxBytes()  --获取通过Mobile连接收到的字节总数，不包含WiFi
getMobileRxPackets()  --获取Mobile连接收到的数据包总数
getMobileTxBytes()  --Mobile发送的总字节数
getMobileTxPackets()  --Mobile发送的总数据包数
getTotalRxBytes()  --获取总的接受字节数，包含Mobile和WiFi等
getTotalRxPackets()  --总的接受数据包数，包含Mobile和WiFi等
getTotalTxBytes()  --总的发送字节数，包含Mobile和WiFi等
getTotalTxPackets()  --发送的总数据包数，包含Mobile和WiFi等
getUidRxBytes(int uid)  --获取某个网络UID的接受字节数
getUidTxBytes(int uid) --获取某个网络UID的发送字节数
--例:TrafficStats.getTotalRxBytes()
```

### 开启关闭WiFi

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
wifi.setWifiEnabled(true)--关闭则false
```

### 断开网络

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
wifi.disconnect()
```

### WiFi是否打开

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
wi = wifi.isWifiEnabled()
```

### WiFi是否连接

```lua
connManager = activity.getSystemService(Context.CONNECTIVITY_SERVICE)
    mWifi = connManager.getNetworkInfo(ConnectivityManager.TYPE_WIFI);
    if tostring(mWifi):find("none)")  then
    --未连接
    else
    --连接
    end
```

### 数据网络是否连接

```lua
manager = activity.getSystemService(Context.CONNECTIVITY_SERVICE);
gprs = manager.getNetworkInfo(ConnectivityManager.TYPE_MOBILE).getState();
if tostring(gprs)== "CONNECTED" then
print"当前数据网络"
end
```

### 获取WiFi信息

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
 wifi.getConfiguredNetworks()
```

### 获取WiFi状态

```lua
import "android.content.Context"
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE)
print(wifi.getWifiState())
```

### IP地址

```lua
--查看某网站IP地址
address=InetAddress.getByName("www.10010.com");

--查看本机IP地址
address=InetAddress.getLocalHost();

--查看IP地址
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getDhcpInfo()
string.match(tostring(wifi),"ipaddr(.-)gate")
```

### 获取Dns

```lua
import "android.content.Context"

--获取Dns1
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getDhcpInfo()
 print(string.match(tostring(wifi),"dns1 (.-) dns2"))

--获取Dns2
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getDhcpInfo()
 dns2 = string.match(tostring(wifi),"dns2 (.-) D")
```

### 获取网络名称

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print(wifiInfo.getSSID())
```

### 获取WiFi加密类型

```lua
wifi = activity.Context.getSystemService(Context.WIFI_SERVICE).getConfiguredNetworks()
print(string.match(tostring(wifi),[[KeyMgmt: (.-) P]]))
```

### 获取网络信号强度

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print(wifiInfo.getRssi())
```

### 获取SSID是否被隐藏

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print(wifiInfo.getHiddenSSID())
```

### 获取Mac地址

```lua
wifiManager=activity.Context.getSystemService(Context.WIFI_SERVICE);
wifiInfo=wifiManager.getConnectionInfo();
print( wifiInfo.getMacAddress())
```

### 打印

```lua
print(打印内容)
```

### 控件被单击

```lua
function 控件ID.onClick()
--事件
end

控件ID.onClick=function()
--事件
end
```

### 控件被长按

```lua
控件ID.onLongClick=function()
--事件
end

function 控件ID.onLongClick()
--事件
end
```

### 控件可视，不可视或隐藏

```lua
--控件可视
控件ID.setVisibility(View.VISIBLE)
--控件不可视
控件ID.setVisibility(View.INVISIBLE)
--控件隐藏
控件ID.setVisibility(View.GONE)
```

### 提示框

```lua
import "android.content.DialogInterface"
local dl=AlertDialog.Builder(activity)
.setTitle("提示框标题")
.setMessage("提示框内容")
.setPositiveButton("按钮标题",DialogInterface
.OnClickListener{
onClick=function(v)
--事件
end
})
.setNegativeButton("按钮标题",nil)
.create()
dl.show()
```

### 读写文件

```lua
--读文件
local file=io.input("地址")
local str=io.read("*a")
io.close()
print(str)
--写文件
local file=io.output("地址")
io.write(写入内容)
io.flush()
io.close()
```

### 加载框示例

```lua
local dl=ProgressDialog.show(activity,nil,'登录中')
dl.show()
local a=0
local tt=Ticker()
tt.start()
tt.onTick=function()
a=a+1
if a==3 then
dl.dismiss()
tt.stop()
end
end
```

### 标题栏菜单按钮

```lua
tittle={"分享","帮助","皮肤","退出"}
function onCreateOptionsMenu(menu)
for k,v in ipairs(tittle) do
if tittle[v] then
local m=menu.addSubMenu(v)
for k,v in ipairs(tittle[v]) do
m.add(v)
end
else
local m=menu.add(v)
m.setShowAsActionFlags(1)
end
end
end
function onMenuItemSelected(id,tittle)
if y[tittle.getTitle()] then
y[tittle.getTitle()]()
end
end

y={}
y["帮助"]=function()
--事件
end

--菜单
function onCreateOptionsMenu(menu)
menu.add("打开").onMenuItemClick=function(a)

end
menu.add("新建").onMenuItemClick=function(a)

end
end
```

### 关闭对话框

```lua
--将dl.show赋值
dialog=dl.show()
--在某按钮点击后关闭这个对话框
function zc.onClick()
dialog.dismiss()
end
```

### 判断是否有网络

```lua
local wl=activity.getApplicationContext().getSystemService(Context.CONNECTIVITY_SERVICE).getActiveNetworkInfo();
if wl== nil then
print("无法连接到服务器")
end
```

### 沉浸状态栏

```lua
--这个需要系统SDK21以上才能用
if Build.VERSION.SDK_INT >= 21 then
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS).setStatusBarColor(0xff4285f4);
end
--这个需要系统SDK19以上才能用
if Build.VERSION.SDK_INT >= 19 then
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
end
```

### 复制文本到剪贴板

```lua
--先导入包
import "android.content.*"
activity.getSystemService(Context.CLIPBOARD_SERVICE).setText(文本)
```

### 安卓跳转动画

```text
android.R.anim.accelerate_decelerate_interpolator
android.R.anim.accelerate_interpolator
android.R.anim.anticipate_interpolator
android.R.anim.anticipate_overshoot_interpolator
android.R.anim.bounce_interpolator
android.R.anim.cycle_interpolator
android.R.anim.decelerate_interpolatoandroid.R.anim.r
android.R.anim.fade_in
android.R.anim.fade_out
android.R.anim.linear_interpolator
android.R.anim.overshoot_interpolator
android.R.anim.slide_in_left
android.R.anim.slide_out_right
```

### TextView文本可选择复制

```lua
--代码中设置
t.TextIsSelectable=true
--布局表中设置
textIsSelectable=true
```

### 取随机数

```text
math.random(最小值,最大值)
```

### 延迟

```lua
--这个会卡进程，配合线程使用
Thread.sleep(延迟时间)
--这个不会卡进程
--500指延迟500毫秒
task(500,function()
--延迟之后执行的事件
end)
```

### 定时器

```lua
--timer定时器
t=timer(function()
--事件
end,延迟,间隔,初始化)
--暂停timer定时器
t.Enable=false
--启动timer定时器
t.Enable=true

--Ticker定时器
ti=Ticker()
ti.Period=间隔
ti.onTick=function()
--事件
end
--启动Ticker定时器
ti.start()
--停止Ticker定时器
ti.stop()
```

### 获取本地时间

```lua
--格式的时间
os.date("%Y-%m-%d %H:%M:%S")
--本地时间总和
os.clock()
```

### EditText文本被改变事件

```lua
控件ID.addTextChangedListener{
onTextChanged=function(s)
--事件
end
}
```

## 来源：AndroLua帮助.txt

### 关于AndroLua

```text
AndroLua是基于LuaJava开发的安卓平台轻量级脚本编程语言工具，既具有Lua简洁优雅的特质，又支持绝大部分安卓API，可以使你在手机上快速编写小型应用。
官方QQ群：236938279；官方QQ2群：621400904
百度贴吧：
http://c.tieba.baidu.com/mo/m?kw=androlua
项目地址：
https://github.com/nirenr/AndroLua_pro
打开链接支持nirenr的工作，一块也可以哦：
https://qr.alipay.com/apt7ujjb4jngmu3z9a

AndroLua使用了以下开源项目部分代码

bson,crypt,md5
https://github.com/cloudwu/skynet

cjson
https://sourceforge.net/projects/cjson/

zlib
https://github.com/brimworks/lua-zlib

xml
https://github.com/chukong/quick-cocos2d-x

luv
https://github.com/luvit/luv
https://github.com/clibs/uv

zip
https://github.com/brimworks/lua-zip
https://github.com/julienr/libzip-android

luagl
http://luagl.sourceforge.net/

luasocket
https://github.com/diegonehab/luasocket

sensor
https://github.com/ddlee/AndroidLuaActivity

canvas
由落叶似秋开发

jni
由nirenr开发
```

### 软件基本操作

```text
工程结构
  init.lua 工程配置文件
  main,lua 工程主入口文件
  layout.aly  工程默认创建的布局文件

  菜单功能
  三角形 运行：执行当前工程
  左箭头 撤销：撤销输入的内容
  右箭头 重做：恢复撤销的内容
  打开：打开文件，在文件列表长按可删除文件
  最近：显示最近打开过的文件

  文件
    保存：保存当前文件
    新建：新建lua代码文件或者aly布局文件，代码文件与布局文件文件名不可以相同
    编译：把当前文件编译为luac文件，通常用不到

  工程
    代开：在工程列表打开工程
    打包：将当前工程编译为apk，默认使用debug签名
    新建：新建一个工程
    导出：将当前工程备份为alp文件
    属性：编辑当前工程的属性，如 名称 权限等

  代码
    格式化：重新缩进当前文件使其更加便于阅读
    导入分析：分析当前文件及引用文件需要导入的java类
    查错：检查当前文件是否有语法错误

  转到
    搜索：搜索指定内容位置
    转到：按行号跳转
    导航：按函数跳转

  插件：使用安装的插件

  其他
    布局助手：在编辑器打开aly文件时用于设计布局，目前功能尚不完善
    日志：查看程序运行时的日志
    java浏览器：用于查看java类的方法
    手册：离线版lua官方手册
    联系作者：加入官方qq群与作者交流
    捐赠：使用支付宝捐赠作者，使软件更好的发展下去
```

### 快速入门

```lua
AndroLua是一个使用Lua语法编写可以使用安卓API的轻型脚本编程工具，使用它可以快速编写安卓应用。
   第一次打开程序默认创建new.lua，并添加以下代码

   require "import"
   import "android.widget."
   import "android.view."

   require "import" 是导入import模块，该模块集成了很多实用的函数，可以大幅度减轻写代码负担，详细函数说明参考程序帮助。
   import "android.widget.*" 是导入Java包。
   这里导入了android的widget和view两个包。

   导入包后使用类是很容易的，新建类实例和调用Lua的函数一样。
   比如新建一个TextView
   tv=TextView(activity)
   activity表示当前活动的context。
   同理新建按钮 btn=Button(activity)

   给视图设置属性也非常简单
   btn.text="按钮"
   btn.backgroundColor=0xff0000ff

   添加视图事件回调函数
   btn.onClick=function(v)
     print(v)
   end
   函数参数v是视图本身。

   安卓的视图需要添加到布局才能显示到活动，一般我们常用LinearLayout
   layout=LinearLayout(activity)

   用addView添加视图
   layout.addView(btn)

   最后调用activity的setContentView方法显示内容
   activity.setContentView(layout)
   这里演示androlua基本用法，通常我们需要新建一个工程来开发，代码的用法是相同的，具体细节请详细阅读后面的内容。
```

### 与标准Lua5.3的不同

```text
打开了部分兼容选项，module，unpack，bit32
添加string.gfind函数，用于递归返回匹配位置
增加tointeger函数，强制将数值转为整数
修改tonumber支持转换Java对象
```

### 参考链接

```text
关于lua的语法和Android API请参考以下网页。
Lua官网：
http://www.lua.org
Android 中文API：
http://android.toolib.net/reference/packages.html
```

### 导入模块

```lua
require "import"
以导入import模块，简化写代码的难度。
目前程序还内置bmob,bson,canvas,cjson,crypt,ftp,gl,http,import,md5,smtp,socket,sensor,xml,zip,zlib等模块。
一般模块导入形式
local http=require "http"
这样导入的是局部变量
导入import后也可以使用
import "http"
的形式，导入为全局变量
```

### 导入包或类

```lua
在使用Java类之前需要导入相应的包或者类，
可以用包名.*的形式导入导入包
import "android.widget.*"
或者用完整的类名导入类
import "android.widget.Button"
导入内部类
import "android.view.View_OnClickListener"
或者在导入类后直接使用内部类
View.OnClickListene
包名和类名必须用引号包围。
导入的类为全局变量，你可以使用
local Burton=import "android.widget.Button"
的形式保存为局部变量，以解决类名冲突问题。
```

### 创建布局与组件

```lua
安卓使用布局与视图管理和显示用户界面。
布局负责管理视图如何显示，如LinearLayout以线性排列视图，FrameLayout则要求自行指定停靠与位置。
视图则显示具体内容，如TextView可以向用户展示文字内容，Button可以响应用户点击事件。

创建一个线性布局
layout=LinearLayout(activity)
创建一个按钮视图
button=Button(activity)
将按钮添加到布局
layout.addView(button)
将刚才的内容设置为活动内容视图
activity.setContentView(layout)

注.activity是当前窗口的Context对象，如果你习惯也可以使用this
button=Button(this)
```

### 使用方法

```lua
button.setText("按钮")

getter/setter
Java的getxxx方法没有参数与setxxx方法只有一个参数时可以简写，
button.Text="按钮"
x=button.Text
```

### 使用事件

```lua
创建事件处理函数
function click(s)
    print("点击")
    end
把函数添加到事件接口
listener=View.OnClickListener{onClick = click}
把接口注册到组件
button.setOnClickListener(listener)

也可以使用匿名函数
button.setOnClickListener(View.OnClickListener {onClick = function(s)
        print("点击")
        end
    })

onxxx事件可以简写
button.onClick=function(v)
    print(v)
    end
```

### 回调方法

```lua
在活动文件添加以下函数，这些函数可以在活动的特定状态执行。
function main(...)
    --...：newActivity传递过来的参数。
    print("入口函数",...)
    end

function onCreate()
    print("窗口创建")
    end

function onStart()
    print("活动开始")
    end

function onResume()
    print("返回程序")
    end

function onPause()
    print("活动暂停")
    end

function onStop()
    print("活动停止")
    end

function onDestroy()
    print("程序已退出")
    end

function onResult(name,...)
  --name：返回的活动名称
  --...：返回的参数
  print("返回活动",name,...)
  end

function onCreateOptionsMenu(menu)
    --menu：选项菜单。
    menu.add("菜单")
    end

function onOptionsItemSelected(item)
    --item：选中的菜单项
    print(item.Title)
    end

function onConfigurationChanged(config)
    --config：配置信息
    print("屏幕方向关闭")
    end

function onKeyDown(keycode,event)
    --keycode：键值
    --event：事件
    print("按键按下",keycode)
    end

function onKeyUp(keycode,event)
    --keycode：键值
    --event：事件
    print("按键抬起",keycode)
    end

function onKeyLongPress(keycode,event)
    --keycode：键值
    --event：事件
    print("按键长按",keycode)
    end

function onTouchEvent(event)
    --event：事件
    print("触摸事件",event)
    end
```

### 按键与触控

```lua
function onKeyDown(code,event)
    print(code event)
    end
function onTouchEvent(event)
    print(event)
    end
支持onKeyDown,onKeyUp,onKeyLongPress,onTouchEvent
函数必须返布尔值
```

### 使用数组

```lua
array=float{1,2,3}
或者
array=int[10]
a=array[0]
array[0]=4
```

### 使用线程

```lua
需导入import模块，参看thread,timer与task函数说明。
线程中使用独立环境运行，不能使用外部变量与函数，需要使用参数和回调与外部交互。
任务

task(str,args,callback)

str 为任务执行代码，args 为参数，callback 为回调函数，任务返回值将传递到回调方法
线程

t=thread(str,args)

str 为线程中执行的代码，args 为初始传入参数
调用线程中方法
call(t,fn,args)
t 为线程，fn 为方法名称，args 为参数
设置线程变量
set(t,fn,arg)
t 为线程，fn 为变量名称，arg 为变量值
线程调用主线程中方法
call(fn,args)
fn 为方法名称，args 为参数
线程设置主线程变量
set(fn,arg)
fn 为变量名称，arg 为变量值

注. 参数类型为 字符串，数值，Java对象，布尔值与nil
线程要使用quit结束线程。

t=timer(func,delay,period,args)

func 为定时器执行的函数，delay 为定时器延时，period 为定时器间隔，args 为初始化参数
t.Enable=false 暂停定时器
t.Enable=true 启动定时器
t.stop() 停止定时器

注意：定时器函数定义run函数时定时器重复执行run函数，否则重复执行构建时的func函数
```

### 使用布局表

```lua
使用布局表须导入android.view与android.widget包。
require "import"
import "android.widget.*"
import "android.view.*"
布局表格式
layout={
    控件类名称,
    id=控件名称,
    属性=值,
    {
        子控件类名称,
        id=控件名称,
        属性=值,
        }
    }

例如：
layout={
  LinearLayout,--视图类名称
  id="linear",--视图ID，可以在loadlayout后直接使用
  orientation="vertical",--属性与值
  {
    TextView,--子视图类名称
    text="hello AndroLua+",--属性与值
    layout_width="fill"--布局属性
  },
}
使用loadlayout函数解析布局表生成布局。
activity.setContentView(loadlayout(layout))
也可以简化为：
activity.setContentView(layout)
如果使用单独文件布局(比如有个layout.aly布局文件)也可以简写为：
activity.setContentView("layout")
此时不用导入布局文件。

布局表支持大全部安卓控件属性，
与安卓XML布局文件的不同点：
id表示在Lua中变量的名称，而不是安卓的可以findbyid的数字id。
ImageView的src属性是当前目录图片名称或绝对文件路径图片或网络上的图片，
layout_width与layout_height的值支持fill与wrap简写，
onClick值为lua函数或java onClick接口或他们的全局变量名称，
背景background支持背景图片，背景色与LuaDrawable自绘制背景，背景图片参数为是当前目录图片名称或绝对文件路径图片或网络上的图片，颜色同backgroundColor，自绘制背景参数为绘制函数或绘制函数的全局变量名称，
控件背景色使用backgroundColor设置，值为"十六进制颜色值"。
尺寸单位支持 px，dp，sp，in，mm，%w，%h。
其他参考loadlayout与loadbitmap
```

### 2D绘图

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.graphics.*"
activity.setTitle('AndroLua')

paint=Paint()
paint.setARGB(100,0,250,0)
paint.setStrokeWidth(20)
paint.setTextSize(28)

sureface = SurfaceView(activity);
callback=SurfaceHolder_Callback{
    surfaceChanged=function(holder,format,width,height)
        end,
    surfaceCreated=function(holder)
        ca=holder.lockCanvas()
        if (ca~=nil) then
            ca.drawRGB(0,79,90);
            ca.drawRect(0,0,200,300,paint)
            end
        holder.unlockCanvasAndPost(ca)
        end,
    surfaceDestroyed=function(holder)
        end
    }
holder=sureface.getHolder()
holder.addCallback(callback)
activity.setContentView(sureface)
```

### Lua类型与Java类型

```lua
在大多数情况下androlua可以很好的处理Lua与Java类型之间的自动转换，但是Java的数值类型有多种(double,float,long,int,short,byte)，而Lua只有number，在必要的情况下可以使用类型的强制转换。
i=int(10)
i就是一个Java的int类型数据
d=double(10)
d是一个Java的double类型
在调用Java方法时androlua可以自动将Lua的table转换成Java的array，Map或interface
Map类型可以像使用Lua表一样简便。
map=HashMap{a=1,b=2}
print(map.a)
map.a=3
取长度运算符#可以获取Java中array，List,Map,Set，String的长度。
```

### canvas模块

```lua
require "import"
import "canvas"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.graphics.*"
activity.setTitle('AndroLua')

paint=Paint()
paint.setARGB(100,0,250,0)
paint.setStrokeWidth(20)
paint.setTextSize(28)

sureface = SurfaceView(activity);
callback=SurfaceHolder_Callback{
    surfaceChanged=function(holder,format,width,height)
        end,
    surfaceCreated=function(holder)
        ca=canvas.lockCanvas(holder)
        if (ca~=nil) then
            ca:drawRGB(0,79,90)
            ca:drawRect(0,0,200,300,paint)
            end
        canvas.unlockCanvasAndPost(holder,ca)
        end,
    surfaceDestroyed=function(holder)
        end
    }
holder=sureface.getHolder()
holder.addCallback(callback)
activity.setContentView(sureface)
```

### OpenGL模块

```lua
require "import"
import "gl"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "android.opengl.*"
activity.setTitle('AndroLua')
--activity.setTheme( android.R.style.Theme_Holo_Light_NoActionBar_Fullscreen)

mTriangleData ={
    0.0, 0.6, 0.0,
    -0.6, 0.0, 0.0,
    0.6, 0.0, 0.0,
    };
mTriangleColor = {
    1, 0, 0, 0,
    0, 1, 0, 0,
    0, 0, 1, 0,
    };

sr=GLSurfaceView.Renderer{
    onSurfaceCreated=function(gl2, config)
        gl.glDisable(gl.GL_DITHER);
        gl.glHint(gl.GL_PERSPECTIVE_CORRECTION_HINT, gl.GL_FASTEST);
        gl.glClearColor(0, 0, 0, 0);
        gl.glShadeModel(gl.GL_SMOOTH);
        gl.glClearDepth(1.0)
        gl.glEnable(gl.GL_DEPTH_TEST);
        gl.glDepthFunc(gl.GL_LEQUAL);
        end,
    onDrawFrame=function(gl2, config)
        gl.glClear(gl.GL_COLOR_BUFFER_BIT | gl.GL_DEPTH_BUFFER_BIT);
        gl.glMatrixMode(gl.GL_MODELVIEW);
        gl.glLoadIdentity();
        gl.glRotate(0,1,1,1)
        gl.glTranslate(0, 0,0);
        gl.glEnableClientState(gl.GL_VERTEX_ARRAY);
        gl.glEnableClientState(gl.GL_COLOR_ARRAY);
        gl.glVertexPointer( mTriangleData,3);
        gl.glColorPointer(mTriangleColor,4);
        gl.glDrawArrays( gl.GL_TRIANGLE_STRIP , 0, 3);
        gl.glFinish();
        gl.glDisableClientState(gl.GL_VERTEX_ARRAY);
        gl.glDisableClientState(gl.GL_COLOR_ARRAY);
        end,
    onSurfaceChanged= function (gl2, w, h)
        gl.glViewport(0, 0, w, h);
        gl.glLoadIdentity();
        ratio =  w / h;
        gl.glFrustum(-rautio, ratio, -1, 1, 1, 10);
        end
    }

glSurefaceView = GLSurfaceView(activity);
glSurefaceView.setRenderer(sr);
activity.setContentView(glSurefaceView);
```

### http 同步网络模块

```lua
body,cookie,code,headers=http.get(url [,cookie,ua,header])
body,cookie,code,headers=http.post(url ,postdata [,cookie,ua,header])
code,headers=http.download(url [,cookie,ua,ref,header])
body,cookie,code,headers=http.upload(url ,datas ,files [,cookie,ua,header])
参数说明
url 网址
postdata post的字符串或字符串数据组表
datas upload的字符串数据组表
files upload的文件名数据表
cookie 网页要求的cookie
ua 浏览器识别
ref 来源页网址
header http请求头

require "import"
import "http"

--get函数以get请求获取网页，参数为请求的网址与cookie
body,cookie,code,headers=http.get("http://www.androlua.com")

--post函数以post请求获取网页，通常用于提交表单，参数为请求的网址，要发送的内容与cookie
body,cookie,code,headers=http.post("http://androlua.com/Login.Asp?Login=Login&Url=http://androlua.com/bbs/index.asp","name=用户名&pass=密码&ki=1")

--download函数和get函数类似，用于下载文件，参数为请求的网址，保存文件的路径与cookie
http.download("http://androlua.com","/sdcard/a.txt")

--upload用于上传文件，参数是请求的网址，请求内容字符串部分，格式为以key=value形式的表，请求文件部分，格式为key=文件路径的表，最后一个参数为cookie
http.upload("http://androlua.com",{title="标题",msg="内容"},{file1="/sdcard/1.txt",file2="/sdcard/2.txt"})
```

### import模块

```lua
require "import"
import "android.widget.*"
import "android.view.*"
layout={
    LinearLayout,
    orientation="vertical",
    {
        EditText,
        id="edit",
        layout_width="fill"
        },
    {
        Button,
        text="按钮",
        layout_width="fill",
        onClick="click"
        }
    }

function click()
    Toast.makeText(activity, edit.getText().toString(), Toast.LENGTH_SHORT ).show()
    end
activity.setContentView(loadlayout(layout))
```

### Http 异步网络模块

```lua
获取内容 get函数
Http.get(url,cookie,charset,header,callback)
url 网络请求的链接网址
cookie 使用的cookie，也就是服务器的身份识别信息
charset 内容编码
header 请求头
callback 请求完成后执行的函数

除了url和callback其他参数都不是必须的

回调函数接受四个参数值分别是
code 响应代码，2xx表示成功，4xx表示请求错误，5xx表示服务器错误，-1表示出错
content 内容，如果code是-1，则为出错信息
cookie 服务器返回的用户身份识别信息
header 服务器返回的头信息

向服务器发送数据 post函数
Http.post(url,data,cookie,charset,header,callback)
除了增加了一个data外，其他参数和get完全相同
data 向服务器发送的数据

下载文件 download函数
Http.download(url,path,cookie,header,callback)
参数中没有编码参数，其他同get，
path 文件保存路径

需要特别注意一点，只支持同时有127个网络请求，否则会出错


Http其实是对Http.HttpTask的封装，Http.HttpTask使用的更加通用和灵活的形式
参数格式如下
Http.HttpTask( url, String method, cookie, charset, header,  callback)
所有参数都是必选，没有则传入nil

url 请求的网址
method 请求方法可以是get，post，put，delete等
cookie 身份验证信息
charset 内容编码
header 请求头
callback 回调函数

该函数返回的是一个HttpTask对象，
需要调用execute方法才可以执行，
t=Http.HttpTask(xxx)
t.execute{data}

注意调用的括号是花括号，内容可以是字符串或者byte数组，
使用这个形式可以自己封装异步上传函数
```

### bmob网络数据库

```lua
b=bmob(id,key)
id 用户id，key 应用key。

b:insert(key,data,callback)
新建数据表，key 表名称，data 数据，callback 回调函数。

b:update(key,id,data,callback)
更新数据表，key 表名称id 数据id，data 数据，callback 回调函数。

b:query(key,data,callback)
查询数据表，key 表名称，data 查询规则，callback 回调函数。

b:increment(key,id,k,v,c)
原子计数，key 表名称，id 数据id，k 数据key，v 计数增加量。

b:delete(key,id,callback)
删除数据，key 表名称,id 数据id，callback 回调函数。

b:sign(user,pass,mail,callback)
注册用户，user 用户名，pass 密码，mail 电子邮箱，callback 回调函数。

b:login(user or mail,pass,callback)
登录用户，user 用户名，pass 密码，mail 电子邮箱，callback 回调函数。

b:upload(path,callback)
上传文件，path 文件路径，callback 回调函数。

b:remove(url,callback)
删除文件，url 文件路径，callback 回调函数。


注：
1，查询规则支持表或者json格式，具体用法参考官方api
2，回调函数的第一个参数为状态码，-1 出错，其他状态码参考http状态码，第二个参数为返回内容。
```

### LuaUtil 辅助库

```text
copyDir(from,to)
复制文件或文件夹，from 源路径，to 目标路径。

zip(from,dir,name)
压缩文件或文件夹，from 源路径，dir 目标文件夹，name zip文件名称。

unZip(from,to)
解压文件，from zip文件路径，to 目标路径。

getFileMD5(path)
获取文件MD5值， path 文件路径。

getFileSha1(path)
获取文件Sha1值， path 文件路径。
```

### LuaAdapter 适配器

```lua
构建方法
adapter=LuaAdapter(activity,data,layout)
构建适配器，activity 当前活动，data 列表数据，layout 列表项目布局。
data格式为{{id=value},{id=value}}格式的数组表。

adapter.add(data)
添加数据，data 为列表项目数据，格式为{id=value}。

adapter.insert(idx,{id=value})
插入数据，idx 为从0计数的插入位置，data 为列表项目数据，格式为{id=value}。

adapter.remove(idx)
删除数据，idx 为从0计数的删除位置。

adapter.clear()
清空数据。

adapter.notifyDataSetChanged()
更新数据。

也可以使用table.insert/table.remove直接对data表操作，table库操作从1开始计数，改操作需要手动更新列表。

在使用LuaAdapter的ListView的onItemClick/onItemLongClick回调函数中，第三个参数为从0开始的项目序号，第四个参数为从1开始的项目序号。
```

### 关于AndroLua打包

```lua
新建工程或在脚本目录新建init.lua文件。
写入以下内容，即可将文件夹下所有lua文件打包，main.lua为程序人口。
appname="demo"
appver="1.0"
packagename="com.androlua.demo"
目录下icon.png替换图标，welcome.png替换启动图。
打包使用debug签名。
```

### 部分函数参考

```lua
[a]表示参数a可选，(...)表示不定参数。函数调用在只有一个参数且参数为字符串或表时可以省略括号。
AndroLua库函数在import模块，为便于使用都是全局变量。
s 表示string类型，i 表示整数类型，n 表示浮点数或整数类型，t 表示表类型，b 表示布尔类型，o 表示Java对象类型，f为Lua函数。
--表示注释。

each(o)
参数：o 实现Iterable接口的Java对象
返回：用于Lua迭代的闭包
作用：Java集合迭代器


enum(o)
参数：o 实现Enumeration接口的Java对象
返回：用于Lua迭代的闭包
作用：Java集合迭代器

import(s)
参数：s 要载入的包或类的名称
返回：载入的类或模块
作用：载入包或类或Lua模块
import "http" --载入http模块
import "android.widget.*" --载入android.widget包
import "android.widget.Button" --载入android.widget.Button类
import "android.view.View$OnClickListener" --载入android.view.View.OnClickListener内部类

loadlayout(t [,t2])
参数：t 要载入的布局表，t2 保存view的表
返回：布局最外层view
作用：载入布局表，生成view
layout={
    LinearLayout,
    layout_width="fill",
    {
        TextView,
        text="Androlua",
        id="tv"
        }
    }
main={}
activity.setContentView(loadlayout(layout,main))
print(main.tv.getText())

loadbitmap(s)
参数：s 要载入图片的地址，支持相对地址，绝对地址与网址
返回：bitmap对象
作用：载入图片
注意：载入网络图片需要在线程中进行

task(s [,...], f)
参数：s 任务中运行的代码或函数，... 任务传入参数，f 回调函数
返回：无返回值
作用：在异步线程运行Lua代码，执行完毕在主线程调用回调函数
注意：参数类型包括 布尔，数值，字符串，Java对象，不允许Lua对象
function func(a,b)
    require "import"
    print(a,b)
    return a+b
    end
task(func,1,2,print)

thread(s[,...])
参数：s 线程中运行的lua代码或脚本的相对路径(不加扩展名)或函数，... 线程初始化参数
返回：返回线程对象
作用：开启一个线程运行Lua代码
注意：线程需要调用quit方法结束线程
func=[[
a,b=...
function add()
    call("print",a+b)
    end
]]
t=thread(func,1,2)
t.add()

timer(s,i1,i2[,...])
参数：s 定时器运行的代码或函数，i1 前延时，i2 定时器间隔，... 定时器初始化参数
返回：定时器对象
作用：创建定时器重复执行函数
function f(a)
    function run()
        print(a)
        a=a+1
        end
    end

t=timer(f,0,1000,1)
t.Enabled=false--暂停定时器
t.Enabled=true--重新定时器
t.stop()--停止定时器

luajava.bindClass(s)
参数：s class的完整名称，支持基本类型
返回：Java class对象
作用：载入Java class
Button=luajava.bindClass("android.widget.Button")
int=luajava.bindClass("int")

luajava.createProxy(s,t)
参数：s 接口的完整名称，t 接口函数表
返回：Java接口对象
作用：创建Java接口
onclick=luajava.createProxy("android.view.View$OnClickListener",{onClick=function(v)print(v)end})

luajava.createArray(s,t)
参数：s 类的完整名称，支持基本类型，t 要转化为Java数组的表
返回：创建的Java数组对象
作用：创建Java数组
arr=luajava.createArray("int",{1,2,3,4})

luajava.newInstance(s [,...])
参数：s 类的完整名称，... 构建方法的参数
作用：创建Java类的实例
b=luajava.newInstance("android.widget.Button",activity)

luajava.new(o[,...])
参数：o Java类对象，... 参数
返回：类的实例或数组对象或接口对象
作用：创建一个类实例或数组对象或接口对象
注意：当只有一个参数且为表类型时，如果类对象为interface创建接口，为class创建数组，参数为其他情况创建实例
b=luajava.new(Button,activity)
onclick=luajava.new(OnClickListener,{onClick=function(v)print(v)end})
arr=luajava.new(int,{1,2,3})
(示例中假设已载入相关类)

luajava.coding(s [,s2 [, s3]])
参数：s 要转换编码的Lua字符串，s2 字符串的原始编码，s3 字符串的目标编码
返回：转码后的Lua字符串
作用：转换字符串编码
注意：默认进行GBK转UTF8

luajava.clear(o)
参数：o Java对象
返回：无
作用：销毁Java对象
注意：仅用于销毁临时对象

luajava.astable(o)
参数：o Java对象
返回：Lua表
作用：转换Java的Array List或Map为Lua表

luajava.tostring(o)
参数：o Java对象
返回：Lua字符串
作用：相当于 o.toString()
```

### activity部分API参考

```text
setContentView(layout, env)
设置布局表layout为当前activity的主视图，env是保存视图ID的表，默认是_G
getLuaDir()
返回脚本当前目录
getLuaDir(name)
返回脚本当前目录的子目录
getLuaExtDir()
返回Androlua在SD的工作目录
getLuaExtDir(name)
返回Androlua在SD的工作目录的子目录
getWidth()
返回屏幕宽度
getHeight()
返回屏幕高度，不包括状态栏与导航栏
loadDex(path)
加载当前目录dex或jar，返回DexClassLoader
loadLib(path)
加载当前目录c模块，返回载入后模块的返回值(通常是包含模块函数的包)
registerReceiver(filter)
注册一个广播接收者，当再次调用该方法时将移除上次注册的过滤器
newActivity(req, path, enterAnim, exitAnim, arg)
打开一个新activity，运行路径为path的Lua文件，其他参数为可选，arg为表，接受脚本为变长参数
result{...}
向来源activity返回数据，在源activity的onResult回调
newTask(func[, update], callback)
新建一个Task异步任务，在线程中执行func函数，其他两个参数可选，执行结束回调callback，在任务调用update函数时在UI线程回调该函数
新建的Task在调用execute{}时通过表传入参数，在func以unpack形式接收，执行func可以返回多个值
newThread(func, arg)
新建一个线程，在线程中运行func函数，可以以表的形式传入arg，在func以unpack形式接收
新建的线程调用start()方法运行，线程为含有loop线程，在当前activity结束后自动结束loop
newTimer(func, arg)
新建一个定时器，在线程中运行func函数，可以以表的形式传入arg，在func以unpack形式接收
调用定时器的start(delay, period)开始定时器，stop()停止定时器，Enabled暂停恢复定时器，Period属性改变定时器间隔
```

### 布局表字符串常量

```lua
布局表支持属性字符串常量
    -- android:drawingCacheQuality
    auto=0,
    low=1,
    high=2,

    -- android:importantForAccessibility
    auto=0,
    yes=1,
    no=2,

    -- android:layerType
    none=0,
    software=1,
    hardware=2,

    -- android:layoutDirection
    ltr=0,
    rtl=1,
    inherit=2,
    locale=3,

    -- android:scrollbarStyle
    insideOverlay=0x0,
    insideInset=0x01000000,
    outsideOverlay=0x02000000,
    outsideInset=0x03000000,

    -- android:visibility
    visible=0,
    invisible=1,
    gone=2,

    wrap_content=-2,
    fill_parent=-1,
    match_parent=-1,
    wrap=-2,
    fill=-1,
    match=-1,

    -- android:orientation
    vertical=1,
    horizontal= 0,

    -- android:gravity
    axis_clip = 8,
    axis_pull_after = 4,
    axis_pull_before = 2,
    axis_specified = 1,
    axis_x_shift = 0,
    axis_y_shift = 4,
    bottom = 80,
    center = 17,
    center_horizontal = 1,
    center_vertical = 16,
    clip_horizontal = 8,
    clip_vertical = 128,
    display_clip_horizontal = 16777216,
    display_clip_vertical = 268435456,
    --fill = 119,
    fill_horizontal = 7,
    fill_vertical = 112,
    horizontal_gravity_mask = 7,
    left = 3,
    no_gravity = 0,
    relative_horizontal_gravity_mask = 8388615,
    relative_layout_direction = 8388608,
    right = 5,
    start = 8388611,
    top = 48,
    vertical_gravity_mask = 112,
    end = 8388613,

    -- android:textAlignment
    inherit=0,
    gravity=1,
    textStart=2,
    textEnd=3,
    textCenter=4,
    viewStart=5,
    viewEnd=6,

    -- android:inputType
    none=0x00000000,
    text=0x00000001,
    textCapCharacters=0x00001001,
    textCapWords=0x00002001,
    textCapSentences=0x00004001,
    textAutoCorrect=0x00008001,
    textAutoComplete=0x00010001,
    textMultiLine=0x00020001,
    textImeMultiLine=0x00040001,
    textNoSuggestions=0x00080001,
    textUri=0x00000011,
    textEmailAddress=0x00000021,
    textEmailSubject=0x00000031,
    textShortMessage=0x00000041,
    textLongMessage=0x00000051,
    textPersonName=0x00000061,
    textPostalAddress=0x00000071,
    textPassword=0x00000081,
    textVisiblePassword=0x00000091,
    textWebEditText=0x000000a1,
    textFilter=0x000000b1,
    textPhonetic=0x000000c1,
    textWebEmailAddress=0x000000d1,
    textWebPassword=0x000000e1,
    number=0x00000002,
    numberSigned=0x00001002,
    numberDecimal=0x00002002,
    numberPassword=0x00000012,
    phone=0x00000003,
    datetime=0x00000004,
    date=0x00000014,
    time=0x00000024,

    --android:ellipsize
    end　　
    start 　　
    middle
    marquee

相对布局rule
    layout_above=2,
    layout_alignBaseline=4,
    layout_alignBottom=8,
    layout_alignEnd=19,
    layout_alignLeft=5,
    layout_alignParentBottom=12,
    layout_alignParentEnd=21,
    layout_alignParentLeft=9,
    layout_alignParentRight=11,
    layout_alignParentStart=20,
    layout_alignParentTop=10,
    layout_alignRight=7,
    layout_alignStart=18,
    layout_alignTop=6,
    layout_alignWithParentIfMissing=0,
    layout_below=3,
    layout_centerHorizontal=14,
    layout_centerInParent=13,
    layout_centerVertical=15,
    layout_toEndOf=17,
    layout_toLeftOf=0,
    layout_toRightOf=1,
    layout_toStartOf=16



尺寸单位
    px=0,
    dp=1,
    sp=2,
    pt=3,
    in=4,
    mm=5
```

## 来源：Intent类.txt

### Intent类介绍

```text
Intent（意图）主要是解决Android应用的各项组件之间的通讯。
Intent负责对应用中一次操作的动作、动作涉及数据、附加数据进行描述.
Android则根据此Intent的描述，负责找到对应的组件，将 Intent传递给调用的组件，并完成组件的调用。

因此，Intent在这里起着一个媒体中介的作用
专门提供组件互相调用的相关信息
实现调用者与被调用者之间的解耦。

例如，在一个联系人维护的应用中，当我们在一个联系人列表屏幕(假设对应的Activity为listActivity)上
点击某个联系人后，希望能够跳出此联系人的详细信息屏幕(假设对应的Activity为detailActivity)
为了实现这个目的，listActivity需要构造一个 Intent
这个Intent用于告诉系统，我们要做“查看”动作，此动作对应的查看对象是“某联系人”
然后调用startActivity (Intent intent)，将构造的Intent传入

系统会根据此Intent中的描述到ManiFest中找到满足此Intent要求的Activity，系统会调用找到的 Activity，即为detailActivity，最终传入Intent，detailActivity则会根据此Intent中的描述，执行相应的操作。
```

### 调用浏览器搜索关键字

```lua
import "android.content.Intent"
import "android.app.SearchManager"
intent =  Intent()
intent.setAction(Intent.ACTION_WEB_SEARCH)
intent.putExtra(SearchManager.QUERY,"Alua开发手册")
activity.startActivity(intent)
```

### 调用浏览器打开网页

```lua
import "android.content.Intent"
import "android.net.Uri"
url="http://www.androlua.cn"
viewIntent =  Intent("android.intent.action.VIEW",Uri.parse(url))
activity.startActivity(viewIntent)
```

### 打开其它程序

```lua
packageName=程序包名
import "android.content.Intent"
import "android.content.pm.PackageManager"
manager = activity.getPackageManager()
open = manager.getLaunchIntentForPackage(packageName)
this.startActivity(open)
```

### 安装其它程序

```lua
import "android.content.Intent"
import "android.net.Uri"
intent = Intent(Intent.ACTION_VIEW)
安装包路径="/sdcard/a.apk"
intent.setDataAndType(Uri.parse("file://"..安装包路径), "application/vnd.android.package-archive")
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
activity.startActivity(intent)
```

### 卸载其它程序

```lua
import "android.net.Uri"
import "android.content.Intent"
包名="com.huluxia.gametools"
uri = Uri.parse("package:"..包名)
intent =  Intent(Intent.ACTION_DELETE,uri)
activity.startActivity(intent)
```

### 播放Mp4

```lua
import "android.content.Intent"
import "android.net.Uri"
intent =  Intent(Intent.ACTION_VIEW)
uri = Uri.parse("file:///sdcard/a.mp4")
intent.setDataAndType(uri, "video/mp4")
activity.startActivity(intent)
```

### 播放Mp3

```lua
import "android.content.Intent"
import "android.net.Uri"
intent =  Intent(Intent.ACTION_VIEW)
uri = Uri.parse("file:///sdcard/song.mp3")
intent.setDataAndType(uri, "audio/mp3")
this.startActivity(intent)
```

### 搜索应用

```lua
import "android.content.Intent"
import "android.net.Uri"
intent = Intent("android.intent.action.VIEW")
intent .setData(Uri.parse( "market://details?id="..activity.getPackageName()))
this.startActivity(intent)
```

### 调用系统设置

```lua
import "android.content.Intent"
import "android.provider.Settings"
intent = Intent(android.provider.Settings.ACTION_SETTINGS)
this.startActivity(intent)

字段列表:
ACTION_SETTINGS	系统设置
CTION_APN_SETTINGS APN设置
ACTION_LOCATION_SOURCE_SETTINGS 位置和访问信息
ACTION_WIRELESS_SETTINGS 网络设置
ACTION_AIRPLANE_MODE_SETTINGS 无线和网络热点设置
ACTION_SECURITY_SETTINGS 位置和安全设置
ACTION_WIFI_SETTINGS 无线网WIFI设置
ACTION_WIFI_IP_SETTINGS 无线网IP设置
ACTION_BLUETOOTH_SETTINGS 蓝牙设置
ACTION_DATE_SETTINGS 时间和日期设置
ACTION_SOUND_SETTINGS 声音设置
ACTION_DISPLAY_SETTINGS 显示设置——字体大小等
ACTION_LOCALE_SETTINGS 语言设置
ACTION_INPUT_METHOD_SETTINGS 输入法设置
ACTION_USER_DICTIONARY_SETTINGS 用户词典
ACTION_APPLICATION_SETTINGS 应用程序设置
ACTION_APPLICATION_DEVELOPMENT_SETTINGS 应用程序设置
ACTION_QUICK_LAUNCH_SETTINGS 快速启动设置
ACTION_MANAGE_APPLICATIONS_SETTINGS 已下载（安装）软件列表
ACTION_SYNC_SETTINGS 应用程序数据同步设置
ACTION_NETWORK_OPERATOR_SETTINGS 可用网络搜索
ACTION_DATA_ROAMING_SETTINGS 移动网络设置
ACTION_INTERNAL_STORAGE_SETTINGS 手机存储设置
```

### 调用系统打开文件

```lua
function OpenFile(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  if Mime then
    intent = Intent()
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
    intent.setAction(Intent.ACTION_VIEW);
    intent.setDataAndType(Uri.fromFile(File(path)), Mime);
    activity.startActivity(intent)
return true
  else
    return false
  end
end
OpenFile(文件路径)
```

### 调用图库选择图片

```lua
import "android.content.Intent"
  local intent= Intent(Intent.ACTION_PICK)
  intent.setType("image/*")
  this.startActivityForResult(intent, 1)
-------

--回调
function onActivityResult(requestCode,resultCode,intent)
  if intent then
    local cursor =this.getContentResolver ().query(intent.getData(), nil, nil, nil, nil)
    cursor.moveToFirst()
import "android.provider.MediaStore"
    local idx = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA)
    fileSrc = cursor.getString(idx)
    bit=nil
    --fileSrc回调路径路径
import "android.graphics.BitmapFactory"
    bit =BitmapFactory.decodeFile(fileSrc)
  --  iv.setImageBitmap(bit)
  end
end--nirenr
```

### 调用文件管理器选择文件

```lua
function ChooseFile()
import "android.content.Intent"
import "android.net.Uri"
import "java.net.URLDecoder"
import "java.io.File"
intent = Intent(Intent.ACTION_GET_CONTENT)
intent.setType("*/");
intent.addCategory(Intent.CATEGORY_OPENABLE)
activity.startActivityForResult(intent,1);
function onActivityResult(requestCode,resultCode,data)
  if resultCode == Activity.RESULT_OK then
  local str = data.getData().toString()
  local decodeStr = URLDecoder.decode(str,"UTF-8")
  print(decodeStr)
  end
end
end

ChooseFile()
```

### 分享文件

```lua
function Sharing(path)
  import "android.webkit.MimeTypeMap"
  import "android.content.Intent"
  import "android.net.Uri"
  import "java.io.File"
  FileName=tostring(File(path).Name)
  ExtensionName=FileName:match("%.(.+)")
  Mime=MimeTypeMap.getSingleton().getMimeTypeFromExtension(ExtensionName)
  intent = Intent()
  intent.setAction(Intent.ACTION_SEND)
  intent.setType(Mime)
  file = File(path)
  uri = Uri.fromFile(file)
  intent.putExtra(Intent.EXTRA_STREAM,uri)
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
  activity.startActivity(Intent.createChooser(intent, "分享到:"))
end

Sharing(文件路径)
```

### 发送短信

```lua
import "android.net.Uri"
import "android.content.Intent"
uri = Uri.parse("smsto:10010")
intent = Intent(Intent.ACTION_SENDTO, uri)
intent.putExtra("sms_body","cxll")
intent.setAction("android.intent.action.VIEW")
activity.startActivity(intent)
```

### 发送彩信

```lua
import "android.net.Uri"
import "android.content.Intent"
uri=Uri.parse("file:///sdcard/a.png") --图片路径
intent= Intent();
intent.setAction(Intent.ACTION_SEND);
intent.putExtra("address",mobile) --邮件地址
intent.putExtra("sms_body",content) --邮件内容
intent.putExtra(Intent.EXTRA_STREAM,uri)
intent.setType("image/png") --设置类型
this.startActivity(intent)
```

### 拨打电话

```lua
import "android.net.Uri"
import "android.content.Intent"
uri = Uri.parse("tel:10010")
intent = Intent(Intent.ACTION_CALL, uri)
intent.setAction("android.intent.action.VIEW")
activity.startActivity(intent)
```

## 来源：Lua教程.txt

### 初识AndroLua

```text
AndroLua可以在安卓平台上的用 Lua 开发安卓程序，不仅支持调用Java API，而且支持编写安卓界面程序，还可以将自己写的 Lua 程序打包成apk安装文件安装。Lua 语言的简单使没有任何编程经验的用户也能在短时间内开发出安卓程序，因此，在学习AndroLua之前我们需要先学习 Lua 语言。
```

### Lua简介

```text
Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。
Lua 是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。
那么我们废话不多说来写第一个 Lua 程序吧！
```

### 第一个 Lua 程序

```lua
接下来我们使用 Lua 来输出"Hello World"

print("Hello World")

运行后，会在屏幕上显示 Hello world
```

### 注释

```lua
单行注释:
两个减号是单行注释:

--注释
```

### 标示符

```text
Lua 表示符用于定义一个变量，函数获取其他用户定义的项。标示符以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头后加上0个或多个字母，下划线，数字（0到9）。
最好不要使用下划线加大写字母的标示符，因为Lua的保留字也是这样的。
Lua 不允许使用特殊字符如 @, $, 和 % 来定义标示符。 Lua 是一个区分大小写的编程语言。因此在 Lua 中 W3c 与 w3c 是两个不同的标示符。以下列出了一些正确的标示符：

mohd         zara      abc     move_name

myname50     _temp     j       a23b9
```

### 关键词

```lua
以下列出了 Lua 的保留关键字。保留关键字不能作为常量或变量或其他用户自定义标示符：

and      break	     do      else    elseif   end       false
for      function  if      in      local    nil	       not
or	      repeat    return	 then    true     until     while

一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量。
```

### 全局变量

```lua
在默认情况下，变量总是认为是全局的。
全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil。

print(b)
--nil
b=10
print(b)
--10

如果你想删除一个全局变量，只需要将变量赋值为nil。

b = 2
b = nil
print(b)
--nil

这样变量b就好像从没被使用过一样。换句话说, 当且仅当一个变量不等于nil时，这个变量即存在。
```

### Lua 数据类型

```lua
Lua是动态类型语言，变量不要类型定义,只需要为变量赋值。 值可以存储在变量中，作为参数传递或结果返回。
Lua中有8个基本类型分别为：

nil、boolean、number、string、userdata、function、thread和table。

我们可以使用type函数测试给定变量或者值的类型：

print(type("Hello world"))
--string
print(type(10.4*3))
--number
print(type(print))
--function
print(type(type))
--function
print(type(true))
--boolean
print(type(nil))
--nil
print(type(type(X)))
--string
```

### nil（空）

```lua
nil 类型表示一种没有任何有效值，它只有一个值 -- nil，例如打印一个没有赋值的变量，便会输出一个 nil 值：

print(type(a))
--nil

对于全局变量和 table，nil 还有一个"删除"作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉。
```

### boolean（布尔）

```text
boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是"假"，其他的都为"真"。
```

### number（数字）

```text
Lua 默认只有一种 number 类型 -- double（双精度）类型（默认类型可以修改 luaconf.h 里的定义），以下几种写法都被看作是 number 类型。
```

### string（字符串）

```lua
字符串由一对双引号或单引号来表示。

string1 = "this is string1"
string2 = 'this is string2'

在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字，字符串连接使用的是 ..如：

print("a" .. 'b')
--ab
print(157 .. 428)
--157428

使用 # 来计算字符串的长度，放在字符串前面，如下实例：

len = "www.androlua.com"
print(#len)
--16
```

### table（表）

```lua
在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表:

-- 创建一个空的 table
local tbl1 = {}

-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}

Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串。不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始。table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。
```

### function（函数）

```lua
在 Lua 中，函数是被看作是"第一类值（First-Class Value）"，函数可以存在变量里:

function factorial1(n)
    if n == 0 then
        return 1
    else
        return n * factorial1(n - 1)
    end
end
print(factorial1(5))
factorial2 = factorial1
print(factorial2(5))

function 可以以匿名函数（anonymous function）的方式通过参数传递:

function anonymous(tab, fun)
    for k, v in pairs(tab) do
        print(fun(k, v))
    end
end
tab = { key1 = "val1", key2 = "val2" }
anonymous(tab, function(key, val)
    return key .. " = " .. val
end)
```

### thread（线程）

```text
在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。
线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。
```

### userdata（自定义类型）

```text
userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。
```

### Lua 变量

```lua
变量在使用前，必须在代码中进行声明，即创建该变量。
编译程序执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。
Lua 变量有三种类型：全局变量、局部变量、表中的域。
Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显示声明为局部变量。
局部变量的作用域为从声明位置开始到所在语句块结束。
变量的默认值均为 nil。

a = 5               -- 全局变量
local b = 5         -- 局部变量

function joke()
    c = 5           -- 全局变量
    local d = 6     -- 局部变量
end
```

### 赋值语句

```lua
赋值是改变一个变量的值和改变表域的最基本的方法。

a = "hello" .. "world"
t.n = t.n + 1

Lua可以对多个变量同时赋值，变量列表和值列表的各个元素用逗号分开，赋值语句右边的值会依次赋给左边的变量。

a, b = 10, 2*x       <-->       a=10; b=2*x

遇到赋值语句Lua会先计算右边所有的值然后再执行赋值操作，所以我们可以这样进行交换变量的值：

x, y = y, x                     -- swap 'x' for 'y'
a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'

当变量个数和值的个数不一致时，Lua会一直以变量个数为基础采取以下策略：
a. 变量个数 > 值的个数             按变量个数补足nil
b. 变量个数 < 值的个数             多余的值会被忽略
例如：

a, b, c = 0, 1
print(a,b,c)             --> 0   1   nil

a, b = a+1, b+1, b+2     -- value of b+2 is ignored
print(a,b)               --> 1   2

a, b, c = 0
print(a,b,c)             --> 0   nil   nil

上面最后一个例子是一个常见的错误情况，注意：如果要对多个变量赋值必须依次对每个变量赋值。

a, b, c = 0, 0, 0
print(a,b,c)             --> 0   0   0

多值赋值经常用来交换变量，或将函数调用返回给变量：

a, b = f()
f()

返回两个值，第一个赋给a，第二个赋给b。
应该尽可能的使用局部变量，有两个好处：
1. 避免命名冲突。
2. 访问局部变量的速度比全局变量更快。
```

### 索引

```lua
对 table 的索引使用方括号 []。Lua 也提供了 . 操作。

t[i]
t.i                 -- 当索引为字符串类型时的一种简化写法
gettable_event(t,i) -- 采用索引访问本质上是一个类似这样的函数调用

例如：

site = {}
site["key"] = "www.androlua.cn"
print(site["key"])
--www.androlua.cn
print(site.key)
--www.androlua.cn
```

### Lua 循环

```lua
很多情况下我们需要做一些有规律性的重复操作，因此在程序中就需要重复执行某些语句。
一组被重复执行的语句称之为循环体，能否继续重复，决定循环的终止条件。
循环结构是在一定条件下反复执行某段程序的流程结构，被反复执行的程序被称为循环体。
循环语句是由循环体及循环的终止条件两部分组成的。
Lua 语言提供了以下几种循环处理方式：

while 循环
在条件为 true 时，让程序重复地执行某些语句。执行语句前会先检查条件是否为 true。

for 循环
重复执行指定语句，重复次数可在 for 语句中控制。

Lua repeat...until
重复执行循环，直到 指定的条件为真时为止

循环嵌套
可以在循环内嵌套一个或多个循环语句（while、for、do..while）
```

### 循环控制语句

```text
循环控制语句用于控制程序的流程， 以实现程序的各种结构方式。
Lua 支持以下循环控制语句：

break 语句
退出当前循环或语句，并开始脚本执行紧接着的语句。
```

### 无限循环

```lua
在循环体中如果条件永远为 true 循环语句就会永远执行下去，以下以 while 循环为例：

while( true )
do
   print("循环将永远执行下去")
end
```

### Lua 流程控制

```lua
Lua 编程语言流程控制语句通过程序设定一个或多个条件语句来设定。在条件为 true 时执行指定程序代码，在条件为 false 时执行其他指定代码。
控制结构的条件表达式结果可以是任何值，Lua认为false和nil为假，true和非nil为真。
要注意的是Lua中 0 为 true

Lua 提供了以下控制结构语句：
if 语句
if 语句由一个布尔表达式作为条件判断，其后紧跟其他语句组成

if...else 语句
if 语句 可以与 else 语句搭配使用, 在 if 条件表达式为 false 时执行 else 语句代码

if 嵌套语句
你可以在if 或 else if中使用一个或多个 if 或 else if 语句
```

### Lua 函数

```lua
在Lua中，函数是对语句和表达式进行抽象的主要方法。既可以用来处理一些特殊的工作，也可以用来计算一些值。
Lua 提供了许多的内建函数，你可以很方便的在程序中调用它们，如print()函数可以将传入的参数打印在控制台上。
Lua 函数主要有两种用途：
1.完成指定的任务，这种情况下函数作为调用语句使用；
2.计算并返回值，这种情况下函数作为赋值语句的表达式使用。
以下实例定义了函数 max()，参数为 num1, num2，用于比较两值的大小，并返回最大值：

function max(num1, num2)

   if (num1 > num2) then
      result = num1;
   else
      result = num2;
   end

   return result;
end
-- 调用函数
print("两值比较最大值为 ",max(10,4))
print("两值比较最大值为 ",max(5,6))

Lua 中我们可以将函数作为参数传递给函数，如下实例：

myprint = function(param)
   print("这是打印函数 -   ##",param,"##")
end

function add(num1,num2,functionPrint)
   result = num1 + num2
   -- 调用传递的函数参数
   functionPrint(result)
end
myprint(10)
-- myprint 函数作为参数传递
add(2,5,myprint)

Lua函数中，在return后列出要返回的值得列表即可返回多值，如：


function maximum (a)
    local mi = 1             -- 最大值索引
    local m = a[mi]          -- 最大值
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end

print(maximum({8,10,23,12,5}))

Lua函数可以接受可变数目的参数，和C语言类似在函数参数列表中使用三点（...) 表示函数有可变的参数。
Lua将函数的参数放在一个叫arg的表中，#arg 表示传入参数的个数。
例如，我们计算几个数的平均值：

function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```

### Lua 运算符

```text
运算符是一个特殊的符号，用于告诉解释器执行特定的数学或逻辑运算。Lua提供了以下几种运算符类型：

算术运算符,关系运算符,逻辑运算符,其他运算符

算术运算符
下表列出了 Lua 语言中的常用算术运算符，设定 A 的值为10，B 的值为 20：

+	加法	A + B 输出结果 30
-	减法	A - B 输出结果 -10
*	乘法	A * B 输出结果 200
/	除法	B / A 输出结果 2
%	取余	B % A 输出结果 0
^	乘幂	A^2   输出结果 100
-	负号	-A    输出结果v -10

关系运算符
下表列出了 Lua 语言中的常用关系运算符，设定 A 的值为10，B 的值为 20：


==	等于，检测两个值是否相等，相等返回 true，
否则返回 false	(A == B) 为 false。
~=	不等于，检测两个值是否相等，相等返回 false，
否则返回 true<	(A ~= B) 为 true。
>	大于，如果左边的值大于右边的值，返回 true，
否则返回 false	(A > B) 为 false。
<	小于，如果左边的值大于右边的值，返回 false，
否则返回 true	(A < B) 为 true。
>=	大于等于，如果左边的值大于等于右边的值，返回 true，
否则返回 false	(A >= B) 返回 false。
<=	小于等于， 如果左边的值小于等于右边的值，返回 true，
否则返回 false	(A <= B) 返回 true。

逻辑运算符
下表列出了 Lua 语言中的常用逻辑运算符，设定 A 的值为 true，B 的值为 false：

and	逻辑与操作符。
如果两边的操作都为 true 则条件为 true。
(A and B) 为 false。
or	逻辑或操作符。
如果两边的操作任一一个为 true 则条件为 true。
(A or B) 为 true。
not	逻辑非操作符。
与逻辑运算结果相反，如果条件为 true，逻辑非为 false。
not(A and B) 为 true。

其他运算符
下表列出了 Lua 语言中的连接运算符与计算表或字符串长度的运算符：

..	连接两个字符串	a..b
#	一元运算符，返回字符串或表的长度。

运算符优先级
从高到低的顺序：


^
not    - (unary)
*      /
+      -
..
<      >      <=     >=     ~=     ==
and
or
```

### Lua 字符串

```lua
字符串或串(String)是由数字、字母、下划线组成的一串字符。
Lua 语言中字符串可以使用以下三种方式来表示：

单引号间的一串字符。
双引号间的一串字符。
[[和]]间的一串字符。

以上三种方式的字符串实例如下：

string1 = "ALua手册"
print("\"字符串 1 是\"",string1)
--字符串 1 是	ALua手册
string2 = 'androlua.cn'
--字符串 2 是	androlua.cn
print("字符串 2 是",string2)
string3 = [["Lua 教程"]]
print("字符串 3 是",string3)
--字符串 3 是	"Lua 教程"
```

### Lua 数组

```text
数组，就是相同数据类型的元素按一定顺序排列的集合，可以是一维数组和多维数组。
Lua 数组的索引键值可以使用整数表示，数组的大小不是固定的。
```

### 一维数组

```lua
一维数组是最简单的数组，其逻辑结构是线性表。一维数组可以用for循环出数组中的元素，如下实例：

array = {"Lua", "Tutorial"}
for i= 0, 2 do
   print(array[i])
end

以上代码执行输出结果为：

nil
Lua
Tutorial

正如你所看到的，我们可以使用整数索引来访问数组元素，如果知道的索引没有值则返回nil。
在 Lua 索引值是以 1 为起始，但你也可以指定 0 开始。
```

### Lua 迭代器

```text
迭代器（iterator）是一种对象，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址

在Lua中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素。
```

### 泛型 for 迭代器

```lua
泛型 for 在自己内部保存迭代函数，实际上它保存三个值：迭代函数、状态常量、控制变量。

泛型 for 迭代器提供了集合的 key/value 对，语法格式如下：

for k, v in pairs(t) do
    print(k, v)
end
上面代码中，k, v为变量列表；pair(t)为表达式列表。

查看以下实例:

array = {"Lua", "Tutorial"}

for key,value in ipairs(array)
do
   print(key, value)
end
以上代码执行输出结果为：

Lua
Tutorial

以上实例中我们使用了 Lua 默认提供的迭代函数 ipairs。

下面我们看看范性for的执行过程：

首先，初始化，计算in后面表达式的值，表达式应该返回范性for需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用nil补足，多出部分会被忽略。
第二，将状态常量和控制变量作为参数调用迭代函数（注意：对于for结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
第三，将迭代函数返回的值赋给变量列表。
第四，如果返回的第一个值为nil循环结束，否则执行循环体。
第五，回到第二步再次调用迭代函数
。在Lua中我们常常使用函数来描述迭代器，每次调用该函数就返回集合的下一个元素。Lua 的迭代器包含以下两种类型：

1，无状态的迭代器
2，多状态的迭代器
```

### 无状态的迭代器

```lua
无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。

每一次迭代，迭代函数都是用两个变量（状态常量和控制变量）的值作为参数被调用，一个无状态的迭代器只利用这两个值可以获取下一个元素。

这种无状态迭代器的典型的简单的例子是ipairs，他遍历数组的每一个元素。

以下实例我们使用了一个简单的函数来实现迭代器，实现 数字 n 的平方：

function square(iteratorMaxCount,currentNumber)
   if currentNumber<iteratorMaxCount
   then
      currentNumber = currentNumber+1
   return currentNumber, currentNumber*currentNumber
   end
end

for i,n in square,3,0
do
   print(i,n)
end

以上实例输出结果为：

1
4
9

迭代的状态包括被遍历的表（循环过程中不会改变的状态常量）和当前的索引下标（控制变量），ipairs和迭代函数都很简单，我们在Lua中可以这样实现：

function iter (a, i)
    i = i + 1
    local v = a[i]
    if v then
       return i, v
    end
end

function ipairs (a)
    return iter, a, 0
end

当Lua调用ipairs(a)开始循环时，他获取三个值：迭代函数iter、状态常量a、控制变量初始值0；然后Lua调用iter(a,0)返回1,a[1]（除非a[1]=nil）；第二次迭代调用iter(a,1)返回2,a[2]……直到第一个nil元素。
```

### 多状态的迭代器

```lua
很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到table内，将table作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在table内，所以迭代函数通常不需要第二个参数。

以下实例我们创建了自己的迭代器：

array = {"Lua", "Tutorial"}

function elementIterator (collection)
   local index = 0
   local count = #collection
   -- 闭包函数
   return function ()
      index = index + 1
      if index <= count
      then
         --  返回迭代器的当前元素
         return collection[index]
      end
   end
end

for element in elementIterator(array)
do
   print(element)
end

以上实例输出结果为：

Lua
Tutorial

以上实例中我们可以看到，elementIterator 内使用了闭包函数，实现计算集合大小并输出各个元素。
```

### Lua 文件 I/O

```lua
Lua I/O 库用于读取和处理文件。分为简单模式（和C一样）、完全模式。

简单模式（simple model）拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作。
完全模式（complete model） 使用外部的文件句柄来实现。它以一种面对对象的形式，将所有的文件操作定义为文件句柄的方法
简单模式在做一些简单的文件操作时较为合适。但是在进行一些高级的文件操作的时候，简单模式就显得力不从心。例如同时读取多个文件这样的操作，使用完全模式则较为合适。

打开文件操作语句如下：

file = io.open (filename , mode)
mode 的值有：

"r"	以只读方式打开文件，该文件必须存在。
"w"	打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。
"a"	以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）
"r+"	以可读写方式打开文件，该文件必须存在。
"w+"	打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。
"a+"	与a类似，但此文件可读可写
"b"	二进制模式，如果文件是二进制文件，可以加上b
```

### I/O 简单模式

```lua
简单模式使用标准的 I/O 或使用一个当前输入文件和一个当前输出文件。

以下为 file.lua 文件代码，操作的文件为test.lua(如果没有你需要创建该文件)，代码如下：

-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 设置默认输入文件为 test.lua
io.input(file)

-- 输出文件第一行
print(io.read())

-- 关闭打开的文件
io.close(file)

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 设置默认输出文件为 test.lua
io.output(file)

-- 在文件最后一行添加 Lua 注释
io.write("--  test.lua 文件末尾注释")

-- 关闭打开的文件
io.close(file)
执行以上代码，你会发现，输出了 test.ua 文件的第一行信息，并在该文件最后一行添加了 lua 的注释。如我这边输出的是：

-- test.lua 文件
在以上实例中我们使用了 io."x" 方法，其中 io.read() 中我们没有带参数，参数可以是下表中的一个：

"*n"	读取一个数字并返回它。例：file.read("*n")
"*a"	从当前位置读取整个文件。例：file.read("*a")
"*l"（默认）	读取下一行，在文件尾 (EOF) 处返回 nil。例：file.read("*l")
number	返回一个指定字符个数的字符串，或在 EOF 时返回 nil。例：file.read(5)
其他的 io 方法有：

io.tmpfile():返回一个临时文件句柄，该文件以更新模式打开，程序结束时自动删除

io.type(file): 检测obj是否一个可用的文件句柄

io.flush(): 向文件写入缓冲中的所有数据

io.lines(optional file name): 返回一个迭代函数,每次调用将获得文件中的一行内容,当到文件尾时，将返回nil,但不关闭文件
```

### I/O 完全模式

```lua
通常我们需要在同一时间处理多个文件。我们需要使用 file:function_name 来代替 io.function_name 方法。以下实例演示了如同同时处理同一个文件:

-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 输出文件第一行
print(file:read())

-- 关闭打开的文件
file:close()

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 在文件最后一行添加 Lua 注释
file:write("--test")

-- 关闭打开的文件
file:close()
执行以上代码，你会发现，输出了 test.ua 文件的第一行信息，并在该文件最后一行添加了 lua 的注释。如我这边输出的是：

-- test.lua 文件
read 的参数与简单模式一致。
```

### I/O 的其他方法

```lua
file:seek(optional whence, optional offset): 设置和获取当前文件位置,成功则返回最终的文件位置(按字节),失败则返回nil加错误信息。参数 whence 值可以是:

"set": 从文件头开始
"cur": 从当前位置开始[默认]
"end": 从文件尾开始
offset:默认为0
不带参数file:seek()则返回当前位置,file:seek("set")则定位到文件头,file:seek("end")则定位到文件尾并返回文件大小
file:flush(): 向文件写入缓冲中的所有数据

io.lines(optional file name): 打开指定的文件filename为读模式并返回一个迭代函数,每次调用将获得文件中的一行内容,当到文件尾时，将返回nil,并自动关闭文件。
若不带参数时io.lines() <=> io.input():lines(); 读取默认输入设备的内容，但结束时不关闭文件,如

for line in io.lines("main.lua") do

　　print(line)

　　end

以下实例使用了 seek 方法，定位到文件倒数第 25 个位置并使用 read 方法的 *a 参数，即从当期位置(倒数第 25 个位置)读取整个文件。

-- 以只读方式打开文件
file = io.open("test.lua", "r")

file:seek("end",-25)
print(file:read("*a"))

-- 关闭打开的文件
file:close()
我这边输出的结果是：

st.lua 文件末尾--test
```

### Lua 错误处理

```text
程序运行中错误处理是必要的，在我们进行文件操作，数据转移及web service 调用过程中都会出现不可预期的错误。如果不注重错误信息的处理，就会照成信息泄露，程序无法运行等情况。
任何程序语言中，都需要错误处理。错误类型有：
1，语法错误
2，运行错误
```

### 语法错误

```lua
语法错误通常是由于对程序的组件（如运算符、表达式）使用不当引起的。一个简单的实例如下：
-- test.lua 文件
a == 2
以上代码执行结果为：

lua: test.lua:2: syntax error near '=='

正如你所看到的，以上出现了语法错误，一个 "=" 号跟两个 "=" 号是有区别的。一个 "=" 是赋值表达式两个 "=" 是比较运算。
另外一个实例:

for a= 1,10
   print(a)
end

执行以上程序会出现如下错误：

lua: test2.lua:2: 'do' expected near 'print'

语法错误比程序运行错误更简单，运行错误无法定位具体错误，而语法错误我们可以很快的解决，如以上实例我们只要在for语句下添加 do 即可：

for a= 1,10
do
   print(a)
end
```

### 运行错误

```lua
运行错误是程序可以正常执行，但是会输出报错信息。如下实例由于参数输入错误，程序执行时报错：

function add(a,b)
   return a+b
end

add(10)

当我们编译运行以下代码时，编译是可以成功的，但在运行的时候会产生如下错误：

lua: test2.lua:2: attempt to perform arithmetic on local 'b' (a nil value)
stack traceback:
    test2.lua:2: in function 'add'
    test2.lua:5: in main chunk
    [C]: ?

以下报错信息是由于程序缺少 b 参数引起的。
```

### 错误处理

```lua
我们可以使用两个函数：assert 和 error 来处理错误。实例如下：

local function add(a,b)
   assert(type(a) == "number", "a 不是一个数字")
   assert(type(b) == "number", "b 不是一个数字")
   return a+b
end
add(10)

执行以上程序会出现如下错误：

lua: test.lua:3: b 不是一个数字
stack traceback:
    [C]: in function 'assert'
    test.lua:3: in local 'add'
    test.lua:6: in main chunk
    [C]: in ?

实例中assert首先检查第一个参数，若没问题，assert不做任何事情；否则，assert以第二个参数作为错误信息抛出。
```

### error函数

```lua
语法格式：

error (message [, level])

功能：终止正在执行的函数，并返回message的内容作为错误信息(error函数永远都不会返回)
通常情况下，error会附加一些错误位置的信息到message头部。
Level参数指示获得错误的位置:
Level=1[默认]：为调用error位置(文件+行号)
Level=2：指出哪个调用error的函数的函数
Level=0:不添加错误位置信息
```

### pcall 和 xpcall、debug

```lua
Lua中处理错误，可以使用函数pcall（protected call）来包装需要执行的代码。
pcall接收一个函数和要传递个后者的参数，并执行，执行结果：有错误、无错误；返回值true或者或false, errorinfo。
语法格式如下

if pcall(function_name, ….) then
-- 没有错误
else
-- 一些错误
end

简单实例：

> =pcall(function(i) print(i) end, 33)
33
true

> =pcall(function(i) print(i) error('error..') end, 33)
33
false        stdin:1: error..
> function f() return false,2 end
> if f() then print '1' else print '0' end
0

pcall以一种"保护模式"来调用第一个参数，因此pcall可以捕获函数执行中的任何错误。
通常在错误发生时，希望落得更多的调试信息，而不只是发生错误的位置。但pcall返回时，它已经销毁了调用桟的部分内容。
Lua提供了xpcall函数，xpcall接收第二个参数——一个错误处理函数，当错误发生时，Lua会在调用桟展看（unwind）前调用错误处理函数，于是就可以在这个函数中使用debug库来获取关于错误的额外信息了。
debug库提供了两个通用的错误处理函数:

debug.debug：提供一个Lua提示符，让用户来价差错误的原因
debug.traceback：根据调用桟来构建一个扩展的错误消息

>=xpcall(function(i) print(i) error('error..') end, function() print(debug.traceback()) end, 33) 33 stack traceback: stdin:1: in function [C]: in function 'error' stdin:1: in function [C]: in function 'xpcall' stdin:1: in main chunk [C]: in ? false nil

xpcall 使用实例 2:

function myfunction ()
   n = n/nil
end

function myerrorhandler( err )
   print( "ERROR:", err )
end

status = xpcall( myfunction, myerrorhandler )
print( status)

执行以上程序会出现如下错误：

ERROR:    test2.lua:2: attempt to perform arithmetic on global 'n' (a nil value)
false
```

### Lua 调试(Debug)

```lua
Lua 提供了 debug 库用于提供创建我们自定义调速器的功能。Lua 本身并未有内置的调速器，但很多开发者共享了他们的 Lua 调速器代码。
Lua 中 debug 库包含以下函数：

sethook ([thread,] hook, mask [, count]):

1.debug():
进入一个用户交互模式，运行用户输入的每个字符串。 使用简单的命令以及其它调试设置，用户可以检阅全局变量和局部变量， 改变变量的值，计算一些表达式，等等。
输入一行仅包含 cont 的字符串将结束这个函数， 这样调用者就可以继续向下运行。

2.getfenv(object):
返回对象的环境变量。

3.gethook(optional thread):
返回三个表示线程钩子设置的值： 当前钩子函数，当前钩子掩码，当前钩子计数

4.getinfo ([thread,] f [, what]):
返回关于一个函数信息的表。 你可以直接提供该函数， 也可以用一个数字 f 表示该函数。 数字 f 表示运行在指定线程的调用栈对应层次上的函数： 0 层表示当前函数（getinfo 自身）； 1 层表示调用 getinfo 的函数 （除非是尾调用，这种情况不计入栈）；等等。 如果 f 是一个比活动函数数量还大的数字， getinfo 返回 nil。

5.debug.getlocal ([thread,] f, local):
此函数返回在栈的 f 层处函数的索引为 local 的局部变量 的名字和值。 这个函数不仅用于访问显式定义的局部变量，也包括形参、临时变量等。

6.getmetatable(value):
把给定索引指向的值的元表压入堆栈。如果索引无效，或是这个值没有元表，函数将返回 0 并且不会向栈上压任何东西。

7.getregistry():
返回注册表表，这是一个预定义出来的表， 可以用来保存任何 C 代码想保存的 Lua 值。

8.getupvalue (f, up)
此函数返回函数 f 的第 up 个上值的名字和值。 如果该函数没有那个上值，返回 nil 。
以 '(' （开括号）打头的变量名表示没有名字的变量 （去除了调试信息的代码块）。

10.将一个函数作为钩子函数设入。 字符串 mask 以及数字 count 决定了钩子将在何时调用。 掩码是由下列字符组合成的字符串，每个字符有其含义：
'c': 每当 Lua 调用一个函数时，调用钩子；
'r': 每当 Lua 从一个函数内返回时，调用钩子；
'l': 每当 Lua 进入新的一行时，调用钩子。

11.setlocal ([thread,] level, local, value):
这个函数将 value 赋给 栈上第 level 层函数的第 local 个局部变量。 如果没有那个变量，函数返回 nil 。 如果 level 越界，抛出一个错误。

12.setmetatable (value, table):
将 value 的元表设为 table （可以是 nil）。 返回 value。

13.setupvalue (f, up, value):
这个函数将 value 设为函数 f 的第 up 个上值。 如果函数没有那个上值，返回 nil 否则，返回该上值的名字。

14.traceback ([thread,] [message [, level]]):
如果 message 有，且不是字符串或 nil， 函数不做任何处理直接返回 message。 否则，它返回调用栈的栈回溯信息。 字符串可选项 message 被添加在栈回溯信息的开头。 数字可选项 level 指明从栈的哪一层开始回溯 （默认为 1 ，即调用 traceback 的那里）。

上表列出了我们常用的调试函数，接下来我们可以看些简单的例子：

function myfunction ()
print(debug.traceback("Stack trace"))
print(debug.getinfo(1))
print("Stack trace end")
    return 10
end
myfunction ()
print(debug.getinfo(1))

执行以上代码输出结果为：

Stack trace
stack traceback:
    test2.lua:2: in function 'myfunction'
    test2.lua:8: in main chunk
    [C]: ?
table: 0054C6C8
Stack trace end

在以实例中，我们使用到了 debug 库的 traceback 和 getinfo 函数， getinfo 函数用于返回函数信息的表。
```

### 调试函数的另一个实例

```lua
我们经常需要调试函数的内的局部变量。我们可以使用 getupvalue 函数来设置这些局部变量。实例如下：

function newCounter ()
  local n = 0
  local k = 0
  return function ()
    k = n
    n = n + 1
    return n
    end
end

counter = newCounter ()
print(counter())
print(counter())

local i = 1

repeat
  name, val = debug.getupvalue(counter, i)
  if name then
    print ("index", i, name, "=", val)
    if(name == "n") then
        debug.setupvalue (counter,2,10)
    end
    i = i + 1
  end -- if
until not name

print(counter())

执行以上代码输出结果为：

1
2
index    1    k    =    1
index    2    n    =    2
11

在以上实例中，计数器在每次调用时都会自增1。实例中我们使用了 getupvalue 函数查看局部变量的当前状态。我们可以设置局部变量为新值。实例中，在设置前 n 的值为 2,使用 setupvalue 函数将其设置为 10。现在我们调用函数，执行后输出为 11 而不是 3。
```

### 调试类型

```text
1，命令行调试
2，图形界面调试

命令行调试器有：RemDebug、clidebugger、ctrace、xdbLua、LuaInterface - Debugger、Rldb、ModDebug。
图形界调试器有：SciTE、Decoda、ZeroBrane Studio、akdebugger、luaedit。
```

### Lua 垃圾回收

```text
Lua 采用了自动内存管理。 这意味着你不用操心新创建的对象需要的内存如何分配出来， 也不用考虑在对象不再被使用后怎样释放它们所占用的内存。
Lua 运行了一个垃圾收集器来收集所有死对象 （即在 Lua 中不可能再访问到的对象）来完成自动内存管理的工作。 Lua 中所有用到的内存，如：字符串、表、用户数据、函数、线程、 内部结构等，都服从自动管理。
Lua 实现了一个增量标记-扫描收集器。 它使用这两个数字来控制垃圾收集循环： 垃圾收集器间歇率和垃圾收集器步进倍率。 这两个数字都使用百分数为单位 （例如：值 100 在内部表示 1 ）。
垃圾收集器间歇率控制着收集器需要在开启新的循环前要等待多久。 增大这个值会减少收集器的积极性。 当这个值比 100 小的时候，收集器在开启新的循环前不会有等待。 设置这个值为 200 就会让收集器等到总内存使用量达到 之前的两倍时才开始新的循环。
垃圾收集器步进倍率控制着收集器运作速度相对于内存分配速度的倍率。 增大这个值不仅会让收集器更加积极，还会增加每个增量步骤的长度。 不要把这个值设得小于 100 ， 那样的话收集器就工作的太慢了以至于永远都干不完一个循环。 默认值是 200 ，这表示收集器以内存分配的"两倍"速工作。
如果你把步进倍率设为一个非常大的数字 （比你的程序可能用到的字节数还大 10% ）， 收集器的行为就像一个 stop-the-world 收集器。 接着你若把间歇率设为 200 ， 收集器的行为就和过去的 Lua 版本一样了： 每次 Lua 使用的内存翻倍时，就做一次完整的收集。
```

### 垃圾回收器函数

```lua
Lua 提供了以下函数collectgarbage ([opt [, arg]])用来控制自动内存管理:

1， collectgarbage("collect"): 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：
2， collectgarbage("count"): 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。
3， collectgarbage("restart"): 重启垃圾收集器的自动运行。
4， collectgarbage("setpause"): 将 arg 设为收集器的 间歇率 （参见 §2.5）。 返回 间歇率 的前一个值。
5， collectgarbage("setstepmul"): 返回 步进倍率 的前一个值。
6， collectgarbage("step"): 单步运行垃圾收集器。 步长"大小"由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。
7， collectgarbage("stop"): 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。

以下演示了一个简单的垃圾回收实例:

mytable = {"apple", "orange", "banana"}

print(collectgarbage("count"))

mytable = nil

print(collectgarbage("count"))

print(collectgarbage("collect"))

print(collectgarbage("count"))

执行以上程序，输出结果如下(注意内存使用的变化)：

20.9560546875
20.9853515625
0
19.4111328125
```

### Lua 面向对象

```text
面向对象编程（Object Oriented Programming，OOP）是一种非常流行的计算机编程架构。
以下几种编程语言都支持面向对象编程：

C++
Java
Objective-C
Smalltalk
C#
Ruby
```

### 面向对象特征

```text
1， 封装：指能够把一个实体的信息、功能、响应都装入一个单独的对象中的特性。
2， 继承：继承的方法允许在不改动原程序的基础上对其进行扩充，这样使得原功能得以保存，而新功能也得以扩展。这有利于减少重复编码，提高软件的开发效率。
3， 多态：同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。在运行时，可以通过指向基类的指针，来调用实现派生类中的方法。
4，抽象：抽象(Abstraction)是简化复杂的现实问题的途径，它可以为具体问题找到最恰当的类定义，并且可以在最恰当的继承级别解释问题。
```

### Lua 中面向对象

```lua
我们知道，对象由属性和方法组成。LUA中最基本的结构是table，所以需要用table来描述对象的属性。
lua中的function可以用来表示方法。那么LUA中的类可以通过table + function模拟出来。
至于继承，可以通过metetable模拟出来（不推荐用，只模拟最基本的对象大部分时间够用了）。

Lua中的表不仅在某种意义上是一种对象。像对象一样，表也有状态（成员变量）；也有与对象的值独立的本性，特别是拥有两个不同值的对象（table）代表两个不同的对象；一个对象在不同的时候也可以有不同的值，但他始终是一个对象；与对象类似，表的生命周期与其由什么创建、在哪创建没有关系。对象有他们的成员函数，表也有：

Account = {balance = 0}
function Account.withdraw (v)
    Account.balance = Account.balance - v
end

这个定义创建了一个新的函数，并且保存在Account对象的withdraw域内，下面我们可以这样调用：

Account.withdraw(100.00)

一个简单实例
以下简单的类包含了三个属性： area, length 和 breadth，printArea方法用于打印计算结果：

-- Meta class
Rectangle = {area = 0, length = 0, breadth = 0}
-- 派生类的方法 new
function Rectangle:new (o,length,breadth)
  o = o or {}
  setmetatable(o, self)
  self.__index = self
  self.length = length or 0
  self.breadth = breadth or 0
  self.area = length*breadth;
  return o
end
-- 派生类的方法 printArea
function Rectangle:printArea ()
  print("矩形面积为 ",self.area)
end

创建对象
创建对象是位类的实例分配内存的过程。每个类都有属于自己的内存并共享公共数据。
r = Rectangle:new(nil,10,20)

访问属性
我们可以使用点号(.)来访问类的属性：
print(r.length)

访问成员函数
我们可以使用冒号 : 来访问类的成员函数：
r:printArea()
内存在对象初始化时分配。

完整实例

以下我们演示了 Lua 面向对象的完整实例：

-- Meta class
Shape = {area = 0}

-- 基础类方法 new
function Shape:new (o,side)
  o = o or {}
  setmetatable(o, self)
  self.__index = self
  side = side or 0
  self.area = side*side;
  return o
end

-- 基础类方法 printArea
function Shape:printArea ()
  print("面积为 ",self.area)
end

-- 创建对象
myshape = Shape:new(nil,10)

myshape:printArea()

执行以上程序，输出结果为：

面积为     100
```

### Lua 继承

```lua
继承是指一个对象直接使用另一对象的属性和方法。可用于扩展基础类的属性和方法。
以下演示了一个简单的继承实例：

 -- Meta class
Shape = {area = 0}
-- 基础类方法 new
function Shape:new (o,side)
  o = o or {}
  setmetatable(o, self)
  self.__index = self
  side = side or 0
  self.area = side*side;
  return o
end
-- 基础类方法 printArea
function Shape:printArea ()
  print("面积为 ",self.area)
end

接下来的实例，Square 对象继承了 Shape 类:

Square = Shape:new()
-- Derived class method new
function Square:new (o,side)
  o = o or Shape:new(o,side)
  setmetatable(o, self)
  self.__index = self
  return o
end

完整实例
以下实例我们继承了一个简单的类，来扩展派生类的方法，派生类中保留了继承类的成员变量和方法：

 -- Meta class
Shape = {area = 0}
-- 基础类方法 new
function Shape:new (o,side)
  o = o or {}
  setmetatable(o, self)
  self.__index = self
  side = side or 0
  self.area = side*side;
  return o
end
-- 基础类方法 printArea
function Shape:printArea ()
  print("面积为 ",self.area)
end

-- 创建对象
myshape = Shape:new(nil,10)
myshape:printArea()

Square = Shape:new()
-- 派生类方法 new
function Square:new (o,side)
  o = o or Shape:new(o,side)
  setmetatable(o, self)
  self.__index = self
  return o
end

-- 派生类方法 printArea
function Square:printArea ()
  print("正方形面积为 ",self.area)
end

-- 创建对象
mysquare = Square:new(nil,10)
mysquare:printArea()

Rectangle = Shape:new()
-- 派生类方法 new
function Rectangle:new (o,length,breadth)
  o = o or Shape:new(o)
  setmetatable(o, self)
  self.__index = self
  self.area = length * breadth
  return o
end

-- 派生类方法 printArea
function Rectangle:printArea ()
  print("矩形面积为 ",self.area)
end

-- 创建对象
myrectangle = Rectangle:new(nil,10,20)
myrectangle:printArea()

执行以上代码，输出结果为：

面积为     100
正方形面积为     100
矩形面积为     200
```

### 函数重写

```lua
Lua 中我们可以重写基础类的函数，在派生类中定义自己的实现方式：

-- 派生类方法 printArea
function Square:printArea ()
  print("正方形面积 ",self.area)
end
```
