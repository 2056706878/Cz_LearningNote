# agora接口使用

引入sdk
```js
import AgoraRTC from 'agora-rtc-sdk-ng';
```


## 创建音视频轨道

+  **单独创建音视频轨道**

采集摄像头
```js
const cameraTrack = await AgoraRTC.createCameraVideoTrack()
```
采集麦克风
```js
const microphoneTrack = await AgoraRTC.createMicrophoneAudioTrack()
```

+ **同时创建音视频轨道**
```js
const [microphoneTrack, cameraTrack] = await AgoraRTC.createMicrophoneAndCameraTracks();
```

+ **创建音视频轨道时可传入参数控制采集行为**
采集麦克风时开启回声消除，噪声抑制和自动增益
```js
let audioOptions = {
	// 开启回声消除
	AEC: true,
	// 开启噪声抑制
	ANS: true,
	// 开启自动增益
	AGC: true,
}
const microphoneTrack = await AgoraRTC.createMicrophoneAudioTrack(audioOptions)
```


## 启用或禁用音视频轨道

```js
setEnabled(bool:boolean)
```

setEnabled在轨道已发布时，禁用音视频轨道后，远端会触发user-unpublished(用户取消发布)事件。重新启用后，远端会触发 user-published(用户发布)事件

```js
setMuted(bool:boolean)
```

setEnabled在轨道已发布时，禁用音视频轨道后，远端会触发user-unpublished(用户取消发布)事件。重新启用后，远端会触发 user-published(用户发布)事件

+ `setEnabled` 和 `setMuted` 不能同时调用
+ 禁用轨道后再启用，音视频恢复时间较慢。
+ Mute 轨道后再 Unmute，恢复时间较快。


## 视频会议开始流程

1.  **创建客户端对象**
```js
let client = AgoraRTC.createClient({mode: "rtc", codec: "vp8"});
// codec(视频编码格式): vp8 \ h264
// mode(频道场景): rtc(通信场景) \ live(直播场景)
```
2.  **加入会议频道**
```js
await client.join(appId,channel,token,uid);
/**
 * appid 在Agora项目管理页面获取到项目的App Id
 * channel  生成临时token的频道名
 * token 在Agora项目管理页面点击生成临时音视频token,输入频道名后获取的临时token
 * uid 用户的id,不传时Agora会自动生成一个Number型的id,频道内uid必须唯一
 */
```
3.  **创建本地音视频轨道**
```js
// 采集麦克风
const localAudioTrack = await AgoraRTC.createMicrophoneAudioTrack();
// 采集摄像头
const localVideoTrack = await AgoraRTC.createCameraVideoTrack();
```
4.  **发布订阅本地音视频**
  +  发布音视频
```js
await client.publish([localAudioTrack, localVideoTrack]);
```

  +  取消发布音视频
```js
await client.unpublish([localAudioTrack, localVideoTrack]);
```

  +  订阅音视频
```js
client.on("user-published", async (user, mediaType) => {
	// 发起订阅 
	await client.subscribe(user, mediaType); 
});
```

  +  取消订阅音视频
```js
// 取消订阅音频
await client.subscribe(user, "audio");
// 取消订阅视频
await client.subscribe(user, "video");
```
5.  **离开会议频道**
```js
await client.leave();
```


## 消息接收和发送

+ **获取用户项目的AppId和App证书[Edit Project (agora.io)](https://console.agora.io/project/UtJI-NvWI)**
+ **选择要加入的用户uid并获取用户临时token[Agora Access Token](https://webdemo.agora.io/token-builder/)**
+ **实现消息发送与接收**
1. 初始化客户端
```js
const client = AgoraRTM.createInstance(appID)
```
2. 初始化频道
```js
let channel = client.createChannel("channelName")
```
3. 登录
```js
let options = {token:'',uid: ''}
await client.login(options)
```
4. 加入频道
```js
await channel.join()
```
5. 登出
```js
await client.logout()
```
6. 离开频道
```js
await channel.leave()
```
7. 监听频道消息发送
```js
channel.on('ChannelMessage', function (message, memberId) {
	// 频道监听事件
})
```
8. 发送频道消息
```js
await channel.sendMessage({ text: '频道消息' })
```


# agora使用中的问题

## 音频回声问题

**使用AgoraRTC.createMicrophoneAudioTrack()采集本地音频轨道后，采集到的音频对象开关状态与本地麦克风同步。如果麦克风是开启状态，音频对象再调用audioTrack.play()方法，此时就会出现音频回声问题。**
+ before
```js
useEffect(() => {
	if (props.audioTrack) {
		props.audioTrack?.play();
	}
	return () => {
		props.audioTrack?.stop();
	};
}, [props.audioTrack]);
```
**解决方案是只播放其他用户的音频，自身的音频对象不播放。**
+  after
```js
useEffect(() => {
	if (props.audioTrack) {
		if (props.uid !== props.clientUid) {
			//是其他用户才播放
			props.audioTrack?.play();
		} else {
			props.audioTrack?.stop();
		}
	}
	return () => {
		props.audioTrack?.stop();
	};
}, [props.audioTrack]);
```