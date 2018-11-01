# JavaScript 基础问题

* [`[Basic]` 类型判断](/sections/common.md#类型判断)
* [`[Basic]` 作用域](/sections/common.md#作用域)
* [`[Basic]` 引用传递](/sections/common.md#引用传递)
* [`[Basic]` 内存释放](/sections/common.md#内存释放)
* [`[Basic]` ES6 新特性](/sections/common.md#es6-新特性)

## 类型判断


JavaScript包含七种数据类型，分别为`number`、`string`、`boolean`、`undefined`、`null`、`object`和`symbol`(ES6新增的)。

`number`、`string`、`boolean`、`symbol`合称为原始类型（primitive type）数据。
`object`是由原始类型数据组成，所以称为复合类型（complex type）数据。
`undefined`和`null`，一般将它们看成两个特殊值。

`object`又可以分成三个子类型: 

 - 狭义的对象（object）
 - 数组（array）
 - 函数（function）