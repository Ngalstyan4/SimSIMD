# SimSIMD

SIMD-accelerated similarity measures, metrics, distance functions for x86 and Arm.
Need something like this in your CMake-based project?

```cmake
FetchContent_Declare(
    simsimd
    GIT_REPOSITORY https://github.com/ashvardanian/simsimd.git
    GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(simsimd)
include_directories(${simsimd_SOURCE_DIR}/include)
```

## Benchmarks

By default, we use GCC12, `-O3`, `-march=native` for benchmarks.
Serial versions imply auto-vectorization pragmas.

---

Cosine distance performance on Arm-based "Graviton 3" CPUs powering AWS `c7g.metal` instances:

| Method | Vectors    | Any Length | Performance |
| :----- | :--------- | :--------- | ----------: |
| Serial | `f32` x16  | ✅          |      5 GB/s |
| Serial | `f32` x256 | ✅          |      5 GB/s |
|        |            |            |             |
| NEON   | `f32` x16  | ❌          |     18 GB/s |
| NEON   | `f32` x256 | ❌          |     29 GB/s |
|        |            |            |             |
| SVE    | `f32` x16  | ✅          |     15 GB/s |
| SVE    | `f32` x256 | ✅          |     39 GB/s |

We only use Arm NEON implementation with vectors lengths that are multiples of 128 bits, avoiding any additional head or tail `for` loops for misaligned data.
SVE looses to NEON on very short vectors, but outperforms on longer sequences.

---

To replicate on your hardware, please run:

```sh
cmake -DCMAKE_BUILD_TYPE=Release -B ./build && make -C ./build && ./build/simsimd_bench
```
