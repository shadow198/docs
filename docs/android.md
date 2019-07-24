<!--
 * @Description: 
 * @Author: yunshan.wang
 * @Version: 
 * @Date: 2019-07-24 17:45:54
 * @LastEditors: yunshan.wang
 * @LastEditTime: 2019-07-24 18:53:12
 -->
# WindSDK 集成文档

* __WindSDK支持最低系统版本：Android SDK Level 14+__

* __Android Support V4 23版本以上__


## SDK版本以下差异说明



更新内容|2.8.1之前|2.8.1以后
---|:--:|---
开屏请求|WindAdRequest| WindSplashAdRequest
开屏接口|public WindSplashAD(Activity activity, ViewGroup adContainer, WindAdRequest adRequest,WindSplashADListener adListener, int fetchDelay)<br>public WindSplashAD(Activity activity, WindAdRequest adRequest, WindSplashADListener adListener, int fetchDelay, String appTitle, String appDesc)| public WindSplashAD(Activity activity, ViewGroup adContainer, WindSplashAdRequest adRequest,WindSplashADListener adListener)



更新内容|2.8.0之前|2.8.0以后
---|:--:|---
激励视频广告请求填充成功回调|onVideoAdLoadReceived|onVideoAdPreLoadSuccess
激励视频广告请求填充失败回调|无|onVideoAdPreLoadFail
激励视频广告视频播放完成回调|无|onVideoAdPlayEnd

更新内容|2.5.1之前|2.5.2以后
---|:--:|---
激励视频广告请求填充回调|无|onVideoAdLoadReceived
SDK初始化|startWithOptions(@NonNull Activity activity , @NonNull WindAdOptions options)| startWithOptions(@NonNull Application application, @NonNull WindAdOptions options)


更新内容|2.4.2之前|2.4.3以后
---|:--:|---
激励视频Error回调|onVideoError|onVideoAdLoadError,onVideoAdPlayError
provider定义|android:authorities="${applicationId}.provider"| android:authorities="${applicationId}.sigprovider"

更新内容|1.7.X|2.X.X
---|:--:|---
广告展示Activity|com.sigmob.sdk.rewardVideoAd.RewardVideoAdPlayerActivity|com.sigmob.sdk.base.common.AdActivity
视频广告准备检查接口|isReady()|isReady(String placementId)


## 1、准备工作

* 解压我们提供的压缩包，把WindAd*.Jar放入app的libs工程中。

* 到需要接入的渠道平台申请对应的appId、appKey、 placementId（先找接口商务提供）。

* 3 到sigmob流量变现管理平台配置app各渠道的参数（目前代为操作，且仅支持Sigmob渠道）。


## 2、 集成步骤
###  加入 Android Support V4 依赖支持库
```
dependencies {

    //windAd SDK jar文件 放入项目libs中
    implementation fileTree(include: ['*.jar'], dir: 'libs')

    //Android Support V4 23版本以上
    implementation 'com.android.support:support-v4:23.0.+'

}
```

### 添加provider
* 在项目结构下的res目录下添加一个xml文件夹，再新建一个sigmob_provider_paths.xml的文件，文件内容如下

```
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 这个下载路径不可以修改，SIGTDOWNLOAD -->
    <external-path name="sigmob_download_path" path="SIGTDOWNLOAD" />
</paths>
```
### 更新 AndroidManifest.xml

```
<manifest>


    <!-- SDK所需要权限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />    <!-- 如果需要精确定位的话请加上此权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />


    <application>

        <!-- targetSDKVersion >= 24时才需要添加这个provider。
       provider的authorities属性的值为${applicationId}.sigprovider -->


        <provider
            android:name="com.sigmob.sdk.SigmobFileProvider"
            android:authorities="${applicationId}.sigprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/sigmob_provider_paths"/>
        </provider>

        <!--广告展示Activity -->

        <activity android:name="com.sigmob.sdk.base.common.AdActivity"
            android:theme="@android:style/Theme.DeviceDefault"
            android:configChanges="keyboardHidden|orientation|screenSize" />

    </application>


</manifest>


```

### 混淆配置
```

# android.support.v4

-dontwarn android.support.v4.**
-keep class android.support.v4.** { *; }
-keep interface android.support.v4.** { *; }
-keep public class * extends android.support.v4.**


# WindAd

-keep class sun.misc.Unsafe { *; }
-dontwarn com.sigmob.**
-keep class com.sigmob.**.**{*;}

```

## 3、 集成示例代码

### SDK初始化及GDPR授权支持（仅用于有GDPR需求的开发者）
```
        WindAds ads = WindAds.sharedAds();

        //enable or disable debug log ouput
        ads.setDebugEnable(BuildConfig.DEBUG);

        /*  欧盟 GDPR 支持
        **  WindConsentStatus 值说明:
        **     UNKNOW("0"),  //未知,默认值，根据服务器判断是否在欧盟区，若在欧盟区则判断为拒绝GDPR授权
        **     ACCEPT("1"),  //用户同意GDPR授权
        **     DENIED("2");  //用户拒绝GDPR授权
        */
        
        ads.setUserGDPRConsentStatus(WindConsentStatus.ACCEPT);

        // start SDK Init with Options
        ads.startWithOptions(mApplication, new WindAdOptions(YOU_APP_ID, YOU_APP_KEY));

```



## 激励视频集成相关
### 设置监听回调
```
        WindRewardedVideoAd windRewardedVideoAd = WindRewardedVideoAd.sharedInstance();

        windRewardedVideoAd.setWindRewardedVideoAdListener(new WindRewardedVideoAdListener() {

            @Override
            public void onVideoAdLoadReceived(String placementId) {
                Toast.makeText(mContext, "激励视频广告数据返回成功", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onVideoAdLoadSuccess(String placementId) {
                Toast.makeText(mContext, "激励视频广告缓存加载成功", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onVideoAdPlayStart(String placementId) {
                Toast.makeText(mContext, "激励视频广告播放开始", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onVideoAdClicked(String placementId) {
                Toast.makeText(mContext, "激励视频广告CTA点击事件监听", Toast.LENGTH_SHORT).show();

            }

            //WindRewardInfo中isComplete方法返回是否完整播放
            @Override
            public void onVideoAdClosed(WindRewardInfo windRewardInfo, String placementId) {
                if(windRewardInfo.isComplete()){
                   Toast.makeText(mContext, "激励视频广告完整播放，给予奖励", Toast.LENGTH_SHORT).show();
                }else{
                   Toast.makeText(mContext, "激励视频广告关闭", Toast.LENGTH_SHORT).show();
                }
            }

            /**
            * 加载广告错误回调
            * WindAdError 激励视频错误内容
            * placementId 广告位
            */
            @Override
            public void onVideoAdLoadError(WindAdError windAdError, String placementId) {
                Toast.makeText(mContext, "激励视频广告错误", Toast.LENGTH_SHORT).show();
            }


            /**
            * 播放错误回调
            * WindAdError 激励视频错误内容
            * placementId 广告位
            */
            @Override
            public void onVideoAdPlayError(WindAdError windAdError, String placementId) {
                Toast.makeText(mContext, "激励视频广告错误", Toast.LENGTH_SHORT).show();
            }

        });
```

### 激励视频广告加载
```
       WindRewardedVideoAd windRewardedVideoAd = WindRewardedVideoAd.sharedInstance();

       //placementId 必填
       WindAdRequest request = new WindAdRequest(PLACEMENT_ID,USER_ID,OPTIONS);
       windRewardedVideoAd.loadAd(request);

```

### 激励视频广告播放
```
        WindRewardedVideoAd windRewardedVideoAd = WindRewardedVideoAd.sharedInstance();

        try {
             //检查广告是否准备完毕
             if(windRewardedVideoAd.isReady(PLACEMENT_ID)){
                 //PLACEMENT_ID必填
                 WindAdRequest request = new WindAdRequest(PLACEMENT_ID,USER_ID,OPTIONS);
                 //广告播放
                 windRewardedVideoAd.show(getActivity(),request);
             }

        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        }
```

## 开屏广告集成相关

**目前开屏广告仅支持竖屏**

### 设置监听回调
```

    // 开屏广告开始展示
    @Override
    public void onSplashAdSuccessPresentScreen() {

    }

    // 开屏广告展示失败
    /**
    * WindAdError 开屏广告错误内容
    * placementId 广告位 
    */
    @Override
    public void onSplashAdFailToPresent(WindAdError error, String placementId) {

    }

    // 开屏广告点击
    @Override
    public void onSplashAdClicked() {


    }
    // 开屏广告关闭
    @Override
    public void onSplashClosed() {

    }
```


### 开屏播放接口
* 此方式自适应广告展示大小，自带LOGO样式展示APP信息，无需开发者处理底部LOGO内容

```
//PLACEMENT_ID必填
WindSplashAdRequest splashAdRequest = new WindSplashAdRequest(PLACEMENT_ID,USER_ID,OPTIONS);

/**
 * 广告结束，广告内容是否自动隐藏.默认是false
 * 若开屏和应用共用Activity，建议false。
 * 若开屏是单独Activity ，建议true。
*/
adRequest.setDisableAutoHideAd(true);

//广告允许最大等待返回时间
splashAdRequest.setFetchDelay(5);

//设置开屏应用LOGO区域
splashAdRequest.setAppTitle(appTitle);

//设置开屏应用LOGO描述
splashAdRequest.setAppDesc(appDesc);

/**
 * 方法:
 *   WindSplashAD(Activity activity,ViewGroup adContainer，WindSplashAdRequest splashAdRequest, WindSplashADListener adListener)
 * 参数说明:
 *   activity 开屏展示Activity
 *   adContainer 开屏内容展示容器,若传null，则默认进行全屏展示
 *   adRequest WindAdRequest广告请求
 *   adListener 开屏事件监听
*/

WindSplashAD mWindSplashAD =  new WindSplashAD(activity,adContainer,splashAdRequest,this);

```

<a name="#xr1">WindAdError错误定义</a>
```
            // 请求的app已经关闭广告服务
            ERROR_REQUEST_APP_IS_CLOSED(500420,"请求的app已经关闭广告服务"),

            // 请求参数缺少设备信息
            ERROR_REQUEST_NEED_DEVICE_INFO(500422,"请求参数缺少设备信息"),

            //缺少设备id相关信息
            ERROR_REQUEST_NEED_DEVICE_ID_INFO(500424,"缺少设备id相关信息"),

            // 缺少广告为信息
            ERROR_REQUEST_NEED_AD_SLOTS_INFO(500428," 缺少广告为信息"),

            // 错误的广告位信息
            ERROR_INVALID_ADSLOT_ID(500430," 错误的广告位信息"),

            // 广告位不存在，或者appid与广告位不匹配
            REQUEST_AD_SLOT_NOT_EXISTS (500432,"广告位不存在，或者appid与广告位不匹配"),

            // 广告位不存在或是已关闭
            REQUEST_AD_SLOT_IS_CLOSED(500433,"广告位不存在或是已关闭"),

            // 设备的操作系统类型，与请求的app的系统类型不匹配，例如，iOS设备使用了安卓的appid发起广告请求（1.9版本实现）
            REQUEST_OS_TYPE_NOT_MATCH_APP_TYPE(500435,"设备的操作系统类型，与请求的app的系统类型不匹配"),

            // 广告单元id与请求的广告类型不匹配（1.9版本实现）
            REQUEST_AD_SLOT_NOT_MATCH__AD_TYPE(500436, "广告单元id与请求的广告类型不匹配"),

            // 请求的app不存在
            ERROR_REQUEST_NO_SUCH_APP(500473,"请求的app不存在"),

            //app未设置聚合策略
            ERROR_REQUEST_APP_NOT_SET_STRATEGY(500700,"app未设置聚合策略"),

            //app未开通任何广告渠道
            ERROR_REQUEST_APP_NOT_SET_CHANNEL(500701,"app未开通任何广告渠道"),

            // dsp无填充错误码 no ad match
            RTB_SIG_DSP_NO_ADS_ERROR(200000,"广告无填充"),

            //网络出错
            ERROR_SIGMOB_NETWORK(600100,"网络错误"),

            //请求出错
            ERROR_SIGMOB_REQUEST(600101,"广告请求出错"),


            //为找到该渠道的适配器
            ERROR_SIGMOB_NOT_FOUD_ADAPTER(600102,"为找到该渠道的适配器"),

            //配置的策略为空
            ERROR_SIGMOB_STRATEGY_EMPTY(600103,"配置的策略为空"),

            //文件下载错误
            ERROR_SIGMOB_FILE_DOWNLOAD(600104,"文件下载错误"),

            //下载广告超时
            ERROR_SIGMOB_AD_TIME_OUT(600105,"下载广告超时"),

            //SDK未初始化
            ERROR_SIGMOB_NOT_INIT(600900,"SDK未初始化"),
            //广告位为空
            ERROR_SIGMOB_PLACEMENTID_EMPTY(600901,"广告位为空"),
            //策略请求失败
            ERROR_SIGMOB_STRATEGY(600902,"策略请求失败"),
            //安装失败
            ERROR_SIGMOB_INSTALL_FAIL(600903,"安装失败"),

            //激励视频播放出错
            ERROR_SIGMOB_AD_PLAY (610002,"激励视频播放出错"),

            //激励视频未准备好
            ERROR_SIGMOB_NOT_READY(610003,"激励视频未准备好"),

            //server下发的广告信息缺失关键信息
            ERROR_SIGMOB_INFORMATION_LOSE(610004,"server下发的广告信息缺失关键信息"),

            //下载的文件校验md5出错
            ERROR_SIGMOB_FILE_MD5(610005,"下载的文件校验md5出错"),

            //激励视频播接口检查出错（广告过期或者未ready)
            ERROR_SIGMOB_AD_PLAY_CHECK_FAIL(610006,"激励视频播接口检查出错（广告过期或者未ready)"),

            //开屏广告加载超时
            ERROR_SIGMOB_SPLASH_TIMEOUT(620001,"开屏广告加载超时"),

            //开屏广告不支持当前方向
            ERROR_SIGMOB_SPLASH_UNSUPPORT_ORIENTATION(620002,"开屏广告不支持当前方向"),

            //开屏广告不支持的资源类型
            ERROR_SIGMOB_SPLASH_UNSUPPORT_RESOURCE(620003,"开屏广告不支持的资源类型"),

            //开屏广告被广告拦截软件拦截
            ERROR_SPLASH_ADBLOCK(620900,"AD BLOCK");

```