# 20230309-Vue3中如何描述VNode节点？配合antd Vue 的Modal.confirm自定义弹窗内容

在使用antd Vue组件时，可以通过函数式`AModal.confirm()`很方便的调出确认提示框，相比于在模板中使用Modal组件，函数式的的调用方法适合展示简单的文本信息，如果需要对文本做一些样式调整是没有那么方便的。 传入参数中,`content`字段默认支持VNode节点，或者直接传入字符串。

下面就讲解：

- 如何使用`createVNode `描述虚拟节点

- 如何使用VNode节点描述嵌套标签
- 如何添加节点样式

## 一、createVNode 的使用

createVNode是Vue 3中用于创建虚拟节点的函数。

1. 首先，在Vue 3中引入createVNode函数：

   ```js
   import { createVNode } from 'vue';
   ```

2. 然后，调用createVNode函数来创建虚拟节点。createVNode函数有三个参数：
   语法：`createVNode(tag,props,children)`

   - tag: 虚拟节点的标签名（例如div、h1、p等）。
   - props: 虚拟节点的属性对象，如class、style等。
   - children: 虚拟节点的子节点，可以是一个字符串、一个数组（包含多个子节点）或者一个组件。

   例如，我们要创建一个div元素，其中包含一个h1元素和一个p元素，代码如下：

   ```js
   const vnode = createVNode('div', { class: 'container' }, [
     createVNode('h1', null, '你好'),
     createVNode('p', null, '欢迎你'),
   ]);
   ```

3. 最后，我们可以使用vnode值来渲染到页面上，例如：

   ```js
   import { render } from 'vue';
   
   render(vnode, document.getElementById('app'));
   ```

   这样就可以将虚拟节点渲染到具有id="app"的元素中。

## 二、如何描述一个嵌套标签？

效果图如下：

![image-20230309170633415](https://s2.loli.net/2023/03/09/uqYLCs5tWGBONo8.png)

这里需要把名字部分加粗展示，所以需要单独用标签包裹名字，并添加样式，需要描述的标签如下：
`<div>xxx <strong> 名字 </strong> xxx</div>`

下面使用VNode描述并使用：

```js
import { ExclamationCircleOutlined } from '@ant-design/icons-vue';	
import { createTextVNode,createVNode } 
	// 创建文本节点
    const textVNode1 = createTextVNode('是否将成员')

    // 创建子节点
    const strongVNode = createVNode('strong', null, ` ${record.fullName} `)

    // 创建文本节点
    const textVNode2 = createTextVNode(`设置为审核员？ \n
    注：设为审核员的成员，姓名及手机号码会在下级机构业务申请时进行展示，方便沟通。`)

    // 创建父节点
    const parentVNode = createVNode('div', 
      {
        // style: 'white-space: pre-line;',
      }, 
      [
        textVNode1,
        strongVNode,
        textVNode2
      ]
    )
    
    AModal.confirm({
      title: '温馨提示',
      content: parentVNode,
      icon: createVNode(ExclamationCircleOutlined),
      okText: '确定',
      cancelText: '取消',
      onOk() {
      },
      onCancel() {},
    });
```



## 问题三：如何添加节点样式？文本如何换行？

直接传入字符串，并且在中间加入`\n`转译字符，也会被当做普通文本展示，无法换行。

```
content:".... \n注：设为审核员的成员，姓名及手机号码会在下级机构业务申请时进行展示，方便沟通。"
```

解决办法就是加上一个样式：`white-space: pre-line;`即可。

```js
// 创建父节点
    const parentVNode = createVNode('div', 
      {
       style: 'white-space: pre-line;',
      }, 
      [
        textVNode1,
        strongVNode,
        textVNode2
      ]
    )
```

效果图：

![image-20230309165724457](https://s2.loli.net/2023/03/09/L8DMcPAf4Sht3Bl.png)

## 如何自定义按钮？

![image-20230426135348181](https://s2.loli.net/2023/04/26/AJRaNtnoxgfUyM2.png)

遇到一些傻逼的测试，连个确认按钮都要统一大小，为了不跟这种人纠缠就花点时间处理算了。

其实下面两个按钮：取消，确认，都是可以传入Vnode的。

![image-20230426135416802](https://s2.loli.net/2023/04/26/ZTS1h9LjF76rkwW.png)

照葫芦画瓢，同样的方法炮制一个VNode，第二个参数写上style。

![image-20230426135655965](https://s2.loli.net/2023/04/26/IQTtO7kFhdJM6CH.png)

效果如下：

![image-20230426135750886](https://s2.loli.net/2023/04/26/mRLfU496TxEaigS.png)

## 简版写法

```js
import { h, createVNode } from 'vue'
import { message as AMessage, Modal as AModal } from 'ant-design-vue'
import { ExclamationCircleOutlined } from '@ant-design/icons-vue'

AModal.confirm({
            title: '删除提醒',
            content: () => {
                return h('div', {}, [
                    h('p', {}, [
                        '角色名称：',
                        h(
                            'span',
                            {
                                style: { color: '#02A7F0' },
                            },
                            [item.templateRoleName],
                        ),
                    ]),
                    h('div', {}, ['删除不影响已有机构的角色；']),
                    h('div', {}, ['如果该地区尚未开设机构，新开设的机构将使用通用模版中的角色。']),
                ])
            },
            icon: createVNode(ExclamationCircleOutlined),
            okText: '立即删除',
            cancelText: '暂不删除',
            onOk() {
                return new Promise((resolve, reject) => {
                    const res = deleteRole(item, resolve, reject)
                    return res
                })
            },
            onCancel() {},
        })
```

