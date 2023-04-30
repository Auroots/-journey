# Rust 权威指南 - 笔记

[TOC]

## 第1章 入门

### 安装

#### Linux \ Mac

```bash
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bash_profile
```



#### Windows

**下载工具**

```bat
# 在下载系统相对应的 Rust 安装程序
https://www.rust-lang.org/learn/get-started
# 一路默认即可
PS C:\Users\NAME> rustup-init.exe
......
Current installation options:

   default host triple: x86_64-pc-windows-msvc
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
# 
rustup toolchain install stable-x86_64-pc-windows-gnu 
rustup default stable-x86_64-pc-windows-gnu
```



#### 使用Cargo创建一个项目

```bash
cargo new hello_cargo
cd hello_cargo
```



**文件 Cargo.toml (Tom's Obvious, Minimal Language)**

```json
# 表示接下来的语句会使用与配置当前的程序包;
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"
author = ["Your Name <you@example.com>"]

# 表示随后的区域用来声明项目的依赖;
# 在Rust中，把代码的集合称作包;
[dependencies]
```

- crate 	  ->    单元包
- package  ->   包

**一个Hello World程序**

```rust
fn main() {
	println!("Hello, world.");
}
```

**文件 Cargo.lock **

- 首次使用命令`cargo build` 的时候会在项目根目录下创建一个名为`Cargo.lock`的新文件;

- 记录当前项目所有依赖库的具体版本号;

**命令 cargo run**

- 构建项目并运行；
- 构建产生的结果会被cargo存储到`target/debug`目录下；

**命令 cargo check**

- 快速检测当前代码是否可以通过编译；

**命令 cargo build --release**

- 在优化模式下构建并生成可执行文件；
- 生成的可执行文件会被存储在`target/release`目录下；
- 此模式会以更长的编译时间为代价来优化代码，使代码拥有更好的运行时性能；

**命令 cargo run --release**



## 第2章 编写一个猜数游戏

```rust
use std::io;

fn main() {
    println!("Guess the number!");
    println!("Please input your guess.");
	// 创建一个存储用户输入数据的地方
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("Failed to read line.");

    println!("you guessed: {}", guess);
}
```

**代码解析**

- `let` ：声明变量的语句，(默认) 声明的变量不能二次改变；
- `let mut guess = 0;` ：声明了一个可变的变量；
- `String::new()`：`String`是标准库中一个字符串类型，内部使用了`UTF-8`，`new`是`String`类型的关联函数(associated function)，它会创建一个新的空白字符串；
- `&`：表示**引用**的意思；
- `&mut guess`：引用`mut` 让`guess`为可变状态，但前提是`guess`是一个可变变量;
- `use std::io` ： `use`用来显式的进行导入声明，`std` 一个`rust`的标准库，我们用来导入了其中`io`模块；
- `io::stdin()`：调用`io`模块中的关联函数`stdin`，它被用作句柄来处理终端中的标准输入；
- `.read_line(&mut guess)` ：调用标准句柄的`read_line`方法来获取用户的输入，它需要一个字符串作为参数，并且传入的变量是可变的，然后将获取的数据传入`guess`变量中，并返回两个`Resule`类型的值`Ok`和`Err`；
- `expect("Failed.")`：用来处理Result类型的表达式，接收`OK`与`Err`，如果返回的是`Err`，则打印`Failed.`；
- `println!("{}", guess);` ：代码中`{}`表示应用变量`guess`，如果有多个，但顺序排列显示；

```rust
let foo = 5;	// foo 是不可变的
let mut bar = 5;// bar 是可变的
```

- `Result`：枚举类型，枚举类型有一系列固定的值组合而成，拥有两个变体`OK`和`Err`，其中`Ok` 表示操作执行**成功**，`Err`表示操作执行**失败**；

### 生成一个随机数

```rust
use std::io;
use rand::Rng;

fn main(){
        println!("Guess the number!");
        let secret_number = rand::thread_rng().gen_range(1, 101);
        println!("The secret number is: {}", secret_number);
    
    println!("Plese input your guess.");
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("Failed to read line!");
    
    println!("You guessed: {}", guess);

}
```

**代码解析**

- `rand::Rng`：定义了随机数生成器需要实现的方法集合；
- `rand::thread_rng()`：会返回一个特定的随机数，它位于本地线程空间，它的随机数空间包含下限但不包含上限；

- `.gen_range(1, 101)`：用该方法来过去指定空间的随机数，包含1，但不包含101，即生成1-100之间的随机数；

### 比较猜测的数字与保密数字

```rust
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main(){
    println!("Guess the number!");
    let secret_number:u32 = rand::thread_rng().gen_range(1, 101);
    println!("The secret number is: {}",  secret_number);

    println!("Plese input your guess.");
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("Failed to read line!");
// -----------添加------------
    let guess: u32 = guess.trim().parse().expect("Please type a number!");
// --------------------------
    println!("You guessed: {}", guess);
// -----------添加------------
    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

**代码解析**

- `std::cmp::Ordering`：枚举类型，与`Result`相同，但是它有3个变体：`Less | Greater | Equal`；
- `match `：基于模式匹配的控制流，用于有条件地运行代码。必须显式地或通过在匹配中使用等通配符彻底地处理每个模式。因为match是一个表达式，所以也可以返回值。
- `guess.cmp(&secret_number)`：`cmp`为任何可比较的值，类型计算出它们比较后的结果；
- `Ordering::Less` :  这里表示`guess`小于`secret_number`;
- `Ordering::Greater` :  这里表示`guess`大于`secret_number`;
- `Ordering::Equal` :  这里表示`guess`等于`secret_number`;
- `.trim()`：返回一个去掉首尾所有空白字符的新字符串；
- `.parse()`：尝试将当前字符串解析为某个数值；
- `u32`：只能通过数字字符转换而来；

### 最后完成所有修改：循环，非法输入，

```rust
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main(){
    println!("Guess the number!");

    let secret_number:u32 = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("Failed to read line!");

        let guess: u32 = match guess.trim().parse() { // 修改
            Ok(num) => num, 	// 修改
            Err(_) => continue,	// 修改
        };

        println!("You guessed: {}", guess);
        
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {		// 修改
                println!("You win!");	// 修改
                break;	// 修改
            }
        }
    }
}
```

**代码解析**

- `loop`：无限反复循环；
- `break`：退出循环；
- `continue`：跳出至下一次循环
- 第15行：使用`match`表达式替换`expect`方法，`parse`会返回一个`Result`类型，只包含`Ok`和`Err`，使用`match`可以像处理`cmp`方法的返回值`Ordering`一样，只是三个返回值，现在变成了两个；



## 第3章 通用编程概念

### 变量与可变性

- Rust 中的变量默认是不可变的；

当一个变量是不可变时，一旦它被绑定到某个值上面，这个值就再也无法被改变：

```rust
fn main(){
    let x = 5;
    println!("The value of x is: {}", x);
// ------ 因 x = 5为不可变变量，所以下面这行赋值将发生编译错误；
	x = 6;
    println!("The value of x is: {}", x);
}
/* ------------ 错误信息 -----------
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:5:2
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
...
5 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
*/
```

- 不能对不可变变量二次赋值
- 在声明的变量名前添加`mut`关键字可以使得变量为可变；

```rust
fn main(){
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
/* 输出结果：
The value of x is: 5
The value of x is: 6
*/
```

### 变量与常量之间的不同

- 常量（constant）：常量不仅是默认不可变，它还总是不可变的；
- 你需要使用`const`关键字来声明一个常量；
- 常量需要标注类型
- 常量可以被声明在任何作用域中，包括全局变量；
- 只能将常量绑定到常量表达式上，不能作为函数返回值；
- **const**：不可改变的值（通常使用这种）。
- **static**：具有 ‘static 生命周期的，可以是可变的变量（须使用 static mut 关键字）。

将数值100000绑定到MAX_POINTS上，在Rust程序中，通常使用过“下划线`_`分割大写字母”来命名一个常量了；

可以使用下划线`_`分割数值，来增加可读性；

##### 常量声明方式

```rust
// const 常量名称:数据类型 = 值;
const MAX_POINTS:u32 = 100_000;
```

##### 变量声明方式

1. 可以包含 **字母**、**数字** 和 **下划线** 。
2. 变量名必须以 **字母** 或 **下划线** 开头。不能以 **数字** 开头。
3. 变量名是 **区分大小** 写的。也就是大写的 Study 和小写的 study 是两个不同的变量。

```rust
let 变量名 = 值;           // 不指定变量类型
let 变量名:数据类型 = 值;   // 指定变量类型

fn main() {
    let Study = "";
    print!("{}",study)
}
报错如下：
    print!("{}",study)
    ^^^^^ help: a local variable with a similar name exists (notice the capitalization): `Study`
```







### 隐藏

- 隐藏（shadow）第一个变量被第二个变量隐藏，意味着随后使用这个变量名称时，它指向的将是第二个变量；

```rust
fn main(){
    let x = 5;
    let x = x + 1;  // x = 6
    let x = x * 2;	// x = 12
    println!("The value of x is: {}", x);
}
/* 输出结果：
The value of x is: 12
*/
```

**代码解析**

首先x绑定到5上，随后又通过`let`语句隐藏了第一个x变量，并将第一个x变量值加上1的运算结果绑定到新的变量x上，此时x的值是6；以此类推；

- 隐藏机制不同于将一个变量声明为`mut`，因为如果不是在使用`let`关键字的情况下重新为这个变量赋值，以上代码将会编译发生错误；
- 通过使用`let`，我们可以对这个值执行一系列的**变换操作**，并且允许这个变量在操作完成后任然**保持无可变性**；

- `shadow`与`mut`另一个区别在于：重复使用`let`创建出新的变量，我们可以复用变量名称的同时改变它的类型；

例如：程序需要用户输入空格数量来决定文本之间的距离，那么我们可以把输入的空格存储为一个独立的数值：

```rust
let spaces = "    ";		// 此时：spaces的类型为 &str
let spaces = spaces.len();	// 此时：spaces的类型为 usize
```

如果使用`mut`模拟类似的效果，将会编译错误，提示要求删除`mut`关键字：

```rust
let mut spaces = "    ";
let spaces = spaces.len(); 

/* 输出结果：
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut spaces = "    ";
  |         ----^^^^^^
  |         |
  |         help: remove this `mut`
*/
```

### 数据类型

- 数据类型子集分两种：**标量类型**(scalar) 和 **复合类型**(compound)；
- Rust中的每一个值都有其特定的数据类型；
- Rust会根据数据的类型来决定应该如何处理它们；
- Rust是一门静态类型语言，它在编译过程中需要中的所有变量的具体类型；
- 大部分情况下编译器可以自动推导出变量的类型；

例如，当我们使用`parse`将一个`String`类型转换为`u32`类型时，就必须显式的在变量名称后面，添加一个类型标注：

```rust
let guess: u32 = "42".parse().expect("Not a number");
```

- 编译器无法自动推导出变量的类型，为了避免混淆，需要手动地添加类型标注；

### 标量类型

- 标量类型是单个值类型的统称；
- Rust中内建了4中基础的标量类型：整数，浮点数，布尔值，字符；

#### 整数类型

- 整数是指那些没有小数部分的数字；
- 整数分为：以( i )开头的有符号整数，以( u )开头的无符号整数；
- Rust中的整数类型有：

|  长度   | 有符号 | 无符号 |
| :-----: | :----: | :----: |
|  8-bit  |   i8   |   u8   |
| 16-bit  |  i16   |  u16   |
| 32-bit  |  i32   |  u32   |
| 64-bit  |  i64   |  u64   |
|  arch   | isize  | usize  |
| 128-bit |  i128  |  u128  |

- 有符号与无符号代表了一个整数是否拥有描述负数的能力；
- 无符号的整数类型，数值永远为正，不需要符号；
- 有符号的整数类型，通过二进制补码的形式来存储，所以可以是正( + )也可以是负( - )；
- 对一个位数为n的有符号整数，它可以存储总 -(2^n-1^) 到 2^n-1^ -1 范围内所有的整数；
    - 例如以`i8`来讲：它可以存储 -(2^8-1^) 到 2^8-1^ -1，也就是总 -128到127之间的所有整数；

- 对一个位数为n的无符号整数，它可以存储总0 到 2^n^ -1 范围内所有的整数；
    - 例如以`u8`来讲：它可以存储 0到 2^8^ -1，也就是总 1到 255 之间的所有整数；


- `isize`与`usize`这两种特殊的整数类型，它们的长度取决于程序运行的目标平台；
    - 在64位架构上就是64位，而在32位架构上，它们就是32位的；

**Rust中的整数字面量**

| 进制     | 整数字面量    | 示例        |
| -------- | ------------- | ----------- |
| 十六进制 | Hex           | 0xff        |
| 十进制   | Decimal       | 98_222      |
| 八进制   | Octal         | 0o77        |
| 二进制   | Binary        | 0b1111_0000 |
| 位(Byte) | Byte(u8 only) | b'A'        |

- 除了Byte，其余所有的字面量都可以使用类型后缀
    - 比如 `57u8` ，代表一个使用`u8`类型的整数`57`；
- 可以使用 下划线`_` 作为分隔符以方便读数；
- Rust对于整数字面量，默认推导类型位`i32`，通常就是最好的选择，即便是在64位系统上也是如此；
- `usize`和`isize` 则主要用作某些集合的索引；

**整数溢出**

假设一个u8类型的变量，存储0 - 255范围的数字，如果修改变量值为256，则会发生整数溢出；

- 当整数变量中的值超出储存范围时，就会发生整数溢出；
- Debug模式下编译时出现整数溢出，会触发程序的恐慌(panic)，描述程序因错误而退出；
- release模式下编译就不会包含那些可能会触发恐慌(panic)的检查代码，作为替代，Rust会在溢出发生时执行二进制补码环绕；换
    - 任何超出类型最大值的数值都会被“环绕”为类型最小值；
    - 以u8为例，256会变成0，257会变成1，以此内推；
    - 以整数溢出时环绕行为的代码都应该杜绝；
    - 如果希望显式地进行环绕行为，可以使用标准库中的类型`Wrapping`；



#### 浮点数类型

- Rust中提供了两种基础浮点数类型：`f32`和`f64`，分别占用`32`位空间和`64`位空间；
- 由于现代CPU中 `f64` 和 `f32` 的运行效率相差无几，`f64`拥有更高的精度，所以Rust默认会将浮点数类型字面量的类型推导为`f64`；

实际代码中浮点数声明：

```rust
fn main(){
    let x = 2.0;		// f64
    let y: f32 = 3.0;	// f32
}
```

- Rust的浮点数使用了 IEEE-754 标准来表述：单精度( f32 )，双精度( f64 )；

**算数运算 - 示例**

```rust
fn main(){
    // 加法
    let sum = 5 + 10;
    // 减法
    let difference = 96.6 - 3.5;
    // 乘法
    let product = 4 * 30;
    // 除法
    let quotient = 56.7 / 32.2;
    // 取余
    let remainder = 43 % 5;
}
```



#### 布尔类型

- Rust的布尔类型只拥有两个可能的值：ture，false；

```rust
fn main(){
    let t = true;
    let f: bool = false; // 附带了显式类型标注的语句
}
```



#### 字符类型

- Rust中`char`类型被用于描述语言中最基础的单个字符；
- `char`类型使用单引号指定；

```rust
fn main(){
    let c = 'z';
    let z = 'Б';
    let s = '😄';
}
```

- `char`类型在4字节，是一个Unicode标量值，可以表述比ASCII更多的字符内容；

- Unicode标量可以描述从`U+0000`到`U+D7FF`，以及从`U+E000`到`U+10FFFF`范围内的所有值；



### 复合类型

- 复合类型可以将多个不同类型的值组合为一个类型；
- Rust提供两种内置的基础复合类型：元组(tuple)，数组(array)；
- 元组类型的索引从0开始；

#### 元组类型(tuple)

- 元组可以将其他不同类型的多个值组合进一个复合类型；

- 元组可以拥有一个固定的长度，你无法在声明结束后增加或减少其中的元素数量；

    - 为了创建元组我们需要把一系列的值使用逗号分隔后放置到一对括号（）中；
    - 元素之间使用逗号 `,` 分隔
    - 每个位置的值都有一个类型，这些类型不需要是相同的；
    - 如果显式指定了元组的数据类型，那么数据类型的个数必须和元组的个数相同

#####  **创建元组**

```rust
fn main(){
    // let tuple变量名称 = (数据1, 数据2，...);
    // let tuple变量名称: (数据类型1, 数据类型2, ...) = (数据1, 数据2，...);
    let tup1 = (500, 6.4, 1);
	let tup: (i32, f64, &str) = (500, 6.4, "abc");
}
```

##### **访问元素**

```rust
// 元组变量.索引数字
fn main() {
    let tup: (i32, f64, &str) = (500, 6.4, "abc");
    println!("{}", tup.2);
}
// 输出
abc
```

##### **元组作参数**

```rust
// fn 函数名称(tuple参数名称: (&str, i32)){}
fn main() {
    let tup: (i32, f64, &str) = (500, 6.4, "abc");
    show_tuple(tup);
}
fn show_tuple(tuple: (i32, f64, &str)) {
    println!("{:?}", tuple);
}
//输出 
(500, 6.4, "abc")
```

##### 元组解构

> 就是在 tuple 中的每一个元素按照顺序一个一个赋值给变量。使用 **=** ，让右边的 tuple 按照顺序给等号左变的变量一个一个赋值。

**使用模式匹配来解构元组** 例子：

```rust
fn main(){
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
    println!("The value of y is: {}", y);
}
/* 输出结果：
The value of y is: 6.4
*/
```
**代码解析：**
    首先创建一个元组，将数值绑定到变量`tup`上，随后let的右侧使用了一个模式将`tup`拆分为3个不同的部分：`x, y, z`，这个创建被称为**解构**(Destructuring)，最后将`y`的值打印出来就是`6.4`。
	
除了解构，还可以**通过索引并使用点号( . )来访问元组中的值**，例子：

```rust
fn main(){
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
}
```

**代码解析：**
	创建一个元组x，随后通过索引访问元组中的各个元素，并将它们的值绑定到新的变量上；



#### 数组类型(array)

- 数组中的每个元素都必须是相同类型的；
- Rust中的数组拥有固定的长度，一旦声明不能随意更改大小；
- 数组由一整块分配在栈上的内存组成；
- 数组类型的索引从0开始；
- 拥有相同类型 T 的对象的集合；
- 内存中是连续存储的；
- 用逗号分隔值，放置在一对方括号`()`中) ;



##### **创建数组**

例子1：创建数组其中有5个元素，而值的类型由Rust自动推导；

例子2：在创建的数组变量名称后面指定该数组为i32类型，并且拥有5个元素，类型与元素数量之间用分号分隔；

```rust
// 例子1
let a = [1, 2, 3, 4, 5]；	
// 例子2
let a: [i32; 5] = [1, 2, 3, 4, 5];	
```



##### **创建一个含有相同元素的数组**

数组变量a将拥有5个元素，这些元素全部都拥有相同的初始值3；

```rust
let a = [3; 5];
// 以上等价于
let a = [3, 3, 3, 3, 3];
```



##### **访问数组的元素**

- 你可以通过索引来访问一个数组中的所有元素；

```rust
let a = [1, 2, 3, 4, 5];

let first = a[0];
let second = a[1];
```



##### **非法的数组元素访问**

```rust
fn main(){
    let a = [1, 2, 3, 4, 5];
    let index = 10;		// 超出索引范围

    let element = a[index];
    println!("The value of element is: {}", element);
}
/* 输出结果：
error: this operation will panic at runtime
 --> src/main.rs:5:19
  |
5 |     let element = a[index];
  |                   ^^^^^^^^ index out of bounds: the length is 5 but the index is 10
  |
  = note: `#[deny(unconditional_panic)]` on by default

error: could not compile `variables` due to previous error
*/
```

**代码解析：**

程序编译发生错误：`a[index]`索引超出范围：长度为5，但索引为10；



##### **数组做参数 - 值传递**

```rust
fn main() {
    let arr1 = ["Rust数据类型", "Rust函数式编程", "Rust所有权"];
    println!("\n{:?}", arr1);
    show_arr(arr1);
    println!("{:?}\n", arr1);
}

fn show_arr(arr: [&str; 3]) {
    let arr_len = arr.len();
    for i in 0..arr_len {
        println!("充电科目: {}", arr[i]);
    }
}
// 输出：
["Rust数据类型", "Rust函数式编程", "Rust所有权"]
充电科目: Rust数据类型
充电科目: Rust函数式编程
充电科目: Rust所有权
["Rust数据类型", "Rust函数式编程", "Rust所有权"]
```



##### **数组做参数 - 引用传递**

>  传递内存的地址给函数，修改数组的任何值都会修改原来的数组。

```rust
fn main() {
    let mut arr2 = ["Rust数据类型", "Rust函数式编程", "Rust所有权"];
    print!("{:?}\n", arr2);
    modify_arr(&mut arr2);
    print!("{:?}\n", arr2);
}
fn modify_arr(arr: &mut [&str; 3]) {
    let arr_len = arr.len();
    for i in 0..arr_len {
        arr[i] = "";
    }
}
// 输出
["Rust数据类型", "Rust函数式编程", "Rust所有权"]
["", "", ""]
```



### 函数

- `main()`函数 程序开始的地方；
- Rust代码使用蛇形命名法(snake case)来作为规范函数和变量名称的风格；
- 蛇形命名法：只使用小写的字母进行命名；并使用下划线分隔单词；

**蛇形命名法**：

```rust
fn main(){
    println!("Hello, world");
    
    another_function();
}
fn another_function(){
    println!("Another function.");
}
/* 输出结果：
Hello, world
Another function.
*/
```

- 函数定义以`fn`关键字开始，紧随函数名称与一对圆括号`()`和一对花括号`{}`；
- 花括号用于标识函数体开始和结束的地方；
- 我们可以使用函数名加圆括号的方式来调用函数；

#### 函数参数

- 可以在函数声明中 定义参数(parameter)；
- 它们是一种特殊的变量，并被视作函数签名的一部分；
- 你需要在调用函数时为这些变量提供具体的值；
- 函数签名中，必须显式地声明每个参数的类型，多个参数使用都好分隔；
- 函数参数可以是不同的类型；
- 函数参数分为两种：实参或实际参数(argument)，形参或形式参数(parameter)；

- 实参：函数被调用时给出的参数包含了实实在在的数据，会被函数内部的代码使用；
- 形参：在函数定义中出现的参数，它没有数据，只能等到函数被调用时接收传递进来的数据；

##### **值传递**

```rust
fn 函数名称(参数: 数据类型) {
   // 函数代码
}
fn 函数名称(参数: 数据类型) -> 返回值 {
   // 函数代码
}
```

实例

```rust
fn another_function(x: i32) {
    // 形参
    println!("The value of x is: {}\n", x);
}
fn another_function_1(x: i32) -> i32 {
    // 形参
    println!("The value of x is: {}\n", x);
    let y = x + 1;
    y
}
fn another_function_2(x: i32, y: i32, z: f64) {
    // 形参
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
    println!("The value of z is: {}", z);
}

fn main() {
    let a = 5; // 实参
    let b = 6; // 实参
    let c = 7.2; // 实参
    another_function(a); // 传参
    another_function_1(b); // 传参
    another_function_2(a, b, c); // 传参
}
/* 输出结果：
The value of x is: 5

The value of x is: 6

The value of x is: 5
The value of y is: 6
The value of z is: 7.2
*/
```

##### **引用传递**

```rust
fn 函数名称(参数: &mut 数据类型) {
   // 执行逻辑代码
}
fn 函数名称(参数: &mut 数据类型) -> 返回值 {
   // 执行逻辑代码
}
```

实例

```rust
fn double_price2(price: &mut i32) {
    *price = *price * 2;
    println!("内部的price是{}", price)
}

fn main() {
    let mut price = 88;
    double_price2(&mut price); //输出 内部的price是176
    println!("外部的price是{}", price); //输出 外部的price是176
}
// 输出
内部的price是176
外部的price是176
```

##### 复合类型传参

> 对于复合类型，比如字符串，如果按照普通的方法传递给函数后，那么该变量将不可再访问。

```rust
fn show_name(name: String) {
    println!("充电科目 :{}", name);
}

fn main() {
    let name: String = String::from("Rust权威指南");
    show_name(name);
    println!("调用show_name函数后: {}", name);
}

// 报错信息
error[E0382]: borrow of moved value: `name`
 --> src/main.rs:8:36
  |
6 |     let name: String = String::from("Rust权威指南");
  |         ---- move occurs because `name` has type `String`, which does not implement the `Copy` trait
7 |     show_name(name);
  |               ---- value moved here
8 |     println!("调用show_name函数后: {}", name);
  |                                       ^^^^ value borrowed here after move
```





#### 函数体中的语句和表达式

- Rust是一门基于表达式的语言
- 与C和Ruby不同，Rust中使用let语句赋值，赋值语句不会返回所赋的值；
- 函数体由若干条语句组成，它可以以一个表达式作为结尾；
- Rust将语句(statement)，表达式(expression)区分为两个不同的概念；
- 语句：是指那些执行操作但不返回值的指令；
- 表达式：是指进行计算并且产生一个值作为结果的指令；

**语句** 例子：

```rust
fn main(){
    let y = 6;	// 正常: 这条语句只进行赋值操作
    let x = (let y = 6);	// 错误: 你不能将一条let语句赋值给另一个变量
}
```

- 调用函数是表达式；
- 调用宏是表达式；
- 使用花括号{}创建新的作用域是表达式

**表达式** 例子：

```rust
fn main(){
    let x = 5;
    
    let y = {	// 表达式是一个代码块，从花括号开始，它计算出4作为结果
        let x =3;
        x + 1	// 这是表达式，假如末尾加分号就变为语句而不会返回任何值
    };
    println!("The value of y is: {}", y);
}
```



#### 函数的返回值

> 函数在代码执行完成后，除了将控制权还给调用者之外，还可以携带值给它的调用者。函数可以返回值给它的调用者。称为 **函数返回值**。

- 函数可以向调用它的代码返回值，不需要为返回值命名，但需要在`->`后面声明它的类型；
- Rust中函数的返回值等同于函数体最后一个表达式；
- 可以使用`return`关键字指定一个值来提前从函数中返回，大多数情况都是隐式返回；
- 返回执行结果，必须和函数定义时的返回数据类型一样

**有 return**

```rust
fn 函数名称() -> 返回类型 {
   return 返回值;
}
```

**没有 return**

> 如果函数代码中没有使用 `return` 关键字，那么函数会默认使用最后一条语句的执行结果作为返回值。

```rust
fn 函数名称() -> 返回类型 {
   // 业务逻辑
   返回值 // 没有分号则表示返回值
}
```



正确例子：

```rust
fn five() -> i32 {
    5	// 作为表达式返回i32类型的数值5
}
fn plus_one(x: i32) -> i32 {
    x + 1	// 将数值加1后，作为表达式返回
}
fn main(){
    let x = five();
    let y = plus_one(5);
    println!("x is: {} \ny is: {}", x, y);
}
/* 输出结果：
x is: 5 
y is: 6
*/
```



错误例子：

```rust
fn plus_one(x: i32) -> i32 {
    x + 1;	// 在末尾加了分号
}
fn main(){
    let y = plus_one(5);
    println!("The value of y is: {}", y);
}
/* 输出结果：
error[E0308]: mismatched types
 --> src/main.rs:1:24
  |
1 | fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
2 |     x + 1;    // 在末尾加了分号
  |          - help: consider removing this semicolon

*/
```

**代码解析：**

错误信息：mismatched types（类型不匹配）；

由于在应该用来返回值的表达式末尾加了分号，让其变成了语句，但是语句不会返回值，所以Rust默认返回了一个空元组，也就是上面描述的`()`，实际的返回值类型与函数定义的类型产生了矛盾，进而触发了编译时错误；

编译器建议我们删除末尾的分号，解决这个问题；



### 控制流 - 条件语句

| 条件判断语句                | 说明                                                        |
| --------------------------- | ----------------------------------------------------------- |
| `if` 语句                   | `if` 语句用于模拟现实生活中的 **如果…就…**                  |
| `if...else` 语句            | `if...else` 语句用于模拟 **如果…就…否则…**                  |
| `else...if` 和嵌套`if` 语句 | 嵌套`if` 语句用于模拟 **如果…就…如果…就…**                  |
| `match` 语句                | `match` 语句用于模拟现实生活中的 **老师点名** 或 **银行叫** |

- 通过条件来执行或重复执行某些代码是大部分编程语言的基础组成部分；
- 在Rust中用来控制程序执行流的结构主要就是 `if`表达式 与 循环表达式；

```rust
if 条件表达式 {
    // 条件表达式为true时要执行的逻辑
}

if 条件表达式 {
   // 如果 条件表达式 为真则执行这里的代码
} else {
   // 如果 条件表达式 为假则执行这里的代码
}

if 条件表达式1 {
   // 当 条件表达式1 为 true 时要执行的语句
} else if 条件表达式2 {
   // 当 条件表达式2 为 true 时要执行的语句
} else {
   // 如果 条件表达式1 和 条件表达式2 都为 false 时要执行的语句
}
```



#### if 表达式

- if表达式允许我们根据条件执行不同的代码分支；
- 假如条件满足则运行这段代码，否则跳过执行；

例子：

```rust
fn main (){
    let number = 6;
    
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is divisible by 4, 3, or 2");
    }
}
/* 输入结果：
number is divisible by 3
*/
```

##### 在let语句中使用if

- if是一个表达式，所以可以在let语句的右侧使用它生成一个值；
- 代码块输出的值就是其中最后一个表达式的值；
- 数字本身也可以作为一个表达式使用；
- 所有if分支可能返回的值都必须是一种类型的；
- 假如分支表达式产生的类型无法匹配，那么就会触发编译时错误；

例子：

```rust
fn main(){
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };
    println!(" The value of number is: {}", number);
}
/* 输出结果：
The value of number is: 5
*/
```



#### 循环

- 循环：即循环执行循环体中的代码直到结尾，并且紧接着回到开头继续执行；
- Rust提供三种循环：
    - `loop`   语句：一种重复执行且永远不会结束的循环；
    - `while` 语句：一种在某些条件为真的情况下就会永远执行下去的循环；
    - `for`     语句：一种有确定次数的循环；




##### loop循环

- 反复执行某一块代码，直到我们显式地声明退出为止；

例子：

```rust
fn main(){
    loop {
        println!("again!");
    }
}
```

**代码解析：**

运行这段程序，除非我们手动强制退出`Ctrl + c` 否则将一直循环往复执行，即死循环；



**从loop循环中返回值，使用break自动终止**

```rust
fn main() {
    let mut counter = 0;
    let result  = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    println!("The result is {}", result);
}
```



##### while 条件循环

```rust
while ( 条件表达式 ) {
    // 执行业务逻辑
}
```



例子： 

```rust
fn main() {
    let mut number = 3;
    while number != 0 {
        println!("{}!",number);
        number = number - 1;
    }
    println!("LIFTOFF!!!");
}
/* 输出结果：
3!
2!
1!
LIFTOFF!!!
*/
```

使用while循环遍历集合中每个元素  例子：

- 此类代码非常容易出错，可能会因为使用了不正确的索引长度而使程序崩溃，而且运行效率低；

```rust
fn main(){
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;
    
    while index < 5 {
        println!("The value is: {}", a[index]);
        index = index + 1;
    }
}
/* 输出结果：
The value is: 10
The value is: 20
The value is: 30
The value is: 40
The value is: 50
*/
```



##### for 条件循环

```rust
for 临时变量 in 左区间..右区间 {
   // 执行业务逻辑
}
```

**使用for来循环遍历集合** 例子：

- `.iter()` ：标准库中提供的Range，用来生成从一个数字开始到另一个数字结束之前的所有数字序列；

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a.iter() {
        println!(" The value is: {}", element);
    }
}
/* 输出结果：
 The value is: 10
 The value is: 20
 The value is: 30
 The value is: 40
 The value is: 50
 */
```

**for如何将循环序列翻转** 例子

- `.rev()`：标准库中提供的Range，将数字序列翻转，配合`.iter()`来生成一个从大到小的数字序列；

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a.iter().rev() {  // 在结尾加.rev()
        println!(" The value is: {}", element);
    }
}
/* 输出结果：
 The value is: 50
 The value is: 40
 The value is: 30
 The value is: 20
 The value is: 10
 */
```

另一种方式：

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}", number);
    }
    println!("LIFTOFF!!!");
}
/* 输出结果：
3
2
1
LIFTOFF!!!E
*/
```

##### for 与迭代器

>iter - 在每次迭代中借用集合中的一个元素。这样集合本身不会被改变，循环之后仍可以使用

```rust
fn main() {
    let studyList = vec!["《Rust权威指南》", "第3章 通用编程概念", "第4章 认识所有权"];
    for name in studyList.iter() {
        match name {
            &"第3章 通用编程概念" => println!("恭喜你进阶到第3阶段-{}!", name),
            _ => println!("学习: {}", name),
        }
    }
}
// 输出
学习: 《Rust权威指南》
恭喜你进阶到第3阶段-第3章 通用编程概念!
学习: 第4章 认识所有权
```

> into_iter - 会消耗集合。在每次迭代中，集合中的数据本身会被提供。一旦集合被消耗了，之后就无法再使用了，因为它已经在循环中被 “移除”（move）了。

```rust
fn main() {
    let studyList2 = vec!["《Rust权威指南》", "第3章 通用编程概念", "第4章 认识所有权"];
    for name in studyList2.into_iter() {
        match name {
            "第4章 认识所有权" => println!("恭喜你进阶到第4阶段-{}!", name),
            _ => println!("学习: {}", name),
        }
    }
}
输出
学习: 《Rust权威指南》
学习: 第3章 通用编程概念
恭喜你进阶到第4阶段-第4章 认识所有权!
```

> iter_mut - 可变地（mutably）借用集合中的每个元素，从而允许集合被就地修改。
> 就是停止本次执行剩下的语句，直接进入下一个循环。

```rust
fn main() {
    let mut studyList3 = vec!["《Rust权威指南》", "第3章 通用编程概念", "第4章 认识所有权"];
    for name in studyList3.iter_mut() {
        *name = match name {
            &mut "第3章 通用编程概念" => "恭喜你进阶到第4阶段---第4章 认识所有权",
            _ => *name,
        }
    }
    println!("studyList3: {:?}", studyList3);
}
// 输出
studyList3: ["《Rust权威指南》", "恭喜你进阶到第4阶段---第4章 认识所有权", "第4章 认识所有权"]
```



## 第4章 认识所有权

- 栈和堆都是代码运行时可以使用的内存空间，但它们通常以不同的结构组织而成；

- 栈(Stack)：以我们放入值时的顺序来存储它们，并以相反的顺序将值取出，也就是所谓的“后进先出”策略；
    - 所有存储在栈中的数据都必须拥有一个已知且固定的大小；
    - 入栈：添加数据；
    - 出栈：溢出数据；

- 堆(Heap)：空间管理较为松散，存储在编译期无法确定大小的数据；
    - 堆分配(分配)：当向堆请求特定大小的空间时，操作系统会根据你的请求在堆上寻找一块足够大的可用空间，将它标记为已使用，并将指向这片空间的地址返回给我们，
    - 如果将值压入栈不叫分配，指针大小是固定的且可以在编译期确定，所以可以将指针存储在栈中，当想要访问指向的数据时，可以通过指针指向的地址来访问；

- 由于访问堆上的数据多了指针跳转的环节，所以相比栈，访问堆上的数据要慢一些；
- 处理器在操作排布紧密的数据(比如在栈上)时要比操作排布稀疏的数据(比如在堆上)有效率的多，另外分配命令本身也可能消耗不少时钟周期；

### 所有权规则

- Rust中的每一个值都有一个对应的变量，作为他的所有者；
- 每个值同时只能有一个所有者；
- 当所有者离开自己的作用域时，它持有的值就会被释放掉；

### 变量作用域

- 变量在进入作用域是会保持自己的有效性，直到自己离开作用域为止；

例子1：

```rust
fn main() {
    // 不可用 变量x 还未生成
    let x = "hello";    // 变量x 的作用域开始
    fn prs() {  // 一个新的函数块，此块中不包含变量x 
        println!("{}", x);  // 这里将编译错误
    }

    println!("{}", x);
}   // 变量x 的作用域终止
```



### String 类型

- 当创建一个String类型时，Rust会将它存储在堆上；
- 可以调用`from`函数根据字符串字面量，创建一个String类型；
    - 这里使用双冒号( :: )运算符，它允许调用String命名空间下特定的from函数

```rust
let S = String::from("hello");
```

还可以生成一个可变的，并在后面继续添加字面量

```rust
fn main(){
    let mut S = String::from("Hello");
    // .push_str() 函数向String空间的尾部添加了一段字面量“,world!”
    S.push_str(", world!");
    println!("{}", S);
}
```



### 内存与分配

```rust
{
    let s = String::from("hello");	// 从这里开始，变量s 变得有效；
    // 执行与s变量的相关操作；
}	// 变量s 在这里离开作用域，即变量失效；
```

- 变量在离开作用域时，Rust会调用drop的特殊函数，将用来存储变量的内存释放；

**变量和数据交互的方式：移动**

将变量x的值重新绑定到变量y上：

```rust
let x = 5;
let y = x;
```



``` rust
let s1 = String::from("hello");
let s2 = s1;
```

- 当创建String类型时，Rust会向系统申请一块堆内存，将指向存放字符串内容的指针(ptr)，长度(len)，及一个容量(capacity)，这三样东西，存放至栈上；



- 当一个变量离开当前作用域时，Rust会自动调用Drop函数，并将变量使用的堆内存释放回收；

```rust
fn main(){
    s1 = 2;
    s2 = s1;
    println!("s1 = {}, s2 = {}", s1, s2);
}
```

- 由于整形类型可以在编译时确定自己的大小，且数据完整地存储在栈上，所有对于这类型值的copy永远都是快速的，与String类型不同，Rust不会阻止整数类型的有效性；

**变量和数据的交互方式：克隆**

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

- 任何**简单标量**的组合类型都可以是Copy的；
- 任何需要分配内存或某种资源的类型都不会是Copy的；

**拥有Copy trait的类型有：**

- 所有的整数类型；
- 仅拥有两种值(true or false)的布尔类型：bool；
- 字符类型：char；
- 所有的浮点类型，；例如f64；
- 元组：`Tuple`，如果元组所包含的所有字段仅拥有以上4点，那么这个元组也可以Copy；



### 所有权与函数

- 将值传递给函数在语义上类型与对变量进行赋值；
- 将变量传递给函数将会触发移动或复制；

```rust
fn main() {
    let s = String::from("hello");  // 变量s 进入作用域
    takes_ownership(s); // s的值被移动至函数
}

fn takes_ownership(some: String) { // 变量some 进入作用域
    println!("{}", some);
}// some 离开作用与，drop函数自动调用，some被清理
```



### 引用与借用(**Borrowing**)

> 就是一个函数中的变量传递给另外一个函数作为参数暂时使用。也会要求函数参数离开自己作用域的时候将**所有权** 还给当初传递给它的变量

- 引用运算符：`&`，允许在不获取所有权的前提下使用值；
- 解引用运算符：`*`


```rust
&变量名  //不可变的借用定义
&mut 变量名  //可变的借用定义
fn main() { 
    let s1 = String::from("hello"); 
    let len = str_len(&s1); 
    println!("The length of '{}' is {}.", s1, len); 
} 
   fn str_len(s: &String) -> usize { 
    s.len() 
}
```



### 切片 Slice

> Rust还有一种不持有所有权的数据类型：切片 （slice）。切片允许我们引用集合中某一段连续的元素序列，而不是 整个集合。
>
> 切片是只向一段连续内存的指针。在 Rust 中，连续内存够区间存储的数据结构：数组(array)、字符串(string)、向量(vector)。切片可以和它们一起使用，切片也使用数字索引访问数据。下标索引从**0**开始。slice 可以指向数组的一部分，越界的下标会引发致命错误（panic）。

```rust
// 切片的定义
let 切片值 = &变量[起始位置..结束位置]
```

1. [起始位置..结束位置]，这是一个左闭右开的区间。
2. **起始位置**最小值是**0**。
3. **结束位置**是数组、向量、字符串的长度。

```rust
fn first_word(s: &String) -> usize { 
    let bytes = s.as_bytes(); 
    for (i, &item) in bytes.iter().enumerate() { 
        if item == b' ' { 
            return i; 
            } 
        } 
    s.len() 
}
```

















## 第5章 使用结构体来组织相关联的数据

**结构体（ `struct` ）**可以由各种不同类型组成。使用 struct 关键字来创建。**struct** 是 **structure** 的缩写。结构体可以作为另一个结构体的字段。结构体是可以嵌套的。

- 元组结构体（tuple struct），事实上就是具名元组而已。

```rust
struct Pair(String, i32);
```

- 经典的 C 语言风格结构体（C struct）。

```rust
struct 结构体名称 {
    ...
}
```

- 单元结构体（unit struct），不带字段，在泛型中很有用。

```rust
struct Unit;
```

##### 定义结构体

```rust
struct 结构体名称 {
    字段1:数据类型,
    字段2:数据类型,
    ...
}
```

##### 创建结构体实例

```rust
let 实例名称 = 结构体名称{
    field1:value1,
    field2:value2
    ...
};

```

结构体初始化，其实就是对 **结构体中的各个元素进行赋值**。

```rust
#[derive(Debug)]
struct Study {
    name: String,
    target: String,
    spend: i32,
}

fn main() {
    let s = Study {
        name: String::from("从0到Go语言微服务架构师"),
        target: String::from("全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。"),
        spend: 3,
    };
    println!("{:?}", s);
    //输出 Study { name: "从0到Go语言微服务架构师", target: "全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。", spend: 3 }
}
```

##### 访问实例属性

```rust
实例名称.属性

println!("{}",s.name);//输出 从0到Go语言微服务架构师
```

##### 修改结构体实例

结构体实例默认是**不可修改**的，如果想修改结构体实例，声明时使用**mut**关键字。

```rust
let mut s2 = Study {
        name: String::from("从0到Go语言微服务架构师"),
        target: String::from("全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。"),
        spend: 3,
    };
 s2.spend=7;
 println!("{:?}",s2);//输出 Study { name: "从0到Go语言微服务架构师", target: "全面掌握Go语言微服务落地，代码级一次性解决微服务和分布式系统。", spend: 7 }
```

##### 结构体做参数

```rust
fn show(s: Study) {
    println!(
        "name is :{} target is {} spend is{}",
        s.name, s.target, s.spend
    );
}
```

##### 结构体作为返回值

```rust
fn get_instance(name: String, target: String, spend: i32) -> Study {
    return Study {
        name,
        target,
        spend,
    };
}

let s3 = get_instance(
    String::from("Go语言极简一本通"),
    String::from("学习Go语言语法，并且完成一个单体服务"),
    5,
);
println!("{:?}", s3);
//输出 Study { name: "Go语言极简一本通", target: "学习Go语言语法，并且完成一个单体服务", spend: 5 }
```

#### 方法

方法（method）是依附于对象的函数。这些方法通过关键字 self 来访问对象中的数据和其他。方法在 impl 代码块中定义。

> 与函数的区别
>
> 函数：可以直接调用，同一个程序不能出现 2 个相同的函数签名的函数，应为函数不归属任何 owner。
>
> 方法：归属某一个 owner，不同的 owner 可以有相同的方法签名。

```rust
impl 结构体{
    fn 方法名(&self,参数列表) 返回值 {
        //方法体
    }
}
```

impl 是 implement 的缩写。意思是 “实现”的意思。

self 是“自己”的意思，&self 表示当前结构体的实例。 &self 也是结构体普通方法固定的第一个参数，其他参数可选。

结构体方法的作用域仅限于结构体内部。

```rust
impl Study {
    fn get_spend(&self) -> i32 {
        return self.spend;
    }
}

println!("spend {}", s3.get_spend());
//输出 spend 5
```

#### 结构体静态方法

```rust
fn 方法名(参数: 数据类型,...) -> 返回值类型 {
      // 方法体
   }

调用方式
结构体名称::方法名(参数列表)
```

```rust
impl Study {
    ...
    fn get_instance_another(name: String, target: String, spend: i32) -> Study {
        return Study {
            name,
            target,
            spend,
        };
    }
}

let s4 = Study::get_instance_another(
    String::from("Go语言极简一本通"),
    String::from("学习Go语言语法，并且完成一个单体服务"),
    5,
);
```

**单元结构体**
unit type 是一个类型，有且仅有一个值：()，单元类型()类似 c/c++/java 语言中的 void。当一个函数并不需要返回值的时候，rust 则返回()。但语法层面上，void 仅仅只是一个类型，该类型没有任何值；而单元类型()既是一个类型，同时又是该类型的值。

实例化一个元组结构体

```rust
let pair = (String::from("从0到Go语言微服务架构师"), 1);
```

访问元组结构体的字段

```rust
println!("pair 包含 {:?} and {:?}", pair.0, pair.1);
```

解构一个元组结构体

```rust
let (study, spend) = pair;
println!("pair contains {:?} and {:?}", study, spend);
```







## 第6章 枚举与模式匹配



### 枚举(Enum)

> 枚举 **enum** 关键字允许创建一个从数个不同取值中选其一的枚举类型（enumeration）。任何一个在 struct 中合法的取值在 enum 中也合法。

#### 枚举的定义

```rust
enum 枚举名称{
 variant1,
 variant2,
 ...
}
```

#### 使用枚举

```rust
// 枚举名称::variant

// #[derive(Debug)] 注解的作用，就是让 派生自Debug`。
#[derive(Debug)]
enum RoadMap {
    认识所有权,
    使用结构体来组织相关联的数据,
    枚举与模式匹配,
}

fn main() {
    let level = RoadMap::枚举与模式匹配;
    println!("level---{:?}", level);
}
```

#### Option 枚举

> `Option` 枚举经常用在函数中的`返回值`，它可以表示有返回值，也可以用于表示没有返回值。如果有返回值。可以使用返回 Some(data)，如果函数没有返回值，可以返回 None。

```rust
enum Option<T> {
   Some(T),      // 用于返回一个值
   None          // 用于返回 null,用None来代替。
}
```

```rust
fn get_discount(price: i32) -> Option<bool> {
    if price > 100 {
        Some(true)
    } else {
        None
    }
}
fn main() {
    let p = 666; //输出 Some(true)
 // let p = 6; //输出 None
    let result = get_discount(p);
    println!("{:?}", result)
}
```



#### match 语句

> 判断一个枚举变量的值，唯一操作符就是 `match`。
>
> Rust 中的 `match` 语句有返回值，它把 **匹配值** 后执行的最后一条语句的结果当作返回值。

```rust
match variable_expression {
   constant_expr1 => "返回值",
   constant_expr2 => {
      // 语句;
   },
   _ => {
      // 默认
      // 其它语句
   }
};
```



```rust
fn print_road_map(r:RoadMap){
    match r {
        RoadMap::Go语言极简一本通=>{
            println!("Go语言极简一本通");
        },
        RoadMap::Go语言微服务架构核心22讲=>{
            println!("Go语言微服务架构核心22讲");
        },
        RoadMap::从0到Go语言微服务架构师=>{
            println!("从0到Go语言微服务架构师");
        }
    }
}
print_road_map(RoadMap::Go语言极简一本通);//输出 Go语言极简一本通
print_road_map(RoadMap::Go语言微服务架构核心22讲);//输出 Go语言微服务架构核心22讲
print_road_map(RoadMap::从0到Go语言微服务架构师);//输出 从0到Go语言微服务架构师
```

##### 带数据类型的枚举

```rust
enum 枚举名称{
    variant1(数据类型1),
    variant2(数据类型2),
    ...
}
```

```rust
#[derive(Debug)]
enum StudyRoadMap{
    Name(String),
}

let level3 = StudyRoadMap::Name(String::from("从0到Go语言微服务架构师"));
match level3 {
    StudyRoadMap::Name(val)=>{
        println!("{:?}",val);
    }
}

//输出 "从0到Go语言微服务架构师"
```













## 第7章 使用包，单元包及模块来管理日渐复杂的项目



















## 第8章 通用集合类型





### 字符串

Rust 语言提供了两种字符串

- Rust 核心内置的数据类型`&str`，字符串字面量 。
- Rust 标准库中的一个 **公开 `pub`** 结构体。字符串对象 `String`。

#### 更新String

- `push_str()`方法 ：把一个字符串切片附加到`String`；

    - ```rust
        let mut s = String::from("foo");
        s.push_str("bar");
        ```

    - ```rust
        let mut s1 = String::from("foo");
        let s2 = "bar";
        s1.push_str(s2);
        println!("s2 is {}", s2);
        ```

- `push()`方法 ：把单个字符附加到`String`；

    - ```rust
        let mut s = String::from("lo");
        s.push('l');
        ```

- `+`运算符 ：拼接字符串

    - ```rust
        let s1 = String::from("Hello, ");
        let s2 = String::from("world!");
        let s3 = s1 + "-" + &s2;
        ```

    - 使用了类似这个签名的方法：

    -  ```rust
        fn add(self, s: &str) -> String {...}
        ```

        - 标准库中`add`方法使用了泛型
        - 只能把`&str`添加到`String`
        - 解引用强制转换(deref coercion)

- `format!`宏：连接多个字符串，不会夺取任何参数的所有权； 

    - ```rust
        let s1 = String::from("tic");
        let s2 = String::from("tac");
        let s3 = String::from("toe");
        
        let s = format!("{}-{}-{}", s1, s2, s3);
        ```

- `println!`宏：将结果打印到屏幕；

#### 字符串对象常用方法

##### String::new()

创建一个新的字符串对象

```rust
let s1 = String::new()
```

##### .push_str()

在字符串末尾追加字符串

```rust
let mut s3 = String::new();
s3.push_str("Rust权威指南");
println!("{}",s3); //输出 Rust权威指南
```

##### .replace()

指定字符串中的子串替换成另一个字符串

```rust
let s4 = String::from("Rust权威指南");
let result = s4.replace("Rust权威指南","www.auroot.cn");
println!("{}",result); //输出 www.auroot.cn
```

##### .len()

获取长度，返回字符串中的 **总字节数**。该方法会统计包括 **制表符 `\t`**、**空格** 、**回车 `\r`**、**换行 `\n`** 和**回车换行 `\r\n`** 等等。

```rust
let s5 = String::from("Rust权威指南\n");
println!("length is {}",s5.len());//输出 17
```

##### .to_string()

将字符串转换为字符串对象，方便以后可以有更多的操作。

```rust
let s6 = "Rust权威指南".to_string();
println!("{}",s6);//输出 Rust权威指南
```

##### .as_str()

返回一个字符串对象的 **字符串** 字面量。

```rust
fn show_name(name:&str){
    println!("充电科目:{}",name);
}

let s7 = String::from("Go语言微服务架构核心22讲");
show_name(s7.as_str()); //输出 充电科目:Go语言微服务架构核心22讲
```

##### .trim()

去除字符串头尾的空白符。空白符是指 **制表符 \t**、**空格**、**回车 `\r`**、**换行 \n** 和**回车换行 `\r\n`** 等等

```rust
let s8 = " \tGo语言极简一本通\tGo语言微服务架构核心22讲 \r\n从0到Go语言微服务架构师\r\n     ";
println!("length is {}",s8.len());//输出 length is 103
println!("length is {}",s8.trim().len());//输出 length is 94
println!("s8:{}",s8);
//输出
s8:     Go语言极简一本通        Go语言微服务架构核心22讲
从0到Go语言微服务架构师
```

##### .split()

将字符串根据某些指定的 **字符串子串** 分割，返回分割后的字符串子串组成的切片上的迭代器。

```rust
let s9 = "Go语言极简一本通、Go语言微服务架构核心22讲、从0到Go语言微服务架构师";
for item in s9.split('、'){
   println!("充电科目: {}",item);
}
//输出
//充电科目: Go语言极简一本通
//充电科目: Go语言微服务架构核心22讲
//充电科目: 从0到Go语言微服务架构师
```

##### .chars()

将一个字符串打散为所有字符组成的数组

```rust
let s10 = "从0到Go语言微服务架构师";
for c in s10.chars(){
   println!("字符: {}",c);
}
//输出
字符: 从
字符: 0
字符: 到
字符: G
字符: o
字符: 语
字符: 言
字符: 微
字符: 服
字符: 务
字符: 架
字符: 构
字符: 师
```





## 第9章 错误处理



















## 第10章 泛型，trait与声明周期

```rust 
// src/lib.rs

// 定义一个trait 名称为：Summary 
pub trait Summary {
    // 定义 summarize 方法
    fn summarize(&self) -> String;
    // fn sumau(&self) -> String;
}

// 定义 NewsArticle 结构体
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}
// impl块只是fn定义的集合
// impl示例1: 为NewsArticle结构体实现Summary trait 集合
impl Summary for NewsArticle {
    // 定义 Summary 中 summarize 方法具体的实现
    fn summarize(&self) -> String {
        format!("{}, by {} ({})",self.headline, self.author, self.location)
    }
}
// impl集合内的函数，都会成为NewsArticle结构体的方法
// impl示例2: 为 NewsArticle 结构体定义方法
impl NewsArticle {
    // 定义方法成员 summarize
    pub fn summarize(&self) -> String {
        format!("{}, by {} ({})",self.headline, self.author, self.location)
    }
}
// 定义 Tweet 结构体
pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}
// 为 Tweet 结构体实现Summary trait
impl Summary for Tweet {
    // 定义 summarize 方法具体的实现
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```



```rust
// src/main.rc

use variables::Summary;
use variables::Tweet;
use variables::NewsArticle;

fn main(){
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };
    let newsa = NewsArticle {
        headline: String::from("Auins.sh"),
        author: String::from("Auroot"),
        location: String::from("2020"),
        content: String::from("2"),
    };
    println!("1 new tweet: {}", tweet.summarize());
    println!("1 new tweet: {}", newsa.summarize());
}
```



















## 第11章 编写自动化测试



















## 第12章 I/O项目：编写一个命令行程序



















## 第13章 函数式语言特性：迭代器与闭包



















## 第14章 进一步认识Cargo及crates.io



















## 第15章 智能指针



















## 第16章 无畏并发



















## 第17章 Rust的面向对象编程特性



















## 第18章 模式匹配



















## 第19章 高级特性



















## 第20章 构建多线程Web服务器













## 运算符

### 算术运算符

> **注：Rust 语言不支持自增自减运算符 `++` 和 `--`**

| 名称 | 运算符 |
| ---- | ------ |
| 加   | +      |
| 减   | -      |
| 乘   | *      |
| 除   | /      |
| 求余 | %      |

### 位运算符

| 名字 | 运算符 | 说明                                           |
| ---- | ------ | ---------------------------------------------- |
| 位与 | &      | 相同位都是 1 则返回 1 否则返回 0               |
| 位或 | \|     | 相同位只要有一个是 1 则返回 1 否则返回 0       |
| 异或 | ^      | 相同位不相同则返回 1 否则返回 0                |
| 位非 | !      | 把位中的 1 换成 0 ， 0 换成 1                  |
| 左移 | <<     | 操作数中的所有位向左移动指定位数，右边的位补 0 |
| 右移 | >>     | 操作数中的所有位向右移动指定位数，左边的位补 0 |

### 关系运算符

| 名称     | 运算符 | 说明                                                     |
| -------- | ------ | -------------------------------------------------------- |
| 大于     | >      | 如果左操作数大于右操作数则返回 true 否则返回 false       |
| 小于     | <      | 如果左操作数小于于右操作数则返回 true 否则返回 false     |
| 大于等于 | >=     | 如果左操作数大于或等于右操作数则返回 true 否则返回 false |
| 小于等于 | <=     | 如果左操作数小于或等于右操作数则返回 true 否则返回 false |
| 等于     | ==     | 如果左操作数等于右操作数则返回 true 否则返回 false       |
| 不等于   | !=     | 如果左操作数不等于右操作数则返回 true 否则返回 false     |

### 逻辑运算符

| 名称   | 运算符 | 说明                                                     |
| ------ | ------ | -------------------------------------------------------- |
| 逻辑与 | &&     | 两边的条件表达式都为真则返回 true 否则返回 false         |
| 逻辑或 | \|\|   | 两边的条件表达式只要有一个为真则返回 true 否则返回 false |
| 逻辑非 | !      | 如果表达式为真则返回 false 否则返回 true                 |



## 个人学习总结

































