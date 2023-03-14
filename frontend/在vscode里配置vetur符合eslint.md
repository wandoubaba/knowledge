# 在vscode里配置vetur符合eslint

## 默认格式化程序

| 配置项 | 值 |
|---|---|
|vetur.format.defaultFormatter.html|prettier|
|vetur.format.defaultFormatter.js|vscode-typescript|
|vetur.format.defaultFormatter.css|prettier|
|vetur.format.defaultFormatter.less|prettier|
|vetur.format.defaultFormatter.scss|prettier|
|vetur.format.defaultFormatter.postcss|prettier|
|vetur.format.defaultFormatter.sass|sass-formatter|

## js函数名与括号之间添加一个空格

| 配置项 | 值 |
|---|---|
|javascript.format.insertSpaceBeforeFunctionParenthesis|true|

> 这个选项只在`vetur.format.defaultFormatter.js`值为`vscode-typescript`时有效。

## 配置缩进

| 配置项 | 值 | 说明 |
|---|---|---|
|vetur.format.scriptInitialIndent|false|&lt;script&gt;内代码初始缩进为0|
|vetur.format.styleInitialIndent|false|&lt;style&gt;内代码初始缩进为0|
|vetur.format.options.tabSize|2|代码缩进单位为2个空格|
|vetur.format.options.useTabs|false|使用空格代替tab|

## js中的字符串使用单引号而不是双引号

只要`vetur.format.defaultFormatter.js`值为`vscode-typescript`就已经可以了。

| 配置项 | 值 |
|---|---|
|vetur.format.defaultFormatter.js|vscode-typescript|

## 取消行尾分号（如果有分号会在格式时自动删除）

| 配置项 | 值 |
|---|---|
|javascript.format.semicolons|remove|

> 这个不是只针对vue的，而是对所有js生效。

## 避免脚本中因使用别名被报错

| 配置项 | 值 |
|---|---|
|vetur.validation.script|false|

## 避免其他过于严格的报错（可选）

| 配置项 | 值 |
|---|---|
|vetur.validation.style|false|
|vetur.validation.template|false|
