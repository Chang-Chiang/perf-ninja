# perf-ninja

## Dev Env

### install perf

```shell
$ perf --version
WARNING: perf not found for kernel 6.14.0-35

  You may need to install the following packages for this specific kernel:
    linux-tools-6.14.0-35-generic
    linux-cloud-tools-6.14.0-35-generic

  You may also want to install one of the following packages to keep up to date:
    linux-tools-generic
    linux-cloud-tools-generic

# no perf in 6.14 kernel
$ sudo ln -sf /usr/lib/linux-tools-6.8.0-87/perf /usr/bin/perf
$ perf --version
perf version 6.8.12

```

### install pmu-tools

```shell
$ cd ~/Projects
$ git clone https://github.com/andikleen/pmu-tools.git
$ vim ~/.bashrc

PATH=$PATH:/home/cc/Projects/pmu-tools

```

### warmup

```shell
$ cd labs/misc/warmup

$ cmake -E make_directory build
$ cd build

# release version
$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..

$ cmake --build . --config Release --parallel 8

# to collect a profile
# debug version
$ cmake -DCMAKE_BUILD_TYPE=Debug .. -DCMAKE_C_FLAGS="-g" -DCMAKE_CXX_FLAGS="-g" -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..

$ cmake --build . --config Debug --parallel 8
```


```shell
$ cmake --build . --target validateLab
[100%] Built target validate
Validation Successful
[100%] Built target validateLab
```


#### release version


```shell
$ cmake --build . --target benchmarkLab
[100%] Built target lab
2025-11-14T21:53:50+08:00
Running ./lab
Run on (12 X 4100 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 2.01, 1.07, 0.77
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           25.8 ns         25.8 ns    112429048
[100%] Built target benchmarkLab
```


```shell
$ sudo cpupower frequency-set --governor performance
[sudo] password for cc: 
Setting cpu: 0
Setting cpu: 1
Setting cpu: 2
Setting cpu: 3
Setting cpu: 4
Setting cpu: 5
Setting cpu: 6
Setting cpu: 7
Setting cpu: 8
Setting cpu: 9
Setting cpu: 10
Setting cpu: 11
```


```shell
$ cmake --build . --target benchmarkLab
[100%] Built target lab
2025-11-14T21:56:59+08:00
Running ./lab
Run on (12 X 3902.34 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.70, 1.00, 0.80
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           23.8 ns         23.8 ns    116277551
[100%] Built target benchmarkLab
```

```shell
$ perf record ./lab
2025-11-14T23:40:45+08:00
Running ./lab
Run on (12 X 3896.87 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 2.48, 1.97, 1.65
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           28.8 ns         28.8 ns     24373687
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.172 MB perf.data (4100 samples) ]
```

#### debug version

```shell
$ cmake --build . --target benchmarkLab
[100%] Built target lab
2025-11-16T06:52:00+08:00
Running ./lab
Run on (12 X 2200.79 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.24, 0.35, 0.25
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           43.6 ns         43.6 ns     64131128
[100%] Built target benchmarkLab
```

after optimize

```shell
$ cmake --build . --target benchmarkLab
[ 33%] Building CXX object CMakeFiles/lab.dir/solution.cpp.o
[ 66%] Linking CXX executable lab
[100%] Built target lab
2025-11-16T07:06:57+08:00
Running ./lab
Run on (12 X 2200.04 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.92, 0.86, 0.54
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           3.14 ns         3.14 ns    811773558
[100%] Built target benchmarkLab
```




```shell
$ perf record ./lab
Error:
Access to performance monitoring and observability operations is limited.
Consider adjusting /proc/sys/kernel/perf_event_paranoid setting to open
access to performance monitoring and observability operations for processes
without CAP_PERFMON, CAP_SYS_PTRACE or CAP_SYS_ADMIN Linux capability.
More information can be found at 'Perf events and tool security' document:
https://www.kernel.org/doc/html/latest/admin-guide/perf-security.html
perf_event_paranoid setting is 4:
  -1: Allow use of (almost) all events by all users
      Ignore mlock limit after perf_event_mlock_kb without CAP_IPC_LOCK
>= 0: Disallow raw and ftrace function tracepoint access
>= 1: Disallow CPU event access
>= 2: Disallow kernel profiling
To make the adjusted perf_event_paranoid setting permanent preserve it
in /etc/sysctl.conf (e.g. kernel.perf_event_paranoid = <setting>)
```


```shell
$ cat /proc/sys/kernel/perf_event_paranoid
4

$ echo 'kernel.perf_event_paranoid = -1' | sudo tee -a /etc/sysctl.conf
[sudo] password for cc: 
kernel.perf_event_paranoid = -1

$ sudo sysctl -p
kernel.perf_event_paranoid = -1

$ cat /proc/sys/kernel/perf_event_paranoid
-1
```

```shell
$ perf record ./lab
WARNING: Kernel address maps (/proc/{kallsyms,modules}) are restricted,
check /proc/sys/kernel/kptr_restrict and /proc/sys/kernel/perf_event_paranoid.

Samples in kernel functions may not be resolved if a suitable vmlinux
file is not found in the buildid cache or in the vmlinux path.

Samples in kernel modules won't be resolved at all.

If some relocation was applied (e.g. kexec) symbols may be misresolved
even with a suitable vmlinux or kallsyms file.

Couldn't record kernel reference relocation symbol
Symbol resolution may be skewed if relocation was used (e.g. kexec).
Check /proc/kallsyms permission or run as root.
2025-11-14T23:37:41+08:00
Running ./lab
Run on (12 X 3899.99 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 2.06, 1.82, 1.56
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           28.1 ns         28.1 ns     25508336
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.159 MB perf.data (4103 samples) ]
```


```shell
$ echo 'kernel.kptr_restrict = 0'      | sudo tee -a /etc/sysctl.conf
kernel.kptr_restrict = 0

$ echo 'kernel.perf_event_paranoid = -1' | sudo tee -a /etc/sysctl.conf
kernel.perf_event_paranoid = -1

$ sudo sysctl -p
kernel.perf_event_paranoid = -1
kernel.kptr_restrict = 0
kernel.perf_event_paranoid = -1
```


## Theory

## Frontend Bound

## Backend Bound

## Bad Speculation

## Retiring

## Practice

```shell
$ ./make_benchmark_library.sh
```





### Data Packing

```shell
$ cd ~/Projects/perf-ninja/labs/memory_bound/data_packing/
```


```shell
$ cmake -E make_directory build
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..
$ cmake --build . --config Release --parallel 8
$ cmake --build . --target validateLab
$ cmake --build . --target benchmarkLab
```


### Loop Interchange 1

```shell
$ cd ~/Projects/perf-ninja/labs/memory_bound/loop_interchange_1
```

```shell
$ cmake -E make_directory build
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..
$ cmake --build . --config Release --parallel 8
$ cmake --build . --target validateLab
$ cmake --build . --target benchmarkLab

```

topdown analysis

```shell
$ perf stat --topdown -a taskset -c 0 ./lab
2025-11-18T23:04:54+08:00
Running ./lab
Run on (12 X 2200 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.65, 0.59, 0.63
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1     1459359992 ns   1458973655 ns            1

 Performance counter stats for 'system wide':

 %  tma_frontend_bound      %  tma_retiring %  tma_backend_bound %  tma_bad_speculation 
                  18.8                 28.8                 49.6                     2.8 

       1.470718902 seconds time elapsed
```

bound by memory and compute

```shell
$ toplev --core S0-C0 -l2 --no-desc taskset -c 0 ./lab
Consider disabling nmi watchdog to minimize multiplexing
(echo 0 | sudo tee /proc/sys/kernel/nmi_watchdog or
 echo kernel.nmi_watchdog=0 >> /etc/sysctl.conf ; sysctl -p as root)
Will measure complete system.
2025-11-18T23:03:17+08:00
Running ./lab
Run on (12 X 2199.98 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.59, 0.52, 0.62
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1     1465541445 ns   1462018800 ns            1
# 5.01-full-perf on Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz [cfl/skylake]
C0    BE               Backend_Bound               % Slots                       62.2   [ 8.0%]
C0    BE/Mem           Backend_Bound.Memory_Bound  % Slots                       28.5   [ 8.0%]
C0    BE/Core          Backend_Bound.Core_Bound    % Slots                       33.7   [ 8.0%]<==
C0-T0 MUX                                          %                              8.00 
C0-T1 MUX                                          %                              8.00 
Run toplev --describe Core_Bound^ to get more information on bottleneck
Add --run-sample to find locations
Add --nodes '!+Core_Bound*/3,+MUX' for breakdown.
```

iteration 10 times

```c++
// bench.cpp
BENCHMARK(bench1)->Iterations(10);
```

```shell
$ cmake --build . --target benchmarkLab
[ 25%] Building CXX object CMakeFiles/lab.dir/bench.cpp.o
[ 50%] Linking CXX executable lab
[100%] Built target lab
2025-11-18T23:10:57+08:00
Running ./lab
Run on (12 X 2200.04 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.61, 0.56, 0.59
---------------------------------------------------------------
Benchmark                     Time             CPU   Iterations
---------------------------------------------------------------
bench1/iterations:10 1439830183 ns   1438716584 ns           10
[100%] Built target benchmarkLab
```

optimize

```shell
$ cmake --build . --target benchmarkLab
[100%] Built target lab
2025-11-19T21:30:18+08:00
Running ./lab
Run on (12 X 3875.55 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.17, 0.37, 0.25
---------------------------------------------------------------
Benchmark                     Time             CPU   Iterations
---------------------------------------------------------------
bench1/iterations:10  143318676 ns    143281242 ns           10
[100%] Built target benchmarkLab
```

### Loop Interchange 2

#### build the project

```shell
$ cd ~/Projects/perf-ninja/labs/memory_bound/loop_interchange_2

$ cmake -E make_directory build
$ cd build

$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..
# cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS="-g" -DCMAKE_CXX_FLAGS="-g" -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..
$ cmake --build . --config Release --parallel 8

$ cmake -DCMAKE_BUILD_TYPE=Debug .. -DCMAKE_C_FLAGS="-g" -DCMAKE_CXX_FLAGS="-g" -DCMAKE_C_COMPILER=clang-17 -DCMAKE_CXX_COMPILER=clang++-17 ..
$ cmake --build . --config Debug --parallel 8

$ cmake --build . --target validateLab
$ cmake --build . --target benchmarkLab
```

```shell
$ cmake --build . --target benchmarkLab
[100%] Built target lab
2025-11-19T21:41:00+08:00
Running ./lab
Run on (12 X 2199.77 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 0.87, 0.41, 0.28
***WARNING*** ASLR is enabled, the results may have unreproducible noise in them.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1      507336869 ns    507292795 ns            5
[100%] Built target benchmarkLab
```



#### topdown analysis

```shell
$ toplev --core S0-C0 -l2 --no-desc --run-sample taskset -c 0 ./lab pexels-pixaby-434334.pbm output.pgm

$ perf report

# 热点
# dot += input[(r - radius + i) * width + c] * kernel[i];
```



#### deeper analysis

```shell
$ toplev --core S0-C0 -l2 --no-desc taskset -c 0 ./lab ../pexels-pixabay-434334.pbm output.pgm
Consider disabling nmi watchdog to minimize multiplexing
(echo 0 | sudo tee /proc/sys/kernel/nmi_watchdog or
 echo kernel.nmi_watchdog=0 >> /etc/sysctl.conf ; sysctl -p as root)
Will measure complete system.
2025-11-19T22:15:42+08:00
Running ./lab
Run on (12 X 3899.03 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 1.14, 1.13, 1.00
***WARNING*** ASLR is enabled, the results may have unreproducible noise in them.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1      335595146 ns    335430564 ns            2
# 5.01-full-perf on Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz [cfl/skylake]
C0    BE               Backend_Bound               % Slots                       70.0   [ 8.0%]
C0    BE/Mem           Backend_Bound.Memory_Bound  % Slots                       44.5   [ 8.0%]<==
C0    BE/Core          Backend_Bound.Core_Bound    % Slots                       25.5   [ 8.0%]
C0-T0 MUX                                          %                              8.00 
C0-T1 MUX                                          %                              8.00 
Run toplev --describe Memory_Bound^ to get more information on bottleneck
Add --run-sample to find locations
Add --nodes '!+Memory_Bound*/3,+MUX' for breakdown.
```

```shell
$ toplev --core S0-C0 -l2 --no-desc --run-sample taskset -c 0 ./lab ../pexels-pixabay-434334.pbm output.pgm
Consider disabling nmi watchdog to minimize multiplexing
(echo 0 | sudo tee /proc/sys/kernel/nmi_watchdog or
 echo kernel.nmi_watchdog=0 >> /etc/sysctl.conf ; sysctl -p as root)
Will measure complete system.
2025-11-19T22:17:50+08:00
Running ./lab
Run on (12 X 3887.08 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 1.05, 1.11, 1.01
***WARNING*** ASLR is enabled, the results may have unreproducible noise in them.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1      354537599 ns    354465237 ns            2
# 5.01-full-perf on Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz [cfl/skylake]
C0    BE               Backend_Bound               % Slots                       67.3   [ 8.0%]
C0    BE/Mem           Backend_Bound.Memory_Bound  % Slots                       44.3   [ 8.0%]<==
C0    BE/Core          Backend_Bound.Core_Bound    % Slots                       22.9   [ 8.0%]
C0-T0 MUX                                          %                              8.00 
C0-T1 MUX                                          %                              8.00 
Run toplev --describe Memory_Bound^ to get more information on bottleneck
Add --nodes '!+Memory_Bound*/3,+MUX' for breakdown.
Sampling:
perf record -g -e 'cycles:pp' -o perf.data -C 0,6 -a taskset -c 0 ./lab ../pexels-pixabay-434334.pgm output.pbm
2025-11-19T22:17:53+08:00
Running ./lab
Run on (12 X 3899.99 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x6)
  L1 Instruction 32 KiB (x6)
  L2 Unified 256 KiB (x6)
  L3 Unified 9216 KiB (x1)
Load Average: 1.20, 1.14, 1.02
***WARNING*** ASLR is enabled, the results may have unreproducible noise in them.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1      351418983 ns    351415108 ns            2
[ perf record: Woken up 2 times to write data ]
[ perf record: Captured and wrote 2.141 MB perf.data (6264 samples) ]
Run `perf report' to show the sampling results
```

```shell
```



#### optimize
