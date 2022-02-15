# 如何修改scroll滚动条样式

效果图：

![img](https://i.loli.net/2021/03/21/W4VA6wgHQYCbfN2.png)

1.给需要设置滚动条的容器加上class:topnav_box   



2.接下来是css.,

```css
.topnav_box::-webkit-scrollbar   //滚动条整体部分

{  
  width: 5px;  

  height:10px;   

  background-color:#b5b1b1;

}  
.topnav_box::-webkit-scrollbar-track    //scroll轨道背景
{  
  -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);  
  border-radius: 10px; 
   background-color:black;   
  
}

.topnav_box::-webkit-scrollbar-thumb  滚动条中能上下移动的小块
{  
  border-radius: 10px;  
  -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);  
  background-color:#b5b1b1;

} 
```



3.自定义滚动条

滚动条组成:

- ::-webkit-scrollbar 滚动条整体部分
- ::-webkit-scrollbar-thumb 滚动条里面的小方块，能向上向下移动（或往左往右移动，取决于是垂直滚动条还是水平滚动条）
- ::-webkit-scrollbar-track 滚动条的轨道（里面装有Thumb）
- ::-webkit-scrollbar-button 滚动条的轨道的两端按钮，允许通过点击微调小方块的位置。
- ::-webkit-scrollbar-track-piece 内层轨道，滚动条中间部分（除去）
- ::-webkit-scrollbar-corner 边角，即两个滚动条的交汇处
- ::-webkit-resizer 两个滚动条的交汇处上用于通过拖动调整元素大小的小控件



## 我自己的样式

代码：

```css
  // 侧边滚轮
      .CodeMirror-vscrollbar::-webkit-scrollbar    //滚动条整体部分
      {
        width: 5px;
        height: 10px;
        background-color: #b5b1b1;
      }
      .CodeMirror-vscrollbar::-webkit-scrollbar-track       //scroll轨道背景
      {
        -webkit-box-shadow: inset 0 0 6px rgba(32, 16, 53, 0.664);
        box-shadow: inset 0 0 6px rgba(32, 16, 53, 0.664);
        border-radius: 10px;
        background-color: #21242b;
      }

      .CodeMirror-vscrollbar::-webkit-scrollbar-thumb   //滚动条中能上下移动的小块
      {
        border-radius: 10px;
        -webkit-box-shadow: inset 0 0 6px rgba(12, 147, 226, 0.596);
        box-shadow: inset 0 0 6px rgba(12, 147, 226, 0.596);
        background-color: #b5b1b1a6;
      }
      // 底部滚轮
      .CodeMirror-hscrollbar::-webkit-scrollbar    //滚动条整体部分
      {
        height: 8px;
        background-color: #b5b1b1;
      }
      .CodeMirror-hscrollbar::-webkit-scrollbar-track       //scroll轨道背景
      {
        -webkit-box-shadow: inset 0 0 6px rgba(32, 16, 53, 0.664);
        box-shadow: inset 0 0 6px rgba(32, 16, 53, 0.664);
        border-radius: 10px;
        background-color: #21242b;
      }

      .CodeMirror-hscrollbar::-webkit-scrollbar-thumb   //滚动条中能上下移动的小块
      {
        border-radius: 10px;
        -webkit-box-shadow: inset 0 0 6px rgba(12, 147, 226, 0.596);
        box-shadow: inset 0 0 6px rgba(12, 147, 226, 0.596);
        background-color: #b5b1b1a6;
      }
```

效果：

![image-20210321224551455](https://i.loli.net/2021/03/21/bdZGTVUsa9rtFHN.png)