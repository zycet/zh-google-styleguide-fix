# 作用域 #

## 局部变量 ##
	将函数变量尽可能置于最小作用域内, 并在变量声明时进行初始化.

-	C++ 允许在函数的任何位置声明变量. 我们提倡在尽可能小的作用域中声明变量, 离第一次使用越近越好. 这使得代码浏览者更容易定位变量声明的位置, 了解变量的类型和初始值. 特别是，应使用初始化的方式替代声明再赋值, 比如:

		int i;
		i = f(); // 坏——初始化和声明分离
		int j = g(); // 好——初始化时声明


-	GCC 正确实现了 `for (int i = 0; i < 10; ++i)` (`i` 的作用域仅限 `for` 循环内), 所以其他 `for` 循环中可以重新使用 `i`. 在 `if` 和 `while` 等语句中的作用域声明也是正确的, 如:

	while (const char* p = strchr(str, ‘/’)) str = p + 1;

-	如果变量是一个对象, 每次进入作用域都要调用其构造函数,每次退出作用域都要调用其析构函数， 则应该将变量声明放在循环外从而提高效率.

		// 低效的实现
		for (int i = 0; i < 1000000; ++i)
		{
			Foo f;				// 构造函数和析构函数分别调用 1000000 次!
			f.DoSomething(i);
		}

		// 高效的实现
		Foo f;					// 构造函数和析构函数只调用 1 次
		for (int i = 0; i < 1000000; ++i)
		{
			f.DoSomething(i);
		}

## 静态和全局变量 ##
-	禁止使用 `class` 类型的静态或全局变量: 它们会导致很难发现的bug和不确定的构造和析构函数调用顺序.
-	静态生存周期的对象, 包括全局变量, 静态变量, 静态类成员变量,以及函数静态变量, 都必须是原生数据类型 (POD : Plain Old Data): 只能是int, char, float, 和 void, 以及 POD 类型的数组/结构体/指针.
-	永远不要使用函数返回值初始化静态变量; 不要在多线程代码中使用非 `const`
的静态变量.
	
	不幸的是, 静态变量的构造函数, 析构函数以及初始化操作的调用顺序在 C++标准中未明确定义, 甚至每次编译构建都有可能会发生变化, 从而导致难以发现的bug. 比如, 结束程序时, 某个静态变量已经被析构了, 但代码还在跑, 其它线程很可能试图访问该变量, 直接导致崩溃.
	所以, 我们只允许 POD 类型的静态变量. 本条规则完全禁止 `vector` (使用 C数组替代), `string` (使用 `const char*`),及其它以任意方式包含或指向类实例的东东, 成为静态变量. 出于同样的理由,我们不允许用函数返回值来初始化静态变量.
-	如果你确实需要一个` class` 类型的静态或全局变量, 可以考虑在 `main()`函数或`pthread\_once()`内初始化一个你永远不会回收的指针.