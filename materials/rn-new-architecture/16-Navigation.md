# Navigation: 页面跳转

1. 目前官方推荐的、主流的导航是 React Navigation，而不是 React Native Navigation。

## 导航基础

1. 实现一个基础的跳转导航，一共需要三步：

- 创建“导航地图”；
- 携带参数跳转页面；
- 页面接收和解析参数。

2. 原生堆栈导航 Stack，是用来创建页面和收集该导航下有哪些页面的。创建页面用的是 Stack.Screen 组件，收集页面用的是 Stack.Navigator 组件。

3. navigation 对象，这个 navigation 对象是从哪里来的呢？

将组件绑定到 Stack.Screen 组件的 component 参数上执行后，该函数组件 Discover 就自动获取到导航对象 navigation 了，该对象的 navigate 方法可以实现从一个页面到另一个页面的跳转。

4. route 对象和 navigation 对象类似，函数组件默认是没有这两个对象的。当你使用 Stack.Screen 创建页面时，用来创建页面的函数组件就会同时获取到 navigation 对象和 route 对象。其中 navigation 对象的主要作用是跳转，route 对象的主要作用是携带自定义参数。

5. 如果要更好地用好 React Navigation ，其中的关键是要理解它的两个配置项和导航路由对象。

## 自定义参数 params

1. 使用 React Navigation 创建出来的页面，有两类属性值比较常用，它们是：

- params：它是开发者自定义参数，通常用来渲染页面主体的数据，它是挂在 route 上的对象；
- options：它是导航相关的配置属性，包括手机顶部的状态栏、页面的标题栏、导航相关手势等等。

2. 当调用 navigation.navigate 方法时，就相当于告诉导航你“目的地”的名字 name 和要携带参数 params。这时导航框架会在它内部的“导航地图”中，也就是 {/_ ... _/} 中，寻找名字是 name 的页面，然后继续找到相关的组件 component，并配合导航配置项 options 和初始化自定义参数 initParams，一起把页面渲染出来。

在一些场景下，params 和 options 并不是固定的，当前页面也可以根据实际情况使用 setParams 和 setOptions 方法，对二者进行重新设置。

**在页面跳转的过程中，initialParams 对象和 params 对象会进行对象合并，而不是覆盖**

**initialParams 属性的作用是，给页面设置默认的且可覆盖的 params 自定义参数。**

3. 如何重置 params 呢？

重置 params 参数，用到的是 navigation.setParams 方法，示例代码如下：

```tsx
// 初始化 params：{symbol: '$', price: 99.9, image: 'dog.png' }
// 重置后 params：{symbol: '￥', price: 629.37, image: 'dog.png' }
function Detail({ route, navigation }) {
  const { price, symbol, image } = route.params;
  return (
    <>
      <Image source={image} />
      <Text>
        {symbol}
        {price}
      </Text>
      <Text
        onPress={() => {
          if (symbol === '￥') return;
          navigation.setParams({
            symbol: '￥',
            price: price * 6.3,
          });
        }}
      >
        切换成￥
      </Text>
    </>
  );
}
```

**使用 navigation.setParams 重置自定义 params 参数时，会将新旧 params 对象进行合并，并使用合并后的 params 重新渲染页面。**

## 导航配置

常用的配置项都列了出来，分成了 3 类。

### header 类

1. title：它是字符串，用于设置导航标题；
2. headerBackTitleVisible：它是布尔值，用于决定返回按钮是否显示回退页面的名字。默认是 true 显示，大多数应用是不显示，因此最好设置为 false（iOS 专属）；
3. headerShown：它是布尔值，用于决定是否隐藏导航头部标题栏；
4. header：它接收一个返回 React 元素的函数作为参数，返回的 React 元素就是新的导航标题栏。

### status 类

1. 控制屏幕顶部状态栏用的，也可使用 React Native 框架提供的 组件进行代替。

- statusBarHidden：它是布尔值，它决定了屏幕顶部状态栏是否隐藏。

### 手势动画类

1. gestureEnabled：它是布尔值，它决定了是否可用侧滑手势关闭当前页面（iOS 专属）；

2. fullScreenGestureEnabled：它是布尔值，它决定了是否使用全屏滑动手势关闭当前页面（iOS 专属）；
3. animation：它是字符串枚举值，它控制了打开或关闭 Stack 页面的动画形式，默认“default”是页面从右到左地推入动画，也可以设置成其他类型的动画，比如“slide_from_bottom”是页面从下到上的推入动画和从上到下的推出动画；

4. presentation：它是字符串枚举值，它控制了页面的展现形式，其主要作用是设置页面弹窗。常用的配置值是 “transparentModal” 它会将页面展示为一个透明弹窗。

### 如何使用？

1. 比如你想让详情页的图片展示更加有沉浸感，你就可以把详情页的 header 给隐藏了，并让它支持全屏返回。

2. options 参数既可以配置，也可以动态设置。比如，点击某个按钮动态更新 title 标题的文案。又比如，有时候在大团队中一个人只负责一个部分，负责该页面的同学不方便修改其他团队维护的全局配置，就可以在当前页面初始化时动态设置 options 参数。

3. 如何在当前页面修改 options 配置呢？

在当前页面重置 options 参数用的方法就是 setOptions

```tsx
function Detail({ navigation }) {
  // React.useEffect() 异步副作用回调，执行 setOptions 会导致闪屏，不推荐使用。

  // 页面初始化时，同步设置
  React.useLayoutEffect(() => {
    navigation.setOptions({
      headerShown: false,
      fullScreenGestureEnabled: true,
    });
  }, [navigation]);

  // 点击按钮后，异步设置
  const handlePress = () => {
    navigation.setOptions({
      title: '新标题',
    });
  };

  return <Text onPress={handlePress}>设置新标题</Text>;
}
```

3. **在初始化时，为了页面不抖动，我们必须使用同步的方法渲染页面。**

React 提供了同步执行的副作用函数 React.useLayoutEffect，把 navigation.setOptions 放在这里面执行，页面初始化的时候会同步地把头部隐藏起来，这样就不会出现页面抖动的现象了。

4. **异步设置 options 参数的场景，多用在有交互的场景**，比如点击某个按钮，改变标题的文案。如示例代码所示，在你点击“设置新标题”的按钮后，就会调用 navigation.setOptions 将标题文案重新设置。

## 各类导航

### Stack Navigator

1. Stack Navigator 和 Native Stack Navigator 都属于堆栈导航，也就是每跳转一次在堆栈的最上面增加一个新页面，每回退一次在堆栈的最上面减少一个老页面

2. 不同的是，Stack Navigator 底层使用的是 Gesture 手势库和 Reanimated 动画库实现的堆栈导航，而 Native Stack Navigator 使用的是 iOS 原生 UINavigationController 和 Android 原生 Fragment 实现的堆栈导航。

3. **一般情况下，我不推荐你使用 Stack Navigator，Native Stack Navigator 的功能更多，性能也更强大**

> [参考](https://reactnavigation.org/docs/stack-navigator/)

### Drawer Navigator:抽屉导航

1. 抽屉导航，也就是从侧边栏推出的导航页面。底层也是用的 Gesture 手势库和 Reanimated 动画库实现的，类似微信首页侧滑查看收起的小程序或公众号文章，这就属于抽屉导航。

### 底部标签导航

1. Bottom Tabs Navigator：底部标签导航。基本上每个 App 底部有好几个 Tab，这种多 Tab 的页面切换的效果在 React Native 中就可以用它来实现

### Material Bottom Tabs Navigator

1. 参考：https://reactnavigation.org/docs/material-bottom-tab-navigator/

### Material Top Tabs Navigator

1. 带 Material 样式的顶部标签导航，它是基于 react-native-tab-view 实现的，你可以把 Material 样式换成你自己的样式，常见的多列表 Tabs 就可以用它来实现

### 使用

1. 无论是 Stack.Screen 还是 Tab.Screen，这些“导航地图”中的 Screen 的 component 属性既接受普通页面函数作为参数，也可以接受“导航地图”函数作为参数。**“导航地图”的 Screen 接收“导航地图”作为 component 参数，就是实现导航嵌套的方法，唯一需要保证的是嵌套的最内层必须是普通页面。**

2. **使用堆栈“导航地图” Stack 作为根元素，使用底部标签“导航地图” Tab 作为子元素的嵌套方案是实现类似微信、淘宝这种多底部标签 App 的最佳实践。**

3. 用好 React Navigator 的关键是，**理解它的两个配置项和导航路由对象。**

4. RN 和 Native 相互跳，用的是 Native 导航。

在混合应用中，RN 只是 Native 的一个页面容器，比如在 Android 中为 Activity/Fragment，在 iOS 中为 ViewController。

跳转是用的 Native 通过 JSI/JS Bridge 暴露给 JS 的 Module/Component 跳转的。

## 示例

1. 官方 Modal 的示例、文档和代码

示例：https://reactnavigation.org/docs/modal/
文档：https://reactnavigation.org/docs/native-stack-navigator/#presentation
代码：https://github.com/react-navigation/react-navigation/blob/main/example/src/Screens/ModalStack.tsx
