# 敲重点，这是小程序的Vant weapp，不是浏览器端的Vant！
根据官方教程，在wxml里键入以下代码：  
```xml
<van-swipe-cell id="swipe-cell" right-width="{{ 65 }}" left-width="{{ 65 }}" async-close bind:close="onClose">
  <view slot="left">选择</view>
  <van-cell-group>
    <van-cell title="单元格" value="内容" />
  </van-cell-group>
  <view slot="right">删除</view>
</van-swipe-cell>
```
，在.js里键入  
```javascript
Page({
  onClose(event) {
    const { position, instance } = event.detail;
    switch (position) {
      case 'left':
      case 'cell':
        instance.close();
        break;
      case 'right':
        Dialog.confirm({
          message: '确定删除吗？'
        }).then(() => {
          instance.close();
        });
        break;
    }
  }
});
```

是得不到官方的效果如下的：  
![img](../image/e3a98534772d7d2a4aab8c5f05383fb5.jpeg)  
你会发现右边的删除是没有样式的，  

因此，需要在wxss里添加上如下样式：  
```xml
.van-swipe-cell__left,
.van-swipe-cell__right {
  display: inline-block;
  width: 65px;
  height: 44px;
  font-size: 15px;
  line-height: 44px;
  color: #fff;
  text-align: center;
  background-color: #f44;
}
```
才能有官方的效果。  
