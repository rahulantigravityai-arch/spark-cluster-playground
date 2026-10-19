# Spark Cluster Playground

An interactive, single-file simulator for building intuition about Spark and Databricks compute. Change the data volume, executors, cores, memory, shuffle partitions and skew, then watch the cluster replay the run and explain what went wrong and how to fix it.

**Live demo:** https://rahulantigravityai-arch.github.io/spark-cluster-playground/

## What it shows

- A hypothetical workload: 1 billion transaction records, one small price plan table, and 400 other tables (big and small). Every value can be edited.
- The cluster as executors (workers) made of cores. Cores turn green when busy, amber when spilling to disk, red when a skewed task holds the run up, and grey when idle.
- Live metrics: total time, slowest task, core utilisation, tasks spilling, data moved on the network, files written, and cost.
- Charts for cores busy over time, task durations in the big stage, and memory per task.
- Plain-language diagnosis with one-click fixes: AQE skew join, salting, broadcast join, more partitions, more memory, optimized writes, and a rough executor count for your time target.
- Scenarios: healthy daily run, too few executors, festive peak (3x data), hot key skew, too few partitions.
- Pin a result as a baseline, change a setting, and see the percentage difference.

## Run it

It is one static file with no build step.

```
git clone https://github.com/rahulantigravityai-arch/spark-cluster-playground.git
cd spark-cluster-playground
open index.html        # or double-click it
```

Fonts load from Google Fonts. Without a network connection the page falls back to system fonts.

## How the model works

This is a teaching model with round numbers. It shows direction and relative size of effects. It is not a benchmark of real jobs.

- Stage 1 joins the transaction table with the price plan table. Stage 2 processes the other tables.
- Time per task is data size divided by the speed of one core (20 MB/s for the join stage, 40 MB/s for plain table work), plus 0.15 s of scheduling overhead.
- A join task needs about 2.5x its data size in memory. Each core gets a share of the executor's execution memory (45% of node memory divided by cores). A task above its share spills, which makes it up to 3x slower.
- Tasks go to whichever core frees up first, so the slowest task sets the finish time.
- A shuffle join moves both tables across the network at 200 MB/s per executor. A broadcast join copies the small table to every executor.
- Each task writes one file, unless optimized writes is on, which targets 128 MB files.

To change the assumptions, edit the `K` constants and the `DEF` and `PRESETS` objects at the top of the script in `index.html`.

## Files

- `index.html`: the whole app (HTML, CSS and JavaScript).
- `README.md`: this file.
