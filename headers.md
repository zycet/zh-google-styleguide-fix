# 头文件 #
	通常每一个 `.cc` 文件都有一个对应的 `.h` 文件.
-	正确使用头文件可令代码在可读性、文件大小和性能上大为改观.
-	下面的规则将引导你规避使用头文件时的各种陷阱.

## #define 保护 ##
	所有头文件都应该使用 `#define` 防止头文件被多重包含, 
-	命名格式应当是:`<PROJECT>_<PATH>_<FILE>_H_`.
-	为保证唯一性, 头文件的命名应该依据所在项目源代码树的全路径.
例如, 项目`foo` 中的头文件 `foo/src/bar/baz.h` 可按如下方式保护:

		#ifndef FOO_BAR_BAZ_H_
		#define FOO_BAR_BAZ_H_
		…
		#endif FOO_BAR_BAZ_H_

## 头文件依赖 ##
	不要包含未使用的头文件
-	当一个头文件被包含的同时也引入了新的依赖, 一旦该头文件被修改, 代码就会被重新编译.
如果这个头文件又包含了其他头文件, 这些头文件的任何改变都将导致所有包含了该头文件的代码被重新编译.
而且如果引用了未使用的头文件会导致代码依赖性分析错误.
因此我们倾向于减少包含头文件, 尤其是在头文件中包含头文件.

## `#include` 的路径及顺序 ##
	使用标准的头文件包含顺序可增强可读性, 避免隐藏依赖: C 库, C++ 库, 其他库的 .h, 本项目内的 .h.
-	项目内头文件应按照项目源代码目录树结构排列,
-	避免使用 UNIX 特殊的快捷目录 `.` (当前目录) 或 `..` (上级目录).
	
	例如, `google-awesome-project/src/base/logging.h`应该按如下方式包含:`#include “base/logging.h”`.

-	又如, `dir/foo.cc` 的主要作用是实现或测试 `dir2/foo2.h` 的功能, `foo.cc` 中包含头文件的次序如下

		1.  `dir2/foo2.h` (优先位置, 详情如下)
		2.  C 系统文件
		3.  C++ 系统文件
		4.  其他库的 `.h` 文件
		5.  本项目内 `.h` 文件

	这种排序方式可有效减少隐藏依赖. 我们希望每一个头文件都是可被独立编译的,最简单的方法是将其作为第一个 `.h` 文件 `#included` 进对应的 `.cc`.