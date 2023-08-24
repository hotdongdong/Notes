# react-native-swipe-list-view 组件使用

### react-native-swipe-list-view 是一个具有侧滑功能的 react-native 的组件

## 安装

```
npm install --save react-native-swipe-list-view
```

## 简单用法

```
import {SwipeListView, SwipeRow} from 'react-native-swipe-list-view';

<SwipeListView
  data={itemList}
  renderItem={({item, index}) => (
    <SwipeRow
      rightOpenValue={-75}
      disableLeftSwipe={false}
      disableRightSwipe={true}>   // 禁止右滑

      <!-- 隐藏内容，当滑动后出现的 -->
      <View style={style.HIDDEN}>
        <TouchableOpacity
          style={style.VIEW}>
          <Text style={style.HIDDEN_TEXT} text={'View'} />
        </TouchableOpacity>
        <TouchableOpacity style={style.SIGN_OUT}>
          <Text style={style.HIDDEN_TEXT} text={'Sign Out'} />
        </TouchableOpacity>
      </View>

      <!-- 显示的内容 -->
      <RenderItem item={item} index={index} />
    </SwipeRow>
  )}
/>
```

## 使用<SwipeRow>特别注意点

- 如果使用一个单独的<SwipeRow>，第一个通过定义 ref 去调用 closeRow()关闭 row
- 使用<SwipeRow>时，必须给<SwipeRow>传递两个元素，一个为隐藏元素，一个是渲染元素
- 想要实现侧滑效果，需要为第二个元素，也就是一开始渲染显示的元素增加 backgroundColor 属性

### 这种滑动实际上是两个组件，一个组件放在下面固定，上面的组件滑动，把下面隐藏的内容露出来，这就是为什么上面滑动的组件需要设置背景色否则就实现不了效果的原因。并且它不能控制最大滑动的距离，只能禁用一侧的滑动，如果滑动超过了下面隐藏组件的范围就会露出空白，虽然手势释放会回到设置的 openValue，但是也不太美观。

## 学习网站

```
官方地址：https://github.com/jemise111/react-native-swipe-list-view

使用分析：https://www.jianshu.com/p/65aa5db3bad8/
```
