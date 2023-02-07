# 全局属性

在所有的组件上，添加`class` / `style` 属性，可以直接添加类名，或者style属性。

![image-20221215152328443](https://f.pz.al/pzal/2022/12/15/839ba8d467be1.png)

![image-20221215152414971](https://f.pz.al/pzal/2022/12/15/3cf03cd7d7e67.png)

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

## 自定义校验规则

```js
const validateCode = async (rule, value) => {
    if (value === '') {
        return Promise.reject('图形验证码不能为空');
    } else if (!/^[a-zA-Z0-9]{4}$/.test(value)) {
        return Promise.reject('图形验证码错误或已失效');
    } else {
        return Promise.resolve();
    }
};

const loginFormRules = {
    //校验手机号
    mobile: [
        { required: true, message: '手机号不能为空', trigger: 'blur' },
        { pattern: /^1[3|4|5|6|7|8|9]\d{9}$/, message: '手机号格式有误', trigger: 'blur' },
    ],
    code: [{ required: true, validator: validateCode, trigger: 'blur' }],
    smsCode: [{ required: true, message: '短信验证码不能为空', trigger: 'blur' }],
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

## 表单校验需要写对应的name

![image-20221221173951725](https://f.pz.al/pzal/2022/12/21/d2bbf74a07c45.png)

## 表单的自动校验

如果在`form-item`中的button组件，设置`html-type='submit'`，那么点击就会自动校验表单，通过后会触发`form`组件的`@finish="handleFinish"`事件。

```html
<a-form-item :wrapper-col="{ span: 14, offset: 4 }">
    <a-button type="primary" html-type="submit">Submit</a-button>
    <a-button style="margin-left: 10px" @click="resetForm">Reset</a-button>
</a-form-item>
```

## 表单移除某一项验证

`clearValidate` : 移除表单项的校验结果。传入待移除的表单项的 name 属性或者 name 组成的数组，如不传则移除整个表单的校验结果

```js
/** 输入框切换可用 */
toggleInput(e, _key){
    const _check = e.target.checked;
    // 不选中清空
    if(!_check){
        this.formData[_key] = null;
    }
    // 移除验证
    if(this.$refs.formRef){
        this.$refs.formRef.clearValidate(_key); //$refs是获取a-form上设置的ref
    }
    // 验证规则切换是否必填
    this.rules[_key][0].required = _check;
},
```

![image-20230115145357668](https://s2.loli.net/2023/01/15/BqfiOAbol726WF9.png)

![image-20221216140203857](https://f.pz.al/pzal/2022/12/16/d761149189190.png)

![image-20221216140226715](https://f.pz.al/pzal/2022/12/16/3f70962044cbf.png)

## 单独检验其中的某一项

```js
loginFormRef.value //loginFormRef表单设置的ref获取的实例
            .validateFields(['mobile', 'code']) //字段名称
            .then(() => {})
```

## 动态表单required校验，星号动态显示

- 如果在rules里面写required，动态设置true/false，则label旁边的星号不会动态显示

- 如果在label-item里写了required属性，则会多出一行提示 xxx is required，跟自己定义的rules的提示不一样，就不符合要求

  ![image-20230130113044480](https://s2.loli.net/2023/01/30/Tot3yFzhWHlIGdE.png)

  ![image-20230130112837254](https://s2.loli.net/2023/01/30/pvKqcUSfrG97PlC.png)

  最终解决方法：使用自定义class控制星号的显示，使用`clearValidate`移除表单项的校验结果(不会影响下一次校验)，再手动设置rules对应字段的required为 true / false 。

![1](https://s2.loli.net/2023/01/30/oNC6FcaYB3UOs7e.gif)

 -  动态设置class样式

    ```less
    .no-required{
        :deep(label.ant-form-item-required::before){
          content: '';
        }
      }
    ```

![image-20230130140432871](https://s2.loli.net/2023/01/30/r9agqpRktoPFGCf.png)

- 默认设置required

![image-20230130140754549](https://s2.loli.net/2023/01/30/LNBRUOg38fwkAdy.png)

```js
 // 交易中心切换为否时，置空两个字段
    watch(
      () => organizationForm.isTradeCenter,
      nVal => {
        // isTradeCenter 1是 0否
        if (nVal != '1') {
          organizationFormRef.value.clearValidate(['complaintCall']);
        }
        organizationFormRules['complaintCall'][0].required = nVal === '1' ? true : false;
      },
    );
```



## 表单的嵌套数据的校验

> [参考嵌套结构的表单](https://2x.antdv.com/components/form-cn#components-form-demo-nested-form)

数据是嵌套结构：

![image-20221230163928506](https://s2.loli.net/2022/12/30/NUt1dbVGefBxOny.png)

校验规则里也要嵌套的写：

![image-20221230164011978](https://s2.loli.net/2022/12/30/Jb9CZU3XkNISj4V.png)

标签的`name`属性也要写成数组的形式:

![image-20221230164052664](https://s2.loli.net/2022/12/30/LetcRJ4SOnoEIpC.png)

## row col -- Grid栅格布局的使用

> [Grid栅格](https://2x.antdv.com/components/grid-cn)
>
> 布局的栅格化系统，我们是基于行（row）和列（col）来定义信息区块的外部框架，以保证页面的每个区域能够稳健地排布起来。下面简单介绍一下它的工作原理：
>
> - 通过`row`在水平方向建立一组`column`（简写 col）
> - 你的内容应当放置于`col`内，并且，只有`col`可以作为`row`的直接元素
> - 栅格系统中的列是指 1 到 24 的值来表示其跨越的范围。例如，三个等宽的列可以使用 `` 来创建
> - 如果一个`row`中的`col`总和超过 24，那么多余的`col`会作为一个整体另起一行排列（换行就设置每个col24即可）

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

## useForm的使用

![image-20221215140053040](https://f.pz.al/pzal/2022/12/15/6ec4850ed72ec.png)

```js
import { Form } from 'ant-design-vue';
setup() {
    const useForm = Form.useForm;
    
     const modelRef = reactive({
          local: '',
          address: '',
        });
        const modelRules = reactive({
          local: [
            {
              required: true,
              validator: validateLocal,
            },
          ],
          address: [
            {
              required: true,
              message: '请输入详细地址',
            },
          ],
        });
	const { resetFields, validate, validateInfos } = useForm(modelRef, modelRules);
    return {
        resetFields,
        validate,
        validateInfos
    }
}

```



## wrapperCol和labelCol属性的使用

> [Ant Design Vue (antdv.com)](https://2x.antdv.com/components/form-cn#API)

| 参数       | 说明                                                         | 类型                                                   |      |      |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------ | ---- | ---- |
| labelCol   | label 标签布局，同 `<Col>` 组件，设置 `span` `offset` 值，如 `{span: 3, offset: 12}` 或 `sm: {span: 3, offset: 12}` | [object](https://2x.antdv.com/components/grid-cn/#Col) |      |      |
| wrapperCol | 需要为输入控件设置布局样式时，使用该属性，用法同 labelCol    | [object](https://2x.antdv.com/components/grid-cn/#Col) |      |      |

### labelCol：控制label的宽度

`:labelCol="{ style: { minWidth: '100px' } }"`

![image-20230113180348482](https://s2.loli.net/2023/01/18/GEzwrAFLPTvuJlj.png)

![image-20230113180324658](https://s2.loli.net/2023/01/13/GE24Qim1n6gyDzY.png)

### wrapperCol：控制控件的宽度

`:wrapperCol="{ span: 17 }"`

![image-20230113180402798](https://s2.loli.net/2023/01/18/OYM1x6ctBRpjHvE.png)

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

## 选中默认项

### label-in-value

> 默认情况下 `onChange` 里只能拿到 value，如果需要拿到选中的节点文本 label，可以使用 `labelInValue` 属性。 选中项的 label 会被包装到 value 中传递给 `onChange` 等函数，此时 value 是一个对象。

这种方法可以设置value为一个对象，而不是字符串，即可设置默认值。

![image-20221216102730785](https://f.pz.al/pzal/2022/12/16/5bc1b44aff907.png)

### 填充默认值

```js
 // 下拉列表
  let optionsOne = ref([
    {
      label:'资源性资产',
      value:'1'
    },
    {
      label:'经营性资产',
      value:'2'
    },
    {
      label:'非经营性资产',
      value:'3'
    },
  ])
  //筛选第一个选项，取出value值
   const firstSelect = computed(()=>{
    const first = optionsOne.value[1]
    return first.value
  })
   // 表单的选项组件数据
  const chooseTypesModelRef = ref({
    typeOne:firstSelect.value,
    typeTwo:null,
    typeThree:null,
    typeFour:null,
    typeFive:null
  })
```

![image-20221216103133770](https://f.pz.al/pzal/2022/12/16/5ae59efb8fd29.png)

而且会在表单校验的输出，返回整个对象，方便取到label的值：

![image-20221216103917068](https://f.pz.al/pzal/2022/12/16/fe7fb38c7f390.png)

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

## 图片上传



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

# divider 分隔线

区隔内容的分割线。

```html
<!-- 分隔线 -->
<a-divider style="border-color: #afafaf" dashed></a-divider>
```

![image-20221215105840382](https://f.pz.al/pzal/2022/12/15/624938b69d558.png)

```html
<a-divider orientation="left">
    <span>同步信息</span>
</a-divider>
```

![image-20221215151330512](https://f.pz.al/pzal/2022/12/15/51af74130f70e.png)

# tabs 标签页

用于有流程步骤的页面，里面的每一块内容用`tab-pane`包裹。在tabs上用`v-model:activeKey="activeTab"`切换激活的标签。

![image-20221215101932027](https://f.pz.al/pzal/2022/12/15/be829f3f21d50.png)

![image-20221215102357524](https://f.pz.al/pzal/2022/12/15/a0bfdac29fe94.png)

## 禁用点击

有些需要完成当前页面的表单才能进入下一步，这时需要禁用，防止用户点击切换。

![image-20221216145803462](https://f.pz.al/pzal/2022/12/16/99dab99e1acc2.png)

# 手写原生table表格

## 套用antd的样式类名

套用antd的样式类名，添加到自己写的table标签上，主要有一下类名：

- ant-table 
- ant-table-default
- ant-table-scroll-position-left
- ant-table-bordered 
- ant-table-thead  
- ant-table-header-column
- ant-table-tbody

![image-20221216191044991](https://f.pz.al/pzal/2022/12/16/6d652931ab157.png)

还可以自己添加类名，对表格添加样式:

![image-20221216191857368](https://f.pz.al/pzal/2022/12/16/e3ab3dc55dcf5.png)

## 在原生表格中插入antd table

![image-20221216191751507](https://f.pz.al/pzal/2022/12/16/9d762565010e7.png)

在表格中插入一张图片：
![image-20221216192217889](https://f.pz.al/pzal/2022/12/16/ef42799c75644.png)

![image-20221216192242099](https://f.pz.al/pzal/2022/12/16/80ec2d785c5d9.png)

# 打包下载

> 路径入口：资产管理->详情页
> http://localhost:5001/globalApply/projectAbnormal?id=1603662498414870531&isGlobalApply=1



![image-20221216191955466](https://f.pz.al/pzal/2022/12/16/ca36fbb62af6a.png)

![image-20221216192017653](https://f.pz.al/pzal/2022/12/16/6df203c7c6fa7.png)

# cascader/select组件

| options | options 数据，如果设置则不需要手动构造 selectOption 节点 | array<{value, label, [disabled, key, title]}> | []   |
| ------- | -------------------------------------------------------- | --------------------------------------------- | ---- |
|         |                                                          |                                               |      |

组件的`options`参数，通常需要缓存到vuex中，后续打开页面，会直接从store中取出，减少请求次数：

![image-20221219151142169](https://f.pz.al/pzal/2022/12/19/e643d429773c1.png)

使用方法：

从外部文件引入方法

```js
import getSelectOptions from './composables/getSelectOptions'; // 获取下拉选项
...
setup(){
	 const { organizationList} = getSelectOptions();
    return {
        organizationList
    }
}
```

数据会涉及到操作vuex，存储到store

```js
//organizationList.js
export default function() {
    // 组织机构列表
  const organizationList = ref([]);
  /** 获取当前用户组织机构及其下属机构 */
  const getOrganization = async () => {
    const res = await store.dispatch('user/getUserOrganizationList');
    if (res && res.code === 200) {
        // 从store获取机构
      const _list = store.state.user.userOrganizationList || [];
      organizationList.value = _list;
      return res;
    } else {
      AMessage.error(res.message);
      return false;
    }
  };
    return {
        organizationList
    }
} 
```

dispatch派发的action: 

1. 会检查数据是否已经存在state中，如果没有，则请求接口获取数据
2. 使用了async await语法，返回的需要是一个promise

![image-20221219181534040](https://f.pz.al/pzal/2022/12/19/91e63632d4ff1.png)

![image-20221219182048832](https://f.pz.al/pzal/2022/12/19/9f0ff9d1da101.png)

# antd提示组件自定义icon

下面的组件都可以使用icon

![image-20221219185234871](https://f.pz.al/pzal/2022/12/19/c5a7ab0b7d824.png)

### Modal.method()

![image-20221219185750650](https://f.pz.al/pzal/2022/12/19/ceed7162e549f.png)

包括：

- `Modal.info`
- `Modal.success`
- `Modal.error`
- `Modal.warning`
- `Modal.confirm`

```js
import { createVNode } from 'vue';
import { ExclamationCircleOutlined } from '@ant-design/icons-vue';
import { message as AMessage, Modal as AModal } from 'ant-design-vue';
/** 清空附件
   * @param {number} ID 资产ID，新增时为空
   */
  const confirmEmptyFile = (ID) => {
    AModal.confirm({
      title: '清空提醒',
      content: '此操作将清空所有上传的文件。',
      icon: createVNode(ExclamationCircleOutlined),
      okText: '立即清空',
      cancelText: '暂不清空',
      onOk() {
        deleteFile(1, {
          assetId: ID,
          fileKeyList: fileList.value.map(c => c.fileKey),
        });
      },
      onCancel() {},
    });
  };
```

# message / Modal.method()的使用

```js
import { message as AMessage, Modal as AModal } from 'ant-design-vue';
import { ExclamationCircleOutlined } from '@ant-design/icons-vue';
import {createVNode } from 'vue';

AMessage.success(res.message);
AMessage.error(res.message);

/** 删除 */
  const handleDelete = (item) => {
    AModal.confirm({
      title: '删除警告',
      content: '注意：删除后将移除该角色，如果成员没有其他角色，则将无法正常登录使用。',
      icon: createVNode(ExclamationCircleOutlined),
      okText: '立即删除',
      cancelText: '暂不删除',
      onOk(){
        return new Promise((resolve, reject) => {
          const res = deleteRole(item, resolve, reject);
          return res;
        });
      },
      onCancel() {},
    });
  };
```



# loadsh库

- pickBy的使用
  ![image-20221219171337388](https://f.pz.al/pzal/2022/12/19/438ac2e369068.png)

# 图片/文件的预览

![image-20221219172622314](https://f.pz.al/pzal/2022/12/19/5b80dd3af0097.png)

# slot插槽的使用





# descriptions 描述列表

> [Descriptions](https://2x.antdv.com/components/descriptions-cn)

![image-20221227173124563](https://f.pz.al/pzal/2022/12/27/b9bec76a42214.png)

 ![image-20221227173313648](https://f.pz.al/pzal/2022/12/27/05e5773e8175a.png)

![image-20230112094829330](https://s2.loli.net/2023/01/12/J7GjvMngHzfoqit.png)

![image-20230112095053611](https://s2.loli.net/2023/01/12/KIZ9gEoezryuADF.png)

# table表格组件

> [Ant Design Vue (antdv.com)](https://2x.antdv.com/components/table-cn#API)

## 设置row-key

> 表格行 key 的取值，可以是字符串或一个函数

```html
 <a-table 
      :dataSource="dataSource" 
      :columns="columns" 
      :pagination="pagination"
      :loading="loading"
      bordered
      :row-key="
        (record, index) => {
          return record[tableRowKey] ? record[tableRowKey] : index;
        }
      "
    >
</a-table>
```



## column数据的构建

多个tab切换的场景，一个父页面需要包含多个table表格时，表头数据column应该如何构建：

![image-20221228141149149](https://s2.loli.net/2022/12/28/Vca6CoJp2D1qry4.png)

![image-20221228141512578](https://s2.loli.net/2022/12/28/6gkPlsZTA3Gi7bU.png)

![image-20221228141600828](https://s2.loli.net/2022/12/28/WLpKHGoj9TeS3au.png)

![image-20221228141655412](https://s2.loli.net/2022/12/28/fz8qJiDCShWoXen.png)

![image-20221228141807910](https://s2.loli.net/2022/12/28/ETN58hzoFASkCMO.png)

## column字段常用的属性

> [Column属性](https://2x.antdv.com/components/table-cn)

```js
let column = [
	{
      title: '操作', //列头显示文字
      dataIndex: 'opera',//列数据在数据项中对应的 key，支持 a.b.c 的嵌套写法
      width: 200,//列宽度
      align: 'center', //设置列内容的对齐方式
      fixed: "right",//列是否固定，可选 true(等效于 left) 'left' 'right'
      slots: { //使用 columns 时，可以通过该属性配置支持 slot 的属性，如 slots: { filterIcon: 'XXX'}
        customRender: 'opera',
      },
    },
]
```

还有其他的属性可以在官网查到；

## slots属性的解疑答惑

slots顾名思义的，是用在自定义插槽的时候，用于自定义显示。

在antd v1 v2 v3版本中，有很大的变化：

- v1: table组件的column中，含有slots和scopedSlots两个属性；
  ![image-20221228151309542](https://s2.loli.net/2022/12/28/K5Hq6QV1dFcm8zu.png)

- v2: 只剩下了slots属性了：

  ![image-20221228151402966](https://s2.loli.net/2022/12/28/gQHdNmjIT5qLcs8.png)

- v3: 直接废弃了column.slots属性，新增 `v-slot:bodyCell`、`v-slot:headerCell`作为代替。
  ![image-20221228151533714](https://s2.loli.net/2022/12/28/Uljk2QvPxJzINnC.png)

## v2版本：column.slots的使用演示

![image-20221228152036340](https://s2.loli.net/2022/12/28/PrWadEmfebR87uG.png)

![image-20221228152121685](https://s2.loli.net/2022/12/28/f1eAbhjoquQpLZW.png)

## v3版本: v-slot:bodyCell的使用演示

> [antd从v2升级到v3_叶子yeah的博客-CSDN博客_antd升级](https://blog.csdn.net/yeqiurong/article/details/124857027)
>
> [[ant-design-vue: Table\] column.slots报错_Cupid510的博客-CSDN博客](https://blog.csdn.net/weixin_44283432/article/details/126101249)
>
> [antd2 vue 升级到 antd3 时 table 报错处理_魂魄修罗的博客-CSDN博客_`column.slots` is deprecated. please use `v-slot:h](https://blog.csdn.net/LIN1475069547/article/details/124271064?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-2-124271064-blog-126101249.pc_relevant_landingrelevant&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-2-124271064-blog-126101249.pc_relevant_landingrelevant&utm_relevant_index=5)

![image-20221228151934352](https://s2.loli.net/2022/12/28/s3geqzKOv49uXWD.png)



# input-number组件



```html
<a-col :span="11">
    <a-row class="project-apl-opt-row" type="flex" justify="start" align="top">
        <a-col :span="16">
            <a-form-item :label="keysProps.minBidRange.name" name="minBidRange">
                <a-input-number
                                v-model:value="formData.minBidRange"
                                :formatter="value => `${value}`.replace(/\B(?=(\d{3})+(?!\d))/g, ',')"
                                :parser="value => value.replace(/\$\s?|(,*)/g, '')"
                                placeholder="请输入"
                                v-bind="keysProps.minBidRange"
                                />
            </a-form-item>
        </a-col>
        <a-col class="project-apl-opt-row_text" :span="8">
            <span>元（需是100的整数倍）</span>
        </a-col>
    </a-row>
</a-col>
```

# Space 间距

设置组件之间的间距。

## 何时使用

避免组件紧贴在一起，拉开统一的空间。

- 适合行内元素的水平间距。
- 可以设置各种水平对齐方式。

![image-20230109163223637](https://s2.loli.net/2023/01/09/RDVeozFTIdSBwx7.png)

![image-20230109163252664](https://s2.loli.net/2023/01/09/Da3zX21LGKHwCPd.png)

![image-20230109163322124](https://s2.loli.net/2023/01/09/aREZn1NctgrTyXS.png)

## API[#](https://2x.antdv.com/components/space-cn#API)

| 参数      | 说明     | 类型                                    | 默认值       | 版本  |
| :-------- | :------- | :-------------------------------------- | :----------- | :---- |
| align     | 对齐方式 | `start` | `end` |`center` |`baseline`   | -            | 1.6.5 |
| direction | 间距方向 | `vertical` | `horizontal`               | `horizontal` | 1.6.5 |
| size      | 间距大小 | `small` | `middle` | `large` | `number` | `small`      | 1.6.5 |

# Popconfirm 气泡确认框

点击元素，弹出气泡式的确认框。

## 何时使用

目标元素的操作需要用户进一步的确认时，在目标元素附近弹出浮层提示，询问用户。 和 'confirm' 弹出的全屏居中模态对话框相比，交互形式更轻量。

![image-20230109164153564](https://s2.loli.net/2023/01/09/ijrE3DQaPF2wmbO.png)

![image-20230109164202720](https://s2.loli.net/2023/01/09/isAuXHTM4kSKd63.png)
