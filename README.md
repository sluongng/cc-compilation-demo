# Benchmark CC rules

Benchmarks are done on Github's Codespace VM (4 CPU, 8G RAM)

## Local build

### Clean run

```bash
$ hyperfine 'bazel build //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build //...
  Time (mean ± σ):     15.422 s ±  0.580 s    [User: 0.012 s, System: 0.010 s]
  Range (min … max):   14.838 s … 16.917 s    10 runs
```

### Local cached run

```bash
$ hyperfine 'bazel build //...' -w 1
Benchmark 1: bazel build //...
  Time (mean ± σ):     205.4 ms ±  41.3 ms    [User: 8.5 ms, System: 8.1 ms]
  Range (min … max):   141.8 ms … 279.4 ms    16 runs
```

### Clean run with repository cache

```bash
$ hyperfine 'bazel build --config=repo-cache //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=repo-cache //...
  Time (mean ± σ):     15.589 s ±  0.548 s    [User: 0.011 s, System: 0.012 s]
  Range (min … max):   14.827 s … 16.640 s    10 runs
```

## With BES enabled

### Clean

```bash
$ hyperfine 'bazel build --config=bes //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=bes //...
  Time (mean ± σ):     17.508 s ±  0.258 s    [User: 0.010 s, System: 0.014 s]
  Range (min … max):   17.144 s … 17.900 s    10 runs
```

### Cached

```bash
$ hyperfine 'bazel build --config=bes //...' -w 1
Benchmark 1: bazel build --config=bes //...
  Time (mean ± σ):      1.880 s ±  0.088 s    [User: 0.007 s, System: 0.010 s]
  Range (min … max):    1.788 s …  2.054 s    10 runs
```

## With Remote Cache enabled

### Clean (locally)

```bash
$ hyperfine 'bazel build --config=remote-cache //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=remote-cache //...
  Time (mean ± σ):     50.616 s ±  0.416 s    [User: 0.017 s, System: 0.015 s]
  Range (min … max):   50.206 s … 51.530 s    10 runs
```

Because our actions are very light weighted (generate source code from template + compile + link),
downloading the cached action result remotely became an overhead, leading to slower build.

But if we were to add `sleep 60 && <gen source code>` into our source code generate action to act as
big, espensive actions, the remote cache should almost certainly be faster.

### Cached

```bash
$ hyperfine 'bazel build --config=remote-cache //...' -w 1
Benchmark 1: bazel build --config=remote-cache //...
  Time (mean ± σ):      2.190 s ±  0.220 s    [User: 0.009 s, System: 0.008 s]
  Range (min … max):    1.993 s …  2.602 s    10 runs
```

## With Remote Build Execution(RBE)

### No remote cache

By setting a random value to `--remote_instance_name` flag of Bazel on each build, we effectively discard all cache from previous builds

```bash
$ hyperfine 'bazel build --config=remote-exec --remote_instance_name=$(openssl rand -hex 12) //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=remote-exec --remote_instance_name=$(openssl rand -hex 12) //...
  Time (mean ± σ):     41.024 s ±  4.564 s    [User: 0.018 s, System: 0.014 s]
  Range (min … max):   35.233 s … 50.490 s    10 runs
```

### Cached (remotely)

We do not set `--remote_instance_name` to allow remote cache hits

```bash
$ hyperfine 'bazel build --config=remote-exec //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=remote-exec //...
  Time (mean ± σ):     10.978 s ±  0.361 s    [User: 0.013 s, System: 0.010 s]
  Range (min … max):   10.270 s … 11.703 s    10 runs
```

## Using RBE without downloading outputs

This is also known as [Build without the Bytes(BwtB)](https://blog.bazel.build/2019/05/07/builds-without-bytes.html).

This would be very relevant if our action outputs are large and take a long time to download over the network to Engineers' laptops.

### No remote cache

```bash
$ hyperfine 'bazel build --config=without-bytes --remote_instance_name=$(openssl rand -hex 12) //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=without-bytes --remote_instance_name=$(openssl rand -hex 12) //...
  Time (mean ± σ):     40.801 s ±  3.366 s    [User: 0.016 s, System: 0.017 s]
  Range (min … max):   36.266 s … 47.615 s    10 runs
```

### Cached (remotely)

```bash
$ hyperfine 'bazel build --config=without-bytes //...' -p 'bazel clean --expunge' -w 1
Benchmark 1: bazel build --config=without-bytes //...
  Time (mean ± σ):     10.027 s ±  0.208 s    [User: 0.013 s, System: 0.009 s]
  Range (min … max):    9.754 s … 10.396 s    10 runs
```
