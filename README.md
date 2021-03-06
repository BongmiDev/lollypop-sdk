# LollypopSDK Instruction
LollypopSDK provides the interfaces to Femometer, Earmo and Growp.
## Tutorial
1. Apply for AppKey from Bongmi.
2. Add dependency, details as follow in the related configuration.
3. Initialize SDK, call `LollypopSDK.getInstance().init()`, (It is suggested to call in application).
4. Call `LollypopSDK.getInstance().registerCallback()` to register callback.
5. Call `LollypopSDK.getInstance().createUser()` to create an account or call `LollypopSDK.getInstance().signIn()` to log in (It just needs call once). Use `LollypopSDK.getInstance().isLogin()` to check if needs to log in.
6. Call `LollypopSDK.getInstance().connect()` to connect the awaken devices (Decap to awaken Femometer, press button to awaken Earmo, Rotate the level ruler to awaken Growp).
7. After successful connection, call `LollypopSDK.getInstance().getDeviceInfo()` to get device info, After measurement is done, the callback method `receiveTemperature()` (`receiveGrowp()` for Growp ) registered in step 3 will be executed. 

## Related configuration
### Gradle
- modify build.gradle in project.
```
allprojects {
  repositories {
    jcenter()
    google()

    maven {
      url 'https://dl.bintray.com/beyondbright/bongmi'
    }
  }
}
```
- Add related dependency in build.gradle in the app module.
```
compile 'com.bm.android:LollypopSDK:2.2.7'
```

### Maven
```
<dependency>
  <groupId>com.bm.android</groupId>
  <artifactId>LollypopSDK</artifactId>
  <version>2.2.7</version>
  <type>pom</type>
</dependency>
```

### Permissions
```
<!-- Bluetooth -->
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-feature
    android:name="android.hardware.bluetooth_le"
    android:required="false"/>
<!-- network -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

### ProGuard
```
-keepattributes Exceptions,InnerClasses,Signature,Deprecated,SourceFile,LineNumberTable,*Annotation*,EnclosingMethod

#SDK
-keep class com.bm.android.thermometer.** { *; }
-keep class com.bm.android.thermometer.sdk.LollypopSDK { *; }
-keep interface com.bm.android.thermometer.sdk.LollypopSDK$LollypopCallback {
    *;
}
-ignorewarnings

#Gson
-keep class cn.lollypop.be.model.** { *; }
-keep class com.google.gson.** { *; }
-dontwarn com.google.gson.**

#Retrofit2
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }

#Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
```

## Related interface
- Initialize SDK. It is suggested to call in Application
```
public void init(Context context)
```
- Create an account. Apply for appKey and create an account to use the SDK.

```
  /**
   * Create an account by phone number
   *
   * @param context  context
   * @param appKey   applied key
   * @param phone    phone number
   * @param password password
   */
  public void createUser(Context context, String appKey, long phone, String password)
```
```
/**
   * Create an account by userId
   *
   * @param context  context
   * @param appKey   allied key
   * @param userId   userId
   */
public void createUser(Context context, String appKey, String userId)
```

- Log in. Log in before connecting Bluetooth.

```
  /**
   * Log in by phone number
   *
   * @param context  context
   * @param appKey   app key
   * @param phone    phone number
   * @param password password
   */
  public void signIn(Context context, String appKey, long phone, String password)
```

```
  /**
   * Log in by userId
   *
   * @param context  context
   * @param appKey   app key
   * @param userId   userId
   */
  public void signIn(Context context, String appKey, String userId)
```

- Check if needs to login.

```
  /**
   * Check if needs to log in
   * @param context context
   * @return true if already logged in
   */
  public boolean isLogin(Context context)
```

- Log out. This function is for switching users.

```
  /**
   * Log out and clear device address.
   *
   * @param context context
   */
  public void signOut(Context context)
```

- Require location permission for system above Android 6.0, refer to demo for instruction.

```
 /**
   * Require access to GPS to scan Bluetooth for system above Android 6.0.
   */
  public void requestLocationPermissions(Activity activity, Callback callback)
```

- Connect devices.

```
  /**
   * Connect to Bongmi devices
   *
   * @param deviceType Femometer: BASAL_THERMOMETER, Earmo: SMARTTHERMO, GROWP: GROWP. Default setting to connect Femometer.
   *
   * @throws LollypopException      Other exceptions.
   * @throws NoPermissionException  No location permission, call requestLocationPermissions() to ask for permission.
   * @throws NotSupportBleException The device doesn’t support BLE.
   * @throws NotEnableBleException  Bluetooth is not enabled.
   * @throws NoDeviceExistException Can not find the device, May it is not awake.
   */
  public void connect(DeviceType deviceType) throws LollypopException, NoPermissionException,
      NotSupportBleException, NotEnableBleException, NoDeviceExistException
```
There are 2 possible reasons if not connected after calling for 40 seconds:

1、The device is dormant and need to be awakened. Call connect to reconnect.
2、Bluetooth crash, restart Bluetooth or phone.

- Disconnect. If call connect in Activity, call disconnect when destroying Activity.

```
  /**
   * Disconnect.
   *
   * @throws LollypopException
   */
  public void disconnect() throws LollypopException
```

- Acquire device info, call the interface to acquire device info after connected.

```
  /**
   * Acquire device info.
   *
   * @param context context
   * @return DeviceInfo {address: bluetooth address，battery: battery，version：version}
   * @throws LollypopException
   */
  public DeviceInfo getDeviceInfo(Context context) throws LollypopException
```

- Set unit of Earmo.
```
  /**
   * Set unit of Earmo.
   *
   * @param isCentigrade. If set to be Celsius degree.
   */
  public void setTemperatureUnit(boolean isCentigrade) throws LollypopException
```

- Set weight unit of Growp.
```
  /**
   * Set weight unit of Growp.
   *
   * @param unit, weight unit
   * @throws LollypopException
   */
  public void setGrowpWeightUnit(WeightUnit unit) throws LollypopException
```

- Set height unit of Growp.
```
  /**
   * Set height unit of Growp.
   *
   * @param unit, height unit
   * @throws LollypopException
   */
  public void setGrowpHeightUnit(HeightUnit unit) throws LollypopException
```

- Regist callback method.

```
  /**
   * Regist callback method.
   *
   * @param lollypopCallback LollypopCallback
   */
  public void registerCallback(LollypopCallback lollypopCallback)

  public interface LollypopCallback {
    // Create user callback
    void createUser(Response response);

    // log in callback
    void login(Response response);

    // connect callback
    void connect(boolean suc);

    // disconnect callback
    void disconnect();

    // Temperature {temperatureInt: type Int (If receive 3655,it means 36.55 Celsius degree), measureTimestamp: time stamp of measurement, calculate: if predicted value, deviceUserId: reserved field}
    void receiveTemperature(Temperature temperature);

    // Growp {createTime: create timestamp, measureTimestamp: timestamp of measuring height and weight, detail: detailed measurement data, format: {"headCircumference":0.0,"height":15.4,"lengthUnit":1,"weight":0.0,"weightUnit":0,"value":0}}
    void receiveGrowp(Growp growp);
  }
```

# LollypopSDK中文说明
LollypopSDK提供了连接棒米体温计(Femometer)、棒米耳温枪(Earmo)、棒米身高体重仪（Growp）的相关接口。
## 使用流程
1. 向棒米官方申请appKey
2. 使用gradle添加依赖，详见下面的相关配置
3. 初始化SDK，调用`LollypopSDK.getInstance().init()`，建议在application中调用
4. 调用`LollypopSDK.getInstance().registerCallback()`注册回调
5. 调用`LollypopSDK.getInstance().createUser()`方法创建用户或者`LollypopSDK.getInstance().signIn()`方法登录，已经登录过就不需要重新登录，是否已经登录可用方法`LollypopSDK.getInstance().isLogin()`来判断
6. 调用`LollypopSDK.getInstance().connect()`方法连接已唤醒的设备（基础体温计需要插拔盖子唤醒，耳温枪需要按一下开关唤醒，身高体重仪需要打开底部盖子）
7. 连接成功后可以调用`LollypopSDK.getInstance().getDeviceInfo()`方法获取设备信息，测温成功会执行在第三步注册的`receiveTemperature()`回调方法，获取身高体重成功会执行在第三步注册的`receiveGrowp()` 回调方法

## 相关配置
### Gradle
- 修改项目的build.gradle
```
allprojects {
  repositories {
    jcenter()
    google()

    maven {
      url 'https://dl.bintray.com/beyondbright/bongmi'
    }
  }
}
```
- 在 app 的 build.gradle 中添加相应依赖
```
compile 'com.bm.android:LollypopSDK:2.2.7'
```

### Maven
```
<dependency>
  <groupId>com.bm.android</groupId>
  <artifactId>LollypopSDK</artifactId>
  <version>2.2.7</version>
  <type>pom</type>
</dependency>
```

### 蓝牙及相关权限申明
```
<!-- 蓝牙 -->
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-feature
    android:name="android.hardware.bluetooth_le"
    android:required="false"/>
<!-- 网络 -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

### 混淆
```
# 并保留源文件名为"Proguard"字符串，而非原始的类名 并保留行号
-keepattributes Exceptions,InnerClasses,Signature,Deprecated,SourceFile,LineNumberTable,*Annotation*,EnclosingMethod

#SDK
-keep class com.bm.android.thermometer.** { *; }
-keep class com.bm.android.thermometer.sdk.LollypopSDK { *; }
-keep interface com.bm.android.thermometer.sdk.LollypopSDK$LollypopCallback {
    *;
}
-ignorewarnings

#Gson
-keep class cn.lollypop.be.model.** { *; }
-keep class com.google.gson.** { *; }
-dontwarn com.google.gson.**

#Retrofit2
-dontwarn retrofit2.**
-keep class retrofit2.** { *; }

#保持 Serializable 不被混淆
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
```

## 相关接口
- 初始化SDK。建议在Application中调用
```
public void init(Context context)
```
- 创建用户。向官方申请appKey之后需要创建用户才能使用该SDK

```
  /**
   * 通过手机号创建用户
   *
   * @param context  context
   * @param appKey 申请的key
   * @param phone    手机号
   * @param password 密码
   */
  public void createUser(Context context, String appKey, long phone, String password)
```
```
/**
   * 通过userId创建用户
   *
   * @param context  context
   * @param appKey 申请的key
   * @param userId  userId
   */
public void createUser(Context context, String appKey, String userId)
```

- 登录，在连接蓝牙之前必须先登录

```
  /**
   * 通过手机号登录
   *
   * @param context  context
   * @param appKey 申请的key
   * @param phone    手机号
   * @param password 密码
   */
  public void signIn(Context context, String appKey, long phone, String password)
```

```
  /**
   * 通过userId登录
   *
   * @param context  context
   * @param appKey 申请的key
   * @param userId   userId
   */
  public void signIn(Context context, String appKey, String userId)
```

- 是否已经登录，已经登录过了就不需要重新登录了

```
  /**
   * 是否已经登录
   * @param context context
   * @return 已经登录则返回true
   */
  public boolean isLogin(Context context)
```
  
- 登出，需要切换用户的时候先调用登出方法，再重新登录

```
  /**
   * 登出，同时清除设备地址
   *
   * @param context context
   */
  public void signOut(Context context)
```

- Android 6.0之后需要定位权限，如何使用参考demo

```
 /**
   * Android 6.0之后扫描蓝牙需要开启GPS定位
   */
  public void requestLocationPermissions(Activity activity, Callback callback)
```

- 连接设备

```
  /**
   * 连接棒米设备，如果之前已经连接上，会直接进行连接，不再扫描
   *
   * @param deviceType 棒米基础体温计：BASAL_THERMOMETER，棒米耳温枪：SMARTTHERMO，棒米身高体重仪：GROWP。不填默认是连接棒米基础体温计。
   *
   * @throws LollypopException      其他异常
   * @throws NoPermissionException  没有定位权限，出现这个异常可以调用 requestLocationPermissions() 方法请求权限
   * @throws NotSupportBleException 该手机不支持BLE
   * @throws NotEnableBleException  蓝牙没打开
   * @throws NoDeviceExistException 硬件设备类型异常，直接调用connect方法不会出现这个异常
   */
  public void connect(DeviceType deviceType) throws LollypopException, NoPermissionException,
      NotSupportBleException, NotEnableBleException, NoDeviceExistException
```

一般调用connect后40秒还未连接成功,可能原因有两种:

1、设备进入休眠状态,需要唤醒。设备唤醒后调用connect方法进行重连。

2、手机蓝牙死机，重启蓝牙或者重启手机恢复。

- 断开连接。如果在Activity中调用connect方法，那在Activity销毁的时候需要调用disconnect方法。

```
  /**
   * 断开连接
   *
   * @throws LollypopException
   */
  public void disconnect() throws LollypopException
```

- 获取设备信息，连接成功后可以调用该接口获取设备信息

```
  /**
   * 获取设备信息
   *
   * @param context context
   * @return DeviceInfo 设备信息{address: 蓝牙地址，battery: 电量，version：固件版本}
   * @throws LollypopException
   */
  public DeviceInfo getDeviceInfo(Context context) throws LollypopException
```

- 设置温度单位
```
  /**
   * 设置耳温枪的单位
   *
   * @param isCentigrade 是否设置成摄氏度
   */
  public void setTemperatureUnit(boolean isCentigrade) throws LollypopException
```

- 设置生长发育仪体重单位
```
  /**
   * 设置身高体重仪的体重单位
   *
   * @param unit 体重单位
   * @throws LollypopException
   */
  public void setGrowpWeightUnit(WeightUnit unit) throws LollypopException
```

- 设置生长发育仪身高单位
```
  /**
   * 设置身高体重仪的身高单位
   *
   * @param unit 身高单位
   * @throws LollypopException
   */
  public void setGrowpHeightUnit(HeightUnit unit) throws LollypopException
```

- 设置回调方法

```
  /**
   * 注册回调方法
   *
   * @param lollypopCallback LollypopCallback
   */
  public void registerCallback(LollypopCallback lollypopCallback)

  public interface LollypopCallback {
    // 创建用户回调
    void createUser(Response response);

    // 登录回调
    void login(Response response);

    // 连接成功回调
    void connect(boolean suc);

    // 断开连接回调
    void disconnect();

    // 收到体温数据 Temperature {temperatureInt: 温度Int型（比如收到3655，就是36.55摄氏度），measureTimestamp：测温的时间戳，calculate：是否是预测值，deviceUserId：预留字段}
    void receiveTemperature(Temperature temperature);
    
    // 收到身高体重数据 Growp {createTime: 创建时间，measureTimestamp：测量身高体重的时间戳，detail：具体的测量数据，格式如下：{"headCircumference":0.0,"height":15.4,"lengthUnit":1,"weight":0.0,"weightUnit":0,"value":0}}
    void receiveGrowp(Growp growp);
  }
```