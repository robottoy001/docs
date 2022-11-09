### related issue 

### Description:

### How to reproduce 

### log before fixing


### log after fixing 

```
1971-04-07T10:44:11+00:00
Running /data/local/tmp/musle
Run on (8 X 2045 MHz CPU s)
Load Average: 14.58, 11.73, 5.98
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
--------------------------------------------------------------------------------------------
Benchmark                                  Time             CPU   Iterations UserCounters...
--------------------------------------------------------------------------------------------
BM_stdio_fwrite/8                       6.14 ns         6.10 ns    100000000 bytes_per_second=1.22074G/s
BM_stdio_fwrite/64                      11.8 ns         11.7 ns     60130995 bytes_per_second=5.08972G/s
BM_stdio_fwrite/512                     55.8 ns         55.5 ns     12661448 bytes_per_second=8.59766G/s
BM_stdio_fwrite/1024                    96.7 ns         96.1 ns      7349564 bytes_per_second=9.92345G/s
BM_stdio_fwrite/8192                     194 ns          193 ns      3627530 bytes_per_second=39.4468G/s
BM_stdio_fwrite/16384                    194 ns          193 ns      3640273 bytes_per_second=79.1532G/s
BM_stdio_fwrite/32768                    194 ns          193 ns      3620036 bytes_per_second=157.822G/s
BM_stdio_fwrite/65536                    194 ns          193 ns      3626092 bytes_per_second=316.921G/s
BM_stdio_fwrite/131072                   194 ns          193 ns      3631068 bytes_per_second=631.8G/s
BM_stdio_fwrite_unbuffered/8             194 ns          193 ns      3635468 bytes_per_second=39.5754M/s
BM_stdio_fwrite_unbuffered/64            194 ns          193 ns      3628255 bytes_per_second=316.386M/s
BM_stdio_fwrite_unbuffered/512           194 ns          193 ns      3635774 bytes_per_second=2.47493G/s
BM_stdio_fwrite_unbuffered/1024          194 ns          193 ns      3627794 bytes_per_second=4.95046G/s
BM_stdio_fwrite_unbuffered/8192          193 ns          193 ns      3635714 bytes_per_second=39.6226G/s
BM_stdio_fwrite_unbuffered/16384         194 ns          193 ns      3632992 bytes_per_second=79.2114G/s
BM_stdio_fwrite_unbuffered/32768         194 ns          193 ns      3634092 bytes_per_second=158.404G/s
BM_stdio_fwrite_unbuffered/65536         195 ns          194 ns      3631598 bytes_per_second=315.241G/s
BM_stdio_fwrite_unbuffered/131072        194 ns          193 ns      3630627 bytes_per_second=633.269G/s
```