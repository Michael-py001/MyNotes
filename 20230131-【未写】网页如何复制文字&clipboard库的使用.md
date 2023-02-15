# 20230131-网页如何复制文字&clipboard库的使用

> [clipboard.js — Copy to clipboard without Flash (clipboardjs.com)](https://clipboardjs.com/)

```js
// 复制文本内容
    handleCopyText: function() {
      $.each($('.copy-btn'), (index, item) => {
        var clipboard = new ClipboardJS(item);
        clipboard.on('success', e => {
          alert('复制成功: ' + e.text);
        });
      });
    },
```

```js
var clipboard = new ClipboardJS('.btn');

clipboard.on('success', function(e) {
    console.info('Action:', e.action);
    console.info('Text:', e.text);
    console.info('Trigger:', e.trigger);

    e.clearSelection();
});

clipboard.on('error', function(e) {
    console.error('Action:', e.action);
    console.error('Trigger:', e.trigger);
});
```

## 核心源码

```js
//copy.js 
/**
 * Create fake copy action wrapper using a fake element.
 * @param {String} target
 * @param {Object} options
 * @return {String}
 */
const fakeCopyAction = (value, options) => {
  const fakeElement = createFakeElement(value);
  options.container.appendChild(fakeElement);
  const selectedText = select(fakeElement);
  command('copy');
  fakeElement.remove();

  return selectedText;
};
```

```js
//command.js 
/**
 * Executes a given operation type.
 * @param {String} type
 * @return {Boolean}
 */
export default function command(type) {
  try {
    return document.execCommand(type); //复制字符串
  } catch (err) {
    return false;
  }
}
```

```js
/**
 * Copy action wrapper.
 * @param {String|HTMLElement} target
 * @param {Object} options
 * @return {String}
 */
const ClipboardActionCopy = (
  target,
  options = { container: document.body }
) => {
  let selectedText = '';
  // 如果是字符串
  if (typeof target === 'string') {
    selectedText = fakeCopyAction(target, options);
  } else if ( //如果是input标签
    target instanceof HTMLInputElement &&
    !['text', 'search', 'url', 'tel', 'password'].includes(target?.type)
  ) {
    // If input type doesn't support `setSelectionRange`. Simulate it. https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setSelectionRange
    selectedText = fakeCopyAction(target.value, options);
  } else {
    // 使用select.js 获取dom的值
    selectedText = select(target);
    command('copy');
  }
  return selectedText;
};
```

## select.js模块

> [select - npm (npmjs.com)](https://www.npmjs.com/package/select)

```html
<script src="dist/select.js"></script>
```

```js
var select = require('select');
```

使用

```js
var input = document.querySelector('input');
var result = select(input);
```

