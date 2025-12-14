# 默认构造函数

> 原文：[`cel.cs.brown.edu/crp/idioms/constructors/default_constructors.html`](https://cel.cs.brown.edu/crp/idioms/constructors/default_constructors.html)

C++ 有一个特殊的默认构造函数概念，用于支持它们在隐式调用时的一些场景。Rust 没有相同的默认构造函数概念。最相似的是 `Default` 特性（[`doc.rust-lang.org/std/default/trait.Default.html`](https://doc.rust-lang.org/std/default/trait.Default.html)）。

```rs
class Person {
    int age;

public:
    // Default constructor
    Person() : age(0) {}
} 
```

```rs

```

#![allow(unused)] fn main() { struct Person {

age: i32,

}

impl Person {

    pub const fn new() -> Self {

        Self { age: 0 }

    }

}

impl Default for Person {

    fn default() -> Self {

        Self::new()

    }

}

}

```rs

```

如果一个结构体有一个有用的默认值（例如，在 C++ 中由默认构造函数构建的值），则类型应提供[两者](https://rust-lang.github.io/api-guidelines/interoperability.html?highlight=default#types-eagerly-implement-common-traits-c-common-traits)——一个不带参数的 `new` 方法和一个 `Default` 的实现。

因为 `Default` 是一个普通特性，所以示例中定义的默认构造函数可以使用调用静态特性方法的常规语法来调用，例如，`Default::default()`。

## 类成员的隐式初始化

在 C++ 中，如果一个成员没有被构造函数显式初始化，则它会被默认初始化。当成员的类型是类时，默认初始化会调用默认构造函数。

在 Rust 中，如果一个结构体的所有字段都实现了 `Default` 特性，那么编译器可以提供一个结构的实现。

```rs
class Person {
  int age;

public:
  Person() : age(0) {}
}

class Student {
  Person person;
} 
```

```rs

```

#![allow(unused)] fn main() { #[derive(Default)]

struct Person {

    age: i32,

}

#[derive(Default)]

struct Student {

    person: Person,

}

}

```rs

```

Rust 中的 `#[derive(Default)]` 宏与以下代码等价。

```rs

```

#![allow(unused)] fn main() { struct Person {

    age: i32,

}

impl Default for Person {

    fn default() -> Self {

        Self {

            age: Default::default()

        }

    }

}

struct Student {

    person: Person,

}

impl Default for Student {

    fn default() -> Self {

        Self {

            person: Default::default()

        }

    }

}

}

```rs

```

与 C++ 不同，在 C++ 中整数的默认初始化值是不确定的，而在 Rust 中，原始整数和浮点类型的默认值[是零](https://doc.rust-lang.org/std/primitive.i32.html#impl-Default-for-i32)。

继承 `Default` 特性对代码简洁性的影响与 C++ 中省略初始化类似。在所有类型都实现了 `Default` 特性，但只有一些字段应该有默认值的情况下，可以使用 [结构体更新语法](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax) 来定义一个构造方法，而不必枚举所有字段的值。

```rs

```

#![allow(unused)] fn main() { #[derive(Default)]

struct Person {

    age: i32,

}

#[derive(Default)]

struct Student {

    person: Person,

    favorite_color: Option<String>,

}

impl Student {

    pub fn with_favorite_color(color: String) -> Self {

        Student {

            favorite_color: Some(color),

            ..Default::default()

        }

    }

}

}

```rs

```

结构体更新语法的性能[取决于优化器](https://godbolt.org/#z:OYLghAFBqd5TKALEBjA9gEwKYFFMCWALugE4A0BIEAZgQDbYB2AhgLbYgDkAjF%2BTXRMiAZVQtGIHgBYBQogFUAztgAKAD24AGfgCsp5eiyahSAVyVFyKxqiIEh1ZpgDC6embZMQAJnLOAGQImbAA5TwAjbFIpAE5yAAd0JWIHJjcPL19E5NShIJDwtiiYnnibbDs0kSIWUiIMz28ea2xbeyEauqICsMjog0tuxqyWofreopKpAEprdDNSVE4uAFIfAGZVgFYAIRxSAgA3bAgAEWwaFjN6Ihmds9WtAEEEswiAaktzOw/VaKUQg%2BqwA7Lsns8PlCPixgJwPgQNn4IaDHi8UZsdvtosdThcrjc7g8IW9Pt8zL8amYcMJgWCIdCPgkAY4/iymOQGdCrkcyMRsAB9DD0MggD4AeQSHSYqw2LhqhxMstwnPRILRzwhBDYCXoHypNKIdPBL0ZpI%2BNCYHwA7sQkAKeXyiILhWQIK6YvqiIrgDMPgBaZX6to0Y1cxlQg3MI2gk2QiMJx2HZ1C9yi/XoDjutOkObhhPQgB0hfx11uIBAOAJtwg91NBdR%2BcbarOXDm9G42343i4OnI6G4ACULEbAYtlsDNnxyERtG25gBrEDbLSGbjSfhsZernt9gdcfhKECr2e9tvkOCwFAYHUMaKUag3hJ3mKkHgADhBq7otwB1Aic7kBEwR1AAntw07AawpCgeKES6JUp7TjeHDCOKTD0OBZ7kDgERmMALgSPQR68PwOBsMYwCSNhhCkIhuIkX22DqJUZjOhB/DBM6HbYfQBARKQYFuDggHetqHHkCcpARMk2AXBRJh8SYc5zDQRjAEoABqBDYNakrMBJgjCGIEicDIcjCMoaiaNh%2BgtEYynmJYhj8UekBzOgUppCR/ouB8nlEP6jAnPQspnD4/DoFJhw4G5tatO0aROEwrjuE0Uh%2BIEwR9MUAzvjkKTSiMzR%2BEkhVpJM/QxPlFRVJ0tT1MVGUJfR9XdJVuXVdYDUNGlox%2BOMPTZVMeVzGOSxme2nbdoB%2B4fE5RCoB8PCFp%2BhZaB8ED4MQZCThsPAzPwp46DMcxINgLAHNQi7bmuXAbuQu6Rdwh7HjOKlTVwEXkFu0jbOtWjSGU748NsPggz4IIAGyPbNL3vWecyXgg8AQNeWB4IQJAUFQtB3qwHCGfIJmSOZRmKCoGiAfofgOaYI4tXV3gQM4TUtFlhRVVILRlXk6R9c0BV8x10w8GMbStUwXSNQLgwS0z0tDZznXc91wyy2MPUiwMYtjQsE3cOsWx7AcuLnJcZZEtsGrmuSvz/KQgJWrG%2BawvCiLIi2KLopiJs4ic5vVlbNvvF83oUkaUa0i79ZQsyjusg7TuqvGUJJvyqYip6krSrK8piUqGwqii6re882q6l61LRmGsdMqHlo2naDosLyyYujm2ZZ2KCrBL6AZBiIIa16nEaMEabBsV8wIbGcwb0DQFZVpbtaynGBZfIW6cph6M9zyImanB69wbOvBZKE2pctuePFdrD2H7sOlhfPrE5G99x3zuQ52XQM8VLiuO6D0twbBBIWbY989zwyPCeFSF5EBozQJmZ8jAcaPmQS%2BEAb5PzfgYM6R2/5AJQTAhJYhME4IITsBJFC0Z0KYUArhfChF6DEQkuRSi1E%2By0XoicRi/BmKsXYqRSgwg2iAT4gJISGNRKHC3MIqSMkVDyQ4X3OBalYRaR0npZkPZpzkxJmZWQ5MrJU1stkOmWCGYSLih5LyQgfJ%2BQCkFbAIUwrfSitEGK2AbGM2lMlVKmRBYcxyqLIWRUNZhIqsNLm4tEptRloEuWcSpZa2iSrTW6tEkZImGk0WetxycB8J9O%2BT1%2BzcHmiOJaK01obS2ljXa79DoIxOmdC6V1/63R4g9Up%2B5XqwMRp9b6W5tjSELLEWI2x3xaA/NIEEH4QQ%2BD%2BpA56B5mlfyXDwLQq4eIbBmg/eGn9TqSQBElaQQA%3D%3D%3D).

## 数组值的隐式初始化

在 C++ 中，未显式初始化的数组使用默认构造函数进行默认初始化。

在 Rust 中，必须提供用于初始化数组的值。

```rs
class Person {
  int age;

public:
  Person() : age(0) {}
};

int main() {
  Person people[3];
  // ...
} 
```

```rs

```

#[derive(Default)]

struct Person {

    age: i32,

}

fn main() {

    // std::array::from_fn 提供回调的索引

    let people: [Person; 3] =

        std::array::from_fn(|_| Default::default());

    // ...

}

```rs

```

如果类型恰好是可以简单复制的，则可以使用简写。

```rs

```

#[derive(Clone, Copy, Default)]

struct Person {

    age: i32,

}

fn main() {

    let people: [Person; 3] = [Default::default(); 3];

    // ...

}

```rs

```

## 容器元素初始化

在 C++ 中，默认构造函数可以用于隐式定义集合类型，例如 `std::vector`。在 C++11 之前，一个值会被默认构造，然后元素会从这个初始元素复制构造。自 C++11 以来，所有元素都进行默认构造。

与数组初始化类似，在 Rust 中必须显式指定值。可以从数组构建向量，使其具有与数组相同的语法。

```rs
#include <vector>

class Person {
    int age;

public:
    Person() : age(0) {}
}

int main() {
    std::vector<Person> people(3);
    // ...
} 
```

```rs

```

#[derive(Default)]

struct Person {

    age: i32,

}

fn main() {

    let people_arr: [Person; 3] =

        std::array::from_fn(|_| Default::default());

    let people: Vec<Person> = Vec::from(people_arr);

    // ...

}

```rs

```

在 Rust 中，向量也可以从[迭代器](https://doc.rust-lang.org/book/ch13-02-iterators.html#methods-that-produce-other-iterators)中构建。

```rs

```

#[derive(Default)]

struct Person {

    age: i32,

}

fn main() {

    let people: Vec<Person> = (0..3).map(|_| Default::default()).collect();

    // ...

}

```rs

```

如果类型实现了 `Clone` 特性，则可以使用 `vec!` 宏来构造数组。有关 `Clone` 的更多详细信息，请参阅复制构造函数章节。

```rs

```

#[derive(Clone, Default)]

struct Person {

    age: i32,

}

fn main() {

    let people: Vec<Person> = vec![Default::default(); 3];

    // ...

}

```rs

```

## 隐式初始化局部变量

在 C++ 中，默认构造函数用于执行未显式初始化的局部变量的默认初始化。

在 Rust 中，局部变量的初始化始终是显式的。

```rs
class Person {
    int age;

public:
    Person() : age(0) {}
};

int main() {
    Person person;
    // ...
} 
```

```rs

```

#[derive(Clone, Default)]

struct Person {

    age: i32,

}

fn main() {

    let person = Person::default();

    // ...

}

```rs

```

## 基类对象的隐式初始化

在 C++ 中，如果没有指定其他构造函数，则使用默认构造函数来初始化基类对象。

```rs
class Base {
  int x;

public:
  Base() : x(0) {}
};

class Derived : Base {
public:
  // Calls the default constructor for Base
  Derived() {}
}; 
```

由于 Rust 没有继承，因此没有与此情况等效的内容。有关替代方案，请参阅实现重用章节或 Rust 书籍中关于[特质的章节](https://doc.rust-lang.org/book/ch10-02-traits.html)。

## `std::unique_ptr`

在 Rust 中，还有一些其他情况下使用 `Default` 特性，但在 C++ 中不使用默认构造函数进行初始化。

Rust 的智能指针通过委托给包含类型的 `Default` 实现来实现 `Default`。

```rs

```

#[derive(Default)]

struct Person {

    age: i32,

}

fn main() {

    let b: Box<Person> = Default::default();

    // ...

}

```rs

```

这与 C++ 中对 `std::unique_ptr` 的处理不同，因为与 `Box` 不同，`std::unique_ptr` 是可空的，因此 `std::unique_ptr` 的默认构造函数会产生一个不拥有任何内容的指针。Rust 中的等效类型是 `Option<Box<Person>>`，其 `Default` 实现产生 `None`。

## 其他 `Default` 的用法

[`Option::unwrap_or_default`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default) 方法利用了 `Default`，这使得在 `Option` 不包含值时获取默认值变得更加方便。

```rs
#include <optional>
#include <string>

void go(std::optional<std::string> x) {
  std::string a =
      x.or_else([]() {
         return std::make_optional<std::string>();
       }).value();
  // if x was nullopt, then a is ""

  // ...
} 
```

```rs

```

#![allow(unused)] fn main() { fn go(x: Option<String>) {

    let a: String = x.unwrap_or_default();

    // 如果 x 是 None，则 a 是 ""

    // ...

}

}

```rs

```

在 C++ 中，`std::optional::value_or` 并不等价，因为它总是会构造默认值，而不仅仅是当 `std::optional` 是 `std::nullopt` 时才构造默认值。等价的做法需要使用 `std::optional::or_else`。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Default constructors)
