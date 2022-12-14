# Form 表单

> [Form 表单](https://2x.antdv.com/components/form-cn#API)

![image-20221214113805744](https://f.pz.al/pzal/2022/12/14/13213f45810bb.png)

## model

定义表单的字段，分为`commonData`公共参数，其他参数，在`composables`目录下

![image-20221214114150762](https://f.pz.al/pzal/2022/12/14/7190db4126498.png)

![image-20221214114209821](https://f.pz.al/pzal/2022/12/14/bf72477708848.png)

## label-col 

| labelCol | label 标签布局，同 `<Col>` 组件，设置 `span` `offset` 值，如 `{span: 3, offset: 12}` 或 `sm: {span: 3, offset: 12}` | [object](https://2x.antdv.com/components/grid-cn/#Col) |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------ |
|          |                                                              |                                                        |

通过 labelCol.style 设置固定宽度

```js
labelCol: { style: { width: '150px' } },
```

实际应用：

```js
//所有的数据逻辑通过外部文件引入
import assetOptForm from './composables/assetOptForm'; // 操作表单
const { labelCol } = assetOptForm();
```

```js
export default function formOperate() {
    // 表单label
    const labelCol = {
        style: { width: '120px'},
    };
    
    return {
     labelCol
    }
  }
```

## rules 校验规则

```js
import assetOptForm from './composables/assetOptForm'; // 操作表单
export default{
    setup(){
        const { commonRules, resourceRules } = assetOptForm();
        // 验证规则
        const rules = Object.assign(commonRules, resourceRules);
        return {
            rules
        }
	}
}

```

### 规则

```js
// 公共验证规则
  const commonRules = {
    sysOrganizationId: [
      { type: 'array', min: 1, required: true, message: '请选择' },
    ],
    assetSelfCode: [
      { required: false, type: 'string', max: 30, message: '最多可输入30个字符' },
    ],
    // 通用验证：限制50个字符
    normalMax: [
      { required: false, type: 'string', max: 50, message: '最多可输入50个字符', trigger: 'blur' },
    ],
    // 通用价格
    normalPrice: [
      { required: false, type: 'number', validator: validPositiveDecimal2, trigger: 'blur' },
    ],
  };
```

规则对象里的每个key值，对应`form-item`中的`name`名字：

```html
<a-col :span="24">
    <a-form-item label="资产自编号" name="assetSelfCode">
        <a-input v-model:value="formData.assetSelfCode" placeholder="请输入" />
    </a-form-item>
</a-col>
```

## row col -- Grid栅格布局的使用

> [Grid栅格](https://2x.antdv.com/components/grid-cn)

![image-20221214142356456](https://f.pz.al/pzal/2022/12/14/a2b8080b80729.png)

### Row

| 成员    | 说明                                                         | 类型                | 默认值  |
| :------ | :----------------------------------------------------------- | :------------------ | :------ |
| align   | flex 布局下的垂直对齐方式：`top` `middle` `bottom`           | string              | `top`   |
| gutter  | 栅格间隔，可以写成像素值或支持响应式的对象写法来设置水平间隔 `{ xs: 8, sm: 16, md: 24}`。或者使用数组形式同时设置 `[水平间距, 垂直间距]`（`1.5.0 后支持`）。 | number/object/array | 0       |
| justify | flex 布局下的水平排列方式：`start` `end` `center` `space-around` `space-between` | string              | `start` |
| wrap    | 是否自动换行                                                 | boolean             | false   |

### Col

| span | 栅格占位格数，为 0 时相当于 `display: none` | number | -    |
| ---- | ------------------------------------------- | ------ | ---- |
|      |                                             |        |      |

剩下的属性请看官方文档

## form-item 表单域

表单一定会包含表单域，表单域可以是输入控件，标准表单域，标签，下拉菜单，文本域等。包含需要验证的内容时，需要使用。

![image-20221214143440141](https://f.pz.al/pzal/2022/12/14/2f4397bec001a.png)

# cascader 级联选择

![image-20221214143610985](https://f.pz.al/pzal/2022/12/14/102ca7ef468be.png)

## options 

数据源

## fieldNames

自定义 options 中 label name children 的字段；

默认值：`{ label: 'label', value: 'value', children: 'children' }`

```js
import getSelectOptions from './composables/getSelectOptions'; // 获取下拉选项
setup() {
	const {orgFieldNames} = getSelectOptions()
    return {orgFieldNames}
}
```

```js
//./composables/getSelectOptions
export default function commonOperate() {
    // 组织机构联动选项对应字段
  const orgFieldNames = {
    label: 'organizationName',
    value: 'sysOrganizationId',
    children: 'sysOrganizationList',
  };
    return {
        orgFieldNames
    }
}
```

## value(v-model) 

指定选中项

`v-model:value="formData.sysOrganizationId"`

## show-search

在选择框中显示搜索框，true/false

![image-20221214144645424](https://f.pz.al/pzal/2022/12/14/35b33f530b6f5.png)

# select 选择器

可以使用`select-option`标签，也可以使用options属性传入列表。

![image-20221214145152806](https://f.pz.al/pzal/2022/12/14/32b8997e55faf.png)

![image-20221214145258327](https://f.pz.al/pzal/2022/12/14/a9904c1f2a75e.png)

## value(v-model)

指定当前选中的条目

省市区联动例子

```vue
<template>
  <a-space>
    <a-select
      v-model:value="province"
      style="width: 120px"
      :options="provinceData.map(pro => ({ value: pro }))"
    >
    </a-select>
    <a-select
      v-model:value="secondCity"
      style="width: 120px"
      :options="cities.map(city => ({ value: city }))"
    >
    </a-select>
  </a-space>
</template>
<script lang="ts">
const provinceData = ['Zhejiang', 'Jiangsu'];
const cityData = {
  Zhejiang: ['Hangzhou', 'Ningbo', 'Wenzhou'],
  Jiangsu: ['Nanjing', 'Suzhou', 'Zhenjiang'],
};
import { defineComponent, reactive, toRefs, computed, watch } from 'vue';
export default defineComponent({
  setup() {
    const province = provinceData[0];
    const state = reactive({
      province,
      provinceData,
      cityData,
      secondCity: cityData[province][0],
    });
    const cities = computed(() => {
      return cityData[state.province];
    });

    watch(
      () => state.province,
      val => {
        state.secondCity = state.cityData[val][0];
      },
    );

    return {
      ...toRefs(state),
      cities,
    };
  },
});
</script>


```

# input 输入框

## input-password 密码框

```html
<a-input-password v-model:value="value" placeholder="input password" />
```

![image-20221214151232429](https://f.pz.al/pzal/2022/12/14/87632e562bf1f.png)

## textarea 文本域

用于多行输入

```html
<a-textarea v-model:value="value" placeholder="Basic usage" :rows="4" />
```

![image-20221214151345591](https://f.pz.al/pzal/2022/12/14/fd1e266f3193d.png)

## input-search 搜索框

```html
<a-input-search
      v-model:value="value"
      placeholder="input search text"
      style="width: 200px"
      @search="onSearch"
    />
<a-input-search
      v-model:value="value"
      placeholder="input search text"
      enter-button="Search"
      size="large"
      @search="onSearch"
    />
```

## input-group 输入框组合

```html
 <a-input-group compact>
      <a-select v-model:value="value5">
        <a-select-option value="Option1">Option1</a-select-option>
        <a-select-option value="Option2">Option2</a-select-option>
      </a-select>
      <a-input style="width: 50%" v-model:value="value6" />
    </a-input-group>
```

input的属性

type:number 输入数字

![image-20221214150723405](https://f.pz.al/pzal/2022/12/14/76d40f67975cd.png)

![image-20221214152003387](https://f.pz.al/pzal/2022/12/14/98e936f227925.png)

# upload 

## 视频上传

视频上传组件位置

![image-20221214154416087](https://f.pz.al/pzal/2022/12/14/1678beb932858.png)

### fileList 

已经上传的文件列表（受控）

`v-model:file-list="fileList"`

### accept

接受上传的文件类型, 详见 [input accept Attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file#accept)

### action

上传的地址

### headers

设置上传的请求头部，IE10 以上有效

![image-20221214162531726](https://f.pz.al/pzal/2022/12/14/ac931ef438086.png)

![image-20221214162558071](https://f.pz.al/pzal/2022/12/14/c0b364c60c8cc.png)

### data

上传所需参数或返回上传参数的方法

![image-20221214162504074](https://f.pz.al/pzal/2022/12/14/9e0bf6afd05c4.png)

### listType

上传列表的内建样式，支持三种基本样式 `text`, `picture` 和 `picture-card`，默认是text

### beforeUpload

上传文件之前的钩子，参数为上传的文件，若返回 `false` 则停止上传。支持返回一个 Promise 对象，Promise 对象 reject 时则停止上传，resolve 时开始上传（ resolve 传入 `File` 或 `Blob` 对象则上传 resolve 传入对象）。**注意：IE9 不支持该方法**。

上传前可以校验文件的格式、大小。

```js
/** 上传前 */
    const beforeUpload = (file) => {
      // 获取文件后缀
      const fileNameArr = file.name.split('.');
      const suffix = fileNameArr[fileNameArr.length - 1];//后缀名
      const isLtMax = file.size / 1024 / 1024 <= fileSizeMax;//最大多少mb

      // 用promise不会进入 changeUpload
      return new Promise((resolve, reject) => {
        if (isLtMax) {
          if (fileTypes.includes(file.type?.toLowerCase()) || fileTypesName.indexOf(suffix?.toLowerCase()) > -1) {
            return resolve();
          } else {
            AMessage.error('请按要求上传！');
            return reject(false);
          }
        } else {
          AMessage.error(`文件大小不能超过 ${fileSizeMax}MB!`);
          return reject(false);
        }
      });
    };
```

## @preview 事件

点击文件链接或预览图标时的回调，`Function(file)`

`@preview="handlePreview"`

```js
  /** 点击预览 */
    const handlePreview = (file) => {
      previewSrc.value = file.videoUrl;
      previewVisible.value = true;
    };
```

使用`modal`组件弹出一个框预览视频、图片：

```html
<!-- 预览 -->
    <a-modal wrapClassName="asset-video-modal" :visible="previewVisible" :footer="null" @cancel="cancelPreview">
      <div class="asset-video-modal--contain">
        <video
          v-if="previewSrc"
          ref="previewRef"
          style="width: 100%;"
          :key="previewSrc"
          width="100%"
          controls
        >
          <source :src="previewSrc" type="video/mp4" />
          <source :src="previewSrc" type="video/ogg" />
          <source :src="previewSrc" type="video/webm" />
          <object :data="previewSrc" width="100%">
            <embed :src="previewSrc" width="100%" />
          </object>
          您的浏览器不支持 video 标签
        </video>
      </div>
    </a-modal>
```

## @change 事件

上传文件改变时的状态，详见 [change](https://2x.antdv.com/components/upload-cn#change)

```js
/** 上传 */
    const changeUpload = ({ file }) => {
      console.log("file:",file)
      // 上传成功
      if (file.status === 'done') {
        if (file.response?.code === 200) {
          AMessage.success(file.name + '上传成功！');
          // 更新路径 - 文件上传完成不确定先后顺序，不能直接用索引
          console.log("fileList.value:",fileList.value)
          fileList.value.map(c => {
            console.log("c.response:",c.response)
            if(c.status === 'done' && !c.videoKey){
              const {
                videoMap: { url: videoUrl, key: videoKey },
                imageMap: { url, key: imgKey }
              } = c.response.data;

              c = Object.assign(c, { videoUrl, videoKey, url, imgKey })
              console.log("c:",c)
            }
          });
        } else {
          AMessage.error(file.name + '上传失败：' + file.response.message);
          // 删除文件
          fileList.value.pop();
        }
      } else if (file.status === 'error') {
        AMessage.error(file.name + '上传失败：' + file.response.message);
        // 删除文件
        fileList.value.pop();
      }
    };
```

![image-20221214164227639](https://f.pz.al/pzal/2022/12/14/92dcdda4b2b53.png)

![image-20221214164438917](https://f.pz.al/pzal/2022/12/14/3d38bbe16f044.png)