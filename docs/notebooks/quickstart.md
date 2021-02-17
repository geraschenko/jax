---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.10.0
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

+++ {"colab_type": "text", "id": "xtWX4x9DCF5_"}

# JAX Quickstart

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.sandbox.google.com/github/google/jax/blob/master/docs/notebooks/quickstart.ipynb)

**JAX is NumPy on the CPU, GPU, and TPU, with great automatic differentiation for high-performance machine learning research.**

With its updated version of [Autograd](https://github.com/hips/autograd), JAX
can automatically differentiate native Python and NumPy code. It can
differentiate through a large subset of Python’s features, including loops, ifs,
recursion, and closures, and it can even take derivatives of derivatives of
derivatives. It supports reverse-mode as well as forward-mode differentiation, and the two can be composed arbitrarily
to any order.

What’s new is that JAX uses
[XLA](https://www.tensorflow.org/xla)
to compile and run your NumPy code on accelerators, like GPUs and TPUs.
Compilation happens under the hood by default, with library calls getting
just-in-time compiled and executed. But JAX even lets you just-in-time compile
your own Python functions into XLA-optimized kernels using a one-function API.
Compilation and automatic differentiation can be composed arbitrarily, so you
can express sophisticated algorithms and get maximal performance without having
to leave Python.

```{code-cell}
:id: SY8mDvEvCGqk

import jax.numpy as jnp
from jax import grad, jit, vmap
from jax import random
```

```{code-cell}
:tags: [remove-cell]

# Execute this to consume & hide the GPU warning.
jnp.arange(10)
```

+++ {"colab_type": "text", "id": "FQ89jHCYfhpg"}

## Multiplying Matrices

+++ {"colab_type": "text", "id": "Xpy1dSgNqCP4"}

We'll be generating random data in the following examples. One big difference between NumPy and JAX is how you generate random numbers. For more details, see [Common Gotchas in JAX].

[Common Gotchas in JAX]: https://jax.readthedocs.io/en/latest/notebooks/Common_Gotchas_in_JAX.html#%F0%9F%94%AA-Random-Numbers

```{code-cell}
:id: u0nseKZNqOoH

key = random.PRNGKey(0)
x = random.normal(key, (10,))
print(x)
```

+++ {"colab_type": "text", "id": "hDJF0UPKnuqB"}

Let's dive right in and multiply two big matrices.

```{code-cell}
:id: eXn8GUl6CG5N

size = 3000
x = random.normal(key, (size, size), dtype=jnp.float32)
%timeit jnp.dot(x, x.T).block_until_ready()  # runs on the GPU
```

+++ {"colab_type": "text", "id": "0AlN7EbonyaR"}

We added that `block_until_ready` because JAX uses asynchronous execution by default (see {ref}`async-dispatch`).

JAX NumPy functions work on regular NumPy arrays.

```{code-cell}
:id: ZPl0MuwYrM7t

import numpy as np
x = np.random.normal(size=(size, size)).astype(np.float32)
%timeit jnp.dot(x, x.T).block_until_ready()
```

+++ {"colab_type": "text", "id": "_SrcB2IurUuE"}

That's slower because it has to transfer data to the GPU every time. You can ensure that an NDArray is backed by device memory using {func}`~jax.device_put`.

```{code-cell}
:id: Jj7M7zyRskF0

from jax import device_put

x = np.random.normal(size=(size, size)).astype(np.float32)
x = device_put(x)
%timeit jnp.dot(x, x.T).block_until_ready()
```

+++ {"colab_type": "text", "id": "clO9djnen8qi"}


The output of {func}`~jax.device_put` still acts like an NDArray, but it only copies values back to the CPU when they're needed for printing, plotting, saving to disk, branching, etc. The behavior of {func}`~jax.device_put` is equivalent to the function `jit(lambda x: x)`, but it's faster.

+++ {"colab_type": "text", "id": "ghkfKNQttDpg"}

If you have a GPU (or TPU!) these calls run on the accelerator and have the potential to be much faster than on CPU.

```{code-cell}
:id: RzXK8GnIs7VV

x = np.random.normal(size=(size, size)).astype(np.float32)
%timeit np.dot(x, x.T)
```

+++ {"colab_type": "text", "id": "iOzp0P_GoJhb"}

JAX is much more than just a GPU-backed NumPy. It also comes with a few program transformations that are useful when writing numerical code. For now, there's three main ones:

 - {func}`~jax.jit`, for speeding up your code
 - {func}`~jax.grad`, for taking derivatives
 - {func}`~jax.vmap`, for automatic vectorization or batching.

Let's go over these, one-by-one. We'll also end up composing these in interesting ways.

+++ {"colab_type": "text", "id": "bTTrTbWvgLUK"}

## Using {func}`~jax.jit` to speed up functions

+++ {"colab_type": "text", "id": "YrqE32mvE3b7"}

JAX runs transparently on the GPU (or CPU, if you don't have one, and TPU coming soon!). However, in the above example, JAX is dispatching kernels to the GPU one operation at a time. If we have a sequence of operations, we can use the `@jit` decorator to compile multiple operations together using [XLA](https://www.tensorflow.org/xla). Let's try that.

```{code-cell}
:id: qLGdCtFKFLOR

def selu(x, alpha=1.67, lmbda=1.05):
  return lmbda * jnp.where(x > 0, x, alpha * jnp.exp(x) - alpha)

x = random.normal(key, (1000000,))
%timeit selu(x).block_until_ready()
```

+++ {"colab_type": "text", "id": "a_V8SruVHrD_"}

We can speed it up with `@jit`, which will jit-compile the first time `selu` is called and will be cached thereafter.

```{code-cell}
:id: fh4w_3NpFYTp

selu_jit = jit(selu)
%timeit selu_jit(x).block_until_ready()
```

+++ {"colab_type": "text", "id": "HxpBc4WmfsEU"}

## Taking derivatives with {func}`~jax.grad`

In addition to evaluating numerical functions, we also want to transform them. One transformation is [automatic differentiation](https://en.wikipedia.org/wiki/Automatic_differentiation). In JAX, just like in [Autograd](https://github.com/HIPS/autograd), you can compute gradients with the {func}`~jax.grad` function.

```{code-cell}
:id: IMAgNJaMJwPD

def sum_logistic(x):
  return jnp.sum(1.0 / (1.0 + jnp.exp(-x)))

x_small = jnp.arange(3.)
derivative_fn = grad(sum_logistic)
print(derivative_fn(x_small))
```

+++ {"colab_type": "text", "id": "PtNs881Ohioc"}

Let's verify with finite differences that our result is correct.

```{code-cell}
:id: JXI7_OZuKZVO

def first_finite_differences(f, x):
  eps = 1e-3
  return jnp.array([(f(x + eps * v) - f(x - eps * v)) / (2 * eps)
                   for v in jnp.eye(len(x))])


print(first_finite_differences(sum_logistic, x_small))
```

+++ {"colab_type": "text", "id": "Q2CUZjOWNZ-3"}

Taking derivatives is as easy as calling {func}`~jax.grad`. {func}`~jax.grad` and {func}`~jax.jit` compose and can be mixed arbitrarily. In the above example we jitted `sum_logistic` and then took its derivative. We can go further:

```{code-cell}
:id: TO4g8ny-OEi4

print(grad(jit(grad(jit(grad(sum_logistic)))))(1.0))
```

+++ {"colab_type": "text", "id": "yCJ5feKvhnBJ"}

For more advanced autodiff, you can use {func}`jax.vjp` for reverse-mode vector-Jacobian products and {func}`jax.jvp` for forward-mode Jacobian-vector products. The two can be composed arbitrarily with one another, and with other JAX transformations. Here's one way to compose them to make a function that efficiently computes full Hessian matrices:

```{code-cell}
:id: Z-JxbiNyhxEW

from jax import jacfwd, jacrev
def hessian(fun):
  return jit(jacfwd(jacrev(fun)))
```

+++ {"colab_type": "text", "id": "TI4nPsGafxbL"}

## Auto-vectorization with {func}`~jax.vmap`
+++ {"colab_type": "text", "id": "PcxkONy5aius"}

JAX has one more transformation in its API that you might find useful: {func}`~jax.vmap`, the vectorizing map. It has the familiar semantics of mapping a function along array axes, but instead of keeping the loop on the outside, it pushes the loop down into a function’s primitive operations for better performance. When composed with {func}`~jax.jit`, it can be just as fast as adding the batch dimensions by hand.

+++ {"colab_type": "text", "id": "TPiX4y-bWLFS"}

We're going to work with a simple example, and promote matrix-vector products into matrix-matrix products using {func}`~jax.vmap`. Although this is easy to do by hand in this specific case, the same technique can apply to more complicated functions.

```{code-cell}
:id: 8w0Gpsn8WYYj

mat = random.normal(key, (150, 100))
batched_x = random.normal(key, (10, 100))

def apply_matrix(v):
  return jnp.dot(mat, v)
```

+++ {"id": "0zWsc0RisQWx", "colab_type": "text"}

Given a function such as `apply_matrix`, we can loop over a batch dimension in Python, but usually the performance of doing so is poor.

```{code-cell}
:id: KWVc9BsZv0Ki

def naively_batched_apply_matrix(v_batched):
  return jnp.stack([apply_matrix(v) for v in v_batched])

print('Naively batched')
%timeit naively_batched_apply_matrix(batched_x).block_until_ready()
```

+++ {"id": "qHfKaLE9stbA", "colab_type": "text"}

We know how to batch this operation manually. In this case, `jnp.dot` handles extra batch dimensions transparently.

```{code-cell}
:id: ipei6l8nvrzH

@jit
def batched_apply_matrix(v_batched):
  return jnp.dot(v_batched, mat.T)

print('Manually batched')
%timeit batched_apply_matrix(batched_x).block_until_ready()
```

+++ {"id": "1eF8Nhb-szAb", "colab_type": "text"}

However, suppose we had a more complicated function without batching support. We can use {func}`~jax.vmap` to add batching support automatically.

```{code-cell}
:id: 67Oeknf5vuCl

@jit
def vmap_batched_apply_matrix(v_batched):
  return vmap(apply_matrix)(v_batched)

print('Auto-vectorized with vmap')
%timeit vmap_batched_apply_matrix(batched_x).block_until_ready()
```

+++ {"colab_type": "text", "id": "pYVl3Z2nbZhO"}

Of course, {func}`~jax.vmap` can be arbitrarily composed with {func}`~jax.jit`, {func}`~jax.grad`, and any other JAX transformation.

+++ {"id": "WwNnjaI4th_8", "colab_type": "text"}

This is just a taste of what JAX can do. We're really excited to see what you do with it!