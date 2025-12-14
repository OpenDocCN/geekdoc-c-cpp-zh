# NRVO 和 RVO

> 原文：[`cel.cs.brown.edu/crp/idioms/rvo.html`](https://cel.cs.brown.edu/crp/idioms/rvo.html)

本章关于 Rust 的一些说法取决于编译器如何优化各种程序的具体细节。除非另有说明，此处展示的结果基于使用 rustc 1.87 的[2024 语言版本](https://doc.rust-lang.org/edition-guide/introduction.html)，以及 C++的`-O2`和 Rust 的`--opt-level=2`。

与 C++不同，Rust 不保证返回值优化（RVO）。两种语言都不保证命名返回值优化（NRVO）。然而，在 Rust 中，RVO 和 NRVO 通常会被应用，就像在 C++中一样。

## RVO

在静态工厂方法（Rust 称为构造方法）中，RVO 和 NRVO 可能最为重要。在以下示例中，C++17 及以后版本保证了 RVO。在 Rust 中，优化是可靠执行的，但并不保证。

```rs
struct Widget {
    signed char x;
    double y;
    long z;
};

Widget make(signed char x, double y, long z) {
    return Widget{x, y, z};
} 
```

```rs

#![allow(unused)] fn main() { struct Widget {

    x: i8,

    y: f64,

    z: i64,

}

impl Widget {

    fn new(x: i8, y: f64, z: i64) -> Self {

        Widget { x, y, z }

    }

}

}

```

可以在[汇编代码](https://godbolt.org/#z:OYLghAFBqd5TKALEBjA9gEwKYFFMCWALugE4A0BIEAZgQDbYB2AhgLbYgDkAjF%2BTXRMiAZVQtGIHgBYBQogFUAztgAKAD24AGfgCsp5eiyahSAVyVFyKxqiIEh1ZpgDC6embZMQANnLOAGQImbAA5TwAjbFIpAFZyAAd0JWIHJjcPL19E5NShIJDwtiiYnnibbDs0kSIWUiIMz28/CqqhGrqiArDI6LjrWvrGrJbBruCe4r6ygEprdDNSVE4uS3M7AGoAdQJMYGwiDYBSAHYAISOtAEEN243NDYIADnJLm7uATxANmh9ZN7uGwAXt8CH9XtdTgARN5vAhsBL0ba7faHU4Xa6Ao4AJgAzEdYmczEwlCwaNgIEx0AB9NjGYCMGYEmGYu40JgbEIAdwgD2e5A2Xx%2B4OBoL%2BMw2AFojrjcMi9gdjucAYC7jsFWjlazVar1BD3jrAR99YbVUCTYboSrblbIScWVcuHN6NxYvxvFwdOR0NwAEoWQ5KBZLbDHPF8chEbROuYAaxAsWxADppD4TrEfLingBOHM8GS4wzcaT8NgJrTkD1en1cfhKEAVqOep3kOCwFAYHD4YhkSjUOiMVgcbgRwTCMQSTgyOTCZRqTTN8j6bGGeloHEXbFnazYWz2RwQZzDbwrwITIolEDZnIpffpdxNEArpK3tLdC99a%2BtO8dIYPrI8Due7VGM769KUAydMeBiWJ0YFTKUcxBosyzcGsZibOqqJKhiBobCkwAhJgGyoEgdT3DKuGApgCwRIwgqUdaGz0EIwDAoxdowriuFvNg6hENEHJYYqdKxhSBFESRZGkPcAo0WYdGhsazGscCErokxpAHIsQkogc6J6oKApAla3GwvaLYulwbqVtG3rcC4G4bvhwbLGGuLYvwTY6DMcxINgLA4DEEDOsWpblrZi41nWDaRtGvnkPGiYpmmGZZrm2b5tIhZWbi7p2dFcXNnMbYIPAEAdugCIMNEfYQBg1WMDEpA8E8JwVgOAmkPWEARHZETBHUXy8PwA2sKQHwAPIRLolRNhGDUcMIk1MPQw1ejgERmMALgSPQ9YjeQOB0iYkiLoQWlVAAbtgB1enxlRmAJdnBAJVlevQBARKQQ1uDgdlEKQ8IjvwN2kBEyTYFC2AnQywSgMVAhGMASgAGoENgXKTQkzAgzOojiJI05jooKgaHZ%2BiAUYJggOYliGF99aQHM6AJHeB2SpKLgbKzRCSowN30DKUKed6YNAzgTMhUBc1pE4TCuP%2B3g8KeCvwZefgvnk96ZMrz65He6t9C0u6y%2B0YzQSrMttEwv7jIU4HZLBf661IK7O/bkwa0hrlTqF1n5VF3AbHTRCoBsPBJm1SZaBsEDdiQMk4riPAzF58WtigCxEAkT11Q1CQ1aQoTsCsofh5H0f3YQie7AYJMTkTsgk3O5OLpT5Bcj9CQg/7NlVvwNaTU9ueHOgNAhwGFdRycMdx24jXRO5qfp8VcYgNIs%2BxB52U%2BDwWj708TyxC8VkluQZaxBWA/2bW1ixd5LalRVaBVYXTX52/RcgMAZSAZ10Qep9UXGNIaeNQETWmrNOweNFrMCICtNadlNrbV2vQfaeNjr0jOhtAgl17A3TuvwB6qAnorAjK9XcdlPrfV%2BlgFYXpAbA0OmDCGKhoaw0%2BjTRGNBkZowxljHGHpRzyEblOZu8hW4Li9MuVcNNUBOS3AzCIUsWZszSBzSwmBhYKK3BuPEUpJqi3QOLXYt14BIVNjbeWitXaATPA7BCBgtZ3ktjebWRsILfhAlBJWMErE/lAueR2gEPZuI9p4qQPsUKcGxH3QO1Zg7qCeD4SUfwNjAFQBXWISYeBxwTmQdycTV4%2BUziAbOo9P6L2LqXbgyTUnpMydk3JxCa5kDriuBuhNxH4ykRTJ8ndu69yLAHSKiSuDDxzk9HmE96lpOkBkrJEccl5IgAvd%2BS9k7FKKj5PyAUgrUH9ufS%2B18CrcBio2eK69N5Jm3tiXe%2B9D7H1PtwPKYzB7nJ2TGEZosb6FUfglMGKRHDSCAA%3D%3D)中看到，对于这两个程序，值都是直接写入调用者提供的目标地址。

```rs
// C++
make(signed char, double, long):
        mov     BYTE PTR [rdi], sil
        mov     rax, rdi
        mov     QWORD PTR [rdi+16], rdx
        movsd   QWORD PTR [rdi+8], xmm0
        ret 
```

```rs
// Rust
new:
        mov     rax, rdi
        mov     byte ptr [rdi + 16], sil
        movsd   qword ptr [rdi], xmm0
        mov     qword ptr [rdi + 8], rdx
        ret 
```

## NRVO

在 C++或 Rust 中，NRVO（不重复赋值优化）并不保证，但优化通常在通常期望的情况下触发。例如，在 C++和 Rust 中创建数组、初始化其内容然后返回时，初始化会直接赋值到返回位置。

```rs
#include <array>

std::array<int, 10> make() {
    std::array<int, 10> v;
    for (int i = 0; i < 10; i++) {
        v[i] = i;
    }
    return v;
} 
```

```rs

#![allow(unused)] fn main() { #[unsafe(no_mangle)]

fn new() -> [i32; 10] {

    let mut v = [0; 10];

    for i in 0..10 {

        v[i] = i as i32;

    }

    v

}

}

```

两个程序版本的 [生成的汇编代码](https://godbolt.org/#z:OYLghAFBqd5TKALEBjA9gEwKYFFMCWALugE4A0BIEAZgQDbYB2AhgLbYgDkAjF%2BTXRMiAZVQtGIHgBYBQogFUAztgAKAD24AGfgCsp5eiyahSAVyVFyKxqiIEh1ZpgDC6embZMDzgDIEmbAA5TwAjbFIQAE5yAAd0JWIHJjcPLwN4xPshf0CQtnDImJtsO2SRIhZSIlTPbx5rbFtspgqqolzgsIjo60rq2vSGy3bO/MLogEprdDNSVE4uAFIAJgBmJYBWACEzJiUWGmwIJnQAfTZjYEZJrYARJa0AQRomAGpAgHcISbeAWiWa1wby22wIaxWgO2bx4WnuIIA7NtHk83mi3owiG82GYsQA3EFrO4gnZwtbQ2H3KEo9FvQSkN4ERnvLQAOlZsMRyOetNpeNBBHhgOJTJYSkZEOpPPRSwRD2laP5z1l8qeXGm9G4m343i4OnI6G4ACULFilLN5tgQes%2BOQiNp1dMANYgTYrVnSABsCM2nrWAA4ooGeDI1oZuNJ%2BGxXVpyLr9YauPwlCBY/a9eryHBYCgMDh8MQyJRqHRGKwONxbYJhGIJJwZHJhMo1JoM%2BR9CtDFc0KtkStto1msknExXO46iA1ptyH4Al0Cj1p5kkkJBt4p3EEiumGNupFpyUykI2gNx%2BkN4eWieOnPxou%2Bu015OD/0b3k967pua5gtuKs1gEqAeDghIuFUpAsAAnoCuAoiiliYCAIDgVBgIuAEVgwmSwKXE6xy/LK3KouiCFISh0FrOhwjkFhMFvPy5I0ui9JvBAGGMoSxJktCTJobR5KMr2vYEUiTG8vRApCkSgmMQqiKqrSpDYEQczvAxREqpmmpcNqcYOga3AuEJ/ZvN%2BlrWhC/DpjokzTEg2AsDgkQ/OGXCRuQ0abLG8b8ImyapnaDq2eQLpuh63q%2BgGQZRCG0hhtpaw6vpfmBRm0zZgg8AQLm6BsLEDARMWEAYHlBWRKQPD%2BgisalkQEQphAoT6aEARVJBlb8C1rCkJBADyoS6KU6a2iVHDCL1TD0O1bY4KEZjAGB9D0CmvD8DglwmJIM0EEpZR4tgK36tg6ilLiiz6hhTT6fQBChBBPVuDg%2BlEKQBDRqt5D7aQoQJNgdzYBt1wBKAaUCEYwBKAAagQ2CfL1sTMB1jaiOIkgNtWigqBo%2Bn6A0RgmCA5iWIYt0ppA0zoLELQrX8fwuG8lNEH8jD7fQworL5X2vTgZMuZew4QM4T6ejOo67gukQNsuLTC5uWTJOLEwNvzx6vrLKutK%2Bis9Mratnt4IsjNU2uS9IX4Wr%2BPAalqSVtombxE0QqAwqyVWslorEFiQDL/lbVlBVmKCzEQsS4kVJX5YwpBBOwiyO87PCuwi7v8NghDewQiENBjtZo7IGPNtjba4%2BQnwQbEHXWzptsJtwvW4qHWLoDQDumgnSfu6xbilVHFl%2B6lNnOiA0jJ5sEJxZ6sKwv6/qbP6rnuZ53nJdw/lpgHGXZWguWR4VVDFTvZUgMAPCbNnDB1aQDVNW2XVtUjd89f1g12Ejo3MEQE1Tfps3zYty1I3WlcLa%2BpCC7XsPtQ6qcTqoDOkjS62l9Q3Tum1R650rKvXeraL6P0VD/UBjdAmoMaDgyhjDOGCNdRVnkLnes%2Bd5CF1bPqDsXYCaoGMgOZBvMKZU2SDTBCwp2H9l7Osf4vUOYGi5pnA68AvxNCGgLIW%2BsQAIlFpgE2IARbS2SE%2BVR2ici3g/IbeRR5NaPmUaojW14NGGz1mkbwlitaGIlpo82P5OArCrrpHyBkuBvHUP6T0fxPTSDeMAVACdNgck9unMgFlPH%2B1BlvYOjdw6HyjjHCsXAAlBJCWEiJMIom2jToWbmIBOw51RnQ5GjCcblNLuXSurlvEry4PXEOuIGYtxycE0J4TInRIgN3XePt1gJIHo6cg9lHI9BctpReMY9J21XtYAK1lJkuhHqyMeKwJ5Ty0DPOerlEpLNrkmCZwVtISJ8SldZwUvqJEcNIIAA%3D%3D) 几乎相同，并且两者都直接在返回位置构建数组。

```rs
// C++
make():
        movdqa  xmm0, XMMWORD PTR .LC0[rip]
        mov     rdx, QWORD PTR .LC2[rip]
        mov     rax, rdi
        movups  XMMWORD PTR [rdi], xmm0
        movdqa  xmm0, XMMWORD PTR .LC1[rip]
        mov     QWORD PTR [rdi+32], rdx
        movups  XMMWORD PTR [rdi+16], xmm0
        ret
.LC0:
        .long   0
        .long   1
        .long   2
        .long   3
.LC1:
        .long   4
        .long   5
        .long   6
        .long   7
.LC2:
        .long   8
        .long   9 
```

```rs
// Rust
.LCPI0_0:
        .long   0
        .long   1
        .long   2
        .long   3
.LCPI0_1:
        .long   4
        .long   5
        .long   6
        .long   7
new:
        mov     rax, rdi
        movaps  xmm0, xmmword ptr [rip + .LCPI0_0]
        movups  xmmword ptr [rdi], xmm0
        movaps  xmm0, xmmword ptr [rip + .LCPI0_1]
        movups  xmmword ptr [rdi + 16], xmm0
        movabs  rcx, 38654705672
        mov     qword ptr [rdi + 32], rcx
        ret 
```

## 确定优化是否发生

可以使用像 [cargo-show-asm](https://github.com/pacak/cargo-show-asm) 这样的工具来显示单个函数的汇编代码，以便确认是否在所需位置应用了 RVO 或 NRVO。

Rust 也有高质量的 [基准测试工具](https://bheisler.github.io/criterion.rs/book/index.html)，可以用来确保更改不会意外地导致性能下降。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=NRVO and RVO)
