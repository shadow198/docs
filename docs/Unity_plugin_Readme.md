<!--
 * @Description: 
 * @Author: yunshan.wang
 * @Version: 
 * @Date: 2019-07-25 12:07:06
 * @LastEditors: yunshan.wang
 * @LastEditTime: 2019-07-25 14:57:14
 -->
# Sigmob SDK - Unity 入门指南


## 使用 Sigmob Unity 插件设置 Unity 项目

### 将 Sigmob Unity 插件添加到 Unity 项目中
在 Unity 中打开您的项目，双击下载的 Sigmob_unity_plugin.unitypackage 文件以将 Sigmob Unity 插件添加到您的应用程序。当“导入 Unity 程序包”窗口打开时，点击“全部”即可选择全部，然后导入

### 在构建设置中选择面向的正确平台
为避免后续步骤中的编译错误，请确保项目的“构建设置”(cmd + Shift + B) 选择面向的是 iOS 或 Android 平台。



## Android 添加 AndroidManifest.xml 

```xml

    //权限定义
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />    <!-- 如果需要精确定位的话请加上此权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>

    
    <application
        <provider android:name="com.sigmob.sdk.SigmobFileProvider"
            android:authorities="${applicationId}.sigprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/sigmob_provider_paths"/>
        </provider>

        <activity android:name="com.sigmob.sdk.base.common.AdActivity"
            android:theme="@android:style/Theme.DeviceDefault"
            android:configChanges="keyboardHidden|orientation|screenSize"/>
    </application>

```

## provider_paths 定义
```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 这个下载路径不可以修改，必须是SIGTDOWNLOAD -->
    <external-path name="sigmob_download_path" path="SIGTDOWNLOAD" />
</paths>
```

## iOS添加相关依赖库
> 可在unity build时写脚本自动配置iOS工程，示例代码如下(仅供参考)：

```C#
public class XcodeProjectUpdater {

	[PostProcessBuild]
	public static void OnPostprocessBuild(BuildTarget target, string pathToBuildProject) {

		if (target != BuildTarget.iOS) return;
		//.xcodeproj的路径
		string projPath = PBXProject.GetPBXProjectPath(pathToBuildProject);
		PBXProject proj = new PBXProject();
		proj.ReadFromString(File.ReadAllText(projPath));

		string buildTarget = proj.TargetGuidByName("Unity-iPhone");

		//--------------------- 
		//1、修改设置
		//添加系统库
		proj.AddFrameworkToProject(buildTarget,"CoreTelephony.framework", false);
		proj.AddFrameworkToProject(buildTarget,"Security.framework", false);  
		proj.AddFrameworkToProject(buildTarget,"MobileCoreServices.framework", false);
		proj.AddFrameworkToProject(buildTarget,"StoreKit.framework", false);
		proj.AddFrameworkToProject(buildTarget,"AdSupport.framework", false);
		proj.AddFrameworkToProject(buildTarget,"ImageIO.framework", false);  
		proj.AddFrameworkToProject(buildTarget,"SafariServices.framework", true);
		proj.AddFrameworkToProject(buildTarget,"WebKit.framework", true);

		//--------------------- 

		//添加tbd
		string fileGuidSqlite = proj.AddFile("usr/lib/libsqlite3.0.tbd", "Libraries/libsqlite3.0.tbd", PBXSourceTree.Sdk);
		proj.AddFileToBuild(buildTarget, fileGuidSqlite);

		string fileGuidLibz = proj.AddFile("usr/lib/libz.tbd", "Libraries/libz.tbd", PBXSourceTree.Sdk);
		proj.AddFileToBuild(buildTarget, fileGuidLibz);

		string fileGuidLibc= proj.AddFile("usr/lib/libc++.tbd", "Libraries/libc++.tbd", PBXSourceTree.Sdk);
		proj.AddFileToBuild(buildTarget, fileGuidLibc);
		//--------------------- 


		//修改属性
     proj.SetBuildProperty(buildTarget,"ENABLE_BITCODE", "YES");
		proj.AddBuildProperty(buildTarget, "OTHER_LDFLAGS", "-ObjC");

		//--------------------- 

		File.WriteAllText (projPath, proj.WriteToString());

	}
}
```

### 1.1 申请应用的AppID和PlacementId
请向Sigmob平台申请AppId和广告位PlacementId

### 1.2 工程设置导入framework
开发者只需将WindSDK.framework文件拖入工程即可。作为聚合SDK,我们为每个渠道SDK提供了一个.a静态库，如果需要使用其它渠道的SDK,那么只需要将对应渠道的SDK和.a静态库文件拖入工程即可。

### 1.3 Xcode编译选项设置

#### 1.3.1 添加“ObjC”链接器标记
在Xcode中选择项目的Targets->Build Settings，配置Other Link Flags 增加 **-ObjC**。

##### 1.3.2 删除iOS状态栏
尽管这不是必需的步骤，但我们建议采取该步骤以确保 WindSDK 的广告互动和演示可以顺利进行。如要删除状态栏，请打开 Info.plist， 添加**View controller-based status bar appearance**，并将其设置为 NO。

#### 1.3.3 添加HTTP权限
工程info.plist文件设置，点击右边的information Property List后边的 "+" 展开
添加 App Transport Security Settings，先点击左侧展开箭头，再点右侧加号，Allow Arbitrary Loads 选项自动加入，修改值为 YES。 SDK API 已经全部支持HTTPS，但是广告主素材存在非HTTPS情况。

```json
<key>NSAppTransportSecurity</key>
    <dict>
         <key>NSAllowsArbitraryLoads</key>
         <true/>
    </dict>
```
#### 1.3.4 添加定位权限
工程info.plist文件设置，点击右边的information Property List后边的 "+" 展开
添加Privacy - Location When In Use Usage Description。

#### 1.3.5 运行环境配置
+ 支持系统 iOS 7.X 及以上;
+ SDK编译环境 Xcode 9.0+, Base SDK 11.0;
+ 支持架构：i386, x86-64, armv7, armv7s, arm64

#### 1.3.6 添加依赖库
```objc
libz.tbd
libc++.tbd
libsqlite3.tbd
libresolv.9.tbd(引入BUAdSDK.framework需加入)
UIKit.framework
GLKit.framework
WebKit.framework
StoreKit.framework
Security.framework
CFNetwork.framework
CoreMedia.framework
MessageUI.framework
CoreVideo.framework
AdSupport.framework
CoreMotion.framework
QuartzCore.framework
Foundation.framework
MediaPlayer.framework
CoreGraphics.framework
AVFoundation.framework
CoreLocation.framework
AudioToolbox.framework****
CoreTelephony.framework
SafariServices.framework
MobileCoreService.framework
SystemConfiguration.framework
```
>注意：如果您 WebKit.framework&SafariServices.framework 将 iOS 7 作为部署目标，请将状态设置为**options**。
>如果您将iOS6作为部署目标，请将UIKit.framework、Foundation.framework、WindSDK.framework设置为**options**。
>由于AppLovinSDK.framework最低至此iOS9，若您最低至此版本小于iOS9，请将AppLovinAdapter.a和AppLovinSDK.framework设置为**options**

## unity插件初始化

```C#
//添加命名空间
using Wind.Advertisements;

#if UNITY_IPHONE
	Advertisement.Init ("your appid", "your apiKey");
        placementId = "reward video AD placement id";
#elif UNITY_ANDROID
        Advertisement.Init ("your appid", "your apiKey");
	placementId = "reward video AD placement id";
#endif

        //激励视频广告初始化
Request request = new Request ();
request.UserId = "unity_user_id";
request.PlacementId = placementId;
adVertisement = WindRewardVideoAdvertisement.Builder(request);

```




## 自定义delegate回调
```c#
//ad server返回广告
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdServerDidSuccess_Delegate))]
private void OnAdServerDidSuccessCallback(string placementId)
{
	Debug.Log ("C# [OnAdServerDidSuccessCallback:]" + placementId);
}
//ad server返回无广告
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdServerDidFail_Delegate))]
private void OnAdServerDidFailsCallback(string placementId)
{
	Debug.Log ("C# [OnAdServerDidFailsCallback:]" + placementId);
}
//广告加载成功，此时ready=true
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdLoadSuccess_Delegate))]
private void OnAdLoadSuccessCallback(string placementId)
{
	Debug.Log ("C# [OnAdLoadSuccessCallback:]" + placementId);
}
//广告开始播放
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdPlay_Delegate))]
private void OnAdPlayStartCallback(string placementId)
{
	Debug.Log ("C# [OnAdPlayStartCallback:]" + placementId);
}
//广告播放出错
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdPlayFailed_Delegate))]
private void OnAdPlayErrorCallback(string placementId, Error error)
{
	Debug.Log ("C# [OnAdPlayErrorCallback:]" + placementId);
}
//视频播放结束
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdVideoPlayEnd_Delegate))]
private void OnAdPlayEndCallback(string placementId)
{
        Debug.Log ("C# [OnAdPlayEndCallback:]" + placementId);
}
//广告被点击
private delegate void OnAdClickedDelegate(string placementId);
[MonoPInvokeCallback(typeof(OnAdClickedDelegate))]
private void OnAdClickedCallback(string placementId)
{
	Debug.Log ("C# [OnAdClickedCallback:]" + placementId);
}
//广告关闭
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdClosed_Delegate))]
private void OnAdClosedCallback(string placementId, RewardInfo info)
{
	Debug.Log ("C# [OnAdClosedCallback:]" + placementId + ", info = " + info.ToString ());
}
//广告加载出错
[MonoPInvokeCallback(typeof(WindRewardVideoAdvertisement.OnAdLoadFailed_Delegate))]
private  void OnAdErrorCallback(string placementId, Error error)
{
	Debug.Log ("C# [OnAdErrorCallback:]" + placementId + ", error = " + error.ToString ());
}
```

## 添加监听

```C#
//设置广告填充成功监听
adVertisement.SetAdServerDidSuccessListener(OnAdServerDidSuccessCallback);
//设置广告填充失败监听
adVertisement.SetAdServerDidFailListener(OnAdServerDidFailsCallback);
//设置广告加载成功监听
adVertisement.SetAdLoadSuccessListener(OnAdLoadSuccessCallback);
//设置广告加载失败监听
adVertisement.SetAdLoadFailedListener (OnAdErrorCallback);
//设置广告播放监听
adVertisement.SetAdPlayListener (OnAdPlayStartCallback);
//设置广告点击监听
adVertisement.SetAdClickedListener (OnAdClickedCallback);
//设置广告关闭监听
adVertisement.SetAdClosedListener (OnAdClosedCallback);
//设置广告视频播放结束监听
adVertisement.SetAdVideoPlayEndListener(OnAdPlayEndCallback);
//设置广告播放失败监听
adVertisement.SetAdPlayFailedListener (OnAdPlayErrorCallback);

```


## 加载广告

```C#
//温馨提示：在加载广告前确保已注册监听
//加载广告
adVertisement.LoadRequest();
```
## 播放广告

```C#
//温馨提示：在播放广告前请使用Ready接口检查广告状态
//播放广告
if (adVertisement.Ready ()) {
    adVertisement.PlayAd ();
}
```

## 广告状态检查

```C#
//广告状态检查
adVertisement.Ready ();
```

 
