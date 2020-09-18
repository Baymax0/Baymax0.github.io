---
layout:     post
title:      "关于iOS端 推送语音播报的实现"
subtitle:   "坑：动态语音、在Extension调取自定义类，Extension中数据共享等等"
date:       2018-03-04 14:29:00
author:     "Leezw"
header-img: "img/post-bg-2015.jpg"
tags:
- iOS
---


公司之前做的收款相关的项目，需要在app内收款，app后台，app被kill掉后都能通过接受推送后发起一个语音播报的功能，进过几个版本迭代后最终实现效果如下：

App在手机在前台：
* 所有系统版本都播报："\*\*收款\*\*元".

App在后台或者被杀死:
* iOS 10及以上：
	* 依然播报："\*\*收款\*\*元".
* iOS 10以下：
	* 播报："\*\*收款成功".(本地固定音频)

之所以会有以上的区别在于，iOS10推出了 [NotificationServiceExtension](https://developer.apple.com/documentation/usernotifications/unnotificationserviceextension?language=objc)

## App在手机在前台

当app在前台时，我们可以在AppDelegate的推送回调方法中执行语音播放的方法。以下是一个简单的语音播报方法：

	//此处使用系统的语音库，需导入#import <AVFoundation/AVFoundation.h>
	+(void)playVoice:(NSString*)text{
	    AVSpeechUtterance *utterance = [AVSpeechUtterance speechUtteranceWithString:text];
	    AVSpeechSynthesizer *synth = [[AVSpeechSynthesizer alloc] init];
	    utterance.volume = 1.0;
	    utterance.pitchMultiplier = 1;
	    [synth speakUtterance:utterance];
	}

## App在后台或被kill了（iOS 10以下）：

由于iOS系统的封闭性，当app在被杀掉后，普通应用在收到推送后无法唤醒应用来对推送的数据进行二次处理，更不能执行语音播报的代码块了，推送只能由后台在在推送的参数中，配置上推送后在通知栏显示的标题、子标题、推送铃声。
苹果也在发现这个问题影响用户体验后推出了很多让系统更开放的Extension，不过由于适配问题，当遇到iOS 10 之前的版本，我们需要做的是：

1. 后台推送时根据当前系统版本，如果版本低于iOS 10，在推送格式中的sound:参数置顶本地的音频文件名称。  
版本号可在登录时传给后台  
音频文件只需要加载到项目内即可  
音频文件下载[百度语音平台-文本转音频](http://developer.baidu.com/vcast)


## App在后台或被kill了（iOS 10以上）：

iOS 10以后的版本我们就可以使用NotificationServiceExtension了，简单的实现步骤是  
1.File -> New -> Target - NotificationServiceExtension  
2.在NotificationService.m文件中处理推送数据，如下：

	- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    	self.contentHandler = contentHandler;
    	self.bestAttemptContent = [request.content mutableCopy];

    	//拿到后台的推送配置的参数
    	NSDictionary *dic = request.content.userInfo;

    	NSString *speakContent = @"收款123元";
    	//播放语音
    	[self playVoice:speakContent];
    	NSString *showContent = @"推送子标题";

    	self.bestAttemptContent.body = showContent;
    	self.bestAttemptContent.sound = nil;
    	self.contentHandler(self.bestAttemptContent);
	}

3.后台在推送的参数中需要配置：

	aps={
		...
		"content-available" = 1;
	}
	


## 关于Extension中的一些问题

##### **Extension 中使用项目中的类或资源**   
NotificationServiceExtension.m文件中，我们可以 #import "myUtils.h"  但是编译器后会报错(因为找不到.m文件)。  
由于项目与Extension是不同的target，当前target创建的文件或资源不能再其他target文件中直接使用，而.h文件是整个项目通用的。  
所以我们只需要选中需要的资源文件 或 类的.m文件 -> 在xcode右侧的属性信息 -> TargetMembership  在扩展前吧勾打上即可，

![](/img/articles/notification/img1.png)

如果当前类有引用其他类，则相关的类也需要进行如上操作，如果是写在pch文件里的系统库，则需要在当前类中手动显式的导入一次(最好不要选择共享复杂的类文件)


#####  **关于数据共享**   
在NotificationServiceExtension中我们不能直接使用原项目中的进行缓存的方法。  
即使像方法1那样将缓存的文件也共享到当前target中，我们读取后也会发现，并不是同一个缓存。 

因Extension与原项目执行时不再同一个沙盒内，这导致NSUserDefaults也无法使用。 
（iOS的沙盒机制使应用不能访问别的沙盒路径，NSUserDefaults实际就是在项目沙盒路径下的一个plist文件）,我们可以使用AppGroup功能。

	//在原项目和Extension中只需要使用同一个SuiteName，就可以拿到同一个缓存对象
	NSUserDefaults *userDefault = [[NSUserDefaults alloc]initWithSuiteName:@"groupKey"];

Appgroup功能还需要在开发账号上进行配置，并不是直接使用的[关于AppGroup的使用](http://blog.csdn.net/shengpeng3344/article/details/52190997)

#####  **审核权限问题**
如果仅仅是实现语音播报的功能，在项目target -> Capabilities -> Background Modes中，不要勾选“Audio,AirPlay,and Picture in Picture”,不需要,否则会导致审核不通过。

---


