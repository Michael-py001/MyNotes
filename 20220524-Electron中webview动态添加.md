# 20220524-Electron中webview动态添加

> [【踩坑记】在electron中使用webview加载第三方登录页面，若页面重定向的网址非我们想要的，如何拉起浏览器去加载？_Wonder233的博客-CSDN博客_electron重定向](https://blog.csdn.net/Wonder233/article/details/109316152?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-4-109316152-null-null.pc_agg_new_rank&utm_term=electron+iframe+webview&spm=1000.2123.3001.4430)

思路：不写静态webview，通过js动态添加，因为动态切换src会报错