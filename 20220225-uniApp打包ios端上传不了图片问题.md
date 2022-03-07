

# 20220225-uniApp打包ios端上传不了图片问题

## uni.uploadFile的formData参数导致

ios端formData不能含有file参数，把 `uni.chooseImage`获取到的图片地址，传到`filePath`中,formData看后端需要什么，反正不能有file参数，安卓机没有影响，可能苹果系统限制的原因

​	完整代码

```js
uni.chooseImage({
					count: 1,
					sizeType: ['original', 'compressed'],
					sourceType: ['album', 'camera'],
					success: res => {
						this.$Request('Common.imageUpload', {
							file: res.tempFilePaths[0],//该字段写了file不影响，在后面统一换成其他名字
							name: 'file'
						}).then(res => {
							this[key] = res.data.fileurl
						})
					}
				})
```



```js
export const UploadFile = ({
	url,
	file,
	data,
	header,
	name = 'file'
}) => {
	return new Promise((resolve, reject) => {
    let {file,name} = data,
        img_path = '',
        _data = {}
    if(file) {
      img_path = JSON.parse(JSON.stringify(file)) 
      delete data.file
    }
    _data['img_path'] = img_path //替代原来的file参数
    _data['name'] = name //_data的字段顺序也是有讲究的，不然后台会判断错误
		uni.uploadFile({
			url: url,
			filePath: img_path,
			name,
			header,
			formData: _data,
			dataType: 'json',
			success: (res) => {
				if (typeof res.data != "object") {
					res.data = JSON.parse(res.data)
				}
				resolve(res)
			},
			fail: err => {
				console.log('Upload fail', err)
				reject(err)
			}
		})
	})
}
```

