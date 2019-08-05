<!--
 * @Description:
 * @Author: yunshan.wang
 * @Version:
 * @Date: 2019-07-15 19:08:18
 * @LastEditors: yunshan.wang
 * @LastEditTime: 2019-07-31 11:36:48
 -->

<!-- 远程加载, 前面标签不换,替换后面链接即可 -->
<!-- [remoteMarkdownUrl](https://raw.githubusercontent.com/docsifyjs/docsify/develop/README.md) -->


# SigMob平台 iOS SDK接入说明

## 1 iOS SDK接入
### 1.1 iOS SDK导入framework

#### 1.1.1 申请应用的AppID和PlacementId
请向Sigmob平台申请AppId和广告位PlacementId

#### 1.1.2 工程设置导入framework
开发者只需将WindSDK.framework文件拖入工程即可。作为聚合SDK,我们为每个渠道SDK提供了一个.a静态库，如果需要使用其它渠道的SDK,那么只需要将对应渠道的SDK和.a静态库文件拖入工程即可。

### 1.2 Xcode编译选项设置

#### 1.2.1 添加“ObjC”链接器标记
在Xcode中选择项目的Targets->Build Settings，配置Other Link Flags 增加 **-ObjC**。

#### 1.2.2 删除iOS状态栏
尽管这不是必需的步骤，但我们建议采取该步骤以确保 WindSDK 的广告互动和演示可以顺利进行。如要删除状态栏，请打开 Info.plist， 添加**View controller-based status bar appearance**，并将其设置为 NO。

#### 1.2.3 添加HTTP权限
工程info.plist文件设置，点击右边的information Property List后边的 "+" 展开
添加 App Transport Security Settings，先点击左侧展开箭头，再点右侧加号，Allow Arbitrary Loads 选项自动加入，修改值为 YES。 SDK API 已经全部支持HTTPS，但是广告主素材存在非HTTPS情况。

```
  <key>NSAppTransportSecurity</key>
    <dict>
         <key>NSAllowsArbitraryLoads</key>
         <true/>
  </dict>
```
#### 1.2.4 添加定位权限
工程info.plist文件设置，点击右边的information Property List后边的 "+" 展开
添加Privacy - Location When In Use Usage Description。

#### 1.2.5 运行环境配置
+ 支持系统 iOS 7.X 及以上;
+ SDK编译环境 Xcode 9.0+, Base SDK 11.0;
+ 支持架构：i386, x86-64, armv7, armv7s, arm64

#### 1.2.5 添加依赖库
```
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


## 2 广告接入
### 2.1 SDK全局设置
**WindAds**类是整个SDK的入口，在使用SDK功能前需要先进行初始化设置。
**注：**初始化前需在管理系统后台获取到**AppID**、**ApiKey** 和 **PlacementId**。
#### 2.1.1 初始化接口
```
+ (void) startWithOptions:(WindAdOptions *)options;
```
#### 2.1.2 代码示例
```
#import <WindSDK/WindSDK.h>
...

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.

    WindAdOptions *options = [WindAdOptions options];
    options.appId = @"your appId";
    options.apiKey = @"your apiKey";
    [WindAds startWithOptions:options];

    return YES;
}
```

### 2.2 开屏广告接入
**类型说明：**开屏广告主要是 APP 启动时展示的全屏广告视图，开发只要按照接入标准就能够展示设计好的视图。

#### 2.2.1 WindSplashAd接口说明


```
@interface WindSplashAd : NSObject

@property (nonatomic,weak) id<WindSplashAdDelegate> delegate;

/**
 *  拉取广告超时时间，默认为3秒
 *  详解：拉取广告超时时间，开发者调用loadAd方法以后会立即展示app的启动图，然后在该超时时间内，如果广告拉
 *  取成功，则立马展示开屏广告，否则放弃此次广告展示机会。
 */
@property (nonatomic, assign) int fetchDelay;

/*
* 第三方游戏 user_id 标识。
*/
@property (nonatomic,strong) NSString *userId;

- (instancetype)initWithPlacementId:(NSString *)placementId;

/**
 *  广告发起请求并展示在Window中（广告全屏展示）
 *  详解：[可选]发起拉取广告请求,并将获取的广告以全屏形式展示在传入的Window参数中
 */
-(void)loadAdAndShow;


/**
 *  广告发起请求并展示在Window中, 同时在屏幕底部设置应用自身的Logo页面或是自定义View
 *  详解：[可选]发起拉取广告请求,并将获取的广告以半屏形式展示在传入的Window的上半部，剩余部分展示传入的bottomView
 *       请注意bottomView需设置好宽高，所占的空间不能过大，并保证广告界面的高度大于360
 *  @param bottomView 自定义底部View，可以在此View中设置应用Logo
 *
 */
-(void)loadAdAndShowWithBottomView:(UIView *)bottomView;



/**
 *  广告发起请求并展示在Window中, 同时在屏幕底部设置应用自身的Logo页面
 *  详解：[logo会自动读取应用图标]
 *
 @param title 设置标题
 @param description 设置描述信息
 */
- (void)loadADAndShowWithTitle:(NSString *)title description:(NSString *)description;

@end
```

#### 2.2.2 WindSplashAdDelegate代理说明


```
@protocol WindSplashAdDelegate <NSObject>

@optional
/**
 *  开屏广告成功展示
 */
-(void)onSplashAdSuccessPresentScreen:(WindSplashAd *)splashAd;

/**
 *  开屏广告展示失败
 */
-(void)onSplashAdFailToPresent:(WindSplashAd *)splashAd withError:(NSError *)error;


/**
 *  开屏广告点击回调
 */
- (void)onSplashAdClicked:(WindSplashAd *)splashAd;

/**
 *  开屏广告将要关闭回调
 */
- (void)onSplashAdWillClosed:(WindSplashAd *)splashAd;

/**
 *  开屏广告关闭回调
 */
- (void)onSplashAdClosed:(WindSplashAd *)splashAd;
```

#### 2.2.3 开屏广告接入实例

备注:**目前开屏广告只针对 iPhone 设备在垂直方向上展示。**

开屏广告提供全屏开屏接口和半屏展示接口。
开屏广告支持开发者自定义设置开屏底部的界面，用以展示应
用 Logo 等。嵌入代码如下:

```
#import <WindSDK/WindSDK.h>

@interface AppDelegate ()<WindSplashAdDelegate>
@property (nonatomic,strong) WindSplashAd *splashAd;
@end
...


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.

    //1. 初始化SDK
    WindAdOptions *options = [WindAdOptions options];
    options.appId = @"xxx";
    options.apiKey = @"xxx";
    [WindAds startWithOptions:options];

    //2. 开屏广告初始化并展示代码
    self.splashAd = [[WindSplashAd alloc] initWithPlacementId:@"ac6b02f04a"];
    self.splashAd.delegate = self;
    self.splashAd.fetchDelay = 3;////开发者可以设置开屏拉取时间，超时则放弃展

    //[可选]拉取并展示全屏开屏广告
    //[self.splashAd loadAdAndShow];

    //设置开屏底部自定义LogoView，展示半屏开屏广告，logo的高度建议不要过高，以免影响广告的展示效果。
    UIView *bottomView = [UIView new];
    bottomView.backgroundColor = [UIColor whiteColor];
    bottomView.frame = CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, 100);
    UIImageView *imgView = [[UIImageView alloc] init];
    imgView.contentMode = UIViewContentModeScaleAspectFit;
    imgView.frame = bottomView.bounds;
    imgView.image = [UIImage imageNamed:@"sigmob_logo"];
    [bottomView addSubview:imgView];
    //调用半屏开屏接口
    [self.splashAd loadAdAndShowWithBottomView:bottomView];

    return YES;
}
```



### 2.3 激励视频广告接入
**类型说明：**激励视频广告是一种全新的广告形式，用户可选择观看视频广告以换取应用内虚拟货币或者应用内物品和内容等等；这类广告的长度为 15-30 秒，默认为不可跳过，如需可跳过，请联系相应的对接人员进行设置，且广告的结束画面会显示结束页面，引导用户进行后续动作。

#### 2.3.1 WindRewardedVideoAd接口说明

```
@interface WindRewardedVideoAd : NSObject

/*
* 设置广告代理
*/
@property (nonatomic,weak) id<WindRewardedVideoAdDelegate> delegate;


/*
* 用于判断某个广告位的广告是否已经缓存完毕，处于和播放的状态
*/
- (BOOL)isReady:(NSString *)placementId;

/*
* 加载广告接口
*/
- (void)loadRequest:(WindAdRequest *)request withPlacementId:(NSString * _Nullable)placementId;

/*
* 播放广告接口
*/
- (BOOL)playAd:(UIViewController *)controller withPlacementId:(NSString * _Nullable)placementId options:(NSDictionary * _Nullable)options error:( NSError *__autoreleasing _Nullable *_Nullable)error;


@end
```

#### 2.3.2 WindRewardedVideoAdDelegate代理说明

```
@protocol WindRewardedVideoAdDelegate<NSObject>


/**
 激励视频广告AdServer返回广告

 @param placementId 广告位Id
 */
- (void)onVideoAdServerDidSuccess:(NSString *)placementId;


/**
 激励视频广告AdServer无广告返回
 表示无广告填充

 @param placementId 广告位Id
 */
- (void)onVideoAdServerDidFail:(NSString *)placementId;

/**
 激励视频广告物料加载成功(此时isReady=YES)
 广告是否加载完成请以改回调为准

 @param placementId 广告位Id
 */
-(void)onVideoAdLoadSuccess:(NSString * _Nullable)placementId;


/**
 激励视频广告开始播放

 @param placementId 广告位Id
 */
-(void)onVideoAdPlayStart:(NSString * _Nullable)placementId;

/**
 激励视频广告视频播放完毕

 @param placementId 广告位Id
 */
- (void)onVideoAdPlayEnd:(NSString *)placementId;



/**
 激励视频广告发生点击

 @param placementId 广告位Id
 */
-(void)onVideoAdClicked:(NSString * _Nullable)placementId;


/**
 激励视频广告关闭

 @param info WindRewardInfo里面包含一次广告关闭中的是否完整观看等参数
 @param placementId 广告位Id
 */
- (void)onVideoAdClosedWithInfo:(WindRewardInfo * _Nullable)info placementId:(NSString * _Nullable)placementId;


/**
 激励视频广告发生错误

 @param error 发生错误时会有相应的code和message
 @param placementId 广告位Id
 */
-(void)onVideoError:(NSError *)error placementId:(NSString * _Nullable)placementId;


/**
 激励视频广告调用播放时发生错误

 @param error 发生错误时会有相应的code和message
 @param placementId 广告位Id
 */
-(void)onVideoAdPlayError:(NSError *)error placementId:(NSString * _Nullable)placementId;

@end
```
#### 2.3.3 加载激励视频广告

**示例代码**

> userId 是第三方游戏 user_id 标识。

```
WindAdRequest *request = [WindAdRequest request];
//userId可选值
request.userId = @"user id";
/* options作为扩展参数为可选参数，如无需求则不传 */
request.options = @{@"test_key":@"test_value"};
//设置delegate
[[WindRewardedVideoAd sharedInstance] setDelegate:self];
//执行加载广告
[[WindRewardedVideoAd sharedInstance] loadRequest:requestwithPlacementId:@"your placementId"];
```

#### 2.3.4 播放激励视频广告

**Note:**在您确定广告已准备好时，您可以使用以下方法来播放广告：

**使用接口**

```
- (BOOL)playAd:(UIViewController *)controller options:(nullable NSDictionary *)options error:(NSError * _Nullable * _Nullable)error
```

**示例代码**

```
/* 使用isReady检查对应广告位的广告是否可以播放 */
BOOL isReady = [[WindRewardedVideoAd sharedInstance] isReady:@"placementId"];
if (isReady) {
    NSError *error = nil;
    [[WindRewardedVideoAd sharedInstance] playAd:self withPlacementId:@"placementId" options:nil error:&error];
    if (error) {
        //your logic
    }
}
```




## 3 iOS SDK日志调试
如果您想要 SDK 输出日志，请使用以下方法：

```
/*
* enable在Debug环境下默认为yes,在Release模式下默认为NO
*/
- (void)setDebugEnable:(BOOL)enable;
```

获取SDK打印的log信息

```
- (void)setDebugCallBack:(WindAdDebugCallBack)callBack;
```
## 4 错误码
> 相关错误信息可参考此翻译表。

| Error Code | Error Message |
| --- | --- |
| 500420  | 请求的app已经关闭广告服务 |
| 500422 | 请求参数缺少设备信息 |
| 500424 | 缺少设备id相关信息 |
| 500428 | 缺少广告为信息 |
| 500430 | 错误的广告位信息 |
| 500432 | 广告位不存在，或者appid与广告位不匹配 |
| 500433 | 广告位不存在或是已关闭 |
| 500435 | 设备的操作系统类型，与请求的app的系统类型不匹配 |
| 500436 | 广告单元id与请求的广告类型不匹配 |
| 500437 | 缺少idfa。仅（iOS） |
| 500473 | 请求的app不存在 |
| 500700 | app未设置聚合策略 |
| 500701 | app未开通任何广告渠道 |
| 200000 | 无广告填充 |
| 600100 | 网络出错 |
| 600101 | 请求出错 |
| 600102 | 未找到该渠道的适配器 |
| 600103 | 配置的策略为空 |
| 600104 | 文件下载错误 |
| 600105 | 下载广告超时 |
| 600106 | 聚合通知给开发者的统一错误码，由于多渠道无法区分具体原因。配置单一渠道时使用该渠道错误码。 |
| 600107 | 不能在后台调用load |
| 600108 | protoBuf协议解析出错 |
| 610002 | 激励视频播放出错 |
| 610003 | 激励视频广告未准备好 |
| 610004 | server下发的广告信息缺失关键信息 |
| 610005 | 下载的文件校验md5出错 |
| 620002 | 开屏广告不支持当前方向 |
| 620003 | 开屏广告不支持的资源类型 |


