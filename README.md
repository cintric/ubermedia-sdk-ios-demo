# Demo app for the ClearBid Header Bidding SDK (v0.6.9)

The ClearBid Header Bidding SDK for iOS allows you to optimize ad revenue by creating an open auction for your ad space instead of using the traditional waterfall method like other mediation SDKs. It is lightweight and optimized to minimize impact on your application.

### Initalize the sdk
To integrate the sdk drag and drop the `ClearBid.framework` and the `ClearBidResources.bundle` files into your project.
Make sure it is included in your "Link Binary With Libraries" section of your target's Build Phases

In your application delegate import the sdk:

`#import <ClearBid/ClearBid.h>`

Then initalize the sdk:
```
objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    // Always initalize the ubermedia sdk before requesting advertisements.
    [ClearBid initWithSDKKey:@"YOUR_SDK_KEY_HERE" andSecret:@"YOUR_SECRET_KEY_HERE"];
    
    // Override point for customization after application launch.
    return YES;
}
```
Put in your own sdk key and secret here.

### Configure your project

It is very critical to enable arbitrary loads in your `Info.plist` file. This is because advertisements pull content from websites that are not using ssl. To do so right click on your `Info.plist` file and select `Open As > Source Code` and paste this directly below the line that says `<dict>` (usually line 4):

```xml
<key>NSAppTransportSecurity</key>
<dict>
	<key>NSAllowsArbitraryLoads</key>
	<true/>
	<key>NSAllowsArbitraryLoadsInWebContent</key>
	<true/>
</dict>
```
**Important:** Do not include the `NSAllowsArbitraryLoadsForMedia` key. Otherwise it will potneitally override `NSAllowsArbitraryLoads` and break critical functionality. If this is a problem the SDK should log a warning.

### Displaying an ad. (Skip this step if you're using an adapter)
To display a banner ad, initalize the UMAdView class like so:
```objective-c
    CGRect frame = CGRectMake(0, [UIApplication sharedApplication].statusBarFrame.size.height, self.view.frame.size.width, 50);
    CBAdView *adView = [[CBAdView alloc] initWithFrame:frame];
    [self.view addSubview:adView];
    self.adView = adView;
    
    // Call this to load your ad (IMPORTANT: change the placement_id or you will only get test ads.)
    [self.adView loadAd:@"test_ad_placement_id" forSize:CGSizeMake(320, 50)];
    
    // The ad will default to a 30 second refresh rate, you can use this method to change the refresh rate. Set to 0 to disable refreshing.
    [self.adView changeAdRefreshRate:60];
```

Optionally, you can also use the storyboard or a nib by setting your view class to `CBAdView` in the interface builder.
You will still need request an ad in your view controller.
```objective-c
[self.adView loadAd:@"test_ad_placement_id" forSize:CGSizeMake(320, 50)];
```

If using the interface builder `self.adView` should be an `IBOutlet` pointing to the view in your interface.

### (Optional) Delegate Methods

You can implement delegate methods via the `CBAdViewDelegate` protocol.

```objective-c
#pragma mark - CBAdViewDelegate methods

/**
 Called when an ad view successfully loads
 
 @param view Reference to the instance of CBAdView which has successfully loaded an ad
 */
- (void)adViewDidLoadAd:(CBAdView *)view;


/**
 Called when an ad view fails to load. This may happen if there is an error, of if no bids are returned for the placement.
 
 @param view Reference to the instance of CBAdView which has failed to load an ad
 @param message A message describing why the ad has failed to load
 */
- (void)adViewDidFailToLoadAd:(CBAdView *)view withMessage:(nullable NSString *)message;


/**
 Called when the ad is clicked. The user will exit the application via the url provided by the advertisement.
 
 @param view Reference to the instance of CBAdView which has been clicked
 */
- (void)willLeaveApplicationFromAd:(CBAdView *)view;

/**
 Called when an ad view will close the ad
 
 @param view Reference to the instance of CBAdView which has successfully loaded an ad
 */
- (void)adViewWillCloseAd:(CBAdView *)view;

/**
 Called when an ad view did close the ad
 
 @param view Reference to the instance of CBAdView which has successfully loaded an ad
 */
- (void)adViewDidCloseAd:(CBAdView *)view;
```

### (Optional) Adapters

To use the SDK with other mediation platforms will require adapters. Included in the [Adapters folder](https://github.com/cintric/ubermedia-sdk-ios-demo/tree/master/Adapters) is the adapter class for the google admob sdk (also compatible with Google's Double Click for Publishers.)

Just include the adapter in your project and set up a mediation layer in the console to invoke the custom class named `UMAdMobCustomEventBanner`. Set the server paramater to your ubermedia placement ID. For testing you can use `test_ad_placement_id`. 

## Line item targeting

If you are using line item targeting for DFP then you will need to pre-cache your ad in the app delegate after you initalize the sdk like so:

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    // Always initalize the ubermedia sdk before requesting advertisements.
    [ClearBid initWithSDKKey:@"YOUR_SDK_KEY_HERE" andSecret:@"YOUR_SECRET_KEY_HERE"];
    
    // Make sure to change your ad placement id.
    [ClearBid preCacheAd:@"test_ad_placement_id" forSize:CGSizeMake(320, 50)];
    
    // Override point for customization after application launch.
    return YES;
}
```

### Using DFP for line item targeting
When you request an ad from DFP you need to set the custom targeting like so:
```objective-c
DFPRequest *request = [DFPRequest request];
// Make sure to change your ad placement id
request.customTargeting = [ClearBid getTargetingParametersForAd:@"test_ad_placement_id"];
[self.dfpBannerView loadRequest:request];
```

Make sure to include the `UMDFPTargetingCustomEventBanner.h` and `UMDFPTargetingCustomEventBanner.m` files from the [Adapters folder](https://github.com/cintric/ubermedia-sdk-ios-demo/tree/master/Adapters) in your project.

### Using MoPub for line item targeting

When you request an ad from MoPub you need to set the custom targeting keywords like so:
```objective-c
// Make sure to change your ad placement id
NSString *keywordsString = [ClearBid getTargetingParametersAsStringForAd:@"test_ad_placement_id"];

// adView should be an MPAdView
self.adView.keywords = keywordsString;
```

Make sure to include the `MPUberMediaBannerCustomEvent.h` and `MPUberMediaBannerCustomEvent.m` files from the [Adapters folder](https://github.com/cintric/ubermedia-sdk-ios-demo/tree/master/Adapters) in your project.

## Using MoPub auto refresh
If you wan't to use mopub auto refresh then you will need to set your viewcontroller to be an `ClearBidDelegate` and set the keywords when receiving the newAdIsCachedForId callback.

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];   
    [ClearBid setClearBidDelegate:self];
}

- (void)newAdIsCachedForId:(NSString *)adUnitId andSize:(CGSize)size {
    if ([adUnitId isEqualToString:MY_CLEARBID_INTERSTITIAL_AD_ID]) {
	mopubInterstitial.keywords = [ClearBid getTargetingParametersAsStringForAd:adUnitId];
    if ([adUnitId isEqualToString:MY_CLEARBID_BANNER_AD_ID]) {
        mopubBanner.keywords = [ClearBid getTargetingParametersAsStringForAd:adUnitId];
    }
}
```

# Interstitials
Using interstitials with an adapter is almost identicial to using banners. You will need to cache your interstitial ad in the app delegate like so:

```objective-c
[ClearBid preCacheAd:MY_CLEARBID_INTERSTITIAL_AD_ID forSize:CGSizeMake(320, 480) interstitial:YES];
```

You will also need to set the targeting keywords either manually when requesting an ad or with the delegate method as can be seen above.

Make sure to include the `ClearBidMPInterstitialCustomEvent.h` and `ClearBidMPInterstitialCustomEvent.m` files from the [Adapters folder](https://github.com/cintric/ubermedia-sdk-ios-demo/tree/master/Adapters) in your project.

