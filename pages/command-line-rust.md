- 通过实现一个命令行项目来学习 [[Rust]]
- Cargo.toml
	- ```toml
	  [dev-dependencies]
	  assert_cmd = "1"  # 在测试用例中，这个库的执行总是返回成功
	  ```
- ## 表达式与操作符
	- ### if
		- Rust 中 `if`是一个表达式。这意味着它会有一个返回值。特别的是：`if`表达式的返回值是`unit`
	- ### ? 操作符
		- `?` 操作符可以用在 `Result` 或者 `Option` 类型表达式后面，简化代码。
		- `?`   操作符的工作原理
			- 当你在一个表达式后面使用 `?` 操作符时，Rust 会自动进行以下操作：
			- 对于`Result` 类型
				- 如果表达式的结果是 `Result::Ok(val)`，则 `?` 操作符会提取 `val` 并继续执行后续代码。
				- 如果表达式的结果是 `Result::Err(err)`，则 `?` 操作符会立即从当前函数返回 `Err(err)`，并且 `err` 会被传递到调用这个函数的地方。
			- 对于 `Option` 类型：
				- 如果表达式的结果是 `Some(val)`，则 `?` 操作符会提取 `val` 并继续执行后续代码。
				- 如果表达式的结果是 `None`，则 `?` 操作符会立即从当前函数返回 `None`。
- ## 基础类型
	- `()` *unit*
	- `Result`
		- ```rust
		  enum Result<T, E> {
		      Ok(T),
		      Err(E),
		  }
		  ```
	- `Option`
		-
- ## *type alias*, *boxed value*, *dyn*
	- `type TestResult = Result<(), Box<dyn std::error:Error>>;`
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202405030134661.png)
	- `Box`
		- 在 heap 上分配空间用于存储*具体*的数据，在 stack 中只是存储了一个指针。由于编译期还不知道具体的数据类型、大小，所以使用 Box “包裹”起来。
		- `Box<dyn T>` 对于编译器来说，一旦 T 确定下来，那么就可以在 stack 中为其对应类型分配一个确定长度的指针空间。实际数据则存储在 heap 中。**Rust 编译器也不允许分配一个长度未确定的 stack 空间存储指针**
		-
	- `dyn T`
		- dyn 表示动态派发，后面通常跟着一个 trait。由于编译期还是不知道 trait 的具体实现，所以需要使用动态派发。
- ## test 相关
	- 测试用例编写
		- ```rust
		  #[test]
		  fn name() {}
		  ```
		- 这里说明这是一个测试函数
	- 运行测试用例
		- 基本命令 `cargo test`
		- 传入命令参数：`cargo test -- ${args}`
		- 运行指定测试函数：`cargo test hello`。会执行所有 hello* 名字的测试函数。
		-
	-