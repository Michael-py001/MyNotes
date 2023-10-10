# 20231008-前端富文本HTML转word文档docx&PDF格式&文本压缩

## HTML转docx

需要依赖`html-docx-js`这个包；

安装：

```
npm install html-docx-js -S
```

需要注意的是在运行`htmlDocx.asBlob()`时会报错，因为 在严格模式下 不支持 with语句 ，需要修改源码：

![image-20231008115916985](https://s2.loli.net/2023/10/08/AHxUYsLyl7tzeuw.png)

修改 node_modules\html-docx-js\dist\html-docx.js 文件中的with语句：

搜索`with(obj||{})`，会出现三处使用的地方，

![img](https://s2.loli.net/2023/10/08/IAEQ81qgZWNflz5.png)

![img](https://s2.loli.net/2023/10/08/DcVWMBurfiyN3t7.png)

然后就可以正常运行了.

```js
import htmlDocx from 'html-docx-js/dist/html-docx';
/**
 * @param {string} htmlContent: html内容
 * @param {string} name: 文件名
 * @return {*}
 */
export function downloadDocx(htmlContent,name) {
  const html = getHtmlString(htmlContent);
  const blob = htmlDocx.asBlob(html);
  let link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = `${name || '内容'}.docx`;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  setTimeout(() => {
    URL.revokeObjectURL(link.href);
  }, 5000);
}
```

其中的`getHtmlString`函数为生成完整html document：

```html
const getHtmlString = content => {
  content = content.replace(/<img/g, '<img width="600" height="400"');
  let html = `<!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
  	  /** 此处可以放css样式*/

      p img {
        max-width: 300px;
        height: 300px;
        padding: 0 24px;
        margin-bottom: 24px;
      }
    </style>
  </head>
  <body>
    <div class="ql-editor">
      ${content}
    </div>
  </body>`;
  return html;
};

```

### 图片大小不能用css控制的问题

需要注意的是**css无法对img标签起作用**，比如设置宽高等都无效，最简单的解决方法是直接在Img标签上使用原生属性width,height设置宽高，但是这个属性的数值**不能带有单位**。

```html
<img width="600" height="400"> </img>
```



下载效果：

![image-20231009100214645](https://s2.loli.net/2023/10/09/q82wkj495PmCaz1.png)

![image-20231009101112318](https://s2.loli.net/2023/10/09/xupyCLsGiQPlKd8.png)

## HTML转PDF

安装html2pdf.js

```
npm i html2pdf.js
```

```js
import html2pdf from 'html2pdf.js';

export function downloadPDF(htmlContent, name) {
  const str = getHtmlString(htmlContent);
  let html = '<!DOCTYPE html>' + str;
  const opt = {
    margin: 1,
    filename: `${name || '内容'}.pdf`,
    image: { type: 'png', quality: 0.98 },
    html2canvas: { scale: 2 },
    jsPDF: { unit: 'in', format: 'letter', orientation: 'portrait' },
    pagebreak: {
      mode: ['avoid-all', 'css', 'legacy'],
    },
  };
  html2pdf().set(opt).from(html).save();
}

```

### 图片不显示问题

富文本中的图片可能是网络链接的图片地址，用这个插件导出不能直接生成图片，需要先转成base64格式才能正常导出显示。

解决方案：

- 编辑富文本插入图片时转base64
- 编辑富文本插入网络链接，提交给后端，详情接口返回网络链接，下载的单独接口转base64后返回

由于图片转成base64后字符串会很长，下面提供一个字符串压缩&解压的方案。

### 字符串压缩、解压

```js
// 压缩
export function zip (data) {
  if (!data) return data
  // 判断数据是否需要转为JSON
  const dataJson = typeof data !== 'string' && typeof data !== 'number' ? JSON.stringify(data) : data

  // 使用Base64.encode处理字符编码，兼容中文
  const str = Base64.encode(dataJson)
  let binaryString = pako.gzip(str);
  let arr = Array.from(binaryString);
  let s = "";
  arr.forEach((item, index) => {
    s += String.fromCharCode(item)
  })
  return btoa(s)
}

// 解压
export function unzip (b64Data) {
  let strData = atob(b64Data);
  let charData = strData.split('').map(function (x) {
    return x.charCodeAt(0);
  });
  let binData = new Uint8Array(charData);

  let data = pako.ungzip(binData);

  // ↓切片处理数据，防止内存溢出报错↓
  let str = '';
  const chunk = 8 * 1024
  let i;
  for (i = 0; i < data.length / chunk; i++) {
    str += String.fromCharCode.apply(null, data.slice(i * chunk, (i + 1) * chunk));
  }
  str += String.fromCharCode.apply(null, data.slice(i * chunk));
  // ↑切片处理数据，防止内存溢出报错↑
        
  const unzipStr = Base64.decode(str);
  let result = ''

  // 对象或数组进行JSON转换
  try {
    result = unzipStr
  } catch (error) {
    console.log("error:",error);
    if (/Unexpected token o in JSON at position 0/.test(error)) {
      // 如果没有转换成功，代表值为基本数据，直接赋值
      result = unzipStr
     }
   }
   return result
}
```

使用方法：

```js
// 提交接口参数时，压缩内容
const params = {
  htmlContent: zip(totalParams.htmlContent),
}
```

```js
//接收接口数据时解压，还原成正常字符串
formData.value.htmlContent = unzip(data.htmlContent);
```

### 图片出现分割成两页的问题

如果有多张图片，很可能出现以下图片被分割在两页的情况。

![image-20231009101028108](https://s2.loli.net/2023/10/09/YlD3CuNAoPyBGxZ.png)

以下是尽可能避免出现这种情况的解决方案：

#### 1. html2pdf配置中设置pagebreak

```js
pagebreak: {
      mode: ['avoid-all', 'css', 'legacy'],
    },
```

#### 2. 给图片标签加上分页属性

```css
p {
  break-inside: avoid;
  page-break-inside: avoid;
}
p img {
  break-inside: avoid;
  max-width: 500px;
  margin: 0 auto;
  padding: 0 24px;
  margin: 24px 0;
  page-break-inside: avoid;
}
```

但是还会出现以下情况，页面底部会有一条图片的分割线，目前没有找到好的办法解决。唯一的办法就是调用系统的打印print()，会自动处理分页问题。

![image-20231010135345458](https://s2.loli.net/2023/10/10/zNY7mVpRJF5whCK.png)

#### 3. 手动计算dom高度

手动计算dom高度，如果超出A4纸的高度，则放到下一页的dom



