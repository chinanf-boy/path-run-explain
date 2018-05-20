## index.js

``` js
// 被 cli.js 
	const { pathRun, IsTruePath} =  require('./index.js');

```

前面说了, path-run 做得事情有两个

1. 其他文件导入自身 的路径更改 `: step1`

2. 自身文件的`require`路径更改 `: step2`


---

- [pathRun](#pathrun)
- [step1](#step1)
- [step2](#step2)
- [IsTruePath](#istruepath)

---

### pathRun

'path-run/index.js` 只有一个文件

``` js
'use strict';
const nodepaths = require("nodepaths") // 
const relative = require('relative'); // 相对路径
const path = require('path')
const chalk = require('chalk') // 颜色

```

- [nodepaths](https://github.com/chinanf-boy/NodePath)

将 输入的文件或目录 下 能被 `require/from` 匹配的 文本输出 

``` js
return {
    file-absolute-name:[
        require-string,
        //...
    ]
}
```

- relative 

相对路径, 有个问题在于 同目录下, 不会

``` js
relative('src/1.js','src/2.js')
// 2.js 
// 没有 './' , 需要自己加上
```

``` js
async function pathRun(options){

	let { InPath, OutPath } = options

	InPath = IsTruePath(InPath) // 
	OutPath = IsTruePath(OutPath) // 验证两个输入 的 正确性, 不对直接抛出错误❌

	const files   = options.cwd

	let replaceMesaages = [] // 返回值

	let pathRunMap = await nodepaths(files) // 拿到 模块路径 描述

	let Ks = Object.keys(pathRunMap)

```

#### step1

``` js
// 1. 其他文件导入自身 的路径更改
	if(Ks && Ks.length) // <==== [] will too many for let i in []
	for( let i in Ks){ // 循环
		let replaceOptions = {}
		let fileAbs = pathRunMap[Ks[i]].map(f =>{
			try{
				return require.resolve(path.resolve(path.dirname(Ks[i]), f))
				// 对每个文件下, 导入的文件做 绝对路径处理

			}catch(err){
				return ""
				// 无法成功的, 直接 ""
				// 比如 webpack 的 @
			}
		})
		let Index = fileAbs.indexOf(InPath) // 对比 输入文件的绝对路径

		if(Index >= 0){ // 有 开始改

			let fileToInPath = pathRunMap[Ks[i]][Index] // file content require relative path : user
			// 需要匹配文本
			let fileToOutPath = relative(Ks[i], OutPath) // file content require relative path : Path-run
			// 获得 新的相对路径
			if(!fileToOutPath.startsWith('.')){
				fileToOutPath = './'+fileToOutPath // 记得加上 './'
			}
			if(fileToInPath === fileToOutPath) continue // 需要一样 就算了

			// replace-in-file 的规则
			replaceOptions = {
				files: Ks[i], // 哪个文件 , from 从 => to 到什么
				from: new RegExp(`( from)([\\s]+)(\\'|\\")+(`+fileToInPath+`)+((\\'|\\")+([^\\S;])?)`,'g'),
				to: ` from '${fileToOutPath}'`
			}
			replaceMesaages.push(replaceOptions) // 添加

			replaceOptions = {
				files: Ks[i],
				from: new RegExp(`(require\\()(\\'|\\")(`+fileToInPath+`)+((\\'|\\")([\\)])+)`,'g'),
				to: `require('${fileToOutPath}')`
			}
			replaceMesaages.push(replaceOptions)
		}
	}

```

#### step2

``` js
// 2. 自身文件的`require`路径更改
	if(Ks && Ks.length){
		let requireList = pathRunMap[InPath] // 拿到自身文件的导入列表
		let replaceOptions = {}
		let fileAbs = requireList.map(f =>{
			try{
				return require.resolve(path.resolve(path.dirname(InPath), f))
				// 同样 导入列表 变绝对路径
			}catch(err){
				return "" // and node module or @/index.vue :vue  == ""
				// 排除 @ 之类的正常导入无法导入的错误❌
			}
		})

		fileAbs.forEach((absPath,index) =>{
			if(!absPath || absPath.includes("node_modules/")){ // 在 node_modules 中的 就不用改了
				return
			}
			let oldRelativePath = requireList[index]

			let newRelativePath = relative(OutPath, absPath)

			if(!newRelativePath.startsWith('.')){
				newRelativePath = './'+newRelativePath // 加 './'
			}

			replaceOptions = { // 规则制定
				files: OutPath,
				from: new RegExp(`(require\\()(\\'|\\")(`+oldRelativePath+`)+((\\'|\\")([\\)])+)`,'g'),
				to: `require('${newRelativePath}')`
			}
			replaceMesaages.push(replaceOptions)

			replaceOptions = {
				files: OutPath,
				from: new RegExp(`( from)([\\s]+)(\\'|\\")+(`+oldRelativePath+`)+((\\'|\\")+([^\\S;])?)`,'g'),
				to: ` from '${newRelativePath}'`
			}
			replaceMesaages.push(replaceOptions)
		})

	}

	return replaceMesaages // 总的 替换规则
};

```

#### IsTruePath

``` js

// 错误❌的路径, 直接抛出错误
function IsTruePath(Abs){
	// Abs == absolute path
	try{

		return require.resolve(Abs)

	}catch(err){
		throw new TypeError(chalk.red('错误路径 ==>>> '+Abs+'\n'))
	}
};

module.exports = {
	IsTruePath, // 方便 测试与使用
	pathRun
}

```