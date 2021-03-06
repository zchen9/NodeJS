##NodeJS 代码组织和部署

####模块路径解析规则

```require``` 函数支持斜杠 ```/``` 或盘符 ```C:``` 开头的绝对路径，也支持 ```./``` 开头的相对路径。但这两种路径在模块之间建立了强耦合关系，一旦某个模块文件的存放位置需要变更，使用该模块的其他模块的代码也需要跟着调整。

因此，```require``` 函数支持第三种形式的路径。

- 1.内置模块
 
    如果传递给 ```require``` 函数的时 ```NodeJS``` 内置模块名称，不做路径解析，直接返回内部模块的导出对象，例如 ```require('fs')```;
    
- 2.node_modules 目录

	```NodeJS``` 定义了一个特殊的 ```Node_modules``` 目录用于存放模块。例如某个模块的绝对路径是 ```/home/user/hello.js```，在该模块中使用 ```require('foo/bar')``` 方式加载模块时，则 ```NodeJS``` 依次尝试使用以下路径。
	
		/home/user/node_modules/foo/bar
		/home/node_modules/foo/bar
		/node_modules/foo/bar
		
- 3.NODE_PATH环境变量

	与 ```PATH``` 环境变量类似，```NodeJS``` 允许通过 ```NODE_PATH``` 环境变量来制定额外的模块搜索路径。```NODE_PATH```环境变量中包含一到多个目录路径，路径之间在Linux下使用 ```:``` 分隔，在Windows下使用 ```;``` 分隔。例如定义了以下 ```NODE_PATH``` 环境变量：
		
		NODE_PATH=/home/user/lib:/home/lib
	
	当使用 ```require('foo/bar')``` 的方式加载模块时，则 ```NodeJS``` 依次尝试以下路径。
	
		/home/user/lib/foo/bar
		/home/lib/foo/bar
		
####包(package)

JS模块的基本单位是单个JS文件，但复杂些的模块往往由多个子模块组成。为了便于管理和使用，我们可以把由多个子模块组成的大模块称作包，并把所有子模块放在同一个目录里。

在组成一个包的所有子模块中，需要一个入口模块，入口模块的导出对象被作为保的导出对象。

	- /home/user/lib/
		- cat/
			head.js
			body.js
			main.js
			
其中cat目录定义了一个包，其中包含了3个子模块。main.js作为入口模块，其内容如下：

	var head = require('./head);
	var body = require('./body);
	
	exports.create = function(name) {
	  return {
	  	name: name,
	  	head: head.create(),
	  	body: body.create()
	  };
	};
	
#####index.js 

当模块的文件名是index.js，加载模块时可以使用模块所在目录的路径代替模块文件路径。

	var cat = require('/home/user/lib/cat');
	var cat = require('/homr/user/lib/cat/index');
	
以上两条语句等价。

这样处理后，就只需要把包目录路径传递给 ```require``` 函数，感觉上整个目录被当作单个模块使用，更有整体感。

package.json

如果想自定义入口模块的文件名和存放位置，就需要在包目录下包含一个 ```package.json``` 文件，并在其中指定入口模块的路径。

	- /home/user/lib/
		- cat/
			+ doc/
			- lib/
				head.js
				body.js
				main.js
			+ tests/
			package.json
			
其中 ```package.json``` 内容如下。

	{
		'name': 'cat',
		'main': './lib/main.js'
	}
	
如此一来，就同样可以使用 ```require('/home/user/lib/cat')``` 的方式加载模块。```NodeJS``` 会根据包目录下的 ```package.json``` 找到入口模块所在位置。

#####工程目录

一个标准的工程目录如下：

	- /home/user/workspace/node-echo/ 	# 工程目录
		- bin/						   	# 存放命令行相关代码
			node-echo					
		+ doc/							# 存放文档
		- lib/							# 存放API相关代码
			echo.js
		- node_modules/					# 存放三方包
			+ argv/
		+ tests/						# 存放测试用例
		package.json					# 元数据文件
		README.md						# 说明文件
		
其中部分文件内容如下：

	/* bin/node-echo */
	var argv = require('argv'),
		echo = require('../lib/echo');
	console.log(echo(argv.join(' ')));
	
	/* lib/echo.js */
	module.exports = function(message) {
		return message;
	}
	
	/* package.json */
	{
		'name': 'node-echo',
		'main': './lib/echo.js'
	}
	
以上例子中分类存放了不同的类型的文件，并通过 ```node_modules``` 目录直接使用三方包名加载模块。此外，定义了 ```package.json``` 之后， ```node-echo``` 目录也可以当做一个包来使用。

