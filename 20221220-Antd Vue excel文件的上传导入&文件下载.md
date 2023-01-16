

# 20221220-Antd Vue excel文件的上传导入&文件下载

## 文件下载

```html
<a-button type="link" style="padding: 0">
    <a title="下载机构导入模版" :href="organizationTempUrl" download="机构导入模板.xlsx">
        下载机构导入模版
    </a>
</a-button>

<a-button type="link" style="padding: 0">
    <a title="下载机构成员导入模版" :href="memberTempUrl" download="成员导入模板.xlsx">
        下载机构成员导入模版
    </a>
</a-button>
```

也可以使用JS，动态创建a标签来模拟点击下载。

### a标签参数

- `href` : http:xxx.com/xxx.xlsx  (可以直接打开的文件的地址)
- `download`: 下载文件的名称，如果不写，会根据地址截取后面的字符作为文件名

![image-20221220141210142](https://f.pz.al/pzal/2022/12/20/0288442a5c3da.png)

![image-20221220141225870](https://f.pz.al/pzal/2022/12/20/7e821ad73b85e.png)

## 文件的上传导入

```html
<a-upload
          v-model:file-list="fileList"
          name="file"
          accept=".xls, .xlsx"
          :headers="uploadHeaders"
          :action="uploadOrganizationAPI"
          :data="uploadData"
          :multiple="false"
          :show-upload-list="false"
          @change="info => handleUploadChange(info, 'organization')"
          >
    <a-button
              :loading="uploadOrganizationLoading"
              :disabled="uploadOrganizationLoading || uploadMemberLoading"
              >
        导入机构
    </a-button>
</a-upload>
```

![image-20230111114613516](https://s2.loli.net/2023/01/11/3DArfyFh98aV4IJ.png)

### action

其中`action`的参数填入后台上传的接口，比如：http://xxx.com/user/importExcel

### change

`change`参数是一个回调函数，在文件上传期间、完成都会触发。

### status

`status`有`uploading`,`done`, `error`三个状态。可以根据`status`的状态进行逻辑处理。

![image-20230109181744511](https://s2.loli.net/2023/01/09/fp5PT3DjUArY8BC.png)

### fileList

已经上传的文件列表（受控）

使用v-model来实时获取上传成功的文件列表：`v-model:file-list="fileList"`。

fileList需要用ref创建。

```js
const fileList = ref([])
```

![image-20230109185759050](https://s2.loli.net/2023/01/09/Jxu3MectNndZ528.png)

上传成功的文件列表，有时候需要用表格展示，这个时候就可以用到了。

![image-20230109190549847](https://s2.loli.net/2023/01/09/GfQHK9sicYy37oS.png)

![image-20230109190640182](https://s2.loli.net/2023/01/09/b4OChXcyJ7Aemgi.png)

![image-20230109190652172](https://s2.loli.net/2023/01/09/3zsCo614vUZ9gpj.png)

## 不使用action，手动上传

手动上传文件，传给接口的数据是用`new FormData()`创建的。可以在里面添加后台约定的参数：

```js
const formData = new FormData();
const last = fileList.value.pop(); // 文件列表中的最后一个
formData.append('file', last?.originFileObj);//添加参数
formData.append('execute', true);//后台约定的参数
```

![image-20230109185835522](https://s2.loli.net/2023/01/09/FzstlpERuTcUiMe.png)

![image-20230109185821671](https://s2.loli.net/2023/01/09/o3S7lVvX8HB4kdx.png)

调用接口：

![image-20230109190502700](https://s2.loli.net/2023/01/09/kLAQyVvle1wdBES.png)