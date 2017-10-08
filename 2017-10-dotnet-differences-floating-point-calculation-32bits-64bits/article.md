---
title: [.NET] Differences in floating point calculations between 32-bit and 64-bit (EN)
published: 2017-10-08
categories: C#/.NET
tags: .NET, EN
url: /2017/10/dotnet-differences-floating-point-calculation-32bit-64bit-en
---

# [.NET] Differences in floating point calculations between 32-bit and 64-bit

As part of the [SwissEph.Net](https://github.com/ygrenier/SwissEphNet) project, I've got some user feedbacks about diffrences in the results of calculation between the .NET library and the original C DLL.

As I have not had feedback from users about used code and diferences found, I spend some time to create a project for testing and compare result values between the differents versions: [https://github.com/ygrenier/SwissEphNet.TestsAndCompare](https://github.com/ygrenier/SwissEphNet.TestsAndCompare).

Once the tests are done, I found there is a difference in floating point calculations in .NET, but not between C code and .NET, the difference is between 32bits and 64bits of the .NET code.  

<!--more-->

# Tests done

The application define some tests and save the result in more values.

I do a automatique load of the 32/64-bit DLL from the platform (like this [french article](http://blog.ygrenier.com/2016/03/pinvoke-utiliser-dll-32bits-64bits-fonction-de-plateforme-mode-any-cpu/) - [Automatic translation here](http://www.microsofttranslator.com/bv.aspx?from=&to=en&a=http%3A%2F%2Fblog.ygrenier.com%2F2016%2F03%2Fpinvoke-utiliser-dll-32bits-64bits-fonction-de-plateforme-mode-any-cpu%2F))

For each platform, the tests are executed and exported to a CSV file.

Then an Excel file was created to merge results and compare the values. 

# Analisys

So the tests was performed on my Windows 10 64-bit machine.

You can find the files in the "tests" folder of the repository.

The result of comparaisons between the differents platforms are following:

| Compare-DLL-32-64 | Compare-.NET-32-64 | Compare-DLL-.NET-32 | Compare-DLL-.NET-64 |
|-------------------|--------------------|---------------------|---------------------|
| true              | false              | false               | true                |

We can see there is no differences between 32/64-bit C DLL, but there is a difference in results with .NET 32-bit version (the 64-bit version have same results than the C DLL).


# Floating-point calculation

First of all, I recall the floats are an approximation, there are always some small errors in calculations.

There format are specified by the [standard IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) and the .NET  use for the `float` type (`System.Single`) a float based on a 32-bit, and for the  `double` type (`System.Double`) a float based on a 64-bit (the `decimal` is stored in an another format).

The floating calculations are quite costly in CPU, that is why we created specific coprocessor named FPU (Floating Point Unit). For a long time the FPU are included in the CPU. 

Since few years too, CPU include new instructions such as [SSE instructions](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions) that optimize the calculation, especially the about the floats.

Here our problems start  :)

The FPU and SSE instructions do not calculate the floats in the same way. The FPU make calculations on 80 bits, while SSE compute on 32 bits (or 64 bits from SSE2).

So if we use the FPU we get a better precision, but as we use `double` (stored in 64 bits) the calculation are truncated to go 80 bits to 64 bits.

This is not the same with the SSE instructions because we are in the same size.

This makes that depending the type of instructions used (FPU or SSE) you will not always get the same values. The difference is usually quite low, but in a complex calculation these differences can muultiply and give considerable discrepancies.

This is one of the reasons why when we do scientific calculations are made, we used specific mathematical libraries that reduce this risk by using fixed point floats. 

# Why this problem is not present with C library ?

Because the C code is compiled to native code, so it's in the compilation we decide how the floats are calulated. In the Swiss Ephemeris case, I think the DLL is compiled in the same way in 32-bit and 64-bit. So we get the same results. 

# And why there is a problem in the .NET ?

I think you are beginning to understand why there is a difference of calculation in the .NET library: **the 32-bit version don't used the same instructions than the 64-bit version**.

To be clear, is the runtime that will compile Just-In-time le CIL code when executing the application, and who will decide to use the FPU if we are in 32-bit or the SSE instructions is we are in 64-bit (and if the SSE are available). 

So because this is the runtime, we can't change this behavior (in any cas not to my knowledge), et some other runtime implementation (like Mono) may have an another behavior.

# This solutions 

To resolve this problem, simply don't use `float`/`double` and instead of use the `decimal` or a third part library using is proper calculations without FPU/SSE influence.

But the `decimal` are floats encoded in 128 bits, and optimised for financial calculations (reducing the rounding problems meets with floating-point). There are a better precision but are a smaller range.

# To finish

Now don't be suprised if you find difference calculations in .NET with differents platforms. 

If this behavior is a problem so you can:
- fix a platform 32/64 bit without permetting choice (this could be a problem for a library)
- using `decimal` if you can
- using third part library resolving this problem 

See you soon,

Yanos

PS: this article is the English translation of the [french article](http://blog.ygrenier.com/2017/10/dotnet-differences-calcul-flottants-32bits-64bits/)
