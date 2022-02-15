### 弹出对话框方法

#### confirm()

> 显示一个带有指定消息和确认及取消按钮的对话框。
>
> 如果访问者点击"确定"，此方法返回true，否则返回false。

```js
function myFunction(){
    var x;
    var r=confirm("按下按钮!");
    if (r==true){
        x="你按下了\"确定\"按钮!";
    }
    else{
        x="你按下了\"取消\"按钮!";
    }
    document.getElementById("demo").innerHTML=x;
}
```

#### prompt()

> 用于显示可提示用户进行输入的对话框。这个方法返回用户输入的字符串。

```js
var num =parseInt( prompt("请输入一个数") ) ; // number
console.log(num);
```

#### alert() 

> 用于显示带有一条指定消息和一个 **确认** 按钮的警告框。

```js
function myFunction(){
    alert("你好，我是一个警告框！");
}
```

### 打开窗口方法

#### window.open()

> 用于打开一个新的浏览器窗口或查找一个已命名的窗口。`window.open(URL,name,specs,replace)`

```
window.open("http://www.runoob.com","_blank","toolbar=yes, location=yes, directories=no, status=no, menubar=yes, scrollbars=yes, resizable=no, copyhistory=yes, width=400, height=400")
```

#### close()

> 关闭浏览器窗口。

```js
function openWin(){
	myWindow=window.open("","","width=200,height=100");
	myWindow.document.write("<p>这是'我的窗口'</p>");
}
function closeWin(){
	myWindow.close();
}
```

### 打印 window.print()



### 定时器

#### setTimeout()

> setTimeout() 方法用于在指定的毫秒数后调用函数或计算表达式。

```js
setTimeout(function(){ alert("Hello"); }, 3000);
```

#### setInterval()

> setInterval() 方法可按照指定的周期（以毫秒计）来调用函数或计算表达式。

```js
setInterval(function(){ alert("Hello"); }, 3000);
```



#### clearTimeout()

> clearTimeout() 方法可取消由 setTimeout() 方法设置的定时操作。
>
> clearTimeout() 方法的参数必须是由 setTimeout() 返回的 ID 值。
>
> **注意:** 要使用 clearTimeout() 方法, 在创建执行定时操作时要使用全局变量：

```js
<script>
var c = 0;
var t;
var timer_is_on = 0;

function timedCount() {
    document.getElementById("txt").value = c;
    c = c + 1;
    t = setTimeout(function(){ timedCount() }, 1000);
}

function startCount() {
    if (!timer_is_on) {
        timer_is_on = 1;
        timedCount();
    }
}

function stopCount() {
    clearTimeout(t);
    timer_is_on = 0;
}
</script>
```

#### clearInterval()

> clearInterval() 方法可取消由 setInterval() 函数设定的定时执行操作。
>
> clearInterval() 方法的参数必须是由 setInterval() 返回的 ID 值。
>
> **注意:** 要使用 clearInterval() 方法, 在创建执行定时操作时要使用全局变量：

```js
<script>
var myVar = setInterval(function(){ setColor() }, 300);
 
function setColor() {
  var x = document.body;
  x.style.backgroundColor = x.style.backgroundColor == "yellow" ? "pink" : "yellow";
}
 
function stopColor() {
  clearInterval(myVar);
}
</script>
```

