---
layout: post
title: 'AVFoundation 音频录制'
subtitle: '音频录制AVAudioRecorder'
date: 2017-04-20
categories: 技术
cover: ''
tags: 音视频
---

上一篇文章我们探讨了`AVFoundation`中最为常用的本地音频播放工具`AVAudioPlayer`,并了解了它的创建, 控制, 监听(代理)方法,今天咱们来研究下它的"亲兄弟"`AVAudioRecorder`.

---
### AVAudioRecorder
`AVAudioRecorder`和`AVAudioPlayer`一样,构建于`Audio Queue Services`之上, 同样支持音频的计量,也是简单易用的Obj-C接口, 并且支持内置和外置麦克风,所以`AVAudioRecorder`是用于本地音频录制的首选方式.

---
### 创建AVAudioRecorder

创建AVAudioRecorder实例需要提供一些信息:
-  用于写入音频文件的URL.
- 用于配置录音会话信息的`NSDictionary`对象.
- 用于返回错误的`NSError`.

代码示例如下
<pre><code class="language-objectivec">
// 保存录音的路径
NSString *directory = NSTemporaryDirectory();
NSString *filePath = [directory stringByAppendingPathComponent:@"tempAudio.m4a"];
NSURL *url = [NSURL fileURLWithPath:filePath];

// 配置音频的参数 (下面有详细描述)
NSDictionary *setting = @{
AVFormatIDKey: @(kAudioFormatMPEG4AAC),
AVSampleRateKey: @(44100),
AVNumberOfChannelsKey:@(1)
};
// 捕捉错误的NSError
NSError *error;
// 创建Recorder实例
AVAudioRecorder *recorder = [[AVAudioRecorder alloc] initWithURL:url settings:setting error:&error];
// 错误判断
if(error) {
    NSLog(@"%@", error); // 在这里我们简单打印错误, 在实际项目中我们要做容错判断
}
</code></pre>

配置音频的`NSDictionary`对象的键值对还是值得探讨一下的,开发者可用的key定义在<AVFoundation/AVAudioSetting.h>中,常用的如下

- `AVFormatIDKey ` 
`AVFormatIDKey`指定了录音会话生成的音频文件的格式.
下面是常用的value (全部的value定义在`CoreAudio/CoreAudioTypes.h`)

    <pre>
    kAudioFormatLinearPCM               = 'lpcm',
    kAudioFormatMPEG4AAC                = 'aac ',
    kAudioFormatAppleIMA4               = 'ima4'
    </pre>
  `kAudioFormatLinearPCM `将会将未压缩的音频直接写入问,极大  限度的保证了质量,但是文件的体积也是最大的.
  `kAudioFormatMPEG4AAC `和`kAudioFormatAppleIMA4`的压缩格式将大大减少文文件大小,还能保证较高的音频质量.
  另外值得一提的是,我们URL参数定义的文件格式类型必须和`AVFormatIDKey `指定的文件类型格式类型相同.

- `AVSampleRateKey `
   `AVSampleRateKey `定义了录音会话的采样率,单位是Hz,越高的采样率,也就意味着更高的音频质量,但是也会增大体积, 虽然没有明确定义,但仍推荐使用标准的采样率.如下:
   
  <pre>
  8000Hz 电话所用采样率，对于人的说话已经足够
  22050Hz 无线电广播所用采样率，广播音质
  44100Hz 音频CD，也常用于MPEG-1音频（VCD，SVCD，MP3）所用采样率
  47250Hz NipponColumbia(Denon)开发的世界上第一个商用PCM录音机所用采样率
  48000Hz miniDV、数字电视、DVD、DAT、电影和专业音频所用的数字声音所用采样率
  </pre>
- `AVNumberOfChannelsKey `
  `AVNumberOfChannelsKey `顾名思义, 指的是录音回话的声道数,指定为1意味着单声道的录制,定义为2则代表立体声录制,除非我们使用外界设备录音,否则应该创建单声道录音.

一个很重要的一点, 因为我们要使用到用户手机的麦克风,所以我们应该添加`NSMicrophoneUsageDescription`到应用info.plist文件,不然会crash.
如图:

![info.plist示例](http://upload-images.jianshu.io/upload_images/4491615-45ba76f7b42ec86c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
### 录音过程的控制
常用的方法如下:
- 开始录制 直接掉用`- (BOOL)record;`方法即可开始录音,和`AVAudioPlayer`一样,苹果依然推荐在录制操作之前调用`- (BOOL)prepareToRecord`方法,作用和`- (BOOL)prepareToPlay;`类似,为录音做准备操作,减小开始录音的延迟.
- 暂停 调用`- (void)pause;`方法即可暂停录制.在暂停状态下可再调用`- (BOOL)record;`即可继续录制.
- 停止 调用`- (void)stop;`即可停止录制.
---
### AVAudioRecorderDelegate
<pre><code class="language-objectivec">
// 录音停止时调用,flag如果为YES则录音成功结束, flag若为NO则录音编码失败. 一般在这里做文件保存操作.
- (void)audioRecorderDidFinishRecording:(AVAudioRecorder *)recorder successfully:(BOOL)flag;

// 如果编码过程中失败, 此方法触发
- (void)audioRecorderEncodeErrorDidOccur:(AVAudioRecorder *)recorder error:(NSError * __nullable)error;

// 录音过程中出现中断会触发(比如打进电话)
- (void)audioRecorderBeginInterruption:(AVAudioRecorder *)recorder;

// 以下三个方法中断结束后调用触发
- (void)audioRecorderEndInterruption:(AVAudioRecorder *)recorder withOptions:(NSUInteger)flags;
- (void)audioRecorderEndInterruption:(AVAudioRecorder *)recorder withFlags:(NSUInteger)flags;
- (void)audioRecorderEndInterruption:(AVAudioRecorder *)recorder;
</code></pre>

---
### 写在最后
大周末依然在加班,忙中偷闲更新一篇.上篇文章和本篇围绕的AVFoundation中最基础也是最常用的两个音频专用类,我们可以实现音频的播放和录制.
下篇文章将围绕AVAudioRecorder和AVAudioPlayer音频计量功能展开研究.

