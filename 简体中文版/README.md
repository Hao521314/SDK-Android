SDK-Android
=================

欢迎使用云子/标签，本质上它们是一种带有 iBeacon 功能的无线智能传感器设备。

**目前我们的 Android SDK，主要提供以下功能：**

1. 扫描周围的传感器设备
2. 读取传感器设备的参数
3. 上传传感器设备状态（电池、UMM等）至 [SENSORO 云平台](https://cloud.sensoro.com)
4. 设置云子传感器

### 安装 SDK
##### 一. 下载
您可以从下面的地址下载最新版的 Android SDK：
  
[下载地址](https://github.com/Sensoro/SDK-Android)

##### 二. 安装与配置

**集成 SDK jar 包**

在工程根目录新建 libs 文件夹，将下载的 SDK 解压到到该文件夹中。

下面是 SDK 所包含的模块：
- sensorocloud-<版本号>.jar
- sensorobeaconkit-<版本号>.jar
- android-async-http-1.4.6.jar
- greendao-1.3.7.jar
- gson-2.3.1.jar

**修改 AndroidManifest.xml**

1.在 **AndroidManifest.xml** 中集成 SDK 所需要的权限

```	
<manifest
	...
	<uses-permission android:name="android.permission.BLUETOOTH" />
  	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
   	<uses-permission android:name="android.permission.INTERNET" />
   	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
   	<uses-permission android:name="android.permission.WAKE_LOCK" />
	...
</manifest>
```
2.在 **AndroidManifest.xml** 中集成 SDK 所依赖的服务

```
<manifest
	...
	<application
		...
		<service android:name="com.sensoro.beacon.kit.BeaconProcessService"/>
		<service android:name="com.sensoro.beacon.kit.BeaconService"/>
		<service android:name="com.sensoro.beacon.kit.IntentProcessorService"/>
		...
	</application>
	...
</manifest>
```

### 开始使用
##### 一、应用程序初始化

首先你需要初始化一个 SDK 的实例，并设置是否上传传感器数据（电池电量、UMM）等，然后启动扫描服务。以下为 SDK 启动的样例代码。

```
SensoroManager sensoroManager = SensoroManager.getInstance(context);
/**
 * 检查蓝牙是否开启
 **/
if (sensoroManager.isBluetoothEnabled()) {
	/**
	 * 设置启用云服务 (上传传感器数据，如电量、UMM等)。如果不设置，默认为关闭状态。
	 **/
	sensoroManager.setCloudServiceEnable(true);
	/**
	 * 启动 SDK 服务
	 **/
	try {
	    sensoroManager.startService();
	} catch (Exception e) {
   	 e.printStackTrace(); // 捕获异常信息
	}
}
```

**提示：**

- SDK 是基于蓝牙 4.0 的服务，启动前请先检查蓝牙是否开启，否则 SDK 无法正常工作。
- SDK 的设计采用单例模式，因此推荐在继承的 **Application** 类中进行初始化。

如果 Android 设备的蓝牙没有打开，请使用下面样例代码请求打开蓝牙：

```
Intent bluetoothIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
startActivityForResult(bluetoothIntent, REQUEST_ENABLE_BT);
```


##### 二、设置传感器数据监听

你可以通过实现并设置 BeaconManagerListener 接口，来检测 Beacon 的出现,显示以及更新。样例代码如下：

```
BeaconManagerListener beaconManagerListener = new BeaconManagerListener() {

    @Override
    public void onUpdateBeacon(ArrayList<Beacon> beacons) {
        // 传感器信息更新                  
    }
    
    @Override
    public void onNewBeacon(Beacon beacon) {
        // 发现一个新的传感器        
    }
    
    @Override
    public void onGoneBeacon(Beacon beacon) {
        // 一个传感器消失     
    }
};
sensoroManager.setBeaconManagerListener(beaconManagerListener);
```
在这个接口中，传感器信息更新频率为 1 秒；发现一个新的传感器后，如果在 8 秒内没有再次扫描到这个设备，则会回调传感器消失。

**提示：**

- 请在 SDK 启动之前设置。

##### 三、监控传感器设备进出状态

通常我们进入或离开某些设备时需要进行一些操作。下面是判断是否进入和离开 SN 为"0117C5456A36"的云子的样例代码：

```
BeaconManagerListener beaconManagerListener = new BeaconManagerListener() {

    @Override
    public void onUpdateBeacon(ArrayList<Beacon> beacons) {
        // 传感器信息更新                  
    }
    
    @Override
    public void onNewBeacon(Beacon beacon) {
        if (beacon.getSerialNumber().equals("0117C5456A36")){
        	// 进入 SN 为"0117C5456A36 的云子
        }       
    }
    
    @Override
    public void onGoneBeacon(Beacon beacon) {
        if (beacon.getSerialNumber().equals("0117C5456A36")){
        	// 离开 SN 为"0117C5456A36 的云子
        }      
    }
};
sensoroManager.setBeaconManagerListener(beaconManagerListener);
```

**提示：**

- 回调函数是在非 UI 线程中运行的，请不要在回调函数中进行任何 UI 的相关相关操作，否则会导致 SDK 运行异常。如有需要，请通过 Handler 或者 Activity.runOnUiThread 方式来运行你的代码。

##### 四、读取传感器状态和更新

下表为云子传感器中常用的重要参数列表。

| 属性                 | 描述                      |
| :------------------ | :----------------------- |
| serialNumber        | SN，设备唯一标识          |
| major               | iBeacon协议中的 major 信息 |
| minor               | iBeacon协议中的 minor 信息 |
| proximityUUID       | iBeacon协议中的 UUID 信息  |
| rssi                | 信号强度                  |
| accuracy            | 距离（米）                 |
| proximity           | 范围（很远，附近，很近，未知）|
| temperature         | 芯片温度                  |
| light               | 光线                      |
| movingState         | 移动状态                  |
| accelerometerCount  | 移动计数器                 |
| batteryLevel        | 电池电量                  |
| hardwareModelName   | 硬件版本                  |
| firmwareVersion     | 固件版本                  |
| measuredPower       | 1 米处测量 rssi           |
| transmitPower       | 广播功率                  |
| advertisingInterval | 广播间隔                  |

应用程序通常会在传感器数据状态发生变化时进行一些操作，如信号强度变化，光线变化，移动状态变化，计数器数值变化。你可以通过遍历传感器更新列表，找到你关心的设备，查看其状态是否有变化。下面是判断 SN 为"0117C5456A36"的云子，运动状态是否有变化的样例代码：

```
BeaconManagerListener beaconManagerListener = new BeaconManagerListener() {

    @Override
    public void onUpdateBeacon(final ArrayList<Beacon> beacons) {
    	// 检查串码为"0117C5456A36"的云子，运动状态是否有变化
    	for(Beacon beacon:beacons){
        	if (beacon.getSerialNumber().equals("0117C5456A36")){
           		if (beacon.getMovingState() == Beacon.MovingState.DISABLED){
            		// 运动传感器禁用
            	} else if (beacon.getMovingState() == Beacon.MovingState.STILL){ 
            		// 传感器静止
            	} else if (beacon.getMovingState() == Beacon.MovingState.MOVING){
            		// 传感器正在运动
            	}
            }
    	}
    }
    
    @Override
    public void onNewBeacon(Beacon beacon) {
        // 发现一个新的传感器        
    }
    
    @Override
    public void onGoneBeacon(Beacon beacon) {
        // 一个传感器消失     
    }
};
```

**提示：**

- 回调函数是在非 UI 线程中运行的，请不要在回调函数中进行任何 UI 的相关相关操作，否则会导致 SDK 运行异常。如有需要，请通过 Handler 或者 Activity.runOnUiThread 方式来运行你的代码。

##### 五、云子/标签防蹭用

你可以为你的云子开启防蹭用功能，使其变成你的专有设备，防止第三方人员查看和使用。使用防蹭用功能需要一个密钥，在[SENSORO 云平台](https://cloud.sensoro.com)中注册后，你可以申请这个密钥，然后将其设置到 SDK 中。样例代码如下：

```
/**
 * 设置云子防蹭用密钥 (如果没有可以不设置)
 **/
sensoroManager.addBroadcastKey("01Y2GLh1yw3+6Aq0RsnOQ8xNvXTnDUTTLE937Yedd/DnlcV0ixCWo7JQ+VEWRSya80yea6u5aWgnW1ACjKNzFnig==");
```

**提示：**

- 请在 SDK 启动之前设置。
- 如果 SDK 无法扫描到设备，请检查设备中设置的密钥和 SDK 设置的是否相同

###总结
至此，您已完成将 SDK 整合进您项目之中的全部工作。后面，您可以使用 SDK 的功能来完成您 App 与 云子传感器 之间的互动，更多技术细节，请参考完整的 SDK 文档，以及我们的示例应用程序源代码。

### 附录

##### Sensoro Beacon 功率档位说明

<table>
	<tr>
  		<th rowspan="2">档位/型号</th>
  		<th colspan="2">A0</th>
		<th colspan="2">B0</th>
		<th colspan="2">C0</th>
	</tr>
	<tr align="center">
		<td>信号强度</td>
		<td>覆盖范围</td>
		<td>信号强度</td>
		<td>覆盖范围</td>
		<td>信号强度</td>
		<td>覆盖范围</td>
	</tr>
	<tr align="center">
		<td>LEVEL0</td>
		<td>-23 dbm</td>
		<td>约 2 m</td>
		<td>-30 dbm</td>
		<td>约 2 m</td>
		<td>微距 -30 dbm</td>
		<td>约 5 cm</td>
	</tr>
	<tr align="center">
		<td>LEVEL1</td>
		<td>-6 dbm</td>
		<td>约 7 m</td>
		<td>-20 dbm</td>
		<td>约 7 m</td>
		<td>微距 -20 dbm</td>
		<td>约 50 cm</td>
	</tr>
	<tr align="center">
		<td>LEVEL2</td>
		<td>0 dbm</td>
		<td>约 15 m</td>
		<td>-16 dbm</td>
		<td>约 10 m</td>
		<td>微距 -16 dbm</td>
		<td>约 80 cm</td>
	</tr>
	<tr align="center">
		<td>LEVEL3</td>
		<td>-</td>
		<td>-</td>
		<td>-12 dbm</td>
		<td>约 15 m</td>
		<td>微距 -12 dbm</td>
		<td>约 1.5 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL4</td>
		<td>-</td>
		<td>-</td>
		<td>-8 dbm</td>
		<td>约 22 m</td>
		<td>-30 dbm</td>
		<td>约 2 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL5</td>
		<td>-</td>
		<td>-</td>
		<td>-4 dbm</td>
		<td>约 27 m</td>
		<td>-20 dbm</td>
		<td>约 7 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL6</td>
		<td>-</td>
		<td>-</td>
		<td>0 dbm</td>
		<td>约 50 m</td>
		<td>-16 dbm</td>
		<td>约 15 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL7</td>
		<td>-</td>
		<td>-</td>
		<td>+4 dbm</td>
		<td>约 90 m</td>
		<td>-12 dbm</td>
		<td>约 20 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL8</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>-8 dbm</td>
		<td>约 25 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL9</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>-4 dbm</td>
		<td>约 45 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL10</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>0 dbm</td>
		<td>约 70 m</td>
	</tr>
	<tr align="center">
		<td>LEVEL11</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>-</td>
		<td>+4 dbm</td>
		<td>约 100 m</td>
	</tr>
</table>


##### Android-SDK 4.0  配置 beacon 错误码表

> Android-SDK 从 4.0 版本开始，使用 *SensoroBeaconConnectionV4* 和 *BeaconConfiguration* 两个类对 beacon 进行配置， 返回值 resultCode 整理如下，以便查询和参考。

| 十六进制(HEX)	| 十进制(DEC)		| 释义		|
| :-----------: |:-------------:| :--------:|
| 0x0000		| 0				| 成功		|
| 0x0100     	| 256     		| MCU 繁忙	|
| 0x0200		| 512      		| LED 繁忙	|
| 0x0300		| 768      		| 没有权限	|
| 0x0400		| 1024      	| 无效的配置指令	|
| 0x0500		| 1280      	| 参数长度错误	|
| 0x0501		| 1281      	| 功率长度错误	|
| 0x0502		| 1282      	| 频率长度错误	|
| 0x0503		| 1283      	| mrssi 长度错误	|
| 0x0504		| 1284      	| 节能模式长度错误	|
| 0x0505		| 1285      	| 密码长度错误	|
| 0x0506		| 1286      	| 工作指示灯长度错误		|
| 0x0511		| 1297      	| iBeacon 开关长度错误	|
| 0x0512		| 1298      	| uuid 长度错误			|
| 0x0513		| 1299      	| major 长度错误			|
| 0x0514		| 1300      	| minor 长度错误			|
| 0x0515		| 1301      	| 广播轮询间隔长度错误		|
| 0x0516		| 1302      	| 后台增强模式长度错误		|
| 0x0517		| 1303      	| 广播密钥长度错误			|
| 0x0521		| 1313      	| 温度采样间隔长度错误		|
| 0x0522		| 1314      	| 光线采样间隔长度错误		|
| 0x0523		| 1315      	| 加速度传感器灵敏度长度错误		|
| 0x0531		| 1329      	| UID 开关长度错误			|
| 0x0532		| 1330      	| URL 开关长度错误			|	
| 0x0533		| 1331      	| TLM 开关长度错误			|
| 0x0534		| 1332      	| UID 长度错误（NID）			|
| 0x0535		| 1333      	| UID 长度错误 (BID)			|
| 0x0536		| 1334      	| URL 长度错误				|
| 0x0537		| 1335      	| TLM 间隔长度错误			|
| 0x0541		| 1345      	| AliBeacon 开关长度错误		|
| 0x0600		| 1536      	| 参数格式错误				|
| 0x0601		| 1537      	| 功率格式错误				|
| 0x0602		| 1538      	| 频率格式错误				|	
| 0x0603		| 1539      	| mrssi 格式错误				|
| 0x0604		| 1540      	| 节能模式格式错误				|
| 0x0605		| 1541      	| 密码格式错误				|
| 0x0606		| 1542      	| 工作指示灯格式错误			|
| 0x0611		| 1553      	| iBeacon 开关格式错误		|	
| 0x0612		| 1554      	| uuid 格式错误				|
| 0x0613		| 1555      	| major 格式错误				|
| 0x0614		| 1556      	| minor 格式错误				|
| 0x0615		| 1557      	| 广播轮询间隔格式错误			|
| 0x0616		| 1558      	| 后台增强模式格式错误			|
| 0x0617		| 1559      	| 广播密钥格式错误				|
| 0x0621		| 1569      	| 温度采样间隔格式错误			|
| 0x0622		| 1570      	| 光线采样间隔格式错误			|
| 0x0623		| 1571      	| 加速度传感器灵敏度格式错误		|
| 0x0631		| 1585      	| UID 开关格式错误			|
| 0x0632		| 1586      	| URL 开关格式错误			|	
| 0x0633		| 1587      	| TLM 开关格式错误			|
| 0x0634		| 1588      	| UID 格式错误（NID）			|
| 0x0635		| 1589      	| UID 格式错误 (BID)			|
| 0x0636		| 1590      	| URL 格式错误				|
| 0x0637		| 1591      	| TLM 间隔格式错误			|
| 0x0641		| 1601      	| AliBeacon 开关格式错误		|
| 0x0700		| 1792      	| 无效的配置项				|		
| 0x0800		| 2048      	| 协议版本错误				|		
| 0x8000		| 32768      	| 蓝牙异常					|
| 0x8100		| 33024      	| 蓝牙通信异常				|
| 0x8200		| 33280      	| 蓝牙连接超时				|
| 0x8300		| 33536      	| 硬件版本不支持				|