# 模式匹配



### 所有模式匹配语法

```
let x = 5;

match x {
	1 => todo, // 字面值
	Some(y) => y, // 匹配命名变量
	2|3|4 => todo, // 多模式匹配
	6..=100 => todo, // 闭区间范围匹配，也可以时char类型
	Point { x, y: 0 } => (x, y), // 结构体解构匹配
	Point { x: a, y: b } => (a, b), // 解构重命名
	Point { x, y } => (x, y),
	Message::Move(x, y) => (x, y), // 解构枚举
	(first, _, third, _) => (f, t), // _忽略值
	(f, ..) => f, // ..忽略剩下值
	(f, .., e) => (f, e), // 填充满中间值并忽略
	Some(y) if y % 2 == 0 => y, // 匹配守卫
	Point { x: a, y: b @ 3..=7} => (a, b), // @绑定
	_ => todo, // 忽略该值 会匹配其他所有情况
}
```



### 用到模式匹配的地方

`match` `if let` `while let` `let` `for` 函数参数