![jsinspect](http://danielstjules.com/github/jsinspect-logo.png)

JavaScript
需要 Node.js 6.0+, ES6, JSX ， Flow. 


Note: 这个项目？？ 不懂 the project has been mostly
rewritten for the 0.10 release and saw several breaking changes.

[![Build Status](https://travis-ci.org/danielstjules/jsinspect.svg?branch=master)](https://travis-ci.org/danielstjules/jsinspect)

* [介绍](#overview)
* [安装](#installation)
* [使用](#usage)
* [整合](#integration)
* [报告？？](#reporters)

## Overview
我们都需要去处理code smell,复制的代码通常是源头-？？ 这种搜索很适合做个好用的CLI 工具

虽有现成的工具，但对一些代码不好用，或不支持JS 生态：ES6 ... 

可以设置最小分析的门槛，找出有相似结构的代码 默认下 它搜索nodes 的对应标识和文字，但可以关掉

-标识包括变量名，方法，属性等

-文字包括string 串，数字等

-可以在很多文件夹里找 (只会去找.js 的文件

-支持常见的boilerplate and repeated logic

![截图](https://cloud.githubusercontent.com/assets/817212/24126139/bd151a34-0da2-11e7-94a8-9742279c8566.png)

## Installation

用 `npm` :

``` bash
npm install -g jsinspect
```

## Usage

```
Usage: jsinspect [options] <路径 ...>


Detect copy-pasted and structurally similar JavaScript code
Example use: jsinspect -I -L -t 20 --ignore "test" ./path/to/src


Options:

  -h, --help                         output usage information
  -V, --version                      版本号
  -t, --threshold <number>           nodes数 (default: 30)
  -m, --min-instances <number>       min instances for a match (default: 2)
  -c, --config [config]              path to config file (default: .jsinspectrc)
  -r, --reporter [default|json|pmd]  specify the reporter to use
  -I, --no-identifiers               do not match identifiers
  -L, --no-literals                  do not match literals
  -C, --no-color                     关颜色
  --ignore <pattern>                 ignore paths matching a regex
  --truncate <number>                length to truncate lines (default: 100, off: 0)
  --debug                            debug信息
```

If a `.jsinspectrc` file is located in the project directory, its values will
be used in place of the defaults listed above. For example:

``` javascript
{
  "threshold":     30,
  "identifiers":   true,
  "literals":      true,
  "color":         true,
  "minInstances":  2,
  "ignore":        "test|spec|mock",
  "reporter":      "json",
  "truncate":      100,
}
```
第一次用时，最好用这个选项，lib/src 路径 不要用别的！

```
jsinspect -t 50 --ignore "test" ./path/to/src
```
可以降低门槛值！找出更多东西  用`-I` 忽视标识 and ignoring literals with `-L`. 

## Integration


It's simple to run jsinspect on your library source as part of a build
process. It will exit with an error code of 0 when no matches are found,
resulting in a passing step, and a positive error code corresponding to its
failure. For example, with Travis CI, you could add the following entries
to your `.travis.yml`:

``` yaml
before_script:
  - "npm install -g jsinspect"

script:
  - "jsinspect ./path/to/src"
```
例子中用的门槛是30，高点也行

To have jsinspect run with each job, but not block or fail the build, you can
use something like the following:

``` yaml
script:
  - "jsinspect ./path/to/src || true"
```

## Reporters

JSON and PMD CPD-style XML reporters are
available. Note that in the JSON example below, indentation and formatting
has been applied. Furthermore, the id property available in these reporters is
useful for parsing by automatic scripts to determine whether or not duplicate
code has changed between builds.

#### JSON

``` json
[{
  "id":"6ceb36d5891732db3835c4954d48d1b90368a475",
  "instances":[
    {
      "path":"spec/fixtures/intersection.js",
      "lines":[1,5],
      "code":"function intersectionA(array1, array2) {\n  array1.filter(function(n) {\n    return array2.indexOf(n) != -1;\n  });\n}"
    },
    {
      "path":"spec/fixtures/intersection.js",
      "lines":[7,11],
      "code":"function intersectionB(arrayA, arrayB) {\n  arrayA.filter(function(n) {\n    return arrayB.indexOf(n) != -1;\n  });\n}"
    }
  ]
}]
```

#### PMD CPD XML

``` xml
<?xml version="1.0" encoding="utf-8"?>
<pmd-cpd>
<duplication lines="10" id="6ceb36d5891732db3835c4954d48d1b90368a475">
<file path="/jsinspect/spec/fixtures/intersection.js" line="1"/>
<file path="/jsinspect/spec/fixtures/intersection.js" line="7"/>
<codefragment>
spec/fixtures/intersection.js:1,5
function intersectionA(array1, array2) {
  array1.filter(function(n) {
    return array2.indexOf(n) != -1;
  });
}

spec/fixtures/intersection.js:7,11
function intersectionB(arrayA, arrayB) {
  arrayA.filter(function(n) {
    return arrayB.indexOf(n) != -1;
  });
}
</codefragment>
</duplication>
</pmd-cpd>
```
