# List: 高性能的无限列表

1. React Native 官方提供的列表组件确实是 FlatList，但是我推荐你优先使用开源社区提供的列表组件 RecyclerListView。因为，开源社区提供的 RecyclerListView 性能更好。

2. 对于列表组件，最应该关心的就是性能。

3. FlatList 在 iOS 端表现很好，但在 Android 低端机还是能感觉到卡顿。

4. 性能更好的列表组件 RecyclerListView。通常评判列表卡顿的指标是 UI 线程的帧率和 JavaScript 线程的帧率。

5. 业内有人实验过，在已经渲染完成的页面中，通过死循环把 JavaScript 线程卡死，页面依旧能够滚动。这是因为滚动本身是在 UI 线程进行的，和 JavaScript 线程无关。但当用户下滑，需要渲染新的列表项时，就需要 JavaScript 线程参与进来了。如果这时候 JavaScript 掉帧了，新的列表项就渲染不出来，即便能滚动，用户看到也是空白项，一样影响用户体验。

6. 即使现在新架构马上要出来了，在这个时间点上，我最推荐你用的还是 RecyclerListView。因为从原理上 RecyclerListView 比 FlatList 强上不少。

7. 为什么开源社区的 RecyclerListView 比官方的 FlatList 性能更好？FlatList、RecyclerListView 的优化原理是什么？

FlatList 和 RecyclerListView 的底层实现都是滚动组件 ScrollView

## ScrollView：渲染所有内容的滚动组件

1. React Native 的 ScrollView 组件在 Android 的底层实现用的是 ScrollView 和 HorizontalScrollView，在 iOS 的底层实现用的是 UIScrollView。

2. 所谓的滚动，解决的是在有限高度的屏幕内浏览无限高度的内容的问题。有限高度的容器是 ScrollView，无限高度，或者说高度不确定的内容是 ScrollView 的 children。

3. 使用 ScrollView 组件时，我们通常并不直接给 ScrollView 设置固定高度或宽度，而是给其父组件设置固定高度或宽度。

一般而言，我们会使用安全区域组件 SafeAreaView 组件作为 ScrollView 的父组件，并给 SafeAreaView 组件设置布局属性 flex:1，让内容自动撑高 SafeAreaView。使用 SafeAreaView 作为最外层组件的好处是，它可以帮我们适配 iPhone 的刘海屏，节约我们的适配成本，

4. 使用 ScrollView 组件时，ScrollView 的所有内容都会在首次刷新时进行渲染。内容很少的情况下当然无所谓，内容多起来了，速度也就慢下来了。

那有什么优化方案吗？你肯定想到了一些优化方案，**比如按需渲染。**

## FlatList：按需渲染的列表组件

1. FlatList 组件底层使用的是虚拟列表 VirtualizedList，VirtualizedList 底层组件使用的是 ScrollView 组件。因此 VirtualizedList 和 ScrollView 组件中的大部分属性，FlatList 组件也可以使用。

2. 列表组件和滚动组件的关键区别是，列表组件把其内部子组件看做由一个个列表项组成的集合，每一个列表项都可以单独渲染或者卸载。而滚动组件是把其内部子组件看做一个整体，只能整体渲染。而自动按需渲染的前提就是每个列表项可以独立渲染或卸载。

3. FlatList 性能比 ScrollView 好的原因是， FlatList 列表组件利用按需渲染机制减少了首次渲染的视图，利用空视图的占位机制回收了原有视图的内存.

4. 实现 FlatList 自动按需渲染的思路具体可以分为三步：

- 通过滚动事件的回调参数，计算需要按需渲染的区域；
- 通过需要按需渲染的区域，计算需要按需渲染的列表项索引；
- 只渲染需要按需渲染列表项，不需要渲染的列表项用空视图代替。

5. 怎么计算按需渲染列表项的索引呢？

两种情况：

- 列表项的高度是确定的情况；
- 另外一种是列表项的高度是不确定的情况。

对于高度未知的情况，FlastList 会启用列表项的布局回调函数 onLayout，在 onLayout 中会有大量的动态测量高度的计算，包括每个列表项的准确高度和整体的平均高度。

在这种列表项高度不确定，而且给定按需渲染区域的情况下，我们可以通过列表项的平均高度，把按需渲染列表项的索引大致估算出来了。即便有误差，比如预计按需渲染区域为上下 10 个屏幕，实际渲染时只有上下 7、8 个屏幕也是能接受的，大部分情况下用户是感知不到的屏幕外内容渲染的。

实际生产中，如果你不填 getItemLayout 属性，不把列表项的高度提前告诉 FlatList，让 FlatList 通过 onLayout 的布局回调动态计算，用户是可以感觉到滑动变卡的。因此，**如果你使用 FlatList，又提前知道列表项的高度，我建议你把 getItemLayout 属性填上。**

## RecyclerListView：可复用的列表组件

1. RecyclerListView 是开源社区提供的列表组件，它的底层实现和 FlatList 一样也是 ScrollView，它也要求开发者必须将内容整体分割成一个个列表项。

2. 在首次渲染时，RecyclerListView 只会渲染首屏内容和用户即将看到的内容，所以它的首次渲染速度很快。在滚动渲染时，只会渲染屏幕内的和屏幕附近 250 像素的内容，距离屏幕太远的内容是空的。

3. React Native 的 RecyclerListView 复用灵感来源于 Native 的可复用列表组件。

在 iOS 中，表单视图 UITableView，实际就是可以上下滚动、左右滚动的可复用列表组件。它可以通过复用唯一标识符 reuseIdentifier，标记表单中的复用单元 cell，实现单元 cell 的复用。

在 Android 上，动态列表 RecyclerView 在列表项视图滚出屏幕时，不会将其销毁，相反会把滚动到屏幕外的元素，复用到滚动到屏幕内的新的列表项上。这种复用方法可以显著提高性能，改善应用响应能力，并降低功耗。

4. **RecyclerListView 在滚动时复用了列表项，而不是创建新的列表项，因此性能好。**

### 使用

1. RecyclerListView 有三个必填参数：

- 列表数据：dataProvider(dp)；
- 列表项的布局方法：layoutProvider；
- 列表项的渲染函数：rowRenderer。

2. 使用列表组件 RecyclerListView 有两个前提：

- 首先是列表项的宽高必须是确定的，或者是大致确定的；
- 第二是列表项的类型必须是可枚举的。

这两个前提，都体现在了列表项的布局方法 layoutProvider 中了。

如果宽高不确定呢？分两种情况，一种就是不确定的，另一种是不确定但可以转换为大致确定的。对于就是不确定的情况，RecyclerListView 是无解的；对于大致确定的情况，我们可以开启 forceNonDeterministicRendering 小幅修正布局位置。

**如果是信息流的内容高度不确定，相差百来个像素，这种大幅修正可能会让用户察觉到，不适合使用 RecyclerListView 。**

类型可枚举。可枚举很好理解，两个列表项的底层 UI 视图必须一样或者大致相似，才能只改列表数据复用列表视图。如果每个列表项的 JSX 结构完全不一样，就不存在复用的可能性。一般来说，一个类型对应一个自定义组件。

## VS. ScrollView、FlatList、RecyclerListView

1. 从底层原理看：

- ScrollView 内容的布局方式是从上到下依次排列的，你给多少内容，ScrollView 就会渲染多少内容；
- FlatList 内容的布局方式还是从上到下依次排列的，它通过更新第一个和最后一个列表项的索引控制渲染区域，默认渲染当前屏幕和上下 10 屏幕高度的内容，其他地方用空白视图进行占位；
- RecyclerListView 性能最好，你应该优先使用它，但使用它的前提是列表项类型可枚举且高度确定或大致确定。

2. 内存上，FlatList 要管理 21 个屏幕高度的内容，而 RecyclerListView 只要管理大概 1 个多点屏幕高度的内容，RecyclerListView 使用的内存肯定少。计算量上，FlatList 要实时地销毁新建 Native 的 UI 视图，RecyclerListView 只是改变 UI 视图的内容和位置，RecyclerListView 在 UI 主线程计算量肯定少。

### 适合场景

1. ScrollView 适合内容少的页面，只有几个屏幕高页面是适合的；

2. FlatList 性能还过得去，但我不推荐你优先使用它，**只有在你的列表项内容高度不能事先确定，或者不可枚举的情况下使用它**；

3. RecyclerListView 性能最好，你应该优先使用它，但使用它的前提是可枚举且高度确定或大致确定。
