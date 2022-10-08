# 关于在开发视频会议时，所遇的问题解决以及总结

### 作者：张钦育

## 所遇问题概述，以及重要点

> <font face="楷体" color="black" size="4">视频通话时,大小流的切换、视频通说过程中,分享屏幕、和其他人进行通讯</font>

## 通用方法

> <font face="楷体" color="black" size="4">自定义方法: useLocalStorage (存储到本地 local)、useDebounce (防抖) umi 方法：useModel (存储到全局的变量)</font>

> <font face="楷体" color="black" size="4">callback 回调函数</font>

- ## 大小流切换
  ```javascript
  HQStreamUserList.forEach(
    (user: number | string) =>
      void handlePromiseList.push(async () => {
        console.log('大流++++++++', user)
        const result = await this.client?.setRemoteVideoStreamType(user, 0)
        return result
      })
  )
  LQStreamUserList.forEach(
    (user: number | string) =>
      void handlePromiseList.push(async () => {
        console.log('小流++++++++', user)
        const result = await this.client?.setRemoteVideoStreamType(user, 1)
        return result
      })
  )
  ```
  通过 api setRemoteVideoStreamType()实现大小流切换
  ```javascript
  client?.setRemoteVideoStreamType(user, 1)
  ```
  第一个参数为要进行大小流缺环的用户，第二个参数为，传递的参数为 number，为 0 时:大流;为 1 时:小流
- ## 分享屏幕

  ```javascript react
  // 开启视频共享
  Shared = async (callback?: Function) => {
    this.screenTrack = await AgoraRTC.createScreenVideoTrack(
      {
        // 配置屏幕共享编码参数，详情请查看 API 文档。
        encoderConfig: '1080p_1',
        // 设置视频传输优化策略为清晰优先或流畅优先。
        optimizationMode: 'detail',
      },
      'auto',
    ).then((localVideoTrack: any) => {
      console.log('@@@@开始共享', localVideoTrack);
      this.localVideoTrack = localVideoTrack;
      this.userList[this.client?.uid as string | number] = {
        ...this.userList[this.client?.uid as string | number],
        videoTrack: localVideoTrack,
      };
      this.centerPhoto = { ...this.centerPhoto, videoTrack: localVideoTrack };
      callback && callback(this.centerPhoto, this.userList);
      return localVideoTrack;
    });
    await this.client?.unpublish();
    await this.client?.publish(this.screenTrack);
    if (this.screenTrack instanceof Array) {
      this.localTracks.screenVideoTrack = this.screenTrack[0];
      this.localTracks.screenAudioTrack = this.screenTrack[1];
    } else {
      this.localTracks.screenVideoTrack = this.screenTrack;
    }

    this.localTracks?.screenVideoTrack?.on('track-ended', async () => {
      await this.client?.unpublish();
      // this.sharedbol = !this.sharedbol;
      this.localTracks.screenVideoTrack && this.localTracks.screenVideoTrack.close();
      this.localTracks.screenAudioTrack && this.localTracks.screenAudioTrack.close();
      this.localTracks.audioTrack && this.localTracks.audioTrack.close();
      await this.createLocalTracks();
      await this.client
        ?.publish([
          this.localAudioTrack as ILocalAudioTrack,
          this.localVideoTrack as ILocalVideoTrack,
        ])
        .then(() => {
          console.log('@@@@停止共享', this.localVideoTrack);
          this.userList[this.client?.uid as string | number] = {
            ...this.userList[this.client?.uid as string | number],
            videoTrack: this.localVideoTrack,
          };
          this.centerPhoto = { ...this.centerPhoto, videoTrack: this.localVideoTrack };
          console.log('@@@@停止共享2222', this.centerPhoto);
          callback && callback(this.centerPhoto, this.userList);
        })
        .catch((fail) => {});
    });
  };

  // 关闭视频共享
  closeShared = async (callback?: Function) => {
    await this.client?.unpublish();
    await this.createLocalTracks();
    await this.client
      ?.publish([
        this.localAudioTrack as ILocalAudioTrack,
        this.localVideoTrack as ILocalVideoTrack,
      ])
      .then(() => {
        this.userList[this.client?.uid as string | number] = {
          ...this.userList[this.client?.uid as string | number],
          videoTrack: this.localVideoTrack,
        };
        this.centerPhoto = { ...this.centerPhoto, videoTrack: this.localVideoTrack };
      })
      .catch((fail) => {});
    callback && callback(this.centerPhoto);
  };

  ```

- ## 和其他人通信

  ```javascript
  //  初始化客户端
  const client = AgoraRTM.createInstance(this._count)
  // 初始化频道
  let channel = client.createChannel('demoChannel')

  const login = (callBack: any) => {
    client.login(option).then((res: any) => {
      message.success('登录成功111')
      // setLoginState({LOGIN: true});
      this.loginStates = true
      joinL()
      callBack(this.loginStates)
    })
  }
  function logout() {
    client.logout().then(() => {
      message.success('退出111')
    })
  }
  function joinL() {
    channel.join().then(() => {
      message.success('加入频道111')
    })
  }
  function leaveL() {
    if (channel != null) {
      channel.leave().then(() => {
        message.success('离开频道111')
      })
    } else {
      console.log('Channel is empty')
    }
  }
  /**发送频道消息**/
  function sendChannelMessage(text: string) {
    if (channel != null) {
      channel.sendMessage({ text: text }).then(() => {
        // message.success('消息发送成功111');
      })
    }
  }
  ```

  使用 client.login() 登录 channel.join() 加入到频道里 使用 channel.sendMessage()进行消息发送

  ```javascript
  /**收到来自对端的点对点消息**/
  client.on('MessageFromPeer', function (message, peerId) {
    console.log('11111-Message from: ' + peerId + ' Message: ' + message)
  })
  /**显示连接状态变化**/
  client.on('ConnectionStateChanged', function (state, reason) {
    console.log('11111-State changed To: ' + state + ' Reason: ' + reason)
  })
  /**显示发送过来到信息**/
  channel.on('ChannelMessage', (message: any, memberId: any) => {
    debounce(() => {
      console.log('11111-ChannelMessage')
      console.log(memberId + ': ' + message?.text || '****')
    })
  })
  /**显示频道**/
  channel.on('MemberJoined', function (memberId) {
    console.log('11111-' + memberId + ' joined the channel')
  })
  /**频道成员**/
  channel.on('MemberLeft', function (memberId) {
    console.log('11111-' + memberId + ' left the channel')
  })
  ```

- ## useLocalStorage（） 存储 local

  ```javascript
  const UseLocalStorage = (key = '', initialValue: any) => {
    const [state, setState] = useState(() => {
      try {
        const item = localStorage.getItem(key)
        return item ? JSON.parse(item) : initialValue
      } catch (error) {
        return initialValue
      }
    })
    const useLocalStorageState = (newState: any) => {
      try {
        const newStateValue =
          typeof newState === 'function' ? newState(state) : newState
        setState(newStateValue)
        window.localStorage.setItem(key, JSON.stringify(newStateValue))
      } catch (error) {
        console.error(`Unable to store new value for ${key} in localStorage.`)
      }
    }
    return [state, useLocalStorageState]
  }
  export default UseLocalStorage
  ```

  使用

  ```javascript
  // 存储登录状态
  const [loginState, setLoginState] = UseLocalStorage('LoginState', loginStates)
  ```

  第一个参数返回，当前的名称为'LoginState'的 local 的值，使用第二个参数发送新的内容到名为'LoginState'的 local 中

- ## useDebounce 防抖器
  ```javascript
  export const debounce = (
    fn: any,
    delay: any = 1000,
    immediate: boolean = false
  ) => {
    let time: any = null
    let isImmediateInvoke: boolean = false
    console.log('-----------函数防抖----------')
    function _debounce(args: any) {
      if (time !== null) {
        clearTimeout(time)
      }
      if (!isImmediateInvoke && immediate) {
        console.log('函数先执行,再计时')
        fn(args)
        isImmediateInvoke = true
      }
      time = setTimeout(() => {
        console.log('函数执行')
        fn(args)
        isImmediateInvoke = false
      }, delay)
    }
    return _debounce
  }
  ```
  使用
  ```javascript
  debounce(() => {
    console.log('11111-ChannelMessage')
    console.log(memberId + ': ' + message?.text || '****')
  })
  ```
  将需要进行防抖的方法放入
- ## useModel 存入全局的变量

  ```javascript
  import { useModel } from 'umi'
  const { Sharedbol, setSharedbol } = useModel('modal')
  ```

  需要在 umi/plugin-model/Provider.tsx 文件中引入和注册

  ```javascript
  import model0 from 'D:/ProjectWarehouse/video/src/models/InformationState'
  import model1 from 'D:/ProjectWarehouse/video/src/models/meetingTime'
  import model2 from 'D:/ProjectWarehouse/video/src/models/modal'
  export const models = {
    '@@initialState': initialState,
    InformationState: model0,
    meetingTime: model1,
    modal: model2,
  }
  ```

  定义全局方法

  ```javascript
  import React, { useState } from 'react'
  export default () => {
    const [Sharedbol, setSharedbol] = useState < boolean > false
    return { Sharedbol, setSharedbol }
  }
  ```

  Sharedbol：获取存在全局的变量的内容，setSharedbol：存储到全局的变量

- ## callback 回调函数

  回调是作为参数传递给另一个函数，并在父函数完成后执行的函数。

  ```javascript
  var clientData = {
    id: 096545,
    fullName: 'Not Set',
    //setUsrName是一个在clientData对象中的方法
    setUserName: function (firstName, lastName) {
      clientData?.fullName = firstName + ' ' + lastName
    },
  }

  function getUserInput(firstName, lastName, callback) {
    //code .....

    //调用回调函数存储
    callback(firstName, lastName)
  }

  getUserInput('Barack', 'Obama', clientData.setUserName)

  console.log(clientData.fullName) //Not Set

  console.log(window.fullName) //Barack Obama
  ```

- ## async 和 await
  async 和 await 是用来处理异步的。即你需要异步像同步一样执行，需要异步返回结果之后，再往下依据结果继续执行。
  async 是“异步”的简写，而 await 可以认为是 async wait 的简写。
  async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。
  ```javascript
  async function testResult() {
    console.log('内部调用前') // 2
    let result = await doubleAfter2seconds(30)
    console.log(result) // 4
    console.log('内部调用后') // 5
  }
  ```
- ## promise 对象

  Promise 简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果

  从语法上说， Promise 是一个对象，从中可以获取异步操作的消息

  Promise 对象共有 3 种状态：

  Pending ：进行中
  Resolved ：又称 Fulfilled ，已完成
  Rejected ：已失败
  状态的改变只有两种可能：从 Pending 变为 Resolved 、从 Pending 变为 Rejected

  Promise 对象主要有如下几个特点：

  - 对象的状态不受外界影响：只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
  - 一旦状态改变，就不会再变，任何时候都可以得到这个结果
  - 无法取消 Promise ，一旦新建它就会立即执行，无法中途取消
  - 如果不设置回调函数， Promise 内部抛出的错误，不会反应到外部
  - 当处于 Pending 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）

  ```javascript
  // 创建Promise示例
  var promise = new Promise((resolve, reject) => {
  // ...
  if (/* 异步操作成功 */) {
    resolve(value)
  } else {
    reject(error)
  }
  })

  // 指定Resolved状态和Reject状态的回调函数
  promise.then(value => {
  // success
  }, error => {
  // failure
  })
  ```

- ## class 类

  class 只是一个语法糖 是构造函数的另一种写法
  (语法糖 是一种为避免编码出错 和提高效率编码而生的语法层面的优雅解决方案 简单说 一种便携写法 而已)

  ```javascript
  /定义类
  class Point {
    constructor(x, y) {
      this.x = x;
      this.y = y;
    }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
  }
  ```
