---
layout: post
title: 'AVFoundation 音频播放'
subtitle: '音频播放 AVAudioPlayer'
date: 2017-04-18
categories: 技术
cover: ''
tags: 音视频
---


音频播放是很多应用中的常见需求, 音频播放用的最多的就是`AVFoundation`为我们提供的`AVAudioPlayer`,  `AVAudioPlayer`提供了非常方便且简单的方法来实现音频播放. 可以是内存或本地中的音频文件,还有我们常用的音频循环, 甚至还提供音频计量,并且是非常友好的Obj-C接口. 
AVAudioPlayer支持的音频格式为：AAC、MP3、ALAC等.
当然AVAudioPlayer也有缺点,那就是只能播放本地的音频.

### 创建AVAudioPlayer

AVAudioPlayer创建有两种方式:

- 是一种是基于内存的`NSData`

<pre><code class="language-objectivec">
NSError *error;
NSData *audioData = // 获取音频data
// 创建AVAudioPlayer实例
AVAudioPlayer *player = [[AVAudioPlayer alloc] initWithData:audioData error:&error];
// 错误判断
if (error) {
    NSLog(@"%@", error);// 在这里我们做了简单的打印,在实际的项目中我们应该做出相应处理
}
</code></pre>

- 另一种是本地文件的`NSURL`

<pre><code class="language-objectivec">
NSError *error;
// 获取音频文件URL
NSURL *url = [[NSBundle mainBundle] URLForResource:@"music" withExtension:@"mp3"];
// 创建AVAudioPlayer实例
AVAudioPlayer *player = [[AVAudioPlayer alloc] initWithContentsOfURL:url error:&error];
// 错误判断
if (error) {
    NSLog(@"%@", error);// 在这里我们做了简单的打印,在实际的项目中我们应该做出相应处理
}
</code></pre>

### AVAudioPlayer播放的控制

- 播放 : 实例直接调用 `play` 方法就可以实现开始播放, 苹果推荐在调用`play`之前先调用`prepareToPlay`方法, 此方法会在播放之前预处理和预加载音频文件, 以减少播放延迟, 如果不调用此方法也可以正常播放,`prepareToPlay`方法也会隐式调用,但是会有些许延迟.
- 停止: 实例调用`pause` 和 `stop`都会停止当前播放, 再调用`play` 也都会继续播放音频, 但是`stop`方法会撤销`prepareToPlay` 方法中做的一些准备工作.
- 进度控制: `currentTime`属性,改属性控制着播放进度, 如果音频正在播放,音频将偏移到指定的进度, 如果音频没在播放状态, `currentTime`决定着开始播放的进度.
- 循环次数: `numberOfLoops`属性决定着音频的重复播放次数,默认值是0, 意味着只会播放一遍, 如果值我们设置为1, 那么会播放2遍, 以此类推,如果我们设置为一个负数, 那么将一直重复播放,直到我们手动停止.
- 音量控制:`volume`属性, 赋值范围是从0.0-1.0, float类型数据.
- 速度控制:`rate`属性是控制音频播放速率, 赋值范围0.5-2.0之间. 1.0为正常, 0.5为半速播放, 2.0为2倍速播放. 在使用`rate`属性之前,应先设置`enableRate`属性为**YES** 激活`rate`,并且必须在`prepareToPlay`之前调用.




### AVAudioPlayerDelegate
<pre><code class="language-objectivec">
// 将在播放完成后触发,如果顺利播放完成flag为YES, 如果返回NO意味着解码失败.
- (void)audioPlayerDidFinishPlaying:(AVAudioPlayer *)player successfully:(BOOL)flag;

// 解码失败会触发此方法
- (void)audioPlayerDecodeErrorDidOccur:(AVAudioPlayer *)player error:(NSError * __nullable)error;

// 发生中断时会触发此方法(比如打进了电话)
- (void)audioPlayerBeginInterruption:(AVAudioPlayer *)player;

// 下面三个为中断结束时触发此方法,苹果推荐最后一个方法
- (void)audioPlayerEndInterruption:(AVAudioPlayer *)player withOptions:(NSUInteger)flags;
- (void)audioPlayerEndInterruption:(AVAudioPlayer *)player withFlags:(NSUInteger)flags;
- (void)audioPlayerEndInterruption:(AVAudioPlayer *)player ;
</code></pre>

### 写在最后

以后呢,我会定期更新有关AVFoundation的文章,如果有技术性错误,或文笔不妥之处,欢迎大家不吝赐教,或有什么开发中的问题都可以探讨学习.可以发邮件给我 :-)

