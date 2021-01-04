

# DEX加固与反编译

[TOC]

## 一、常用的`Android`反编译工具

### 1.1 编译与反编译

#### 1.1.1 编译

* 将`java`代码转换为`Dalvik`字节码；
* 将`res`资源文件、`AndroidManifest.xml`等配置文件编译为二进制文件。

#### 1.1.2 反编译

* 将`DEX`文件转换为`jar`包或者`Smali`文件；
* 将二进制资源文件还原为资源源码文件。

### 1.2 反编译工具

* `Apktool`
* `dex2jar`
* `jd-gui`

#### 1.2.1 `Apktool`

作用：反编译`DEX`为`smali`文件，反编译资源文件，支持重打包。

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/apktool.png)

```bash
# 解包apk
java -jar apktool.jar d demo.apk -o out
# 重新打包
java -jar apktool.jar b out  # out 为上面的输出目录
```

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/apktool2.png)

#### 1.2.2 `dex2jar`

```bash
# 把DEX转换为jar包
d2j-dex2jar.bat demo.apk
```

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/dex2jar.png)

#### 1.2.3 `JD-GUI`

`jar`包的图形化阅读工具：

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/jd_gui.png)

## 二、反编译带来的安全威胁与保护方案

### 2.1 应用安全与反编译

#### 2.1.1 `Android`应用反编译威胁

* 逆向分析：漏洞挖掘、协议分析；
* 二次打包：盗版、破解。

#### 2.1.2 保护方案

* 代码混淆：`Java`代码、`C/C++`代码、`JS/HTML`代码；
* 应用加固：`DEX`文件、`SO`文件、资源文件。

### 2.2 应用开发安全生态链

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/security_chain.png)

## 三、`DEX`加固的常见方案与原理

### 3.1 `DEX`加固方案演进

动态加载 --> `DEX`内存加载 --> `DEX`指令抽取 --> 虚拟机加固 --> `JAVA2C`

#### 3.1.1 `DEX`内存加载实现原理

`Android`加壳框架原理为`Proxy/Delegate Application`，即使用自定义的代理`Application`类作为程序入口（修改`AndroidManifest.xml`），在代理`Application`中完成壳的解密操作，最后启动原来的`Application`。

* `ProxyApplication`：框架会提供一个`ProxyApplication`抽象基类（`abstract class`）,使用者需要继承这个类，并重载其`initProxyApplication()`方法，在其中改变`surrounding`，如替换`ClassLoader`等；
* `DelegateApplication`：即应用原有的`Application`，应用从`getApplicationContext()`等方法中取到的都是`DelegateApplication`。

修改`AndroidManifext.xml`入口：

```xml
<!-- old AndroidManifest.xml -->
<Application
             android:name=".MyApplication"
             android:icon="@drawable/icon"
             android:label="@string/app_name"></Application>

<!-- new AndroidManifest.xml -->
<Application
             android:name=".MyProxyApplication"
             android:icon="@drawable/icon"
             android:label="@string/app_name"></Application>
```

代理`Application`：

```java
public abatract class ProxyApplication extends Application {
  protected abstract void initProxyApplication();
  
  @Override
  protected void attachBaseContext(Context context) {
    super.attachBaseContext(context);
    initProxyApplication();
  }
  //...
}
```

`initProxyApplication`实现内容：

> 1. 内存加载`DEX`：加载原`Application`;
> 2. `ClassLoader`设置；
> 3. `Application`引用替换。

### 3.2 壳启动流程

* 内存加载`DEX`文件：通过`Dalvik`、`ART`虚拟机`JNI`接口内存加载被加密隐藏的`DEX`文件；
* 设置`ClassLoader`：将`DEX`文件内存加载产生的`mCookie`放入到`ClassLoader`中（`MutiDex`）;
* 加载`Application`：基于替换后的`ClassLoader`查找原始`Application`类并生成实例；
* `Application`还原：将换`API`层的所有`Application`引用替换为原始`Application`.

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/shell_launch_process.png)

### 3.3 `DEX`加固效果

![image](https://github.com/tianyalu/NeDexPrevent/raw/master/show/dex_result.png)