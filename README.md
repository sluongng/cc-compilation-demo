# Benchmark CC rules

These results were captured in a poor network environment and thus, may not reflect the actual result.

TODO(sluongng): re-do these tests in an isolated environment.


## Clean run

```bash
> hyperfine 'bazel build //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build //...
  Time (mean ± σ):     13.700 s ±  0.891 s    [User: 0.026 s, System: 0.030 s]
  Range (min … max):   12.841 s … 15.735 s    10 runs
```

# Local cached run

```bash
> hyperfine 'bazel build //...' -w 1
Benchmark 1: bazel build //...
  Time (mean ± σ):      75.6 ms ±   5.3 ms    [User: 10.4 ms, System: 12.5 ms]
  Range (min … max):    67.2 ms …  87.8 ms    34 runs
```

# Clean run with repository cache

```bash
> hyperfine 'bazel build --config=repo-cache //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=repo-cache //...
  Time (mean ± σ):     13.403 s ±  0.434 s    [User: 0.024 s, System: 0.028 s]
  Range (min … max):   12.944 s … 14.179 s    10 runs
```

# With BES enabled

### Clean

```bash
> hyperfine 'bazel build --config=bes //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=bes //...
  Time (mean ± σ):     15.646 s ±  1.031 s    [User: 0.025 s, System: 0.029 s]
  Range (min … max):   14.632 s … 18.042 s    10 runs
```

### Cached

```bash
> hyperfine 'bazel build --config=bes //...' -w 1
Benchmark 1: bazel build --config=bes //...
  Time (mean ± σ):      2.385 s ±  0.243 s    [User: 0.017 s, System: 0.020 s]
  Range (min … max):    2.174 s …  2.964 s    10 runs
```

# With Remote Cache enabled

### Clean (locally)

```bash
> hyperfine 'bazel build --config=remote-cache //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=remote-cache //...
  Time (mean ± σ):     54.059 s ± 12.697 s    [User: 0.033 s, System: 0.034 s]
  Range (min … max):   40.634 s … 83.073 s    10 runs
```

### Cached

```bash
> hyperfine 'bazel build --config=remote-cache //...' -w 1
Benchmark 1: bazel build --config=remote-cache //...
  Time (mean ± σ):      4.114 s ±  0.402 s    [User: 0.017 s, System: 0.020 s]
  Range (min … max):    3.651 s …  4.837 s    10 runs
```

# With Remote Build Execution(RBE)

### No remote cache

By setting a random value to `--remote_instance_name` flag of Bazel on each build, we effectively discard all cache from previous builds

```bash
> hyperfine 'bazel build --config=remote-exec --remote_instance_name=$(openssl rand -hex 12) //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=remote-exec --remote_instance_name=$(openssl rand -hex 12) //...
  Time (mean ± σ):     40.005 s ±  2.026 s    [User: 0.031 s, System: 0.038 s]
  Range (min … max):   37.424 s … 44.717 s    10 runs
```

### Cached (remotely)

We do not set `--remote_instance_name` to allow remote cache hits

```bash
> hyperfine 'bazel build --config=remote-exec //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=remote-exec //...
  Time (mean ± σ):     28.026 s ±  9.137 s    [User: 0.016 s, System: 0.020 s]
  Range (min … max):   18.308 s … 51.079 s    10 runs
```

# Using RBE without downloading outputs

This is also known as [Build without the Bytes(BwtB)](https://blog.bazel.build/2019/05/07/builds-without-bytes.html)

```bash
> hyperfine 'bazel build --config=without-bytes //...' -s 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=without-bytes //...
  Time (mean ± σ):      7.813 s ±  4.900 s    [User: 0.022 s, System: 0.029 s]
  Range (min … max):    5.081 s … 21.467 s    10 runs
```
