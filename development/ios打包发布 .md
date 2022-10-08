# IOS 应用打包发布

## 打包

1. 通过 XCode 打开文件
2. 打包之前可以先运行项目，看一下代码是否有问题
3. 点击 _general_，设置属性

- Display Name 属性，设置 app 名称
- Bundle Identifier 属性，设置 app 包名(**与项目文件目录中 app.json 文件里的 ios->bundleIdentifier 字段的值一致**)
- Version 属性，设置版本
- Build 属性，当前版本打包次数(**每次打包时，此字段的值必须要更新**)

4. 点击 Signing & Capabilities

- Signing -> Team 测试人员账号

5. 点击选择模拟器，选择 _Any ios Device(arm64)_
6. 点击桌面左上方 _Product_，选择 _Archive_ 开始打包
7. 等待模拟器右边文件编译完成
8. 在新出来的 Archives 页面，可以查看历史编译版本(**有时会出现版本没有变化，可以重新打开 XCode 再试一次**)
9. 点击 _Distribute App_ 开始发布版本，选择打包版本，然后一直点击 next，直到打包完成。

- App Store Connect (上传 app Store)
- Ad Hoc ()
- Enterprise ()
- Development ()

<!-- 10. 或者点击 validate App 开始打包(一般用于测试) -->

## 发布

1. 打开 ios 管理平台[https://developer.apple.com/account/resources/identifiers/list](管理平台)
2. 点击 _Identifiers_,找到对应的项目包名,点击列表进入详情页
3. 如果列表中没有，需要新建项目：点击 Identifiers 右侧加号,选择 App IDs，点击 _Continue_,选择 App，再点击 _Continue_，输入 Bundle,即项目包名，再点击 _Continue_。
4. 打开 ios App Store [https://appstoreconnect.apple.com/login](AppStore)
5. 点击 App 右侧加号,创建新的 App

- 平台 选择为 _ios_
- 名称 为项目名称
- 语言 为简体中文
- 套装 ID 为第三步创建的项目 ID
- SKU 为项目文件 app.json 中 _slug_ 字段的值
- 用户访问权限 可选择 _有限访问权限_ (**此处可选择添加开发者**)或 _完全访问权限_ (**一般选择完全访问权限**)
  全部选择好后，点击创建

6. 此时 App 页面可以看见新添加的 App，点击新的 App 图标，修改详细信息

- ios 预览和截屏 选择相应尺寸(6.5 英寸和 5.5 英寸)，并拖动对应图片上传
- 描述 输入一段描述 app 功能的文本
- 关键词 用于应用商店搜索时的相关词汇
- 技术支持网址 (可使用默认网址)
- 营销网址 (可使用默认网址)
- App 审核信息 需要提供联系人以及可用与测试的管理员账号
- 版本发布 选择 _手动发布此版本_
- ios 构建版本 在打包并选择上传 app Store 完成之后，可点击上方 TestFlight 查看版本，可启用内部测试，创建测试群组并添加测试员，若提示缺少合规证明，点击管理，选择 _否_ 点击 _开始内部测试_。此时可在 App Store 中选择相应的构建版本
  按流程输入其他信息，版权信息、价格等级、年龄分级...
  ，输入完相应信息后，点击存储，再点击 _添加以供审核_ ,确认后点击 _提交至 App 审核_(**每输入完一个信息后都可以点击存储**)
