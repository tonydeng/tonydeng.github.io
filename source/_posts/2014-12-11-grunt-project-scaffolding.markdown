---
layout: post
title: "Grunt项目脚手架"
date: 2014-12-11 14:04:00 +0800
published: true
comments: true
categories: [效率]
tags: [grunt,front-end,nodejs,engineered]
keywords: grunt, scaffold, front-end, nodejs, engineered, grunt-init
---
> 原文： [https://github.com/gruntjs/grunt-docs/blob/master/Project-Scaffolding.md](https://github.com/gruntjs/grunt-docs/blob/master/Project-Scaffolding.md)

# 1. grunt-init

grunt-init 是一个脚手架工具，用于自动创建项目。基于当前的环境和几个问题的答案创建一个完整的目录结构。确切的文件和内容取决于所选的模板，和模板提出的问题的答案。

注：本独立程序曾经作为“init”任务内置于 Grunt。关于该变化的更多信息查看 Grunt 从 0.3 升级到 0.4 指南。

<!-- more -->

# 2. 安装

为了使用 **grunt-init** ，你需要全局安装它。

```
npm install -g grunt-init
```

这将把将命令 `grunt-init` 放入系统路径中，从而可以从任意目录运行它。

注：你可能需要使用指令 sudo 或者以管理员身份运行命令解释程序来做到这一点。

# 3. 用法

* 使用 `grunt-init --help` 获取程序帮助和可用的模板列表。
* 使用 `grunt-init TEMPLATE` 基于一个可用的模板创建一个项目。
* 使用 `grunt-init /path/to/TEMPLATE` 基于一个任意位置的模板创建一个项目。

需要注意的是，大多数模板在当前目录中生成它们的文件，因此一定要先切换到一个新目录，如果不希望覆盖已有文件的话。

# 4. 安装模板

一旦模板被安装到 `~/.grunt-init/` 目录（在 Windows 上是 `(%USERPROFILE%\.grunt=init\）`，就可以通过 grunt-init 使用它们。使用你使用 git 拷贝一个模范到该目录中。例如，可以这样安装 [grunt-init-jquery](https://github.com/gruntjs/grunt-init-jquery) 模板：

```
git clone git@github.com:gruntjs/grunt-init-jquery.git ~/.grunt-init/jquery
```

注：如果你想以*“foobarbaz”*在本地使用模板，你可以在拷贝时指定`~/.grunt-init/foobarbaz`。grunt-init 将使用模板在目录 `~/.grunt-init/` 中的实际目录名。

一些 grunt-init 模板由官方维护：

* [grunt-init-commonjs](https://github.com/gruntjs/grunt-init-commonjs) - 创建一个 commonjs 模块，包括 Nodeunit 单元测试。（[生成的样本库](https://github.com/gruntjs/grunt-init-commonjs-sample/tree/generated) | [创建记录](https://github.com/gruntjs/grunt-init-commonjs-sample#project-creation-transcript)）
* [grunt-init-gruntfile](https://github.com/gruntjs/grunt-init-gruntfile) - 创建一个基本的 Gruntfile。（[生成的样本库](https://github.com/gruntjs/grunt-init-gruntfile-sample/tree/generated) | [创建记录](https://github.com/gruntjs/grunt-init-gruntfile-sample#project-creation-transcript)）
* [grunt-init-gruntplugin](https://github.com/gruntjs/grunt-init-gruntplugin) -创建一个 Grunt 插件，包含 Nodeunit 单元测试。（[生成的样本库](https://github.com/gruntjs/grunt-init-gruntplugin-sample/tree/generated) | [创建记录](https://github.com/gruntjs/grunt-init-gruntplugin-sample#project-creation-transcript)）
* [grunt-init-jquery](https://github.com/gruntjs/grunt-init-jquery) - 创建一个 jQuery 插件，包含 Nodeunit 单元测试。（[生成的样本库](https://github.com/gruntjs/grunt-init-jquery-sample/tree/generated) | [创建记录](https://github.com/gruntjs/grunt-init-jquery-sample#project-creation-transcript)）
* [grunt-init-node](https://github.com/gruntjs/grunt-init-node) - 创建一个 Node.js 模块，包含 Nodeunit 单元测试。（[生成的样本库](https://github.com/gruntjs/grunt-init-node-sample/tree/generated) | [创建记录](https://github.com/gruntjs/grunt-init-node-sample#project-creation-transcript)）


# 5. 自定义模板

你可以创建和使用自定义模板。你的模板必须遵循与上述模板同样的结构。

一个命名为 `my-template` 的模板要遵循下面的通用文件结构：

* my-template/template.js - 主模板文件.
* my-template/rename.json - 模板特定的命名规则，作为模板处理。
* my-template/root/ - 拷贝到目标位置的文件。

假设这些文件已经在 `/path/to/my-template`，可以用命令 `grunt-init /path/to/my-template` 处理该模板。多个不重名的模板可以存在于同一个目录项，就像内置模板一样。

> 没有在内置模板中找到多个不重名模板，疑问中

此外，如果将自定义模板放入 `~/.grunt-init/` 目录（Windows 上是 `%USERPROFILE%\.grunt-init\`），运行 `grunt-init my-template` 将自动被使用。

## 5.1 拷贝文件

当 init 模板运行时，只要一个模板使用方法 `init.filesToCopy` 和 `init.copyAndProcess`，目录 `root/` 中的所有文件将被复制到当前目录。

请注意，所以拷贝文件会被当作模板处理，模板 `{ % % }` 将在收集的 `props` 数据对象下处理，除非设置了选项 `noProcess`。看看 [jQuery 模板](https://github.com/gruntjs/grunt-init-jquery)例子。

## 5.2 重命名或不包括模板文件

`rename.json` 描述 `sourcepath` 到 `destpath` 的重命名映射。`sourcepath` 必须是 `root/` 文件夹下将被拷贝文件的路径，而 `destpath` 值可以包含 `{ % % }` 模板，描述目标路径是什么。

如果指定 `destpath` 为 `false`，该文件不会被复制。此外，`srcpath` 支持通配符。

# 6. 指定默认提示答案

没有初始化提示或者有一个硬编码的默认值，或者产看当前环境来尝试判断默认值。如果你想覆盖个别提示的默认值，你可以在选项文件 `~/.grunt-init/defaults.json`（OS X 或 Linux）或 `%USERPROFILE%\.grunt-init\defaults.json`（Window）中这么做。

例如，我的 `defaults.json` 文件看起来就像这样，因为我想用一个稍微不同的名称而不是默认名称，想要排除我的电子邮件地址，想自动指定一个作者网址。

```json
{
  "author_name": "\"Cowboy\" Ben Alman",
  "author_email": "none",
  "author_url": "http://benalman.com/"
}
```

注：在所有内置提示被文档化之前，你可以在[源代码](https://github.com/gruntjs/grunt-init/blob/master/tasks/init.js)中找到它们的名称和默认值。

# 7. 定义一个初始化模板

> 对照 ~/.grunt-init/node/template.js 理解。

## 7.1 exports.description

当用户运行 `grunt init` 或 `grunt-init` 来显示所有有效的初始化模板列表时，简短的模板描述随模板名称一起显示。

```javascript
exports.description = descriptionString;
```
## 7.2 exports.notes

如果指定了，该可选的扩展描述将在所有提示之前显示。这是一个很好的向用户提供一点点帮助解释命名约定的地方，哪些提示是必须或可选的，等等。

```javascript
exports.notes = notesString;
```

## 7.3 exports.warnOn

如果该可选（但是推荐）的通配符或通配符数组被（已存在的文件或目录）匹配，Grunt 将中止，并导致一条警告信息，用户可以使用 `--force` 覆盖该行为。这些初试化模板可能潜在的覆盖已存在文件时非常有用。

```javascript
exports.warnOn = wildcardPattern;
```
而最常见的值是 '*'，匹配所有文件或文件夹，使用的 [minimatch](https://github.com/isaacs/minimatch) 通配符语法允许很大的灵活性。例如：

```javascript
exports.warnOn = 'Gruntfile.js';    // Warn on a Gruntfile.js file.
exports.warnOn = '*.js';            // Warn on any .js file.
exports.warnOn = '*';               // Warn on any non-dotfile or non-dotdir.
exports.warnOn = '.*';              // Warn on any dotfile or dotdir.
exports.warnOn = '{.*,*}';          // Warn on any file or dir (dot or non-dot).
exports.warnOn = '!*/**';           // Warn on any file (ignoring dirs).
exports.warnOn = '*.{png,gif,jpg}'; // Warn on any image file.
//
// This is another way of writing the last example.
exports.warnOn = ['*.png', '*.gif', '*.jpg'];
```

## 7.4 exports.template

虽然 `exports` 属性定义在该函数之外，但是所有实际的初始化代码在该函数内指定。三个参数被传给该函数。参数 `grunt` 指向 grunt，包含了所有的 [grunt 方法和库](http://gruntjs.com/api/grunt)。参数 `init` 是一个包含了特定于该初始化模板的方法和属性的对象。参数 `done` 是一个函数，当初始化模板执行完成时必须调用该函数。

```javascript
exports.template = function(grunt, init, done) {
  // See the "Inside an init template" section.
};
```

# 8. 初始化模板内部
## 8.1 init.addLicenseFiles

添加适当命名的许可证文件到 files 对象。

```javascript
var files = {};
var licenses = ['MIT'];
init.addLicenseFiles(files, licenses);
// files === {'LICENSE-MIT': 'licenses/LICENSE-MIT'}
```

## 8.2 init.availableLicenses

返回有效许可证数组。

```javascript
var licenses = init.availableLicenses();
// licenses === [ 'Apache-2.0', 'GPL-2.0', 'MIT', 'MPL-2.0' ]
```

## 8.3 init.copy

复制一个文件，指定一个绝对或相对源路径和一个可选的相对目标路径，可以通过传入的回调函数处理复制的文件。
```javascript
init.copy(srcpath[, destpath], options)
```
## 8.4 init.copyAndProcess

遍历传入的对象中的所有文件，拷贝源文件到目标地址，并处理文件内容。
```javascript
init.copyAndProcess(files, props[, options])
```
## 8.5 init.defaults

`defaults.json` 中用户指定的默认初始值。
```javascript
init.defaults
```
## 8.6 init.destpath

绝对目标文件路径。
```javascript
init.destpath()
```

## 8.7 init.expand

与 [grunt.file.expand](https://github.com/gruntjs/grunt/wiki/grunt.file#wiki-grunt-file-expand) 一致。

返回一个不重复数组，含有匹配通配符模式的所有文件和目录路径。该方法接收逗号分割的痛佩服模式，或者通配符模式数组。匹配以 ! 开头的模式的路径将从返回值中排除。模式被顺序处理，所以包含和排除的顺序是重要的。

```javascript
init.expand([options, ] patterns)
```
## 8.8 init.filesToCopy

返回一个包含了待复制文件的对象，包含绝对源路径和相对目标路径，按照 `rename.json`（如果存在的话）中的规则重命名（或忽略）。

```javascript
var files = init.filesToCopy(props);
/* files === { '.gitignore': 'template/root/.gitignore',
  '.jshintrc': 'template/root/.jshintrc',
  'Gruntfile.js': 'template/root/Gruntfile.js',
  'README.md': 'template/root/README.md',
  'test/test_test.js': 'template/root/test/name_test.js' } */
```

## 8.9 init.getFile

返回单个任务文件路径。

```javascript
init.getFile(filepath[, ...])
```

## 8.10 init.getTemplates

返回含有所有有效模板的对象。
```javascript
init.getTemplates()
```
## 8.11 init.initSearchDirs

初始化目录来搜索初始化模板。这里的`模板`是指模板路径。会包括 `~/.grunt-init/` 和 `grunt-init` 中的核心初始化任务。
```javascript
init.initSearchDirs([filename])
```
## 8.12 init.process

启动进程，开始提示输入。

```javascript
init.process(options, prompts, done)

init.process({}, [
  // Prompt for these values
  init.prompt('name'),
  init.prompt('description'),
  init.prompt('version')
], function(err, props) {
  // All finished, do something with the properties
});
```

## 8.13 init.prompt

提示用户输入。

```javascript
init.prompt(name[, default])
```
## 8.14 init.prompts

含有所有提示的对象。

```javascript
var prompts = init.prompts;
```
## 8.15 init.readDefaults

从任务文件（如果存在的话）读取 JSON 默认值，并合并到一个数据对象中。

```javascript
init.readDefaults(filepath[, ...])
```
## 8.16 init.renames

模板的重命名规则。

```javascript
var renames = init.renames;
// renames === { 'test/name_test.js': 'test/\{\%= name \%\}_test.js' }
```

## 8.17 init.searchDirs

目录数组，用于在其中搜索模板。
```javascript
var dirs = init.searchDirs;
/* dirs === [ '/Users/shama/.grunt-init',
  '/usr/local/lib/node_modules/grunt-init/templates' ] */
```

## 8.18 init.srcpath

通过文件名搜索初始化模板路径，并返回一个绝对路径。

```javascript
init.srcpath(filepath[, ...])
```

## 8.19 init.userDir

返回用户模板路径的绝对路径。
```javascript
var dir = init.userDir();
// dir === '/Users/shama/.grunt-init'
```

## 8.20 init.writePackageJSON

保存一个 package.json 文件到目标目录。回调函数可以用来后置处理属性，或添加或移除或其他任何操作。
```javascript
init.writePackageJSON(filename, props[, callback])
```
# 9. 内置提示
## 9.1 author_email

Author's email address to use in the `package.json`. Will attempt to find a default value from the user's git config.

## 9.2 author_name

Author's full name to use in the `package.json` and copyright notices. Will attempt to find a default value from the user's git config.

## 9.3 author_url

A public URL to the author's website to use in the `package.json`.

## 9.4 bin

A relative path from the project root for a cli script.

## 9.5 bugs

A public URL to the project's issues tracker. Will default to the github issue tracker if the project has a github repository.

## 9.6 description

A description of the project. Used in the package.json and README files.

## 9.7 grunt_version

A valid semantic version range descriptor of Grunt the project requires.

## 9.8 homepage

A public URL to the project's home page. Will default to the github url if a github repository.

## 9.9 jquery_version

If a jQuery project, the version of jQuery the project requires. Must be a valid semantic version range descriptor.

## 9.10 licenses

The license(s) for the project. Multiple licenses are separated by spaces. The licenses built-in are: MIT, MPL-2.0, GPL-2.0, and Apache-2.0. Defaults to MIT. Add custom licenses with init.addLicenseFiles.
## 9.11 main

The primary entry point of the project. Defaults to the project name within the lib folder.

## 9.12 name

The name of the project. Will be used heavily throughout the project template. Defaults to the current working directory.

## 9.13 node_version

The version of Node.js the project requires. Must be a valid semantic version range descriptor.

## 9.14 npm_test

The command to run tests on your project. Defaults to grunt.

## 9.15 repository

Project's git repository. Defaults to a guess of a github url.
9.16 title ⬆

A human readable project name. Defaults to the actual project name altered to be more human readable.

## 9.17 version

The version of the project. Defaults to the first valid semantic version, 0.1.0.
