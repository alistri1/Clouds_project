# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->246.294<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->13.997<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->436092.2<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->15184.6<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.443<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2656.581 ms
2 threads: 2083.108 ms
4 threads: 2145.662 ms
8 threads: 2226.579 ms

The fastest configuration was 2 threads, with an execution time of 2083.108 ms.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive attention implementation uses Python loops, causing interpreter overhead and inefficient memory access. The vectorized version uses optimized NumPy operations that execute compiled instructions and utilize CPU vector units (SIMD), improving cache efficiency and reducing unnecessary operations. As a result, execution time decreased from 246.294 ms to 13.997 ms, achieving a 17.6× speedup while performing the same mathematical calculations.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
Performance improved from 2656.581 ms with one thread to 2083.108 ms with two threads. However, using four or eight threads increased execution time to 2145.662 ms and 2226.579 ms. The Colab runtime has limited vCPU resources, so additional threads compete for CPU time and shared memory bandwidth. Thread scheduling and synchronization introduce overhead, making more than two threads less efficient for this workload.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training involves large matrix multiplications in attention and feed-forward layers, which GPUs execute efficiently using thousands of parallel arithmetic units. The NVIDIA T4 completed 100 training steps in 15184.6 ms, compared with 436092.2 ms on the CPU, achieving a 28.7× speedup. Small sequential workloads would not benefit similarly because GPU kernel-launch overhead and CPU–GPU data transfers could outweigh the advantages of parallel computation.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism stops helping when hardware limitations, sequential dependencies, memory transfers, and coordination overhead outweigh the benefits of dividing work.
In the vectorization experiment, optimized NumPy operations reduced execution time from 246.294 ms to 13.997 ms, achieving a 17.6× speedup. However, sequential dependencies and memory access still limit further acceleration.
In the CPU multithreading experiment, two threads performed best at 2083.108 ms. Increasing to eight threads raised execution time to 2226.579 ms because limited CPU resources, synchronization, and memory bandwidth caused additional overhead.
In the GPU experiment, 100 training steps took 15184.6 ms, compared with 436092.2 ms on the CPU, achieving a 28.7× speedup. Although GPUs accelerate parallel matrix operations, sequential computations, kernel-launch overhead, and CPU–GPU data transfers limit further improvements.
Therefore, parallelism is most effective when sufficient independent work is available and communication costs remain smaller than the performance gains.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would benchmark GPU training using different batch sizes while keeping the total processed data constant. Larger batches should initially improve GPU utilization, but eventually memory limitations and overhead may reduce performance gains. If execution time continued improving proportionally, it would suggest that the original workload did not fully utilize the GPU.
<!--/answer-->
