## 开发问题总结文档

1. **useReducer 使用问题**

```typescript
// 定义赋值函数
const reducer = (state: any, action: any) => {
  switch (action.type) {
    case 'payload': {
      return { ...state, ...action.payload };
    }
    case 'updateListItem': {
      let arr = [...state['data1']];
      arr[action.index][action.label] = action.info;
      return {
        ...state,
        ['data1']: [...arr],
      };
    }
    case 'updateDetails': {
      return { ...state, ['details']: action.details };
    }
    default:
      return state;
  }
};
```

```typescript
// 设置初始值
const initialState = {
  data1: {},
  data2: [],
  details: {},
};
// 声明state变量
const [state, dispatch] = useReducer(reducer, initialState);

// 使用dispatch赋值
dispatch({ type: 'updateCarDetails', details: res });
```

2. **ReactNative 中 textInput 的点击事件和 ScrollView 的滑动事件冲突问题**
   当在 ScrollView 容器中存在着 textInput 元素，并且 textInput 设置了 textAlign={'right'|| 'center'}属性，此时 ScrollView 的滑动事件与 textInput 的点击事件将会冲突，无法进行滑动。

```tsx
<ScrollView>
  <TextInput style={{ textAlign: 'right' }} />
  <View style={{ height: 2000 }} />
</ScrollView>
```

- 解决方案
  设置多行属性 multiline={true}可以解决

```tsx
<ScrollView>
  <TextInput style={{ textAlign: 'right',multiline={true} }} />
  <View style={{ height: 2000 }} />
</ScrollView>
```

3. **因 expo 更新导致的 android 版本不一致的问题**
   expo 官方库版本更新，导致其他依赖库报错 android 版本不一致

- 错误信息

```gradle
The minCompileSdk (31) specified in a
dependency’s AAR metadata (META-INF/com/android/build/gradle/aar-metadata.properties)
is greater than this module’s compileSdkVersion (android-30)
```

- 解决方案
在 android/build.gradle 文件中,找到 allprojects {},在上方同级目录添加代码:

```gradle
  def REACT_NATIVE_VERSION = new File(['node', '--print', "JSON.parse(require('fs').readFileSync(require.resolve('react-native/package.json'), 'utf-8')).version"].execute(null, rootDir).text.trim())
```

在 allprojects {}中添加代码

```gradle
allprojects {
  ...

  configurations.all {
    resolutionStrategy {
      force "com.facebook.react:react-native:" + REACT_NATIVE_VERSION
    }
  }

  ...
}
```
