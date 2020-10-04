---
title: "Type sizes are not your friends"
date: 2020-10-04T14:57:33+01:00
---

I've recently “inherited” a codebase, said codebase was poorly adapted from C to compile with C++.
It used raw pointers everywhere, all the libraries used were aimed at C and more...
Said codebase did a lot of socket networking, communicating using a custom “ad hoc” data format.

The format is fairly simple and follows a pattern of sending the length first and then the content of said length.
This is practical because it allows dealing with dynamic data sizes on the fly.
And so, my problem is not about the use of the custom data format,
my problem is about the code that handles the format.

Bellow is the code used to receive one of such data packets.

```cpp
size_t in_len;
untrusted_util::socket_receive(socket, &in_len, sizeof(size_t));
void *in_buffer = malloc(in_len);
untrusted_util::socket_receive(socket, in_buffer, in_len);
```

As previously stated, the length is read first and then the content.
The length is encoded as a `size_t` variable and the content is just a block of bytes.

The problem here is that `size_t` or rather any non-fixed integer type do not have a set size according to the standard.

> A size modifier specifies the width in bits of the integer representation used.
> The language supports `short`, `long`, and `long long` modifiers.
> A `short` type must be at least 16 bits wide.
> A `long` type must be at least 32 bits wide.
> A `long long` type must be at least 64 bits wide.
> The standard specifies a size relationship between the integral types.

As we can see [here](https://docs.microsoft.com/en-us/cpp/cpp/fundamental-types-cpp?view=vs-2019#integer-types),
such integer types are defined as “at least X bits wide”, which means it is up to the compiler to decide how many bits to use,
according to the specification, all integer types could be 64-bit wide and that would be valid!

**So, why is this important?**

Because one cannot expect every device in the network to be the same.
Unless you're very clear about the compilers and architectures your code supports,
you must be strict about details like this.

The following snippet demonstrates the differences in integer type sizes between architectures:

```c
#include <stdio.h>
void main() {
    printf("int %ld\n", sizeof(int));
    printf("long %ld\n", sizeof(long));
    printf("short %ld\n", sizeof(short));
    printf("size_t %ld\n", sizeof(size_t));
}
```

When compiling with `gcc test.c -o test` in a 64-bit system the output produces:

```text
int 4
long 8
short 2
size_t 8
```

However, when compiling for 32-bit with `gcc test.c -m32 -o test` the output produces:

```text
int 4
long 4
short 2
size_t 4
```

So, when compiling a client/server with different architectures,
if you want to make communication between them compatible you should use fixed-width integers (e.g. [`stdint.h`](https://www.cplusplus.com/reference/cstdint/)).

The example bellow yields the same result whether compiled for 32 or 64-bit architectures as well as different compilers.

```c
#include <stdio.h>
#include <stdint.h>
void main() {
    printf("int8_t %ld\n", sizeof(int8_t));
    printf("int16_t %ld\n", sizeof(int16_t));
    printf("int32_t %ld\n", sizeof(int32_t));
    printf("int64_t %ld\n", sizeof(int64_t));
}
```