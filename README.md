# libcubwt

The libcubwt is a library for fast (see [Benchmarks](#benchmarks) below) GPU-based Burrows-Wheeler transform construction on the CUDA platform using prefix doubling + Skew/DC3 approach as described in the following papers:
* Vitaly Osipov, *Parallel Suffix Array Construction for Shared Memory Architectures*, 2012
* Leyuan Wang, Sean Baxter, and John D. Owens *Fast Parallel Suffix Array on the GPU*, 2015
* Florian Büren, Daniel Jünger, Robin Kobus, Christian Hundt, Bertil Schmidt *Suffix Array Construction on Multi-GPU Systems*, 2019

Copyright (c) 2022-2023 Ilya Grebnov <ilya.grebnov@gmail.com>

## Introduction
The libcubwt provides simple API to construct Burrows-Wheeler transform from a given string over constant-size alphabet using 20.5n bytes of GPU memory.

> * The libcubwt works with any CUDA compatible GPU, but I recommend SM 8.9+ Ada Lovelace (CUDA 11.8 and later) due to very large L2 cache.
> * The libcubwt is sensitive to fast GPU memory and might not be suitable for some workloads. Please benchmark yourself.

## Suffix array computation
> * Starting with version 1.5.0, libcubwt algorithm was changed and no longer supports the computation of suffix arrays or inverse suffix arrays. To compute these arrays, please consider using a version prior to 1.5.0.

## License
The libcubwt is released under the [Apache License Version 2.0](LICENSE "Apache license") and is considered suitable for production use. However, no warranty or fitness for a particular purpose is expressed or implied.

## Changes
* March 24, 2023 (1.5.0)
  * Reduced memory usage and improved performance.
* February 10, 2023 (1.0.0)
  * Initial public release of the libcubwt.

## Examples of APIs (see [libcubwt.cuh](libcubwt.cuh) for complete APIs list)
```c
    /**
    * Allocates storage on the CUDA device that allows reusing allocated memory with each libcubwt operation.
    * @param max_length The maximum length of string to support.
    * @return LIBCUBWT_NO_ERROR if no error occurred, libcubwt error code otherwise.
    */
    int64_t libcubwt_allocate_device_storage(void ** device_storage, int64_t max_length);

    /**
    * Destroys the previously allocated storage on the CUDA device.
    * @param device_storage The previously allocated storage on the CUDA device.
    * @return LIBCUBWT_NO_ERROR if no error occurred, libcubwt error code otherwise.
    */
    int64_t libcubwt_free_device_storage(void * device_storage);

    /**
    * Constructs the Burrows-Wheeler Transform (BWT) of a given string.
    * @param device_storage The previously allocated storage on the CUDA device.
    * @param T [0..n-1] The input string.
    * @param L [0..n-1] The output string (can be T).
    * @param n The length of the input string.
    * @return The primary index if no error occurred, libcubwt error code otherwise.
    */
    int64_t libcubwt_bwt(void * device_storage, const uint8_t * T, uint8_t * L, int64_t n);
```

---

# Benchmarks

## Methodology ##
  * Input files were capped at 352MB due to GPU memory limit.
  * All timings exclude initialization and memory allocations. However, the time for data transfer in and out of GPU is included.
  * The timings are minimum of five runs measuring multi-threading performance of Burrows-Wheeler transform construction.

## Specification (x86-64 architecture) ##
  * OS: Microsoft Windows 10 Pro 64-Bit
  * MB: MSI MPG Z390M GAMING EDGE AC, PCIe 3.0 x16
  * CPU: Intel Core i7-9700K Processor (12M Cache, 5GHz all cores)
  * RAM: 2x8 GB dual-channel DDR4 (4133 MHz, 17-17-17-37)
  * GPU: ASUS TUF Gaming GeForce RTX 4070 Ti 12GB GDDR6X OC Edition
  * Compiler: Microsoft Visual C++ compiler v19.34.31937 for x64
  * Optimizations (CPU): /MD /DNDEBUG /O2 /GL /arch:AVX2 /openmp
  * Optimizations (GPU): -arch=native --use_fast_math

### Silesia Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|         dickens |    10192446 |   **0.012 sec ( 844.66 MB/s)** |     0.115 sec (  88.27 MB/s)   |     0.317 sec (  32.12 MB/s)   |
|         mozilla |    51220480 |   **0.107 sec ( 479.70 MB/s)** |     0.722 sec (  70.91 MB/s)   |     1.296 sec (  39.51 MB/s)   |
|              mr |     9970564 |   **0.044 sec ( 228.12 MB/s)** |     0.124 sec (  80.11 MB/s)   |     0.283 sec (  35.21 MB/s)   |
|             nci |    33553445 |   **0.097 sec ( 347.06 MB/s)** |     0.378 sec (  88.67 MB/s)   |     1.141 sec (  29.40 MB/s)   |
|         ooffice |     6152192 |   **0.007 sec ( 924.93 MB/s)** |     0.085 sec (  72.66 MB/s)   |     0.117 sec (  52.59 MB/s)   |
|            osdb |    10085684 |   **0.012 sec ( 841.83 MB/s)** |     0.152 sec (  66.35 MB/s)   |     0.299 sec (  33.72 MB/s)   |
|         reymont |     6627202 |   **0.008 sec ( 834.20 MB/s)** |     0.077 sec (  86.48 MB/s)   |     0.199 sec (  33.23 MB/s)   |
|           samba |    21606400 |   **0.044 sec ( 488.45 MB/s)** |     0.277 sec (  77.87 MB/s)   |     0.502 sec (  43.04 MB/s)   |
|             sao |     7251944 |   **0.006 sec (1275.49 MB/s)** |     0.145 sec (  50.11 MB/s)   |     0.158 sec (  46.04 MB/s)   |
|         webster |    41458703 |   **0.065 sec ( 640.98 MB/s)** |     0.603 sec (  68.71 MB/s)   |     1.768 sec (  23.46 MB/s)   |
|           x-ray |     8474240 |   **0.007 sec (1297.70 MB/s)** |     0.149 sec (  56.84 MB/s)   |     0.198 sec (  42.90 MB/s)   |
|             xml |     5345280 |   **0.009 sec ( 594.05 MB/s)** |     0.054 sec (  98.87 MB/s)   |     0.117 sec (  45.77 MB/s)   |


### Large Canterbury Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|       bible.txt |     4047392 |   **0.004 sec (1063.62 MB/s)** |     0.042 sec (  97.15 MB/s)   |     0.106 sec (  38.16 MB/s)   |
|          E.coli |     4638690 |   **0.006 sec ( 771.33 MB/s)** |     0.040 sec ( 116.30 MB/s)   |     0.166 sec (  27.98 MB/s)   |
|    world192.txt |     2473400 |   **0.003 sec ( 885.03 MB/s)** |     0.027 sec (  90.86 MB/s)   |     0.053 sec (  46.41 MB/s)   |


### Maximum Compression Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|         A10.jpg |      842468 |   **0.001 sec (1338.95 MB/s)** |     0.020 sec (  42.90 MB/s)   |     0.018 sec (  45.71 MB/s)   |
|    AcroRd32.exe |     3870784 |   **0.007 sec ( 577.69 MB/s)** |     0.059 sec (  65.08 MB/s)   |     0.070 sec (  55.50 MB/s)   |
|     english.dic |      465211 |   **0.001 sec ( 567.47 MB/s)** |     0.007 sec (  70.86 MB/s)   |     0.008 sec (  54.87 MB/s)   |
|     FlashMX.pdf |     4526946 |   **0.004 sec (1104.21 MB/s)** |     0.099 sec (  45.58 MB/s)   |     0.086 sec (  52.37 MB/s)   |
|          FP.LOG |    20617071 |   **0.047 sec ( 437.20 MB/s)** |     0.261 sec (  79.07 MB/s)   |     0.715 sec (  28.82 MB/s)   |
|       MSO97.DLL |     3782416 |   **0.003 sec (1178.36 MB/s)** |     0.061 sec (  62.34 MB/s)   |     0.072 sec (  52.74 MB/s)   |
|         ohs.doc |     4168192 |   **0.006 sec ( 674.87 MB/s)** |     0.051 sec (  81.72 MB/s)   |     0.094 sec (  44.17 MB/s)   |
|      rafale.bmp |     4149414 |   **0.004 sec (1060.26 MB/s)** |     0.043 sec (  96.65 MB/s)   |     0.083 sec (  49.81 MB/s)   |
|       vcfiu.hlp |     4121418 |   **0.005 sec ( 797.61 MB/s)** |     0.046 sec (  89.91 MB/s)   |     0.078 sec (  52.78 MB/s)   |
|     world95.txt |     2988578 |   **0.004 sec ( 836.76 MB/s)** |     0.033 sec (  91.77 MB/s)   |     0.067 sec (  44.32 MB/s)   |


### Large Text Compression Benchmark Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|          enwik8 |   100000000 |   **0.147 sec ( 681.07 MB/s)** |     1.633 sec (  61.22 MB/s)   |     4.616 sec (  21.67 MB/s)   |
|          enwik9 |   369098752 |   **0.579 sec ( 637.83 MB/s)** |     6.363 sec (  58.01 MB/s)   |    20.633 sec (  17.89 MB/s)   |


### The Gauntlet Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|            abac |      200000 |     0.004 sec (  47.69 MB/s)   |     0.002 sec (  89.59 MB/s)   |   **0.002 sec ( 105.77 MB/s)** |
|            abba |    10500596 |   **0.038 sec ( 278.77 MB/s)** |     0.092 sec ( 114.41 MB/s)   |     0.494 sec (  21.26 MB/s)   |
|        book1x20 |    15375420 |   **0.076 sec ( 201.57 MB/s)** |     0.192 sec (  80.14 MB/s)   |     0.535 sec (  28.75 MB/s)   |
|   fib_s14930352 |    14930352 |   **0.111 sec ( 134.06 MB/s)** |     0.181 sec (  82.57 MB/s)   |     1.164 sec (  12.83 MB/s)   |
|           fss10 |    12078908 |   **0.073 sec ( 166.32 MB/s)** |     0.135 sec (  89.26 MB/s)   |     0.885 sec (  13.65 MB/s)   |
|            fss9 |     2851443 |   **0.014 sec ( 205.39 MB/s)** |     0.027 sec ( 107.11 MB/s)   |     0.132 sec (  21.53 MB/s)   |
|         houston |     3839141 |     0.023 sec ( 164.68 MB/s)   |   **0.020 sec ( 188.82 MB/s)** |     0.028 sec ( 138.15 MB/s)   |
|       paper5x80 |      956322 |   **0.002 sec ( 403.97 MB/s)** |     0.010 sec (  94.51 MB/s)   |     0.021 sec (  46.35 MB/s)   |
|           test1 |     2097152 |   **0.006 sec ( 358.13 MB/s)** |     0.023 sec (  92.50 MB/s)   |     0.082 sec (  25.46 MB/s)   |
|           test2 |     2097152 |   **0.006 sec ( 356.80 MB/s)** |     0.022 sec (  96.95 MB/s)   |     0.055 sec (  37.91 MB/s)   |
|           test3 |     2097088 |   **0.004 sec ( 474.53 MB/s)** |     0.035 sec (  60.43 MB/s)   |     0.049 sec (  42.93 MB/s)   |


### Manzini Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|       chr22.dna |    34553758 |   **0.093 sec ( 369.82 MB/s)** |     0.436 sec (  79.18 MB/s)   |     1.837 sec (  18.81 MB/s)   |
|         etext99 |   105277340 |   **0.199 sec ( 528.25 MB/s)** |     1.780 sec (  59.14 MB/s)   |     5.590 sec (  18.83 MB/s)   |
|     gcc-3.0.tar |    86630400 |   **0.221 sec ( 392.32 MB/s)** |     1.215 sec (  71.32 MB/s)   |     2.682 sec (  32.30 MB/s)   |
|           howto |    39422105 |   **0.065 sec ( 603.01 MB/s)** |     0.565 sec (  69.75 MB/s)   |     1.366 sec (  28.86 MB/s)   |
|          jdk13c |    69728899 |   **0.174 sec ( 400.57 MB/s)** |     1.031 sec (  67.66 MB/s)   |     2.734 sec (  25.50 MB/s)   |
| linux-2.4.5.tar |   116254720 |   **0.252 sec ( 461.40 MB/s)** |     1.698 sec (  68.45 MB/s)   |     3.808 sec (  30.53 MB/s)   |
|        rctail96 |   114711151 |   **0.254 sec ( 451.11 MB/s)** |     1.864 sec (  61.54 MB/s)   |     5.452 sec (  21.04 MB/s)   |
|             rfc |   116421901 |   **0.228 sec ( 510.92 MB/s)** |     1.725 sec (  67.48 MB/s)   |     4.510 sec (  25.81 MB/s)   |
|     sprot34.dat |   109617186 |   **0.211 sec ( 519.63 MB/s)** |     1.756 sec (  62.42 MB/s)   |     4.810 sec (  22.79 MB/s)   |
|            w3c2 |   104201579 |   **0.286 sec ( 364.16 MB/s)** |     1.603 sec (  65.01 MB/s)   |     3.885 sec (  26.82 MB/s)   |


### Pizza & Chilli Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|        dblp.xml |   296135874 |   **0.558 sec ( 530.99 MB/s)** |     4.860 sec (  60.93 MB/s)   |    13.964 sec (  21.21 MB/s)   |
|             dna |   369098752 |   **0.591 sec ( 624.79 MB/s)** |     5.837 sec (  63.24 MB/s)   |    30.630 sec (  12.05 MB/s)   |
|  english.1024MB |   369098752 |   **0.758 sec ( 486.98 MB/s)** |     6.794 sec (  54.33 MB/s)   |    24.330 sec (  15.17 MB/s)   |
|         pitches |    55832855 |   **0.097 sec ( 578.30 MB/s)** |     0.807 sec (  69.19 MB/s)   |     1.521 sec (  36.70 MB/s)   |
|        proteins |   369098752 |   **0.740 sec ( 499.06 MB/s)** |     7.085 sec (  52.10 MB/s)   |    20.326 sec (  18.16 MB/s)   |
|         sources |   210866607 |   **0.391 sec ( 539.50 MB/s)** |     3.254 sec (  64.80 MB/s)   |     7.826 sec (  26.94 MB/s)   |


### Pizza & Chilli Repetitive Corpus ###

|  file           |    size     |      libcubwt 1.5.0 (GPU)      |      libsais 2.7.1 (CPU)       |     divsufsort 2.0.2 (CPU)     |
|:---------------:|:-----------:|:------------------------------:|:------------------------------:|:------------------------------:|
|            cere |   369098752 |   **1.788 sec ( 206.43 MB/s)** |     5.648 sec (  65.35 MB/s)   |    24.792 sec (  14.89 MB/s)   |
|       coreutils |   205281778 |   **0.892 sec ( 230.01 MB/s)** |     3.097 sec (  66.29 MB/s)   |     8.828 sec (  23.25 MB/s)   |
| einstein.de.txt |    92758441 |   **0.547 sec ( 169.56 MB/s)** |     1.363 sec (  68.03 MB/s)   |     4.038 sec (  22.97 MB/s)   |
| einstein.en.txt |   369098752 |   **2.195 sec ( 168.17 MB/s)** |     5.845 sec (  63.15 MB/s)   |    21.941 sec (  16.82 MB/s)   |
|Escherichia_Coli |   112689515 |   **0.302 sec ( 372.84 MB/s)** |     1.695 sec (  66.48 MB/s)   |     6.832 sec (  16.49 MB/s)   |
|       influenza |   154808555 |   **0.527 sec ( 293.57 MB/s)** |     2.234 sec (  69.31 MB/s)   |     8.538 sec (  18.13 MB/s)   |
|          kernel |   257961616 |   **1.063 sec ( 242.78 MB/s)** |     3.856 sec (  66.89 MB/s)   |    10.715 sec (  24.08 MB/s)   |
|            para |   369098752 |   **1.335 sec ( 276.43 MB/s)** |     5.725 sec (  64.47 MB/s)   |    25.627 sec (  14.40 MB/s)   |
|   world_leaders |    46968181 |   **0.312 sec ( 150.45 MB/s)** |     0.457 sec ( 102.68 MB/s)   |     1.119 sec (  41.98 MB/s)   |
|dblp.xml.00001.1 |   104857600 |   **0.424 sec ( 247.09 MB/s)** |     1.739 sec (  60.31 MB/s)   |     5.610 sec (  18.69 MB/s)   |
|dblp.xml.00001.2 |   104857600 |   **0.432 sec ( 242.87 MB/s)** |     1.729 sec (  60.63 MB/s)   |     5.669 sec (  18.50 MB/s)   |
| dblp.xml.0001.1 |   104857600 |   **0.371 sec ( 282.50 MB/s)** |     1.743 sec (  60.17 MB/s)   |     5.600 sec (  18.73 MB/s)   |
| dblp.xml.0001.2 |   104857600 |   **0.372 sec ( 282.22 MB/s)** |     1.729 sec (  60.64 MB/s)   |     5.585 sec (  18.77 MB/s)   |
|       dna.001.1 |   104857600 |   **0.275 sec ( 381.66 MB/s)** |     1.571 sec (  66.73 MB/s)   |     6.097 sec (  17.20 MB/s)   |
|   english.001.2 |   104857600 |   **0.284 sec ( 369.03 MB/s)** |     1.795 sec (  58.43 MB/s)   |     4.912 sec (  21.35 MB/s)   |
|  proteins.001.1 |   104857600 |   **0.490 sec ( 214.02 MB/s)** |     1.898 sec (  55.26 MB/s)   |     4.256 sec (  24.64 MB/s)   |
|   sources.001.2 |   104857600 |   **0.300 sec ( 349.51 MB/s)** |     1.717 sec (  61.07 MB/s)   |     4.295 sec (  24.42 MB/s)   |
|           fib41 |   267914296 |   **2.480 sec ( 108.03 MB/s)** |     3.898 sec (  68.73 MB/s)   |    40.566 sec (   6.60 MB/s)   |
|           rs.13 |   216747218 |   **2.100 sec ( 103.23 MB/s)** |     3.062 sec (  70.79 MB/s)   |    34.326 sec (   6.31 MB/s)   |
|            tm29 |   268435456 |   **2.343 sec ( 114.58 MB/s)** |     4.134 sec (  64.93 MB/s)   |    37.548 sec (   7.15 MB/s)   |
