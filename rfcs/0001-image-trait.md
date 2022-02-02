
* RFC PR: [rust-cv/rfcs#N](https://github.com/rust-cv/rfcs/pull/N)
* Tracking Issue [rust-cv/rfcs#N](https://github.com/rust-cv/rfcs/issues/N)

# Summary

An `Image` trait to allow interoperability of image data.

# Motivation

Multiple existing crates implement concrete types that hold image data either
explicitly or implicitly. If each of these implemented a single, fundamental
trait that allowed specification of how to access image data across
implementations, image operations could be written generically that could
operate on any of the data types. This would enable, for example, image
processing routines to be written which would work equally well regardless of
how the data types is defined.

# Prior Art

## [`machine_vision_formats::ImageStride` v0.1.1](https://docs.rs/machine-vision-formats/0.1.1/machine_vision_formats/trait.ImageStride.html)

Here we consider
[`machine_vision_formats::ImageStride`](https://docs.rs/machine-vision-formats/0.1.1/machine_vision_formats/trait.ImageStride.html)
together with its supertraits
[`machine_vision_formats::ImageData`](https://docs.rs/machine-vision-formats/0.1.1/machine_vision_formats/trait.ImageData.html)
and
[`machine_vision_formats::Stride`](https://docs.rs/machine-vision-formats/0.1.1/machine_vision_formats/trait.Stride.html).

### What's good

A simple mostly-trait only. Provides a "view" of existing structure and enum
definitions, allowing low-cost consumption of image data from a variety of
sources. This trait be implemented by crates which provide their own data types
that could be used to hold image data.

### What's not so good

No colorspace definition (e.g. sRGB vs linear RGB).

The version number will have to be bumped (although just unbreaking bumps) when new pixel formats are added.

Should there be a single fundamental trait instead of three? The present design
provides the possibility to implement `ImageData` for a type, and thus provide
access to the raw image data, without also implementing `Stride`. Does the
ability to allow access to raw image data and pixel format information, but
without providing information about stride,

(Minor nit) The static pixel format types are redundant with the dynamic types. The naming of the two could be improved.

### Where is this unidiomatic or unergonomic for Rust

The
[`width()`](https://docs.rs/machine-vision-formats/0.1.1/machine_vision_formats/trait.ImageData.html#tymethod.width)
and `height()` methods return `u32` but should perhaps return `usize`. (The [`stride()`](https://docs.rs/machine-vision-formats/0.1.1/machine_vision_formats/trait.Stride.html#tymethod.stride)
method does return `usize`.)

## [`ndarray_image::NdImage` v0.3.0](https://docs.rs/ndarray-image/0.3.0/ndarray_image/)

### What's good

### What's not so good

A struct of its own, this would require implementations of the
[`From`](https://doc.rust-lang.org/nightly/std/convert/trait.From.html) trait
for each supporting library, potentially incurring conversion overhead rather
than merely providing a view of existing data.

### Where is this unidiomatic or unergonomic for Rust


## [`nshare v0.0.0`](https://docs.rs/nshare/0.8.0/nshare/)

### What's good

### What's not so good

Not specialized for images, provides no definitions for pixel format or
colorspace.

### Where is this unidiomatic or unergonomic for Rust


# Use case

 - explicit image types (e.g.
   [`image::DynamicImage`](https://docs.rs/image/0.23.14/image/enum.DynamicImage.html))
 - implicit image types (e.g. when using
[`nalgebra::Matrix`](https://docs.rs/nalgebra/0.29.0/nalgebra/base/struct.Matrix.html)
to hold image data).

Use in `no_std` environments.

# Design


# Impact

We would want to define the base trait extremely carefully because (breaking)
version changes will cause widespread ripple it this trait because widely
implemented.

# Unresolved Questions

##  Utility for specifying batched GPU operations

## Dynamic vs. static pixel formats

When possible, it is useful to specify pixel formats statically (at compile
time) so that image processing functions can be specialized to operate on, for
example, images of a specific format (e.g. Mono8 or RGB8). However, often one
wants to process images from a dynamic source such as a file. Therefore, it
seems desirable to allow specifying pixel formats both at compile time and
dynamically (at run time).

## Colorspaces

To be maximally useful, we need to define pixel format (e.g. YUV packed, YUV
planar, RGB packed, RGB planar, Mono8, Mono10 packed, Mono 10 unpacked, Bayer
patterns) and also the colorspace (e.g. that of BT.709, sRGB, "undefined" and
perhaps "custom").

## Strides (Packed vs Planar representations)

## Hyperspectral data.

## Volume images (e.g. confocal microscopy).
