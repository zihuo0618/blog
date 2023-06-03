## Substrate宏

### 前言：

​		最近在学substrate，发现里面宏用的太多了，另外宏里面的代码语法还是自定义的（不是标准rust语法），导致整个代码看起来相当的奇怪，为了搞清楚substrate宏到底干了啥，使用cargo expand宏展开后研究了一番，得到了一些收获，现在仅使用rust语法来简单展示下substrate这些奇怪的宏到底在底层做了什么。



### 项目文件介绍：

下面的代码是演示如何去调用pallet中的方法，工程目录结构如下：

```
main.rs          // 入口文件
randomness.rs    //生成随机数的trait定义文件
kitties.rs       // kitties pallet模块
system.rs        // 公共Config trait
```



### 代码实现：

#### randomness.rs

首先是定义下随机数trait,代码如下：

```rust
pub trait Randomness<Output> {
    fn random() -> Output;
}
```

#### system.rs

这个文件是模拟substrate中的frame_system::Config trait

```rust
use std::fmt::Debug;

pub trait Config {
    type Account: Debug + Clone + Copy + ToString;
}
```

代码没啥实际意义，只是做演示用

#### kitties.rs

```rust
pub use pallet::*;
pub mod pallet {
    use std::marker::PhantomData;
    use crate::randomness::Randomness;
    use crate::system;

    // 模拟pallet中的config结构
    pub trait Config: system::Config {
        type Randomness: Randomness<u32>;
    }

    // 模拟 	#[pallet::pallet]
	//       pub struct Pallet<T>(_);
    pub struct Pallet<T:Config>(PhantomData<T>);

    impl <T:Config> Pallet<T> {
        pub fn create_kitty() {
            println!("hello kitty!!!  {}", T::Randomness::random());
        }
    }
}
```



#### main.rs  部分代码，省略了use这些

**模拟runtime中做的部分事情**

```rust
// 生成的Runtime结构
pub struct Runtime;
impl system::Config for Runtime { type Account = u32; }

// Randomness trait的fake实现
pub struct MyRandom;
impl randomness::Randomness<u32> for MyRandom {
    fn random() -> u32 {
        123123
    }
}

// construct_runtime! 宏展开
pub type KittiesPallet = kitties::Pallet<Runtime>;

// 指定kitties Config中的random
impl kitties::Config for Runtime {
    type Randomness = MyRandom;
}

fn main() {
    // 调用pallet的方法
    KittiesPallet::create_kitty();
}
```

真相就是这么简单！！！



