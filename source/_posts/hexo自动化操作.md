---
title: hexo自动化操作
date: 2016-03-30 19:42:54
categories: 其他
tags: hexo
---

上一篇把hexo的源文件托管到了github，但是每次写完后既要deploy，又要push上去，另外每次创建完新的文章后，要手动切换目录去打开，总觉得很麻烦有没有。好在hexo提供了一些监听动作的方法可以让我们把一些操作自动化。本文参考了这位的[文章](http://notes.xiamo.tk/categories/Hexo/)。

<!--more-->

## hexo新建文章后自动打开编辑器
hexo作者在github上回答了类似的[问题](https://github.com/hexojs/hexo/issues/1007)，在结合网上搜到的，步骤如下：

* 先在hexo根目录下scripts目录(没有就新建一个)创建一个JS文件，文件名可以任意取
* JS文件中复制代码并保存

这是hexo作者提供的方案，直接用vim打开编辑。

```javascript
var exec = require('child_process').exec;

// Hexo 2.x
hexo.on('new', function(path){
	exec('vi', [path]);
});

// Hexo 3
hexo.on('new', function(data){
  exec('vi', [data.path]);
});
```

当然要用其他应用也是可以的，做如下修改：

```javascript
//以hexo 3版本做例子，如果是2.x的版本，参考上述即可

//windows 
exec('start  "markdown编辑器绝对路径.exe" ' + data.path);

//mac
//mac的app安装目录为/Applications
exec('open -a "markdown编辑器绝对路径.app" ' + data.path);
```

保存后，再执行hexo new "new page"后就会自动打开编辑器了。

补充一句，这里用的是NodeJs的exec，作者在github回答用的是spawn，这两者的区别可以看[这里](https://segmentfault.com/a/1190000002913884)，不过不影响我们的使用，两者皆可。

## 自动备份hexo源文件
上述自动打开编辑器是利用了监听hexo new的事件，[hexo文档](https://hexo.io/api/events.html)上还有一些其他的事件可以利用，见下表，监听其中的deployAfter就可以在部署完后执行我们需要的操作了，比如提交、上传这次改动。

| 事件            | 说明                              |
| -------------- | ---------------------------------:|
| deployBefore   | 在部署完成前触发                     |
| deployAfter    | 在部署成功后触发                     |
| exit           | 在Hexo结束前触发                     |
| generateBefore | 在静态文件生成前触发                  |
| generateAfter  | 在静态文件生成后触发                  |
| new            | 在文章文件建立后触发, 该事件返回文章参数 |

首先安装一个插件，这个是用来js调用本地shell命令方法的，想了解的可自行百度

```shell
npm install --save shelljs
```
在hexo根目录下的scripts目录(没有创建一个)中新建一个js脚本文件，文件名任意，内容代码如下：

```javascript
require('shelljs/global');


try {
	hexo.on('deployAfter', function() {
		run();
	});
} catch (e) {
	console.log("产生了一个错误<(￣3￣)> !，错误详情为：" + e.toString());
}

function run() {
	if (!which('git')) {
		echo('Sorry, this script requires git');
		exit(1);
	} else {
		echo("======================Auto Backup Begin===========================");
		cd('hexo目录路径');    //此处修改为Hexo根目录路径
		if (exec('git add --all').code !== 0) {
			echo('Error: Git add failed');
			exit(1);
		}

		//这个commit的信息可以根据自己需要修改，比如像我这样加一个时间戳等等
		var commit_log = "From auto backup script's commit at " + new Date().toLocaleString();
		if (exec('git commit -m "' + commit_log + '"' ).code !== 0) {
			echo('Error: Git commit failed');
			exit(1);
		}

		//这里的git push命令可以改成git push origin(仓库名) branch_name(分支名)
		//如果目录下已经切换好分支了，那直接用git push也可以
		if (exec('git push').code !== 0) {
			echo('Error: Git push failed');
			exit(1);

		}
		echo("==================Auto Backup Complete============================")
	}
}
```
上述需要注意的是：

* 第18行你的hexo目录路径
* 第32行，可选仓库名和分支名

保存执行hexo d后者deploy后就会自动commit并且push了，是不是觉得方便不少，再也不用担心有没有备份了。

当然，这一段还可以修改成在hexo new之后立马就备份上去，只需要把触发事件deployAfter改成new就可以了，看个人需要修改就好。
