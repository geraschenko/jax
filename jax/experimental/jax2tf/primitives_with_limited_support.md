# Primitives with limited support

*Last generated on (YYYY-MM-DD): 2020-09-10*

## Updating the documentation

This file is automatically generated by running the jax2tf tests in
`jax/experimental/jax2tf/tests/primitives_test.py` with the
`JAX2TF_OUTPUT_LIMITATIONS` environment variable set.

## Generated summary of primitives with limited support

The following JAX primitives are converted to Tensorflow but the result of the
conversion may trigger runtime errors when run on certain devices and with
certain data types. Note that some of these primitives have unimplemented
cases even for JAX, e.g., sort does not work for complex numbers on TPUs.
This table includes only those cases that **work** in JAX but **fail** after
conversion to Tensorflow.

| Affected primitive | Type of limitation | Description | Devices affected |
| --- | --- | --- | --- |
| acosh | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| acosh | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| add | Missing TF support | Primitive is unimplemented for dtype uint16 | CPU, GPU, TPU |
| add | Missing TF support | Primitive is unimplemented for dtype uint32 | CPU, GPU, TPU |
| asinh | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| asinh | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| atan2 | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU, TPU |
| atan2 | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| atanh | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| atanh | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| bessel_i0e | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| bessel_i1e | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| cosh | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| digamma | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| erf | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| erf_inv | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| erf_inv | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| erfc | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| lgamma | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| max | Missing TF support | Primitive is unimplemented for dtype bool | CPU, GPU, TPU |
| max | Missing TF support | Primitive is unimplemented for dtype complex64 | CPU, GPU, TPU |
| max | Missing TF support | Primitive is unimplemented for dtype int8 | CPU, GPU, TPU |
| max | Missing TF support | Primitive is unimplemented for dtype uint16 | CPU, GPU, TPU |
| max | Missing TF support | Primitive is unimplemented for dtype uint32 | CPU, GPU, TPU |
| min | Missing TF support | Primitive is unimplemented for dtype bool | CPU, GPU, TPU |
| min | Missing TF support | Primitive is unimplemented for dtype complex64 | CPU, GPU, TPU |
| min | Missing TF support | Primitive is unimplemented for dtype int8 | CPU, GPU, TPU |
| min | Missing TF support | Primitive is unimplemented for dtype uint16 | CPU, GPU, TPU |
| min | Missing TF support | Primitive is unimplemented for dtype uint32 | CPU, GPU, TPU |
| mul | Missing TF support | Primitive is unimplemented for dtype uint32 | CPU, GPU, TPU |
| nextafter | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU, TPU |
| nextafter | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| population_count | Missing TF support | Primitive is unimplemented for dtype uint32 | CPU, GPU, TPU |
| qr | Missing TF support | Primitive is unimplemented for dtype complex64 | CPU, GPU, TPU |
| reduce_window_sum | Missing TF support | Primitive is unimplemented for dtype uint16 | CPU, GPU, TPU |
| reduce_window_sum | Missing TF support | Primitive is unimplemented for dtype uint32 | CPU, GPU, TPU |
| rem | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU, TPU |
| rem | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| round | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| rsqrt | Missing TF support | Primitive is unimplemented for dtype bfloat16 | CPU, GPU |
| scatter-add | Missing TF support | Primitive is unimplemented for dtype complex64 | TPU |
| scatter-mul | Missing TF support | Primitive is unimplemented for dtype complex64 | TPU |
| select_and_gather_add | Missing TF support | Primitive is unimplemented for dtype float32 | TPU |
| sinh | Missing TF support | Primitive is unimplemented for dtype float16 | CPU, GPU, TPU |
| sort | Missing TF support | Primitive is unimplemented for dtype bool; sorting 2 arrays where the first one is an array of booleans is not supported for XlaSort | CPU, GPU, TPU |
| sort | Missing TF support | Primitive is unimplemented for dtype complex64 | CPU, GPU, TPU |
| sort | Missing TF support | Primitive is unimplemented; only sorting on last dimension is supported for XlaSort | CPU, GPU, TPU |
| sort | Missing TF support | Primitive is unimplemented; sorting more than 2 arrays is not supported for XlaSort | CPU, GPU, TPU |
| sort | Missing TF support | Primitive is unimplemented; stable sort not implemented for XlaSort | CPU, GPU, TPU |
| svd | Missing TF support | Primitive is unimplemented for dtype complex64; this works on JAX because JAX uses a custom implementation | CPU, GPU |

## Not yet implemented primitive conversions

The conversion of the following JAX primitives is not yet implemented:

`after_all`, `all_to_all`, `axis_index`, `cholesky`, `create_token`, `cummax`, `cummin`, `custom_linear_solve`, `eig`, `eigh`, `igamma_grad_a`, `infeed`, `lu`, `outfeed`, `pmax`, `pmin`, `ppermute`, `psum`, `random_gamma_grad`, `reduce`, `rng_uniform`, `triangular_solve`, `xla_pmap`