---
title: "Here, Have a Fractal"
layout: post
---

# Not feeling it

Today isn't great.
I don't like it.
So here, to take minds off things, have a fractal.

![Mandelbrot Set]({{ "/public/mandelbrot.png" | absolute_url }})

Ugly Rust code, using `image` and `num_complex`:


```rust
use image::{ImageBuffer, Rgb};
use num_complex::Complex;

fn main() {
    let img = 3200;
    let scale = 3.0 / img as f32;
    let max_iter = 42.0;
    let imgbuf = ImageBuffer::from_par_fn(img, img, |x, y| {
        let c = Complex::new(x as f32 * scale - 2.25, y as f32 * scale - 1.5);
        let (mut z, mut i) = (Complex::<f32>::ZERO, 0.0);
        while i < max_iter && z.norm_sqr() <= 16.0 {
            z = z * z + c;
            i += 1.0;
        }
        let n = (i as f32) + 1.0 - (0.5 * z.norm().ln()).log2();
        let h = (n * 360.0 / max_iter + 210.0) % 360.0;
        let v = (i < max_iter) as u8 as f32;
        let f = |n: f32| {
            let k = (n + (h / 60.0)) % 6.0;
            (255.0 * (v - 0.5 * v * 0f32.max(k.min(4.0 - k).min(1.0)))) as u8
        };
        Rgb([f(5.0), f(3.0), f(1.0)])
    });
    imgbuf.save("mandelbrot.png").unwrap();
}
```
