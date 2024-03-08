# Cocoon Artifact Overview
## Introduction

| Claim | Supported by Artifact? | Explanation |
| ----- | ---------------------- | ----------- |
| Cocoon is a Rust library that applications can use to provide fine-grained IFC | Yes | Source code for Cocoon and for several applications and benchmarks using Cocoon |
| Cocoon adds negligible run-time and code size overhead to applications | Yes | Performance experiments |
| Cocoon adds compile-time overhead proportional to the amount of code in secret blocks and side-effect-free functions | Yes | Performance experiments |

## Hardware (and OS) Dependencies
No special hardware is required. We think 8 GB of RAM should be sufficient.

The artifact requires Linux to work correctly. It mostly works on macOS/OSX (Intel chip), except for some of the commands in the evaluation scripts. Several benchmark games specify x86_64 architecture and cannot be run on macOS M1/M2 (Apple Silicon) without modification. 

## Getting Started Guide
The following steps describe how to install Rust, run the motivating example from Section 2 of the paper, and perform a general-purpose check that all examples compile.

1. There are two options for using Cocoon: the Docker image (`19-docker-image.tgz`) or the source code (`19-source-code.tgz`). The source code allows users to edit scripts or examples. Both compressed files are located inside `19.tgz`. Use the following command to unzip. 

        $ tar -xzf 19.tgz

    If using the Docker image, ensure that [Docker](https://docs.docker.com/get-docker/) has been installed and started. Then, load and run the provided image. 

        $ docker load -i 19-docker-image.tgz
        $ docker container run -ti cocoon 

    At this point, the shell should should show a root prompt similar to `root@075bdd84e164:/usr/src#`. The remaining steps will be run from the Docker container's shell. 

    Otherwise, if not using Docker, extract the source code and navigate to the root directory. 

        $ tar -xzf 19-source-code.tgz

2. The artifact's root directory contains rust-toolchain.toml, which forces the Rust compiler to use version `1.69.0-nightly` when compiling anything in the artifact. However, this functionality may not be supported by older versions of Rust or may inadvertently be overridden in your Rust installation. To check that the correct version is being used, run the following commands from the artifact:

        $ rustc --version
        rustc 1.69.0-nightly (44cfafe2f 2023-03-03)

        $ cargo --version
        cargo 1.69.0-nightly (9880b408a 2023-02-28)

    In the event that these commands do NOT report the correct versions, set the default Rust version with this command:

        $ rustup default nightly-2023-03-04

3. All examples are located in `ifc_examples` and can be run via `cargo run`. For instance, to run the calendar program used as a motivating example throughout the paper, use the following commands. There may be some warnings about unused functions, which can be ignored.

        $ cd ifc_examples/paper_calendar
        $ cargo run

4. Every example in the `ifc_examples` folder can be tested using the script `tests/test-all.sh`, which runs `cargo` to ensure that each example compiles and runs as expected. This script comes with several different argument options: clean, check, build, or run. The arguments correspond to the same-named `cargo` commands.  For example, to confirm that each example compiles, use the following command:

        $ tests/test-all.sh build

## Step by Step Instructions
This section details how to reproduce experiments from the paper. Note that all experiments in the paper were run on a quiet machine with a 16-core Intel Xeon Gold 5218 at 2.3 GHz with 187 GB RAM running Linux. (The paper submission says that the machine has 18 GB of RAM because of a typo, which we'll fix in the camera-ready version. We think 8 GB of RAM should be sufficient, e.g., we can run the experiments on a laptop with 8 GB of RAM.) The following steps assume usage of the Docker container. 

### Motivating Example (Section 2)
This example computes the number of overlapping days of availability on two calendars, belonging to Alice and Bob. In this example, whether Alice or Bob is available on a given day is secret.

This example can be found in `ifc_examples/paper_calendar/src/main.rs`. The expansion of the secret block shown in Figure 7 of the paper can be reproduced using the `expand.sh` script located in `ifc_examples/paper_calendar` (as shown below).

        $ cd ifc_examples/paper_calendar
        $ ./expand.sh

The resulting expanded code takes a few seconds to generate and will be located in `ifc_examples/paper_calendar/expanded.rs`.

### Spotify TUI (Section 5.1)
Spotify TUI is a Spotify client written in Rust that allows an end user to interact with Spotify from a terminal. Spotify TUI is a popular GitHub project with over 12,000 lines of code and 90 contributors. Cocoon is used to protect the client secret, which is similar to a user account password.

The original Spotify TUI code, downloaded from GitHub, is located in `ifc_examples/spotify-tui/spotify-tui-original` while the modified Cocoon version is located in `ifc_examples/spotify-tui/spotify-tui-cocoon`.

The empirical results reported in Tables 1 and 2 can be reproduced using the `tests/eval-spt.sh` script. Note that the original experiment used 100,000 trials to measure run time, to account for high run-to-run variability, which takes a couple hours to complete on the machine we used. In the artifact we've set `RUNTIME_TRIALS` to 10,000 to use less time. The number of run-time trials can be changed by modifying the value of `RUNTIME_TRIALS` near the top of `tests/eval-spt.sh`.
```
# from root directory
$ tests/eval-spt.sh > tests/eval_results/spotify-results.txt
```

The resulting file (`tests/eval_results/spotify-tui_samples/spotify-results.txt`) has the following format.
```
COMPILE TIME
tests/eval-results/spotify-tui-original-compile-results.csv: duration (s): [mean] +- [95% confidence interval]
tests/eval-results/spotify-tui-original-compile-results.csv: elapsed (s): [mean] +- [95% confidence interval]
tests/eval-results/spotify-tui-cocoon-compile-results.csv: duration (s): [mean] +- [95% confidence interval]
tests/eval-results/spotify-tui-cocoon-compile-results.csv: elapsed (s): [mean] +- [95% confidence interval]

RUN TIME
Running 10000 trials...
spotify-tui-original-results.csv: elapsed (ns): [mean] +- [95% confidence interval]
spotfiy-tui-cocoon-results.csv elapsed (ns): [mean] +- [95% confidence interval]
Finished 10000 trials...

LINES OF CODE
[relative path to modified file]  |  [number of lines changed] +-
[...]
[#] files changed, [#] insertions(+), [#] deletions(-)

EXECUTABLE SIZE
ifc_examples/spotify-tui/spotify-tui-cocoon/ executable size (bytes): [size]
ifc_examples/spotify-tui/spotify-tui-original/ executable size (bytes): [size]
```

`eval-spt.sh` also generates files similar to `tests/eval_results/spotify-tui_samples/spotify-tui-cocoon-compile-results-sample.csv` and `tests/eval_results/spotify-tui_samples/spotify-tui-original-results-sample.csv`. The former gives the output from compile time trials for the Cocoon-integrated version of Spotify TUI while the latter gives the compile time trials for the original implementation. Both files have the following format. 
```
duration (s),elapsed (s)
[#],[#]
[...]
```

### Servo (Section 5.2)
Servo is Mozilla's browswer engine written in Rust [Linux Foundation 2022]. At the time of writing, it is the sixth-most popular Rust project on GitHub and has been influential in the development of Rust. Mozilla's experiences in creating Servo contributed to Rust's early design. In this example, we use Cocoon to protect responses recieved from an origin server following an HTTP request to prevent other origins, such as JavaScript program, from reading the response body.

Due to its size, Servo's code is not located within the artifact. Instructions for accessing the Cocoon-integrated Servo code and reproducing our results are located in `ifc_examples/servo/README.md`. Evaluating Servo takes several hours and produces four files.

`tests/eval_results/servo_samples/` contain sample output files from the Servo evaluation. `modified_builds-sample.csv` and `modified_tests-sample.csv` give the compile times and run times, respectively, for Servo with Cocoon modifications over several trials. `original_builds-sample.csv` and `original_tests-sample.csv` give the compile and run times for the original Servo implementation. All four files having the following format.
```
time,max_rss
[Trial 1: time (s)],[maximum memory usage (KB)]
[Trial 2: time (s)],[maximum memory usage (KB)]
[...]
```

### Battleship (Section 5.3)
This example demonstrates Cocoon's ability to protect secret values over a communication channel, using the game of Battleship. This example was also used in Jif and JRIF [Kozyri et al. 2016; Myers et al. 2006] and has been converted to Rust and integrated with Cocoon. In this example, the secret information is the placement of each player's ships.

The non-Cocoon and Cocoon versions of Battleship can be found in `ifc_examples/battleship-no-ifc/src/main.rs` and `ifc_examples/battleship/src/main.rs`, respectively.

The empirical results reported in Tables 1 and 2 can be reproduced using the `tests/eval-battleship.sh` script. The script takes approximately 5-10 minutes to run.
```
# from root directory
$ tests/eval-battleship.sh > tests/eval_results/battleship-results.txt
```

The resulting file (`tests/eval_results/battleship-results.txt`) has the following format.
```
COMPILE TIME
[file path]/battleship-no-ifc-compile-results.csv: duration: [mean] +- [95% confidence interval]
[file path]/battleship-no-ifc-compile-results.csv: elapsed (s): [mean] +- [95% confidence interval]
[file path]/battleship-compile-results.csv: duration: [mean] +- [95% confidence interval]
[file path]/battleship-compile-results.csv: duration (s): [mean] +- [95% confidence interval]

RUN TIME
tests/eval_results/battleship-no-ifc-runtime-results.csv: run time: [mean] +- [95% confidence interval]
test/eval_results/battleship-runtime-results.csv: run time: [mean] +- [95% confidence interval]

LINES OF CODE
[relative file path]   |  [number of lines modified] +-
[...]
[#] files changed, [#] insertions(+), [#] deletions(-)

EXECUTABLE SIZE
ifc_examples/battleship-no-ifc executable size (bytes): [size]
ifc_examples/battleship executable size (bytes): [size]
```

`eval-battleship.sh` also generates four output files, samples of which can be found in `tests/eval_results/battleship_samples/`. `battleship-compile-results-sample.csv` and `battleship-no-ifc-compile-results-sample.csv` give the compile times from each trial for Battleship with and without Cocoon integration, respectively. Both files have the following format. 
```
duration (s),elapsed (s)
[#],[#]
[...]
```

`battleship-runtime-results-sample.csv` and `battleship-no-ifc-runtime-results-sample.csv` give the run time for each trial for the Cocoon and original implementations, respectively. The format for these files is given below. 
```
run time
[#]
[...]
```

### Benchmark Games (Section 5.4)
Section 5.4 refers to submitted supplementary material. This material can be found in our artifact as `perf-eval.pdf`.

The Computer Language Benchmark Games [Debian benchmarksgame-team 2022] contains nine programs: `binary-trees`, `fannkuch-redux`, `fasta`, `k-nucleotide`, `mandelbrot`, `n-body`, `pidigits`, `regex-redux`, and `spectral-norm`. To assess the peformance impacts of Cocoon, we modified each of these programs to perform all computation inside `secret_block`s.

The original and Cocoon versions of the Benchmark Games are located in `ifc_examples/benchmark_games/originals` and `ifc_examples/benchmark_games`, respectively.

To reproduce the run time, compile time, and executable size results reported in Table 3 of the paper, execute the `tests/eval-bm-games.sh` script. The script takes approximately 30 minutes to run.
```
# from root directory
$ tests/eval-bm-games.sh > tests/eval_results/bm-games-results.txt
```

The resulting file (`tests/eval_results/bm-games-results.txt`) has the following format.
```
COMPILE TIME
tests/[package name]-compile-results.csv: compile_time_package: [mean] +- [95% confidence interval]
tests/[package name]-compile-results.csv: compile_time_total: [mean] +- [95% confidence interval]
[...]

RUN TIME
Benchmarking [package name]: (1 / 10)
Benchmarking [package name]: (2 / 20)
[...]
Start benchmark for original implementation.
Benchmarking [package name]: (1 / 10)
Benchmarking [package name]: (2 / 20)
[...]

LINES OF CODE
[relative file path]   |  [number of lines modified] +-
[...]
[#] files changed, [#] insertions(+), [#] deletions(-)

EXECUTABLE SIZE
[package name] executable size (bytes): [size]
[...]
```

For each package, four files are produced: `[package name]-compile-results-original-sample.csv`, `[package  name]-compile-results-sample.csv`, `[package name]-run-results-sample.csv`, and `originals/[package name]-sample.csv`. The first two files give the output of running compile time trials for the original and Cocoon implementations, respectively. These files have the following format. 
```
compile_time_package,compile_time_total
[#],[#]
[...]
```

`[package name]-run-results-sample.csv` has the run time trial results for the Cocoon-integrated implementation, and `originals/[package name]-sample.csv` contains run time results for the original implementation. These files have the following format, where `max_rss` corresponds to the maximum memory usage in KB. 
```
time,max_rss
[#],[#]
[...]
```

