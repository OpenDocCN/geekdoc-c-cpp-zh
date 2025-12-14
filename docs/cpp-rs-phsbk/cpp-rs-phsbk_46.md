# UnknownVar(&'a str),

> ) -> Result<i32, EvalError<'a>> {

body: Box<Exp>,

在 C++ 中，访问者模式通常用于在不修改类定义的情况下向类型添加行为。在 Rust 中，传统上通过使用 Rust 枚举来实现相同的目标，这些枚举类似于 C++ 的标记联合。虽然关于标记联合的章节比较了使用 Rust 枚举与 C++ `std::variant`，但本章比较了在 C++ 中使用访问者模式与使用 Rust 枚举。

在 C++ 中，有时会使用访问者模式来扩展数据和行为，而不修改原始定义（即解决[表达式问题](https://cs.brown.edu/~sk/Publications/Papers/Published/kff-synth-fp-oo/))。其他方法，得益于 Rust 的特性和泛型，更可能在 Rust 中使用（见变化的数据和行为）。

## Var(String),

let e = Let {

```rs
#include <exception>
#include <iostream>
#include <memory>
#include <string>
#include <unordered_map>

// Declare types that visitor can visit
class Lit;
class Plus;
class Var;
class Let;

// Define abstract class for visitor
struct Visitor {
  virtual void visit(Lit &e) = 0;
  virtual void visit(Plus &e) = 0;
  virtual void visit(Var &e) = 0;
  virtual void visit(Let &e) = 0;
  virtual ~Visitor() = default;

protected:
  Visitor() = default;
};

// Define abstract class for expressions
struct Exp {
  virtual void accept(Visitor &v) = 0;
  virtual ~Exp() = default;
};

// Implement each expression variant
struct Lit : public Exp {
  int value;

  Lit(int value) : value(value) {}

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

struct Plus : public Exp {
  std::unique_ptr<Exp> lhs;
  std::unique_ptr<Exp> rhs;

  Plus(std::unique_ptr<Exp> lhs,
       std::unique_ptr<Exp> rhs)
      : lhs(std::move(lhs)), rhs(std::move(rhs)) {
  }

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

struct Var : public Exp {
  std::string name;

  Var(std::string name) : name(name) {}

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

struct Let : public Exp {
  std::string name;
  std::unique_ptr<Exp> exp;
  std::unique_ptr<Exp> body;

  Let(std::string name, std::unique_ptr<Exp> exp,
      std::unique_ptr<Exp> body)
      : name(std::move(name)),
        exp(std::move(exp)),
        body(std::move(body)) {}

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

// Define Visitor for evaluating expressions

// Exception for representing expression
// evaluation errors
struct UnknownVar : std::exception {
  std::string name;

  UnknownVar(std::string name) : name(name) {}

  const char *what() const noexcept override {
    return "Unknown variable";
  }
};

// Define type for evaluation environment
using Env = std::unordered_map<std::string, int>;

// Define evaluator
struct EvalVisitor : public Visitor {
  // Return value. Results propagate up the stack.
  int value = 0;

  // Evaluation environment. Changes propagate
  // down the stack
  Env env;

  // Define behavior for each case of the
  // expression.
  void visit(Lit &e) override { value = e.value; }
  void visit(Plus &e) override {
    e.lhs->accept(*this);
    auto lhs = value;
    e.rhs->accept(*this);
    auto rhs = value;
    value = lhs + rhs;
  }
  void visit(Var &e) override {
    try {
      value = env.at(e.name);
    } catch (std::out_of_range &ex) {
      throw UnknownVar(e.name);
    }
  }
  void visit(Let &e) override {
    e.exp->accept(*this);
    auto orig_env = env;
    env[e.name] = value;
    e.body->accept(*this);
    env = orig_env;
  }
};

int main() {
  // Construct an expression
  auto x = Plus(std::make_unique<Let>(
                    std::string("x"),
                    std::make_unique<Lit>(3),
                    std::make_unique<Var>(
                        std::string("x"))),
                std::make_unique<Lit>(2));

  // Construct the evaluator
  EvalVisitor visitor;

  // Run the evaluator
  x.accept(visitor);

  // Print the output
  std::cout << visitor.value << std::endl;
} 
```

```rs

// 构造一个表达式

var: String,

// 运行评估器

}),

rhs: Box::new(Lit(2)),

use std::collections::HashMap;

    body: Box::new(Plus {

    //

    }

        .ok_or(EvalError::UnknownVar(x)),

        rhs: Box<Exp>,

    },

    let rv = eval(env, rhs)?;

        exp: Box<Exp>,

        Ok(lv + rv)

        }

    Plus {

}

对于第一种情况，其中变体是固定的但行为不是，Rust 中的惯用方法是将数据结构实现为枚举，而不是作为具有公共接口的许多结构体。这与在 C++ 中使用 `std::variant` 类似。

lhs: Box<Exp>,

Exp::Let { var, exp, body } => {

enum EvalError<'a> {

    }

}

// 定义评估环境类型

var: "x".to_string(),

// 定义评估器

如果出于某种原因仍然需要访问者模式，它可以像在 C++ 中那样实现。这可以使使用访问者模式的程序的直接移植更加可行。然而，基于枚举的实现仍然应该被优先考虑。

    // 这涵盖了前 3 个部分

    type Env<'a> = HashMap<&'a str, i32>;

Let {

    Exp::Lit(n) => Ok(*n),

        Exp::Var(x) => env

            .get(x.as_str())

            println!("{:?}", res);

            // 表示表达式的例外

        e: &'a Exp,

        }

            Exp::Plus { lhs, rhs } => {

            // 评估错误

            use Exp::*;

        }

        访问者

            let lv = eval(env, lhs)?;

            #[derive(Debug)]

            env.insert(var, val);

            .cloned()

        Lit(i32),

    }

};

fn main() {

    // 打印输出

    由于访问者模式和双重分派可能还有其他用途，还提供了一个示例的 Rust 访问者模式版本。

    // C++ 版本。

        原文：[`cel.cs.brown.edu/crp/patterns/visitor.html`](https://cel.cs.brown.edu/crp/patterns/visitor.html)

        env: &Env<'a>,

        // 定义表达式。

            enum Exp {

            },

        eval(&env, body)

    lhs: Box::new(Var("x".to_string())),

    let mut env = env.clone();

    let val = eval(env, exp)?;

    let res = eval(&HashMap::new(), &e);

    match e {

fn eval<'a>(

```

## exp: Box::new(Lit(3)),

访问者模式和双重分派

以下示例展示了如何使用 Rust 中的访问者实现与上一个示例相同的程序，但使用了 Rust。C++ 程序与上一个示例相同。

示例还演示了在 Rust 中使用双分派与 trait 对象。表达式表示为 `dyn Exp` trait 对象，它们接受一个 `dyn Visitor` trait 对象，然后调用访问者针对表达式类型的特定方法。

```rs
#include <exception>
#include <iostream>
#include <memory>
#include <string>
#include <unordered_map>

// Declare types that visitor can visit
class Lit;
class Plus;
class Var;
class Let;

// Define abstract class for visitor
struct Visitor {
  virtual void visit(Lit &e) = 0;
  virtual void visit(Plus &e) = 0;
  virtual void visit(Var &e) = 0;
  virtual void visit(Let &e) = 0;
  virtual ~Visitor() = default;

protected:
  Visitor() = default;
};

// Define abstract class for expressions
struct Exp {
  virtual void accept(Visitor &v) = 0;
  virtual ~Exp() = default;
};

// Implement each expression variant
struct Lit : public Exp {
  int value;

  Lit(int value) : value(value) {}

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

struct Plus : public Exp {
  std::unique_ptr<Exp> lhs;
  std::unique_ptr<Exp> rhs;

  Plus(std::unique_ptr<Exp> lhs,
       std::unique_ptr<Exp> rhs)
      : lhs(std::move(lhs)), rhs(std::move(rhs)) {
  }

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

struct Var : public Exp {
  std::string name;

  Var(std::string name) : name(name) {}

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

struct Let : public Exp {
  std::string name;
  std::unique_ptr<Exp> exp;
  std::unique_ptr<Exp> body;

  Let(std::string name, std::unique_ptr<Exp> exp,
      std::unique_ptr<Exp> body)
      : name(std::move(name)),
        exp(std::move(exp)),
        body(std::move(body)) {}

  void accept(Visitor &v) override {
    v.visit(*this);
  }
};

// Define Visitor for evaluating expressions

// Exception for representing expression
// evaluation errors
struct UnknownVar : std::exception {
  std::string name;

  UnknownVar(std::string name) : name(name) {}

  const char *what() const noexcept override {
    return "Unknown variable";
  }
};

// Define type for evaluation environment
using Env = std::unordered_map<std::string, int>;

// Define evaluator
struct EvalVisitor : public Visitor {
  // Return value. Results propagate up the stack.
  int value = 0;

  // Evaluation environment. Changes propagate
  // down the stack
  Env env;

  // Define behavior for each case of the
  // expression.
  void visit(Lit &e) override { value = e.value; }
  void visit(Plus &e) override {
    e.lhs->accept(*this);
    auto lhs = value;
    e.rhs->accept(*this);
    auto rhs = value;
    value = lhs + rhs;
  }
  void visit(Var &e) override {
    try {
      value = env.at(e.name);
    } catch (std::out_of_range &ex) {
      throw UnknownVar(e.name);
    }
  }
  void visit(Let &e) override {
    e.exp->accept(*this);
    auto orig_env = env;
    env[e.name] = value;
    e.body->accept(*this);
    env = orig_env;
  }
};

int main() {
  // Construct an expression
  auto x = Plus(std::make_unique<Let>(
                    std::string("x"),
                    std::make_unique<Lit>(3),
                    std::make_unique<Var>(
                        std::string("x"))),
                std::make_unique<Lit>(2));

  // Construct the evaluator
  EvalVisitor visitor;

  // Run the evaluator
  x.accept(visitor);

  // Print the output
  std::cout << visitor.value << std::endl;
} 
```

```rs

// 这**不是**一个惯用的翻译。这是

// 以下是一个使用 Rust 枚举的先前示例。

use std::collections::HashMap;

// 定义访问者可以访问的类型

struct Lit(i32);

struct Plus {

    lhs: Box<dyn Exp>,

    rhs: Box<dyn Exp>,

}

struct Var(String);

struct Let {

    name: String,

    exp: Box<dyn Exp>,

    body: Box<dyn Exp>,

}

// 定义表达式的 trait

trait Exp {

    // 类似于 C++ 不能有虚拟模板

    // 方法，Rust 不能有 trait 对象

    // 其中 trait 有泛型方法。

    //

    // 因此访问者必须

    // 可变以收集结果或

    // 接受方法必须专门化到一个

    // 具体的返回类型。

    fn accept<'a>(&'a self, v: &mut dyn Visitor<'a>);

}

// 定义访问者的 trait

trait Visitor<'a> {

    fn visit_lit(&mut self, e: &'a Lit);

    fn visit_plus(&mut self, e: &'a Plus);

    fn visit_var(&mut self, e: &'a Var);

    fn visit_let(&mut self, e: &'a Let);

}

// 为每个表达式变体实现 accept 行为

impl Exp for Lit {

    fn accept<'a>(&'a self, v: &mut (dyn Visitor<'a>));

        v.visit_lit(self);

    }

}

impl Exp for Plus {

    fn accept<'a>(&'a self, v: &mut dyn Visitor<'a>) {

        v.visit_plus(self);

    }

}

impl Exp for Var {

    fn accept<'a>(&'a self, v: &mut dyn Visitor<'a>) {

        v.visit_var(self);

    }

}

impl Exp for Let {

    fn accept<'a>(&'a self, v: &mut dyn Visitor<'a>) {

        v.visit_let(self);

    }

}

// 定义用于评估表达式的访问者

// 用于表示表达式评估的错误

// 错误。

//

// 因为它借用了一个生命周期参数

// 从表达式中获取名称。

#[derive(Debug)]

enum EvalError<'a> {

    UnknownVar(&'a str),

}

// 定义评估环境的类型

//

// 因为它借用了一个生命周期参数

// 从表达式中获取名称。

type Env<'a> = HashMap<&'a str, i32>;

// 定义评估器

struct EvalVisitor<'a> {

    // 返回值。结果会向上传播到堆栈。

    env: Env<'a>,

    // 评估环境。更改会传播

    // 向下传播到堆栈

    value: Result<i32, EvalError<'a>>,

}

// 定义每个情况的特定行为

// 表达式。

impl<'a> Visitor<'a> for EvalVisitor<'a> {

    fn visit_lit(&mut self, e: &'a Lit) {

        self.value = Ok(e.0);

    }

    fn visit_plus(&mut self, e: &'a Plus) {

        e.lhs.accept(self);

        let Ok(lv) = self.value else {

            return;

        };

        e.rhs.accept(self);

        let Ok(rv) = self.value else {

            return;

        };

        self.value = Ok(lv + rv);

    }

    fn visit_var(&mut self, e: &'a Var) {

        self.value = self

            .env

            .get(e.0.as_str())

            .ok_or(EvalError::UnknownVar(&e.0))

            .copied();

    }

    fn visit_let(&mut self, e: &'a Let) {

        e.exp.accept(self);

        let Ok(val) = self.value else {

            return;

        };

        let orig_env = self.env.clone();

        self.env.insert(e.name.as_ref(), val);

        e.body.accept(self);

        self.env = orig_env;

    }

}

fn main() {

    // Construct an expression

    let x = Plus {

        lhs: Box::new(Let {

            name: "x".to_string(),

            exp: Box::new(Lit(3)),

            body: Box::new(Var("x".to_string())),

        }),

        rhs: Box::new(Lit(2)),

    };

    // Construct the evaluator

    let mut visitor = EvalVisitor {

        value: Ok(0),

        env: HashMap::new(),

    };

    // Run the evaluator

    x.accept(&mut visitor);

    // Print the output

    println!("{:?}", visitor.value);

}

```

## Varying data and behavior

In C++, extensions to the visitor pattern are sometimes used to handle situations where both data and behavior and vary. However, those solutions also make use of dynamic casting. In Rust, that requires opting into RTTI by making `Any` a supertrait of the trait for the visitors, so they can be downcast. While this extension to the visitor pattern is possible to implement, the ergonomics of the approach make other approaches more common in Rust.

One of the alternative approaches, adopted from functional programming and leveraging the design of traits and generics in Rust, is called ["data types à la carte"](https://www.cambridge.org/core/services/aop-cambridge-core/content/view/14416CB20C4637164EA9F77097909409/S0956796808006758a.pdf/data-types-a-la-carte.pdf).

The following example shows a variation on the earlier examples using this pattern to make it so that two parts of the expression type can be defined separately and given evaluators separately. This approach can lead to performance problems (in large part due to the indirection through nested structures) or increases in compilation time, so its necessity should be carefully evaluated before it is used.

```rs

use std::collections::HashMap;

// A type for combining separately-defined

// expressions. Defining individual expressions

// completely separately and then using an

// application-specific sum type instead of nesting

// Sum can improve performance.

enum Sum<L, R> {

    Inl(L),

    Inr(R),

}

// Define arithmetic expressions

enum ArithExp<E> {

    Lit(i32),

    Plus { lhs: E, rhs: E },

}

// Define let bindings and variables

enum LetExp<E> {

    Var(String),

    Let { name: String, exp: E, body: E },

}

// Combine the expressions

type Sig<E> = Sum<ArithExp<E>, LetExp<E>>;

// Define the fixed-point for recursive

// expressions.

struct Exp(Sig<Box<Exp>>);

// Define an evaluator

// The evaluation environment

type Env<'a> = HashMap<&'a str, i32>;

// Evaluation errors

#[derive(Debug)]

enum EvalError<'a> {

    UndefinedVar(&'a str),

}

// A trait for expressions that can

// be evaluated.

trait Eval {

    fn eval<'a>(&'a self, env: &Env<'a>) -> Result<i32, EvalError<'a>>;

}

// Implement the evaluator trait for

// the administrative types

impl<L: Eval, R: Eval> Eval for Sum<L, R> {

    fn eval<'a>(&'a self, env: &Env<'a>) -> Result<i32, EvalError<'a>> {

        match self {

            Sum::Inl(left) => left.eval(env),

            Sum::Inr(right) => right.eval(env),

        }

    }

}

impl<E: Eval> Eval for Box<E> {

    fn eval<'a>(&'a self, env: &Env<'a>) -> Result<i32, EvalError<'a>> {

        self.as_ref().eval(env)

    }

}

// 为所需的变体实现特质的实现。

impl<E: Eval> Eval for ArithExp<E> {

    fn eval<'a>(&'a self, env: &Env<'a>) -> Result<i32, EvalError<'a>> {

        match self {

            ArithExp::Lit(n) => Ok(*n),

            ArithExp::Plus { lhs, rhs } => Ok(lhs.eval(env)? + rhs.eval(env)?),

        }

    }

}

impl<E: Eval> Eval for LetExp<E> {

    fn eval<'a>(&'a self, env: &Env<'a>) -> Result<i32, EvalError<'a>> {

        match self {

            LetExp::Var(x) => env

                .get(x.as_str())

                .copied()

                .ok_or(EvalError::UndefinedVar(x)),

            LetExp::Let { name, exp, body } => {

                let arg = exp.eval(env)?;

                let mut env = env.clone();

                env.insert(name, arg);

                body.eval(&env)

            }

        }

    }

}

// 由于特质为所有内容实现了

// 在 Exp 中，它可以针对 Exp 实现。

impl Eval for Exp {

    fn eval<'a>(&'a self, env: &Env<'a>) -> Result<i32, EvalError<'a>> {

        self.0.eval(env)

    }

}

// 构造表达式的辅助函数

fn lit(n: i32) -> Exp {

    Exp(Sum::Inl(ArithExp::Lit(n)))

}

fn plus(lhs: Exp, rhs: Exp) -> Exp {

    Exp(Sum::Inl(ArithExp::Plus {

        lhs: Box::new(lhs),

        rhs: Box::new(rhs),

    }))

}

fn var(x: &str) -> Exp {

    Exp(Sum::Inr(LetExp::Var(x.to_string())))

}

fn elet(name: &str, val: Exp, body: Exp) -> Exp {

    Exp(Sum::Inr(LetExp::Let {

        name: name.to_string(),

        exp: Box::new(val),

        body: Box::new(body),

    }))

}

fn main() {

    let e = elet("x", lit(3), plus(var("x"), lit(2)));

    println!("{:?}", e.eval(&HashMap::new()));

}

```

值得注意的是，上述实现中不需要动态分发。

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Visitor pattern and double dispatch)
