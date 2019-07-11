# NexPlayer SDK Quick Start with __Widevine__

This tutorial demonstrates how you are able to handle the NexPlayer SDK in iOS applications with Widevine.  In this tutorial, a sample player will be implemented in Objective-C with several buttons which control the player with Widevine.

First thing you have to keep in mind before implementing NexPlayer SDK is that most of player methods behave __asynchronously__. There are numerous methods available in _NexPlayerDelegate.h_ which listen to what is happening in NexPlayer. Asynchronous approach is strongly recommended in NexPlayer SDK.

__NOTE__: If you are familiar with NexPlayerSDK, you can start from step #8. It is is same as [NexPlayer SDK QuickStart](https://github.com/JohnKim9316/NexPlayer_iOS) from step #1 to #7

---

## Step #1. Create a new project
1. Open Xcode from the __/Applications__ directory.

2. Click “Create a new Xcode project” (or select File > New > Project).

3. Select iOS at the top of the dialog.

4. In the Application section, select Single View Application and then click Next.

5. Set the following values to name your app and choose additional options for your project.

	Product Name: NexPlayerQuickStart

	Team: If this is not automatically filled in, set the team to None.

	Organization Name: The name of your organization or your own name. You can leave this blank.

	Organization Identifier: Your organization identifier, if you have one.

	Bundle Identifier: This value is automatically generated based on your product name and organization identifier.

	Language: Objective-C

	Use Core Data: Unselected.

	Include Unit Tests: Unselected.

	Include UI Tests: Unselected.

6. Click Next.

7. Choose a location to save your project and click Create


## Step #2. Import required frameworks
1. Copy __NexPlayerSDK.framework__ to your Xcode project folder.

2. Go to Navigation Area in Xcode > XCODE PROJECT FILE(indicated by blue icon) > TARGETS > General > In Linked Frameworks and Libraries section, click the plus sign(+) to add frameworks and libraries on the list

	To import NexPlayerSDK, click the plus sign(+) in Linked Frameworks and Libraries section > Add Other > Click NexPlayerSDK in your Xcode folder.

>- AVKit
>- AVFoundation
>- VideoToolbox
>- AudioToolbox
>- CoreAudio
>- CoreMedia
>- SystemConfiguration
>- NexPlayerSDK


3. Go to Navigation Area in Xcode > XCODE PROJECT FILE(indicated by blue icon) > PROJECT > Build Settings > All | Combined > Search __Other Linker Flags__ > Add __-lc++__ in __Other Linker Flags__


---

## Step #3. Setup PlayerView


### _ViewController.h_
```javascript
#import <UIKit/UIKit.h>
#import <NexPlayerSDK/NexPlayerSDK.h>

@interface ViewController : UIViewController <NXPlayerDelegate> {
}

@property (nonatomic, strong) NXPlayerView *playerView;

@end
```

### _ViewController.m_
```javascript
#import "ViewController.h"
#import <MediaPlayer/MediaPlayer.h>
#import <NexPlayerSDK/NexPlayerSDK.h>

@interface ViewController ()

@end

@implementation ViewController

@synthesize playerView;

- (void)viewDidLoad {
	[super viewDidLoad];

	// playerView should be set up first.
	// If playerView is set up after buttons are allocated, buttons will be hidden.
	[self setupPlayerView];
}


- (void)setupPlayerView
{
	self.view = [[UIView alloc] init];

	//playerView will be fit in self.view automatically with black background color.
	self.playerView = [[NXPlayerView alloc] initWithFrame:self.view.bounds];
	self.playerView.autoresizingMask =
	UIViewAutoresizingFlexibleWidth|UIViewAutoresizingFlexibleHeight;
	self.playerView.backgroundColor = [UIColor blackColor];
	self.playerView.autoScaling = NXScale_FitInView;

	//This is the way how user can interact with player.
	self.playerView.player.delegate = self;

	[self.view addSubview:self.playerView];
}

@end
```
When the video starts, it is played on the playerView, which means the player you are going to cope with is actually on the playerView. You can consider playerView as one sort of _layer_. Most of processes in NexPlayer operate asynchronously through delegate and that is the reason why NexPlayerDelegate is added in ViewController.h

---
## Step #4. Setup Buttons
### _ViewController.h_
```javascript
...

@property (nonatomic, strong) UIStackView *buttonStackView;
@property (nonatomic, strong) UIButton *startButton;
@property (nonatomic, strong) UIButton *resumeButton;
@property (nonatomic, strong) UIButton *pauseButton;
@property (nonatomic, strong) UIButton *stopButton;
```

### _ViewController.m_
```javascript
...

@synthesize buttonStackView;
@synthesize startButton;
@synthesize resumeButton;
@synthesize pauseButton;
@synthesize stopButton;

...

- (void)viewDidLoad
{
	...
	[self setupButtons];
}

// setupButtons will set up four buttons by using UIStackView called buttonStackView.
// UIStackView is used to locate four buttons in the middle of self.view.
- (void)setupButtons
{
	//First, set up all the buttons.
	[self setupStartButton];
	[self setupResumeButton];
	[self setupPauseButton];
	[self setupStopButton];

	//Second, Initialize buttonStackView.
	buttonStackView = [[UIStackView alloc] init];
	buttonStackView.axis = UILayoutConstraintAxisHorizontal;
	buttonStackView.distribution = UIStackViewDistributionEqualSpacing;
	buttonStackView.alignment = UIStackViewAlignmentCenter;
	buttonStackView.spacing = 16;

	//Third, Add all the buttons you have already set up into buttonStackView.
	[buttonStackView addArrangedSubview:startButton];
	[buttonStackView addArrangedSubview:resumeButton];
	[buttonStackView addArrangedSubview:pauseButton];
	[buttonStackView addArrangedSubview:stopButton];

	// Auto Layout is adapted.
	buttonStackView.translatesAutoresizingMaskIntoConstraints = false;

	[self.view addSubview:buttonStackView];

	// It will be located in the middle of self.view.
	[buttonStackView.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor].active = YES;
	[buttonStackView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor constant:-16].active = YES;
}


- (void)setupStartButton
{
	// In this QuickStart, simple title is used to indicate startButton.
	startButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
	[startButton setTitle:@"START" forState:UIControlStateNormal];
	[startButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
	[startButton addTarget:self action:@selector(touchUpStartButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpStartButton:(UIButton *)startButton
{
	// In this QuickStart if player stops,
	// completedAsyncCmdStopWithResult method will be called asynchronously
	// and then close the player.

	// completedAsyncCmdStopWithResult will be explained later in this QuickStart.

	// When the player is closed, it should be opened first to start the player.
	// The state of player will be changed asynchronously by buttons.
	if(self.playerView.player.state == NXPlayerStateClose)
	{
		[self.playerView.player open:url];
	}
	else
	{
		[self.playerView.player start];
	}
}


- (void)setupResumeButton
{
	resumeButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
	[resumeButton setTitle:@"RESUME" forState:UIControlStateNormal];
	[resumeButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
	[resumeButton addTarget:self action:@selector(touchUpResumeButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpResumeButton:(UIButton *)resumeButton
{
	// Resume is closely related to Pause.
	// Bear in mind that this is different from start.
	// [self.playerView.player start] starts from the beginning of the content,
	// whereas Resume starts from the place where you paused.
	[self.playerView.player resume];
}


- (void)setupPauseButton
{
	pauseButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
	[pauseButton setTitle:@"PAUSE" forState:UIControlStateNormal];
	[pauseButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
	[pauseButton addTarget:self action:@selector(touchUpPauseButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpPauseButton:(UIButton *)pauseButton
{
	[self.playerView.player pause];
}


- (void)setupStopButton
{
	stopButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
	[stopButton setTitle:@"STOP" forState:UIControlStateNormal];
	[stopButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
	[stopButton addTarget:self action:@selector(touchUpStopButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpStopButton:(UIButton *)stopButton
{
	// In this QuickStart, stop will not only stop the player
	// but also close the player by completedAsyncCmdStopWithResult.

	// The scenario of the player is totally depending on how you design the player.
	[self.playerView.player stop];
}

```
Let's set the buttons in order to control the NexPlayer. Normally two buttons are enough to handle basic the operations in the player but in this QuickStart, four buttons are implemented to show you how it works directly. Four buttons will be located in UIStackView named buttonStackView which is for handy UI setting.

UIStackView is totally optional to implement NexPlayer since the purpose of UIStackView is to locate UIButtons.

The way how you are able to modify the state of player is to call methods such as start, resume, pause and stop. Bear in mind that the method named start means starting the video from the beginning.

---

## Step #5. Open URL
### _ViewController.h_
```javascript
...

@property (nonatomic, strong) NSString *url;

...

```
### _ViewController.m_
```javascript
...

- (void)viewDidLoad {

	...
	url = @"https://storage.googleapis.com/wvmedia/cenc/h264/tears/tears.mpd";
	[self.playerView.player open:url];
}

...
```
You are able to open URL by the method called _open_.

## Step #6. Implement asynchronously


### _ViewController.m_
```javascript
...


// This method will be called asynchronously when the [self.playerView.player open] completes.
- (void)nexPlayer:(NXPlayer *)nxplayer completedAsyncCmdOpenWithResult:(NXError)result playbackType:(NXPlaybackType)type
{
	// Check if there is no error
	if(result == NXErrorNone)
	{
		// Then start the player.
		[self.playerView.player start];
	}
	else
	{
		// If there is error, call encounteredError method with result.
		[self nexPlayer:nxplayer encounteredError:result];
	}
}


// This method will be called asynchronously when the [self.playerView.player stop] completes by stopButton.
- (void)nexPlayer:(NXPlayer *)nxplayer completedAsyncCmdStopWithResult:(NXError)result
{
	if(result == NXErrorNone)
	{
		// In this QuickStart, when the user stops the player, it will be closed.
		[self.playerView.player close];
	}
	else
	{
		[self nexPlayer:nxplayer encounteredError:result];
	}
}


// You are able to handle the error in this method.
// If you would like to process specific error, you can find more information by errorCode.
- (void)nexPlayer:(NXPlayer *)nxplayer encounteredError:(NXError)errorCode
{
	NSLog(@"encounteredError");
}

```
When the player completes to open the URL, nxplayer completedAsyncCmdOpenWithResult method will be called asynchronously and there is possibility to encounter error while opening the player. For this situation, the error needs to be handled appropriately by encounteredError method and there are several types of NXError declared in _NXError.h_

The way how player plays a content depends on what kind of content it is. If you would like to know how exactly the player works, you can find out in _NexPlayerDelegate.h_ by calling all the methods in your ViewController.


If you would like to handle specific NXError, you can find more precise information by (NXError)errorCode in encounteredError method.


## Step #7. Setup URL label

### _ViewController.h_
```javascript
...

@property (nonatomic, strong) UILabel *urlLabel;

...
```

### _ViewController.m_
```javascript
...

@synthesize urlLabel;

...

- (void)viewDidLoad {

	...
	[self setupUrlLabel:url];
}

...

// This label is to check which url is currently playing on the player.
- (void)setupUrlLabel:(NSString *)url
{
	urlLabel = [[UILabel alloc] init];
	urlLabel.text = url;
	urlLabel.textColor = UIColor.whiteColor;
	urlLabel.numberOfLines = 0;

	[self.view addSubview:urlLabel];

	// Auto Layout is adapted.
	urlLabel.translatesAutoresizingMaskIntoConstraints = false;

	[urlLabel.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor constant:16].active = YES;
	[urlLabel.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor constant:-16].active = YES;
	[urlLabel.topAnchor constraintEqualToAnchor:self.view.topAnchor constant:16].active = YES;
}

...
```
In order to identify which url is playing at the moment, let's put the url label. That is all for this Quick Start. As you may know, Those buttons and labels are totally optional.


---

## Widevine
Now it is time to set up the Widevine. The first thing that you have to be aware of Widevine frameworks is that there are three types of Widevine Frameworks (develop, simulator, and release). In this tutorial, widevine_cdm_sdk_release.framework will be used in this tutorial on __actual device__ to implement Widevine DRM with NexPlayer.

The device should follow these two requirements.
1. iOS version 9.0+
2. arm64 CPU architecture

## Step #8. Import Widevine Integration
1. Copy __WidevineIntegration.framework__ and __widevine_cdm_sdk_release.framework__ to your Xcode project folder.

2. Go to Navigation Area in Xcode > XCODE PROJECT FILE(indicated by blue icon) > TARGETS > General > In Embedded Binaries, click the plus sign(+) > Add __WidevineIntegration.framework__ and __widevine_cdm_sdk_release.framework__ in your Xcode project directory.

3. Check the order of frameworks in Linked Frameworks and Libraries section. Order is extremely important in Linked Frameworks and Libraries section like below.

>- AVKit
>- AVFoundation
>- VideoToolbox
>- AudioToolbox
>- CoreAudio
>- CoreMedia
>- SystemConfiguration
>- __widevine_cdm_sdk_release__
>- __WidevineIntegration__
>- __NexPlayerSDK__

NOTE: If the NexPlayerSDK is not located __under__ both widevine_cdm_sdk_release and WidevineIntegration, WidevineIntegration is not able to call NexPlayerSDK.

4. Go to Navigation Area in Xcode > XCODE PROJECT FILE(indicated by blue icon) > PROJECT > Build Settings > All | Combined > Search __Allow Non-Modular Includes In Framework Modules__ > YES

5. Go to Navigation Area in Xcode > XCODE PROJECT FILE(indicated by blue icon) > PROJECT > Build Settings > All | Combined > Search __Enable Bitcode__ > No


## Step #9. Setup Widevine

### _ViewController.h_
```javascript
...

#import <WidevineIntegration/WidevineIntegration.h>

@interface ViewController : UIViewController <NXPlayerDelegate> {
}

...

@property (nonatomic, strong) WidevineIntegration *widevine;
@property (nonatomic, strong) ClientInfo *clientInfo;

```

### _ViewController.m_
```javascript
...

#import <WidevineIntegration/WidevineIntegration.h>
#import <WidevineIntegration/NexWidevineDelegate.h>

...

@synthesize widevine;
@synthesize clientInfo;


...

- (void)viewDidLoad {

  	...
    // Set up LicenseServerURL
    self.widevine.licenseServerUrl = @"https://proxy.uat.widevine.com/proxy?provider=widevine_test";

    // Start WidevineIntegration before opening the content.
    [self.widevine start];

    //Keep in mind that player is in playerView.
    [self.playerView.player open: url];
}

...

- (void)setupPlayerView
{
    ...

    // Widevine Setting
    self.clientInfo = [[ClientInfo alloc] init];
    clientInfo.productName = @"WidevineGuide";
    clientInfo.companyName = @"Nexstreaming";
    clientInfo.deviceName = @"iPhone";
    clientInfo.modelName = @"6+";
    clientInfo.archName = @"arm64";
    clientInfo.buildInfo = @"0.1";



    // Create WidevineIntegration with clientInfo
    self.widevine = [[WidevineIntegration alloc] initWithNXPlayer:self.playerView.player clientInfo:
                     clientInfo];
    self.widevine.nexWVdelegate = self;

    ...
}

...

- (void)nexPlayer:(NXPlayer *)nxplayer completedAsyncCmdStopWithResult:(NXError)result
{
    if(result == NXErrorNone)
    {
        // widevine should stop before closing the player.
        [self.widevine stop];

        // In this QuickStart, when the user stops the player, it will be closed.
        [self.playerView.player close];
    }
    else
    {
        [self nexPlayer:nxplayer encounteredError:result];
    }
}

...

- (void)touchUpStartButton:(UIButton *)startButton
{
    // If player stops, completedAsyncCmdStopWithResult method will be called asynchronously and then close the player in this QuickStart.
    // It should be opened first in order to start the player when the player is closed, .
    // The state of player will be changed asynchronously by buttons.
    if(self.playerView.player.state == NXPlayerStateClose)
    {
        // Start WidevineIntegration before open the content.
        [self.widevine start];

        [self.playerView.player open:url];
    }
    else
    {
        [self.playerView.player start];
    }
}

...

```

Client information is needed to initialize Widevine object. Before start API of WidevineIntegration is called, the license server URL should be set.

Keep in mind that start API of WidevineIntegration should be called before open API of NexPlayerSDK, and stop API of WidevineIntegration should be called before close API of NexPlayerSDK.

For further information, please refer to _NexPlayerSDK_with_Widevine_Integration_For_iOS_Technical_Reference_Manual.pdf_
 document.
 
---
Here is the entire code in NexPlayerQuickStart with Widevine.

## NexPlayerQuickStart with Widevine

### _ViewController.h_
```javascript
#import <UIKit/UIKit.h>
#import <NexPlayerSDK/NexPlayerSDK.h>
#import <WidevineIntegration/WidevineIntegration.h>

@interface ViewController : UIViewController <NXPlayerDelegate> {
}

@property (nonatomic, strong) NXPlayerView *playerView;

@property (nonatomic, strong) UILabel *urlLabel;
@property (nonatomic, strong) NSString *url;

@property (nonatomic, strong) UIStackView *buttonStackView;
@property (nonatomic, strong) UIButton *startButton;
@property (nonatomic, strong) UIButton *resumeButton;
@property (nonatomic, strong) UIButton *pauseButton;
@property (nonatomic, strong) UIButton *stopButton;

@property (nonatomic, strong) WidevineIntegration *widevine;
@property (nonatomic, strong) ClientInfo *clientInfo;

@end
```

### _ViewController.m_
```javascript
#import "ViewController.h"
#import <MediaPlayer/MediaPlayer.h>
#import <NexPlayerSDK/NexPlayerSDK.h>
#import <WidevineIntegration/WidevineIntegration.h>

#import <WidevineIntegration/NexWidevineDelegate.h>

@interface ViewController () <NexWidevineDelegate>

@end

@implementation ViewController

@synthesize playerView;

@synthesize url;
@synthesize urlLabel;

@synthesize buttonStackView;
@synthesize startButton;
@synthesize resumeButton;
@synthesize pauseButton;
@synthesize stopButton;

@synthesize widevine;
@synthesize clientInfo;

- (void)viewDidLoad {
    [super viewDidLoad];

    // playerView should be set up first.
    // If playerView is set up after buttons are allocated, buttons will be hidden.
    [self setupPlayerView];
    [self setupButtons];

    // URL should be included Widevine to check if it works
    url = @"https://storage.googleapis.com/wvmedia/cenc/h264/tears/tears.mpd";

    [self setupUrlLabel:url];

    // Set up LicenseServerURL
    self.widevine.licenseServerUrl = @"https://proxy.uat.widevine.com/proxy?provider=widevine_test";

    // Start WidevineIntegration before open the content.
    [self.widevine start];

    //Keep in mind that player is in playerView.
    [self.playerView.player open: url];

}


- (void)setupPlayerView
{
    self.view = [[UIView alloc] init];

    //playerView will be fit in self.view automatically with black background color.
    self.playerView = [[NXPlayerView alloc] initWithFrame:self.view.bounds];
    self.playerView.autoresizingMask =
    UIViewAutoresizingFlexibleWidth|UIViewAutoresizingFlexibleHeight;
    self.playerView.backgroundColor = [UIColor blackColor];
    self.playerView.autoScaling = NXScale_FitInView;

    // Widevine Setting
    self.clientInfo = [[ClientInfo alloc] init];
    clientInfo.productName = @"WidevineGuide";
    clientInfo.companyName = @"Nexstreaming";
    clientInfo.deviceName = @"iPhone";
    clientInfo.modelName = @"6+";
    clientInfo.archName = @"arm64";
    clientInfo.buildInfo = @"0.1";



    // Create WidevineIntegration with clientInfo
    self.widevine = [[WidevineIntegration alloc] initWithNXPlayer:self.playerView.player clientInfo:
                     clientInfo];
    self.widevine.nexWVdelegate = self;

    //This is the way how user can interact with player.
    self.playerView.player.delegate = self;

    [self.view addSubview:self.playerView];
}


// setupButtons will set up four buttons by using UIStackView called buttonStackView.
// UIStackView is used to locate four buttons in the middle of self.view.
- (void)setupButtons
{
    //First, set up all the buttons.
    [self setupStartButton];
    [self setupResumeButton];
    [self setupPauseButton];
    [self setupStopButton];

    //Second, Initialize buttonStackView.
    buttonStackView = [[UIStackView alloc] init];
    buttonStackView.axis = UILayoutConstraintAxisHorizontal;
    buttonStackView.distribution = UIStackViewDistributionEqualSpacing;
    buttonStackView.alignment = UIStackViewAlignmentCenter;
    buttonStackView.spacing = 16;

    //Third, Add all the buttons you have already set up into buttonStackView.
    [buttonStackView addArrangedSubview:startButton];
    [buttonStackView addArrangedSubview:resumeButton];
    [buttonStackView addArrangedSubview:pauseButton];
    [buttonStackView addArrangedSubview:stopButton];

    // Auto Layout is adapted.
    buttonStackView.translatesAutoresizingMaskIntoConstraints = false;

    [self.view addSubview:buttonStackView];

    // It will be located in the middle of self.view.
    [buttonStackView.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor].active = YES;
    [buttonStackView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor constant:-16].active = YES;
}


- (void)setupStartButton
{
    // In this QuickStart, simple title is used to indicate startButton.
    startButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    [startButton setTitle:@"START" forState:UIControlStateNormal];
    [startButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
    [startButton addTarget:self action:@selector(touchUpStartButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpStartButton:(UIButton *)startButton
{
    // If player stops, completedAsyncCmdStopWithResult method will be called asynchronously and then close the player in this QuickStart.
    // It should be opened first in order to start the player when the player is closed, .
    // The state of player will be changed asynchronously by buttons.
    if(self.playerView.player.state == NXPlayerStateClose)
    {
        // Start WidevineIntegration before opening the content.
        [self.widevine start];

        [self.playerView.player open:url];
    }
    else
    {
        [self.playerView.player start];
    }
}


- (void)setupResumeButton
{
    resumeButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    [resumeButton setTitle:@"RESUME" forState:UIControlStateNormal];
    [resumeButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
    [resumeButton addTarget:self action:@selector(touchUpResumeButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpResumeButton:(UIButton *)resumeButton
{
    // Resume is closely related to pause.
    // Bear in mind that this is different from start.
    // [self.playerView.player start] starts from the beginning of the content,
    // whereas resume starts from the place where you paused.
    [self.playerView.player resume];
}


- (void)setupPauseButton
{
    pauseButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    [pauseButton setTitle:@"PAUSE" forState:UIControlStateNormal];
    [pauseButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
    [pauseButton addTarget:self action:@selector(touchUpPauseButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpPauseButton:(UIButton *)pauseButton
{
    [self.playerView.player pause];
}


- (void)setupStopButton
{
    stopButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    [stopButton setTitle:@"STOP" forState:UIControlStateNormal];
    [stopButton setTitleColor:UIColor.whiteColor forState:UIControlStateNormal];
    [stopButton addTarget:self action:@selector(touchUpStopButton:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)touchUpStopButton:(UIButton *)stopButton
{
    // The scenario of the player is totally depending on how you design the player.
    [self.playerView.player stop];
}


// This label is to check which url is currently playing on the player.
- (void)setupUrlLabel:(NSString *)url
{
    urlLabel = [[UILabel alloc] init];
    urlLabel.text = url;
    urlLabel.textColor = UIColor.whiteColor;
    urlLabel.numberOfLines = 0;

    [self.view addSubview:urlLabel];

    // Auto Layout is adapted.
    urlLabel.translatesAutoresizingMaskIntoConstraints = false;

    [urlLabel.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor constant:16].active = YES;
    [urlLabel.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor constant:-16].active = YES;
    [urlLabel.topAnchor constraintEqualToAnchor:self.view.topAnchor constant:16].active = YES;
}


// This method will be called asynchronously when the [self.playerView.player open] completes.
- (void)nexPlayer:(NXPlayer *)nxplayer completedAsyncCmdOpenWithResult:(NXError)result playbackType:(NXPlaybackType)type
{
    // Check if there is no error
    if(result == NXErrorNone)
    {
        // Then start the player.
        [self.playerView.player start];
    }
    else
    {
        // If there is error, call encounteredError method with result.
        [self nexPlayer:nxplayer encounteredError:result];
    }
}


// This method will be called asynchronously by stopButton when the [self.playerView.player stop] completes.
- (void)nexPlayer:(NXPlayer *)nxplayer completedAsyncCmdStopWithResult:(NXError)result
{
    if(result == NXErrorNone)
    {
        // widevine should stop before closing the player.
        [self.widevine stop];

        // In this QuickStart, when the user stops the player, it will be closed.
        [self.playerView.player close];
    }
    else
    {
        [self nexPlayer:nxplayer encounteredError:result];
    }
}


// You are able to handle the error in this method.
// If you would like to process specific error, you can find more information by errorCode.
- (void)nexPlayer:(NXPlayer *)nxplayer encounteredError:(NXError)errorCode
{
    NSLog(@"encounteredError");
}


- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
}

@end

```
---

## Step #10. Run Xcode and check your first NexPlayer with Widevine on actual device.
Connect your actual device and select your device on the tool bar and click the Run button. Now let's enjoy watching the content on your device.
