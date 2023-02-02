# 20230103-jQuery的API使用案例及技巧

## .text(val)

用于设置元素内容的文本。

```js
$("p").text("Hello world!");
```

## .find(xxx)

搜索所有与指定表达式匹配的元素。这个函数是找出正在处理的元素的后代元素的好方法。

所有搜索都依靠jQuery表达式来完成。这个表达式可以使用CSS1-3的选择器语法来写。

参数可以是：

- #### **jQuery object**  一个用于匹配元素的jQuery对象

- #### **element**  一个DOM元素

```html
<p><span>Hello</span>, how are you?</p>
```

```js
$("p").find("span")

//结果
// [ <span>Hello</span> ]
```

## .hide() / .show()

用于设置元素的显示和隐藏，基于`dispaly:none / block`。

## .empty()

清空这个元素内的所有内容，包括html节点和文本节点

## .done(()=>{})

当延迟成功时调用一个函数或者数组函数，一般用于`$.ajax`后面处理接口返回的数据

```js
$.get("test.php").done(function() { 
  alert("$.get succeeded"); 
});
```

```js
$.ajax({
    url: 'xxx',
    type: 'post',
    dataType: 'json',
    contentType: 'application/x-www-form-urlencoded',
    data: {
        code: 'ASSET_FIXED_CATEGORY' 
    }
})
    .done(response => {
    // console.log("获取到的字典数据：", response);
    if (response.code === 200 && response.data) {
        // xxx
    } else if (response.code !== 200) {
        // xxx
    }
})
    .fail(error => {
    // xxx
});
```



## .fail(()=>{})

当延迟失败时调用一个函数或者数组函数.。

```js
$.get("test.php")
  .done(function(){ alert("$.get succeeded"); })//延迟成功
  .fail(function(){ alert("$.get failed!"); });//延迟失败
```

## .prop()

用于获取 / 设置元素上的属性。

- 获取

  ```js
  $("input[type='checkbox']").prop("checked");
  ```

  

- 设置：

```js
$("input[type='checkbox']").prop("disabled", true);
$("input[type='checkbox']").prop("checked", true);
```

## radio标签修改了checked为什么不触发change事件？

> [使用jQuery更改.prop不会触发.change事件 - IT屋-程序员软件开发技术分享社区 (it1352.com)](https://www.it1352.com/1480149.html)

原因是需要自己手动触发，使用`.change()`或`.trigger("change")`：

```js
$('input[type="checkbox"][name="something"]').prop("checked", false).change();
```

```js
$(secondChilds[0]).children('.def-radio_input').prop('checked', true).change()
```

监听的事件就能触发了：

```js
$(document).on('change', '.trade-detail-page_filters .def-radio_input', function (e) {
    //xxx
})
```

## .index()

```js
var _index = $(this).index()
```



## .attr()

## .eq()

## .addClass()

## .removeClass()

## .is()

```js
var _empty = $('.property-page-table_sheet').eq(activeTab).find('tbody').is(':empty');
```



## .parents()



## $.extend()



## 在$上自定义方法

### $.utilPreTimeDown()

```js
var timeDown = $.utilPreTimeDown('#' + item.downTimeId, handleSendServer);
```

![image-20230105101923938](https://s2.loli.net/2023/01/05/WtMzSaycuC9wrJU.png)

## .unbind()

```js
$slider.unbind('mouseenter').unbind('mouseleave');// 取消 slider的 hover事件，避免自动滑动
```

## .off()

```js
$('.trade-product-page-contain_left_opera').off('click', handleSliderPlay);
```

