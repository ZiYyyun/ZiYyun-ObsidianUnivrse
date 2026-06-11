### 变量的声明

- 在Rust 中，使用`let`关键字声明变量：

```rust
fn main(){
  // 声明变量name并初始化赋值
  let name = "junmajinlong.com";
  println!("{}", name);  // 使用"{}"格式化输出
}
```

- **Rust会对未使用的变量发出警告信息**。如果确实想保留从未被使用过的变量，可在变量名前加上`_`前缀。

  ```rust
  fn main(){
    let name = "junmajinlong.com";
    println!("{}", name);
  
    let gender = "male";   // 警告，gender未使用
    let _age = 18;     // 加_前缀的变量不被警告
  }
  ```

  

- **Rust允许声明未被初始化(即未被赋值)的变量，但不允许使用未被赋值的变量**。多数情况下，都是声明的时候直接初始化的。

  ```rust
  fn main() {
    let name;  // 只声明，未初始化
    // println!("{}", name);  // 取消该行注释，将编译错误
    
    name = "junmajinlong.com";
    println!("{}", name);
  }
  ```

  ### 变量的使用

- Rust允许**重复声明**同名变量，后声明的变量将**遮盖**(shadow)前面已声明的变量。需注意的是，遮盖不是覆盖，被遮盖的变量仍然存在，而如果是被覆盖则不再存在(也即，覆盖时，原数据会被销毁)。

  ```rust
  fn main() {
    let name = "junmajinlong.com";
    // 注释下行，将警告：name变量未被使用 因为name仍然存在，只是被遮盖了
    println!("{}", name);  
    
    let name = "gaoxiaofang.com";  // 遮盖已声明的name变量
    println!("{}", name);
  }
  ```

  ```
  注：下图内存布局并不完全正确，此图仅为说明变量遮盖
           +---------+       +--------------------+
           |  Stack  |       |        Heap        |
           +---------+       +--------------------+
  name --> | 0x56789 |  ---> | "gaoxiaofang.com"  |
           |         |       +--------------------+
  name --> | 0x01234 |  ---> | "junmajinlong.com" |
           +---------+       +--------------------+
  
  ```

  ### 变量的修改

- **变量初始化后，默认不允许再修改该变量**。注意，修改变量是直接给变量赋值，而不是再次let声明该变量，再次声明变量是允许的，它会遮盖原变量。

  ```rust
  fn main() {
    let name = "junmajinlong.com";
    // 取消下行注释将编译错误，默认不允许修改变量
    // name = "gaoxiaofang.com";
    
    let name = "gaoxiaofang.com";  // 再次声明变量，遮盖变量
    println!("{}", name);
  }
  ```

  - **如果想要修改变量的值，需要在声明变量时加上`mut`标记**(mutable)表示该变量是可修改的。

    ```rust
    fn main() {
      let mut name = "junmajinlong.com";
      println!("{}", name);
      
      name = "gaoxiaofang.com";   // 修改变量
      println!("{}", name);
    }
    ```

### 变量解构

```rust
fn main() {
    let (a, mut b): (bool,bool) = (true, false);
    // a = true,不可变; b = false，可变
    println!("a = {:?}, b = {:?}", a, b);

    b = true;
    assert_eq!(a, b);
}
```


### 常量

- 常量不允许使用 `mut`。**常量不仅仅默认不可变，而且自始至终不可变**，因为常量在编译完成后，已经确定它的值。
- 常量使用 `const` 关键字而不是 `let` 关键字来声明，并且值的类型**必须**标注。

下面是一个常量声明的例子，其常量名为 `MAX_POINTS`，值设置为 `100,000`。（Rust 常量的命名约定是全部字母都使用大写，并使用下划线分隔单词，另外对数字字面量可插入下划线以提高可读性）：
```rust
const MAX_POINTS: u32 = 100_000;
```

>常量可以在任意作用域内声明，包括全局作用域，在声明的作用域内，常量在程序运行的整个过程中都有效。对于需要在多处代码共享一个不可变的值时非常有用，例如游戏中允许玩家赚取的最大点数或光速。

### 变量遮蔽
> Rust 允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的

```rust
fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;

    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

	这和 `mut` 变量的使用是不同的，第二个 `let` 生成了完全不同的新变量，两个变量只是恰好拥有同样的名称，涉及一次内存对象的再分配 ，而 `mut` 声明的变量，可以修改同一个内存地址上的值，并不会发生内存对象的再分配，性能要更好。

> 变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字，而不用绞尽脑汁去想更多的名字。

例如，假设有一个程序要统计一个空格字符串的空格数量：
```rust
// 字符串类型
let spaces = "   ";
// usize数值类型
let spaces = spaces.len();
```

这种结构是允许的，因为第一个 `spaces` 变量是一个字符串类型，第二个 `spaces` 变量是一个全新的变量且和第一个具有相同的变量名，且是一个数值类型。所以变量遮蔽可以帮我们节省些脑细胞，不用去想如 `spaces_str` 和 `spaces_num` 此类的变量名；相反我们可以重复使用更简单的 `spaces` 变量名。如果你不用 `let` :
```rust
let mut spaces = "   ";
spaces = spaces.len(); //会报错 
```
显然，Rust 对类型的要求很严格，不允许将整数类型 `usize` 赋值给字符串类型。`usize` 是一种 CPU 相关的整数类型。
### 数学运算

```rust
fn main() { 
    let sum = 5 + 10; // 加 
    let difference = 95.5 - 4.3; // 减 
    let product = 4 * 30; // 乘 
    let quotient = 56.7 / 32.2; // 除 
    let remainder = 43 % 5; // 求余
}
```

- Rust 不支持 **++** 和 **--**，因为这两个运算符出现在变量的前后会影响代码可读性，减弱了开发者对变量改变的意识能力。