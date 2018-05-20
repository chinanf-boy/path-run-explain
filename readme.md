# Path-run

「 更改项目文件-被其他文件导入的所在位置 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain) << ⬅️ more explain
    
Explanation

> "version": "0.0.5"

[github source](https://github.com/chinanf-boy/Path-run)

[中文](./readme.md) | ~~[english](./readme.en.md)~~

---

作为 项目中的 成员 可能会被其他文件 `require`, 但是当想更改在项目中的位置

那么需要注意的有两点

1. 自身文件的`require`路径更改

2. 其他文件导入自身 的路径更改

所有就有了 `path-run`


---

本目录

- [package.json](#package-json)
- [cli.js 开始](#cli)
- [require](#require)
- [meow-cli](#meow-cli)
- [cli-main](#cli-main)

---


### package.json


``` js
	"main": "./index.js",
	"bin": "cli.js",

```

从 `cli.js` 开始吧

### cli


#### require

``` js
#!/usr/bin/env node
(async () => {
	"use strict";
	const meow = require("meow");
	const path = require("path");
	const chalk = require("chalk");
	const replace = require('replace-in-file');
	const { pathRun, IsTruePath} =  require('./index.js');

```

#### meow-cli

命令行定义

可以看出只有两个 `一进一出`

``` js
	const cli = meow(`
	Usage
	  $ path-run [input] [output]

	Options
		input  要更改的路径
		output 变成的路径

		Examples
        $ path-run './index' './lib/index'

  will change all process.cwd()/* files require Path 'index' => './lib/index'`);

const CWD = process.cwd()

if (!cli.input || cli.input.length !== 2) {
	console.log(cli.help);
	process.exit(1);
}

let In = cli.input[0];
let Out = cli.input[1];

let InPath = path.resolve(CWD, In) // 获得文件绝对值
let OutPath = path.resolve(CWD, Out)

```

#### cli-main

``` js
try{

	let replaceMesaages = await pathRun({InPath, OutPath, cwd:CWD}) // 主要 函数

	if(replaceMesaages.length) 
	for(let i in replaceMesaages){

		let changes = await replace(replaceMesaages[i]) // 文件替换工具

		changes.length > 0 && (console.log(chalk.green(changes),chalk.blue(" >>> Done")))
	}

	if(process.env.RUN_DEBUG){
		console.log(replaceMesaages)
	}

	console.log(chalk.yellow("All done"))

}catch(err){
	throw new Error("cli"+err)
}
})();

```

- [pathRun](./index.md)

返回 提供给 replace 使用的 数据

- replace

[replace-in-file 更改文件中文本的工具](https://github.com/adamreisnz/replace-in-file) 
