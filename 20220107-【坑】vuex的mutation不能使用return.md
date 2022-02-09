# 20220107-vuex的mutation不能使用return

mutation里的一个方法

```js
//mutation里的一个方法	
getGlobalTabBrowserByName(state,name) {
		if(state.GlobalTabBrowserIdObj[name]) {
			console.log("要取出的name id:",name,state.GlobalTabBrowserIdObj[name].id)
			return name,state.GlobalTabBrowserIdObj[name].id
		}
		return null
	}
```

在外面获取

```js
let id = this.getGlobalTabBrowserByName(this.fromTabName)
```

结果是 undefind 

只能使用this.$store.state.xxx去获取