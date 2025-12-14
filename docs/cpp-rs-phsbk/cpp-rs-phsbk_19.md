# 空数组

> 原文：[`cel.cs.brown.edu/crp/idioms/null/zero_length_arrays.html`](https://cel.cs.brown.edu/crp/idioms/null/zero_length_arrays.html)

在以 C 风格编写的 C++代码库或使用 C 库的情况下，空指针可能被用来表示空数组。

在 Rust 中，任意大小的数组被表示为[slices](https://doc.rust-lang.org/book/ch04-03-slices.html)。这些切片可以有零长度。由于[Rust 向量可以转换为 slices](https://doc.rust-lang.org/std/vec/struct.Vec.html#impl-Deref-for-Vec%3CT,+A%3E)，定义与切片一起工作的函数使得它们也可以与向量一起使用。

```rs
#include <cstddef>
#include <cassert>

int c_style_sum(std::size_t len, int arr[]) {
    int sum = 0;
    for (size_t i = 0; i < len; i++) {
        sum += arr[i];
    }
    return sum;
}

int main() {
    int sum = c_style_sum(0, nullptr);
    assert(sum == 0);
} 
```

```rs

```

fn sum_slice(arr: &[i32]) -> i32 {

    let mut sum = 0;

    for x in arr {

        sum += x;

    }

    sum

}

fn main() {

    let sum = sum_slice(&[]);

    assert!(sum == 0);

    let sum2 = sum_slice(&vec![]);

    assert!(sum2 == 0);

}

```rs

```

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Zero-length arrays)
