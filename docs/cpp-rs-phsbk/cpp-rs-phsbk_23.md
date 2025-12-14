# 私有成员和朋友

> 原文：[`cel.cs.brown.edu/crp/idioms/encapsulation/private_and_friends.html`](https://cel.cs.brown.edu/crp/idioms/encapsulation/private_and_friends.html)

## 私有成员

在 C++ 中，封装的单位是类。访问指定符（`private`、`protected` 和 `public`）控制成员的访问，并在类边界上强制执行。

在 Rust 中，模块是封装的单位。项目可见性（Rust 的访问指定符类似物）控制模块边界上的项目访问。

```rs
#include <iostream>
#include <string>

class Person {
  int age;

public:
  std::string name;

  // Because age is private, a public constructor
  // method is needed to create instances.
  Person(std::string name, int age)
      : name(name), age(age) {}

  // Free functions cannot access private members,
  // so this has to be a member function.
  static void example() {
    Person alice{"Alice", 42};
    std::cout << alice.name << std::endl;
    // The private field is visible here, within
    // the class.
    std::cout << alice.age << std::endl;
  }
};

int main() {
  Person alice("Alice", 42);
  std::cout << alice.name << std::endl;
  // compilation error
  // std::cout << alice.age << std::endl;
} 
```

```rs

```

mod person {

    pub struct Person {

        pub name: String,

        // 此字段是私有的

        age: i32,

    }

    impl Person {

        // 因为年龄是私有的，需要一个公共

        // 构造方法需要创建

        // 模块外部的值。

        pub fn new(

            name: String,

            age: i32,

        ) -> Person {

            Person { name, age }

        }

    }

    // 同一模块中的自由函数可以在

    // 访问私有字段，因为访问的

    // 封装是模块，而不是

    // 结构体。

    fn example() {

        let alice =

            Person::new("Alice".to_string(), 42);

        println!("{}", alice.name);

        // 私有字段在这里可见，

        // 在模块内部。

        println!("{}", alice.age);

    }

}

use person::Person;

fn main() {

    let alice =

        Person::new("Alice".to_string(), 42);

    println!("{}", alice.name);

    // 编译错误

    // println!("{}", alice.age);

}

```rs

```

在 Rust 的示例中，`Person` 的构造函数是私有的因为其中一个字段是私有的。

## 朋友

因为在 Rust 中封装是在模块级别，因此类型的相关方法可以访问同一模块中定义的其他类型的内部。这涵盖了 C++ `friend` 声明的大多数用途。

例如，在 C++ 中定义二叉树需要表示树节点的类将主二叉树类声明为友元，以便它可以访问内部方法，同时将它们对其他使用保持私有。即使 `TreeNode` 类被定义为 `BinaryTree` 的内部类，也需要这样做。

然而，在 Rust 中，这两种类型可以在同一个模块中定义，并且可以访问彼此的私有字段和方法。整个模块提供了一组类型、方法和函数，共同定义了一个封装的概念。

```rs
#include <memory>

class BinaryTree {
  // This needs to be an inner class in order for
  // it to be private.
  class TreeNode {
    friend class BinaryTree;

    int value;
    std::unique_ptr<TreeNode> left;
    std::unique_ptr<TreeNode> right;

  public:
    TreeNode(int value)
        : value(value), left(nullptr),
          right(nullptr) {}

  private:
    static void
    insert(std::unique_ptr<TreeNode> &node,
           int value) {
      if (node) {
        node->insert(value);
      } else {
        node = std::make_unique<TreeNode>(value);
      }
    }

    void insert(int value) {
      if (value < this->value) {
        insert(this->left, value);
      } else {
        insert(this->right, value);
      }
    }
  };

  std::unique_ptr<TreeNode> root;

public:
  BinaryTree() : root(nullptr) {}

  void insert(int value) {
    TreeNode::insert(root, value);
  }
};

int main() {
  BinaryTree b;
  b.insert(42);

  return 0;
} 
```

```rs

```

mod binary_tree {

    pub struct BinaryTree {

        // 此字段在模块外部不可见

        // 在模块内部。

        root: Option<Box<TreeNode>>,

    }

    impl BinaryTree {

        pub fn new() -> BinaryTree {

            BinaryTree { root: None }

        }

        pub fn insert(&mut self, value: i32) {

            insert(&mut self.root, value);

        }

    }

    // 此结构和所有其字段均不可见

    // 在模块外部可见。

    struct TreeNode {

        value: i32,

        left: Option<Box<TreeNode>>,

        right: Option<Box<TreeNode>>,

    }

    impl TreeNode {

        fn new(value: i32) -> TreeNode {

            TreeNode {

                value,

                left: None,

                right: None,

            }

        }

        fn insert(&mut self, value: i32) {

            if value < self.value {

                insert(&mut self.left, value);

            } else {

                insert(&mut self.right, value);

            }

        }

    }

    // 此自由函数在模块外部不可见

    // 在模块外部可见。

    fn insert(

        node: &mut Option<Box<TreeNode>>,

        value: i32,

    ) {

        match node {

            None => {

                *node = Some(Box::new(

                    TreeNode::new(value),

                ));

            }

            Some(ref mut left) => {

                left.insert(value);

            }

        }

    }

}

// 这将（公开）类型引入作用域。

use binary_tree::BinaryTree;

fn main() {

    let mut b = BinaryTree::new();

    b.insert(42);

}

```rs

```

## Passkey 习语

在之前的 C++ 示例中，`TreeNode` 构造函数必须公开才能与 `make_unique` 一起使用。幸运的是，构造函数仍然在包含类之外不可访问，但并非所有辅助类都可以是内部类。

要使构造函数在不可能的情况下实际上私有，可能需要使用像 [Passkey 习语](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/patterns/passkey.md) 这样的编程模式。

Passkey 习语有时也用于提供比友元声明更细粒度的对成员的访问控制。在任一情况下，效果是通过模拟一个能力类系统来实现的。

在 Rust 中，可以通过相同的习语来表达相同的效果，以实现相同的效果。

```rs
#include <iostream>
#include <memory>
#include <string>

class Person {
  int age;

  class Passkey {};

public:
  std::string name;

  Person(Passkey, std::string name, int age)
      : name(name), age(age) {}

  static std::unique_ptr<Person>
  createPerson(std::string name, int age) {
    // Other uses of make_unique are not possible
    // because the Passkey type cannot be
    // constructed.
    return std::make_unique<Person>(Passkey(),
                                    name, age);
  }
}; 
```

```rs

```

pub trait Maker<K, B> {

    fn make(passkey: K, args: B) -> Self;

}

// 我们希望能够调用的泛型辅助器。

// 一个否则私有的函数或方法。

fn alloc_thing<K, B, T: Maker<K, B>>(

    passkey: K,

    args: B,

) -> Box<T> {

    Box::new(Maker::<K, B>::make(passkey, args))

}

mod person {

    use super::*;

    use std::marker::PhantomData;

    pub struct Person {

        pub name: String,

        age: u32,

    }

    // 一个零大小的类型，用作 passkey。

    pub struct Passkey {

        // 此字段为零大小。它也是

        // 私有，这阻止了构造

        // 的人之外的 Passkey

        // 模块。

        _phantom: PhantomData<()>,

    }

    impl Person {

        // 私有方法将被公开

        // 使用 passkey 包装器。

        fn new(name: String, age: u32) -> Person {

            Person { name, age }

        }

        // 使用外部辅助器的方法的实现

        // 需要访问另一个

        // 否则私有的方法。

        fn alloc(

            name: String,

            age: u32,

        ) -> Box<Person> {

            alloc_thing(

                Passkey {

                    _phantom: PhantomData {},

                },

                MakePersonArgs { name, age },

            )

        }

    }

    // 需要辅助结构来使特性

    // 提供接口泛型。

    pub struct MakePersonArgs {

        pub name: String,

        pub age: u32,

    }

    // 实现该特性，暴露

    // 需要 passkey 的方法。

    impl Maker<Passkey, MakePersonArgs> for Person {

        fn make(

            _passkey: Passkey,

            args: MakePersonArgs,

        ) -> Person {

            Person::new(args.name, args.age)

        }

    }

}

fn main() {}

```rs

```

然而，Passkey 习语在 Rust 中可能不太可能被使用，因为

+   配对的类型通常定义在同一个模块中（或可以使用 `pub (in path)` 声明），这使得它变得不必要，并且

+   它需要接口的合作，通过该接口调用函数将使用一个类型。

第二点与上面使用`std::make_unique`的情况形成对比，后者能够在`std::make_unique`定义点不知道其存在的情况下转发到底层的构造函数。虽然下面的示例并不实用（因为`alloc_thing`不是一个有用的辅助函数），但它确实展示了为了达到使用 C++中惯用语的相同效果，需要定义哪些类型。

## 友元和测试

另一种常见的友元声明的用途是使类的内部结构对单元测试可用。尽管这种做法在 C++中通常被劝阻，但在测试其他情况下私有的辅助内部类或辅助方法时，有时是必要的。

在 Rust 中，测试通常定义在正在测试的代码相同的模块中。由于模块的内容对子模块是可见的，这使得模块的所有内容都可以用于测试。

```rs
// Using Boost.Test
// https://www.boost.org/doc/libs/1_84_0/libs/test/doc/html/index.html
#include <string>

class Person {
public:
  std::string name;

private:
  int age;

  friend class PersonTest;

public:
  Person(std::string name, int age)
      : name(name), age(age) {}

  void have_birthday() {
    this->age = this->age + 1;
  }
};

#define BOOST_TEST_MODULE PersonTestModule
#include <boost/test/included/unit_test.hpp>

class PersonTest {
public:
  static void test_have_birthday() {
    Person alice("Alice", 42);
    BOOST_CHECK_EQUAL(alice.age, 42);

    alice.have_birthday();
    BOOST_CHECK_EQUAL(alice.age, 43);
  }
};

BOOST_AUTO_TEST_CASE(have_birthday_test) {
  PersonTest::test_have_birthday();
} 
```

```rs

```

#![allow(unused)] fn main() { pub struct Person {

    pub name: String,

    age: u32,

}

impl Person {

    pub fn new(name: String, age: u32) -> Person {

        Person { name, age }

    }

    pub fn have_birthday(&mut self) {

        self.age = self.age + 1;

    }

}

#[cfg(test)]

mod test {

    use super::Person;

    #[test]

    fn test_have_birthday() {

        let mut alice =

            Person::new("alice".to_string(), 42);

        assert_eq!(alice.age, 42);

        alice.have_birthday();

        assert_eq!(alice.age, 43);

    }

}

}

```rs

```

## Rust 特质方法的可见性

因为 Rust 中的特质旨在定义接口，所以由特质声明的某些类型的声明方法，在特质的类型都可见时是可见的。换句话说，不可能有私有的特质方法。

Rust 中特质方法的默认可见性与 Rust 结构体不同，其中默认可见性是定义模块的私有。

## 私有构造函数和友元

在 C++中，可以通过将所有构造函数设为私有，然后声明可能从它派生的类为友元，来控制哪些类可以从特定类派生。

在 Rust 中，可以通过使用[密封特质模式](https://predr.ag/blog/definitive-guide-to-sealed-traits-in-rust/)来实现控制哪些类型可以实现特质的类似目标。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Private members and friends)
