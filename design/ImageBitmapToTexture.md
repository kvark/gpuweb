# WebGPU + ImageBitmap

```webidl
dictionary GPUImageBitmapCopyView {
    required ImageBitmap imageBitmap;
    GPUOrigin2D origin;
};

partial interface GPUCommandEncoder {
    void copyImageBitmapToTexture(
        GPUImageBitmapCopyView source,
        GPUTextureCopyView destination,
        // For now, copySize.z must be 1.
        GPUExtent3D copySize);
};
```

`copyImageBitmapToTexture` encodes a copy from a source sub-rectangle of an `ImageBitmap` into a destination sub-resource of a `GPUTexture`.
When the command buffer is submitted, the `ImageBitmap` must not be detached.
If it is, a validation error is generated.

## Alternatives Considered

  * Creating a `GPUTexture` directly from an `ImageBitmap`, attempting to avoid copies, is impractical because it requires the GPUTexture's format to match the internal representation of the `ImageBitmap`, which is not exposed to the Web platform.
    Additionally, `ImageBitmap`s may be GPU- or CPU-backed, and wrapping a CPU-backed `ImageBitmap` is a significant meta-operation that requires an additional copy to be submitted.

## Issues

  * Some of the `ImageBitmap` creation options, such as `"flipY"`, have semantics that have to match the target graphics API where the data is intended to be used.
    For WebGL, `imageOrientation: "flipY"` is necessary to ensure that the resulting `WebGLTexture` is oriented correctly.
    For WebGPU, it may be the case that texture origins are defined differently from WebGL, necessitating `imageOrientation: "none"`.
    These cases will have to be thoroughly tested.

  * The browser may choose an internal representation for an `ImageBitmap` which is not ideal for usage by WebGPU (or, for that matter, [by WebGL](https://crbug.com/831740)).
    This could result in texture uploads being significantly more expensive than necessary due to per-pixel data swizzling during upload.
    Providing any hint about the intended usage of the `ImageBitmap` during its construction (for example "for use with WebGL" or "for use with this WebGPU adapter") would require changes to the HTML specification.
    Attempts to change Chrome's internal representation of `ImageBitmap` have not yet been successful; it's not clear how feasible it would be in other browsers.
