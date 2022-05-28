# Image: 选择合适的图片加载方式

1. 有时候，我们可能会忽视对这些基础知识的琢磨。在日常开发中，图片是影响用户体验的关键因素之一，它很常见，基本上哪里都有它。而且相对于文字，图片也更容易抓住用户的眼球。图片组件很重要，但要用好却不那么容易。

2. RN 的 Image 组件支持 4 种加载图片和方法：

- 静态图片资源；
- 网络图片；
- 宿主应用图片；
- Base64 图片；

## 静态图片资源

1. 静态图片资源（Static Image Resources）是一种使用内置图片的方法。静态图片资源中的“静态”指的是每次访问时都不会变化的图片资源。

2. 如果图片每次都不会变化，那么你就可以把这张图片作为静态图片资源，内置在 App 中。

一张网络图片从下载到展示的耗时通常需要 100ms 以上，而一张内置图片从读取到展示的耗时通常只有几 ms，甚至更低，二者耗时相差了两个数量级。

**在一些高性能场景下，你应该选择把这些不经常变动的静态图片资源内置到 App 中。当用户打开 App 时，这些图片就能够立刻展示出来了。**

3. 怎么使用静态图片资源的呢？

首先，把图片放到 React Native 的代码仓库中，然后通过 require 的方式引入图片，最后把图片的引用值传给 source 属性。Image.source 属性是用来设置图片加载来源的。

tips: **require 函数的入参必须是字面常量，而不能是变量。**

4. 为什么使用 require 函数引入静态图片资源时，require 入参，也就是图片的相对路径，必须用字面常量表示，而不能用变量表示？

#### 静态图片资源的加载原理

从编译时到运行时，加载静态资源图片的全过程，一共分为三步：

1. **编译：**在编译过程中，图片资源本身是独立于代码文件之外的文件，图片资源本身是不能编译到代码中的，所以，我们需要把图片资源的路径、宽高、格式等信息记录到代码中，方便后面能从代码中读取到图片。

在你引入静态图片资源完成后，可以先本地试试图片是否能正常展示。如果展示没有问题，直接运行 react-native bundle 的打包命令，开始打包编译：

```sh
npx react-native bundle --entry-file index.tsx --dev false --minify false --bundle-output ./build/index.bundle --assets-dest ./build
```

说明：

- 新建一个 build 目录(如果没有的话)；
- 根目录的 index.tsx 文件为入口（entry file）；
- dev=false - 产出 release 环境的包；
- minify=false - 这个包不用压缩；
- 并将这个包命名为 ./build/index.bundle，同时将静态资源编译产物放到 ./build 目录

在 index.bundle 文件，搜索 图片名称 关键字，可以找到一个和 【图片名称】 关键字相关的独立模块，这个模块的作用就是将静态图片资源的路径、宽高、格式等信息，注册到一个全局管理静态图片资源中心。这个独立模块的代码如下：

```js
module.exports = _$$_REQUIRE(_dependencyMap[0]).registerAsset({
  __packager_asset: true,
  httpServerLocation: '/assets/src/assets/images',
  width: 1024,
  height: 1024,
  scales: [1],
  hash: '14ad75f6ff2cb977d546dc710e38279c',
  name: 'running',
  type: 'png',
});
```

**注意：**

- 在使用 require 函数引入静态图片资源时，图片的相对路径必须用字面常量表示的原因是，字面常量'./running.jpg'提供的是一个直接的明确的图片相对路径，打包工具很容易根据字面常量'./running.jpg' 找到真正的图片，提取图片信息。而变量 path 提供的是一个间接的可变化的图片路径，你光看 require(path) 这段代码是不知道真正的图片放在哪的，打包工具也一样，更别提自动提取图片信息了。
- 第一步“编译时”生成的图片注册函数和其注册的信息，我们在后面的第三步“运行时”还会用到。

2. **构建：**编译后的 Bundle 和静态图片资源，会在构建时内置到 App 中

使用命令构建时，默认构建的是调试包，而我们想要的是正式包，因此我们还需要在命令后面加一行配置--configuration Release。

```sh
$ npx react-native run-ios --configuration Release
```

编译后的 Bundle，包括 Bundle 中的静态图片资源信息，和真正的静态图片资源都已经内置到 App 中了。

**命令 react-native run-ios 既包括第一步的编译 react-native bundle 又包括第二步的构建。**

3. **运行：**在运行时，require 引入的并不是静态图片资源本身，而是静态图片资源的信息。Image 元素要在获取到图片路径等信息后，才会按配置的规则加载和展示图片。

可以通过 Image.resolveAssetSource 方法来获取图片信息

**在 Image 组件底层，使用的就是 Image.resolveAssetSource 来获取图片信息的，**, 然后，再根据这些图片信息，找到“构建时”内置在 App 中的静态图片资源，并将图片加载和显示的。这就是静态图片资源的加载原理。

4. 小结一下：

- 编译时，提前获取图片的宽高信息；
- 构建时，内置了静态图片资源；
- 运行时，程序可以获取图片宽高和真正的图片资源。

## 网络图片

1. 网络图片（Network Images）指的是使用 http/https 网络请求加载远程图片的方式。

2. 在使用网络图片时，我建议你将宽高属性作为一个必填项来处理。

### 缓存与预加载

1. React Native Android 用的是 Fresco 第三方图片加载组件的缓存机制，iOS 用的是 NSURLCache 系统提供的缓存机制。

第一次访问时，网络图片是先加载到内存中，然后再落盘存在磁盘中的。后续如果我们需要再次访问，图片就会从缓存中直接加载，除非超出了最大缓存的大小限制。

2. iOS 的 NSURLCache 遵循的是 HTTP 的 Cache-Control 缓存策略，同时当 CDN 图片默认都已经设置了 Cache-Control 时，iOS 图片就是有缓存的。

- NSURLCache 的默认最大内存缓存为 512kb，最大磁盘缓存为 10MB，如果缓存图片的体积超出了最大缓存的大小限制，那么一些老的缓存图片就会被删除。

3. 图片缓存机制有什么用呢？

通过图片缓存机制和预加载机制的配合，我们可以合理地利用缓存来提高图片加载速度，这能进一步地提升用户体验。

4. 使用图片预加载机制，可以提前把网络图片缓存到本地。对于用户来说，提前缓存的图片是第一次看到的，但对于系统缓存来说图片是第二次加载，它的加载速度是毫秒级的甚至亚秒级的。这就是预加载机制，提升图片加载性能的原理。

5. 在这种无限滚动的长列表场景中，图片预加载就非常适合了。React Native 也提供了非常方便的图片预加载接口 Image.prefetch：

## 宿主应用图片

1. 宿主应用图片（Images From Hybrid App’s Resources​）指的是 React Native 使用 Android/iOS 宿主应用的图片进行加载的方式。在 React Native 和 Android/iOS 混合应用中，也就是一部分是原生代码开发，一部分是 React Native 代码开发的情况下，你可能会用到这种加载方式。

2. 在 React Native 中，我们为什么要用 URI ，比如 { uri: 'app_icon' } ，来代表图片，而不是用更常用的 URL，比如 { url: 'app_icon' } ， 代表图片呢？

这是因为，URI 代表的含义更广泛，它既包括 URN 这种用名称代表图片的方式，也包括用 URL 这种地址代表图片的方式。

3. 使用 Android drawable 或 iOS asset 文件目录中的图片资源时，我们可以直接通过统一资源名称 URN（Uniform Resource Name）进行加载。不过，使用 Android asset 文件目录中图片资源时，我们需要在指定它的统一资源定位符 URL（Uniform Resource Locator）。

4. **在我们国内，绝大多数的 React Native 应用都是混合应用，都是把 React Native 当做一个支持动态更新的跨端框架来使用的。**这种情况下，我们在 React Native 中直接用宿主应用图片资源不是更好吗？

在实际工作中，我不推荐你在 React Native 中使用宿主应用图片资源。首先，这种加载图片的方法没有任何的安全检查，一不小心就容易引起线上报错。第二，大多数 React Native 是动态更新的，最新代码是跨多个版本运行的，而 Native 应用是发版更新的，应用的最新代码只在最新版本运行，这就导致 React Native 需要确切知道 Native 图片到底内置在哪些版本中，才能安全地使用，这对图片管理要求太高了，实现起来太麻烦了。

## Base64 图片

1. Base64 指的是一种基于 64 个可见字符表示二进制数据的方式，Base64 图片指的是使用 Base64 编码加载图片的方法，它适用于那些图片体积小的场景。

2. 通常我们看的图片资源 .jpg、.png 都是二进制格式的，二进制格式的图片是以独立文件存在的。而当二进制图片 Base64 化后，就变成了一段由字母、数字和符号组成的字符串。

3. **由于 Base64 图片是嵌套在 Bundle 文件中的，所以 Base64 图片的优点是无需额外的网络请求展示快，缺点是它会增大 Bundle 的体积。**

在动态更新的 React Native 应用中，Base64 图片展示快是以 React Native 页面整体加载慢为代价的。原因就是它会增加 Bundle 的体积，增加 Bundle 的下载耗时，从而导致 React Native 页面展示变慢。

4. Base64 从 ASCII 256 个字符中选取了 64 个可见字符作为基础，这样就二进制就能以 Base64 的格式转换为 ASCII 字符串了。

5. **建议是 Base64 图片只适合用在体积小的图片或关键的图片上。**

## 最佳实践

这套最佳实践，适用于那些将 React Native 当做一个动态更新框架来使用的应用中。

1. 首先是静态图片资源。如果你使用的是自研的热更新平台，就需要注意图片资源一定要先于 bundle 或和 bundle 一起下发，因为在执行 bundle 时，图片资源是必须已经存在的。

2. 网络图片和 Base64 图片。这两类图片之所以放在一起说，是因为它们单独管理起来都不方便，一张张手动上传网络图片不方便，一张张手动把图片 Base64 化也不方便，所以我们需要一个自动化的工具来管理它们。

- 可以把需要上传到网络的图片放在代码仓库的 assets/network 目录，把需要 Base64 化的图片放在 assets/base64 目录。

3. 宿主应用图片，这种加载图片的方式我不建议你使用

## 小结

1. 发版更新的 React Native 应用，使用内置图片的最佳方式是静态图片资源，但对于动态更新的 React Native 应用而言，需要注意静态图片资源并不是真正的“内置”，而是必须和 Bundle 执行文件“同步”的加载。

2. 推荐你自研一个图片管理工具，把设计师给你的图片管理起来，并按照指定的配置规则转换为 Base64 图片或网络图片，这样可以提高你的开发效率。

3. React Native 框架对图片的默认缓存处理并不是最优的方案，社区中提供了替代方案 FastImage，它是基于 SDWebImage (iOS) 和 Glide (Android) 实现的性能和效果会更好一些。