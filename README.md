# iOS-NetworkExtension-NEHotspotHelper
简书地址：http://www.jianshu.com/p/aa6219925c2e
>NOTE
- 应用程序的Info.plist必须添加一个包含“网络认证”的UIBackgroundModes数组
- 应用程序必须设置“com.apple.developer.networking.HotspotHelper'作为它的权利之一。该权利的值是一个布尔值true
- 要申请这个权利，请发送E-MAIL到networkextension@apple.com或点击申请链接[https://developer.apple.com/contact/network-extension/](https://developer.apple.com/contact/network-extension/)
- 更多信息请参阅苹果的*[Hotspot Network Subsystem Programming Guide](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/Hotspot_Network_Subsystem_Guide/Contents/Introduction.html#//apple_ref/doc/uid/TP40016639)*

---
>NEHotsportHelper  Custom authentication for Wi-Fi Hotspots
Register with the system as a Hotspot Helper
From the backgroud
- Claim Hotspots with a level of confidence
- Perform the initial authentication
- Maintain the authentication session
- Annotate Wifi networks in the WiFi network scanner

### Register a Hotspot Helper

```
+ (BOOL)registerWithOptions:(NSDictionary<NSString *, NSObject *> *)options 
                    queue:(dispatch_queue_t)queue 
                    handler:(NEHotspotHelperHandler)handler
```
```
@param options
    <key>kNEHotspotHelperOptionDisplayName</key>
    <string>WIFI的注释tag字符串</string>

@param queue 
    dispatch_queue_t 用来调用handle的block

@param handler
    NEHotspotHelperHandler block 用于执行处理 helper commands.

@return 注册成功YES, 否则NO.

@discussion
    一旦这个API调用成功，应用程序有资格在后台启动，并参与各种热点相关的功能。
    当应用程序启动此方法应该调用一次。再次调用它会不会产生影响，并返回NO。
```
---
### Manage Hotspot Networks
    + (BOOL)logoff:(NEHotspotNetwork *)network

```
@param network 
    对应当前关联的WiFi网络NEHotspotNetwork

@return 注销命令已成功进入队列YES, 否则NO.

@discussion
    调用此方法使kNEHotspotHelperCommandTypeLogoff型的NEHotspotHelperCommand向应用程序发出的“handler”模块
    网络参数必须符合当前关联的WiFi网络，即它必须来自对NEHotspotHelperCommand网络属性或方法supportedInterfaces
```

    + (NSArray *)supportedNetworkInterfaces

```
@return
    如果没有网络接口被管理，返回nil。否则，返回NEHotspotNetwork对象数组。

@discussion
    每个网络接口由NEHotspotNetwork对象表示。当前返回的数组包含一个NEHotspotNetwork对象代表Wi-Fi接口。
    这种方法的主要目的是当没有得到一个命令来处理它时，让一个热点助手偶尔提供在UI里其准确的状态。
    此方法加上NEHotspotNetwork的isChosenHelper方法允许应用程序知道它是否是当前处理的网络。
```
---
### Data Types
    typedef void (^NEHotspotHelperHandler)(NEHotspotHelperCommand * cmd)
```
@discussion
    当调用方法registerWithOptions:queue:handler:时，Hotspot Helper app提供这种类型的blcok。
    每次有要处理的命令时调用block。
```
---
### 实现代码
```
NSDictionary *options = [NSDictionary dictionaryWithObjectsAndKeys:@"WI-FI TAG", kNEHotspotHelperOptionDisplayName, nil];
    
    dispatch_queue_t queue = dispatch_queue_create("com.myapp.ex", 0); 
    
    BOOL returnType = [NEHotspotHelper registerWithOptions:options queue:queue handler: ^(NEHotspotHelperCommand * cmd) {

        if(cmd.commandType == kNEHotspotHelperCommandTypeEvaluate || 
           cmd.commandType == kNEHotspotHelperCommandTypeFilterScanList) 
        {
            for (NEHotspotNetwork* network  in cmd.networkList) 
            {
                if ([network.SSID isEqualToString:@"WX_moses"]) 
                {
                    [network setConfidence:kNEHotspotHelperConfidenceHigh];
                    [network setPassword:@"mypassword"];
                    NSLog(@"Confidence set to high for ssid: %@ (%@)\n\n", network.SSID, network.BSSID);
//                    NSMutableArray *hotspotList = [NSMutableArray new];     
//                    [hotspotList addObject:network];

                    // This is required
                    NEHotspotHelperResponse *response = [cmd createResponse:kNEHotspotHelperResultSuccess];
                    [response setNetwork:network];
                    [response deliver];
                }
            }
        }
    }];

```
