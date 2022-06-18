# 20220606-前端上传文件校验md5与秒传

相关文章

> [element ui 文件上传 + MD5 + axios_Holly1945的博客-CSDN博客](https://blog.csdn.net/Holly1945/article/details/121080257?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-121080257-blog-106280009.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-121080257-blog-106280009.pc_relevant_paycolumn_v3&utm_relevant_index=1)
>
> [基于vue框架下使用Element-UI获取文件MD5值并上传_哔咔哔咔~的博客-CSDN博客_vue 获取文件md5](https://blog.csdn.net/weixin_45493798/article/details/106280009?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-106280009-null-null.pc_agg_new_rank&utm_term=elementui+获取上传文件md5&spm=1000.2123.3001.4430)
>
> [vue-simple-uploader + spark-md5 大文件分片断点续传（本篇只做分片上传）_seatownzhang的博客-CSDN博客_sparkmd5 分片](https://blog.csdn.net/qq_33462132/article/details/108474351)
>
> [基于vue-simple-uploader封装文件分片上传、秒传及断点续传的全局上传插件 - 夏大师 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiahj/p/vue-simple-uploader.html)
>
> [Vue2.0结合webuploader实现文件分片上传 - 夏大师 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiahj/p/8529545.html)
>
> [vue项目实现分片上传及断点续传_我是槑槑的博客-CSDN博客_vue断点续传插件](https://blog.csdn.net/weixin_42132044/article/details/120850327)
>
> [element ui 文件上传 + MD5 + axios_Holly1945的博客-CSDN博客](https://blog.csdn.net/Holly1945/article/details/121080257?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-121080257-blog-106280009.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-121080257-blog-106280009.pc_relevant_paycolumn_v3&utm_relevant_index=1)

优秀插件

> [GitHub - simple-uploader/vue-uploader: A Vue.js upload component powered by simple-uploader.js](https://github.com/simple-uploader/vue-uploader)

## 整体流程

目前采用的上传是整个文件上传，没有分片，但是md5计算是有分片计算的。

整个流程如下：

1. 选择文件
2. 计算md5
3. 与后台比对，如果后台存在，则直接使用返回图片路径，不存在，继续上传

## md5的计算

使用的第三方插件`spark-md5`

> [GitHub - satazor/js-spark-md5: Lightning fast normal and incremental md5 for javascript](https://github.com/satazor/js-spark-md5)

项目使用el-upload上传组件，在`on-change`里监听文件改动，得到的参数file包含raw属性。

```html
<el-upload class="upload-demo" ref="elUpload" :disabled="disabled" :auto-upload="false" :show-file-list="false" drag multiple accept="image/*" :limit="limit" :on-change="onChange">
```

md5计算，根据`spark-md5`的文档进行了修改

`chunkSize`是分割文件每个分片的大小，`5 * 1024 * 1024`就是5m。

```js
import SparkMD5 from 'spark-md5'
export const computedMD5 = (file) => {
  return new Promise((resolve, reject) =>{
    let blobSlice = File.prototype.slice || File.prototype.mozSlice || File.prototype.webkitSlice,
    chunkSize = 5 * 1024 * 1024,                             // Read in chunks of 2MB
    chunks = Math.ceil(file.size / chunkSize),
    currentChunk = 0,
    spark = new SparkMD5.ArrayBuffer(),
    fileReader = new FileReader(),
    time = new Date().getTime(),
    md5 = '';
    loadNext();
    let that = this
    fileReader.onload = function (e) {
        console.log('read chunk nr', currentChunk + 1, 'of', chunks);
        spark.append(e.target.result);                   // Append array buffer
        currentChunk++;

        if (currentChunk < chunks) {
            loadNext();
        } else {
            console.log('finished loading');
            console.info('computed hash', spark.end());  // Compute hash
            file.raw.md5 = spark.end()
            file.md5 = spark.end()
            md5 = spark.end()
            console.log(`MD5计算完毕：${file.name} \nMD5：${md5} \n分片：${chunks} 大小:${(file.size/1024).toFixed(2)}kb 用时：${new Date().getTime() - time} ms`);
            resolve(file)
        }
    };

    fileReader.onerror = function () {
        console.warn('oops, something went wrong.');
        reject(false)
    };

    function loadNext() {
        let start = currentChunk * chunkSize,
            end = ((start + chunkSize) >= file.size) ? file.size : start + chunkSize;
        fileReader.readAsArrayBuffer(blobSlice.call(file.raw, start, end));
    }
  })
}
```

## 使用

```js
	async onChange(file) {
        console.log("file:",file);

        if (this.fileList.length > this.limit) {
            return false
        }

        const isImage = ['image/jpg', 'image/jpeg', 'image/png', 'image/gif'].includes(file.raw.type)
        const isLt = file.size / 1024 / 1024 < this.maxSize

        if (!isImage) {
            return this.$showToast('格式错误')
        }

        if (!isLt) {
            return this.$showToast('图片太大')
        }
        let resultFile = await this.$Tool.computedMD5(file) //计算md5

        let flag = await this.$Tool.compareMD5(resultFile.md5) //与后台比对

        if(!flag) { //不存在，则上传
            console.log("不存在，继续上传");
            // this.uploadFile(resultFile)
        }
        else {
            console.log("存在，秒传");
            ...
        }
   },
```

## 文件处理相关方法

```js

/**
 * base64转File
 * @param { string } dataurl
 * @param { string } filename
 */
export const dataURLtoFile = (dataurl, filename) => { // 将base64转换为文件
  var arr = dataurl.split(',')
  var mime = arr[0].match(/:(.*?);/)[1]
  var bstr = atob(arr[1])
  var n = bstr.length
  var u8arr = new Uint8Array(n)
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n)
  }
  return new File([u8arr], filename, {
    type: mime
  })
}

/**
 * 压缩图片
 * @param { File } file
 */
import lrz from 'lrz'
export const compressImage = async (file) => {
  const lrz_file = await lrz(file)
  // 压缩后将base64转file
  return dataURLtoFile(lrz_file.base64, lrz_file.origin.name)
}
```

