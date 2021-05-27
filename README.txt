Abstract: 

We present the artifact for the accepted paper - Parallel Shadow Execution to Accelerate the Debugging of Numerical
Errors. This artifact provides the link to the source code and step-by-step instructions to reproduce the performance
graphs and case study from the
accepted paper. We also provide the test harness to evaluate the correctness of our tool. In this artifact, we provide
the scripts and instructions required to execute the different parts of the experiment automatically. 
Experiments are mainly divided into three parts. The first part focuses on correctness evaluation, the second part focuses on
the case study, and the third part focuses on performance evaluation. Complete performance evaluation can require more than
24 hours to run the benchmarks. Hence, we have provided the shorter version, which produces the partial performance
results and does not take more than an hour. 

* HARDWARE & SOFTWARE RECOMMENDATION:

	We recommend evaluating our artifact with a machine that has:
	Disk Storage: at least 10 GB available
	RAM memory: 16GB
	Processor: 4GHz
	OS: Ubuntu 18.04
	We ran all of our experiments on a machine with similar
	specification with 126GB of RAM and 64 cores.
	We have disabled turbo-boost and hyper-threading before conducting the experiments.

__________________________________________________________________

* DOWNLOADING PREBUILT DOCKER IMAGE

   	 We have prebuilt a docker image and hosted it in the docker hub.

(1) Install docker if not already installed by following the
   	 installation documentation in this link:
   	 https://docs.docker.com/install/

	 We recommend installing Docker and evaluating our artifact on a machine with Ubuntu. Although docker can be used with
	 Windows or macOS, docker may run on top of a Linux VM.

(2) Download the prebuilt docker image by using the command:
	 
	 $ sudo docker pull sc1696/pfsan_artifact:latest

	 The docker image is roughly 1.21GB

(3) Run the docker image:

	 $ sudo docker run -it --cap-add=SYS_PTRACE --security-opt seccomp=unconfined sc1696/pfsan_artifact:latest

* STEP BY STEP INSTRUCTIONS TO EVALUATE THE ARTIFACT

(1) Testing the correctness of PFSan (Section 5:Ability to detect FP errors):

  This process should take less than 4 minutes. This script will run microbenchmarks with PFSan and report numerical errors. It runs 43 benchmarks and reports numerical errors in these benchmarks, 
        i.e., catastrophic cancellation, NaN, Inf, branch flips, and integer conversion error. The
	  script also outputs whether this error is correctly found (expected
	  with green letters) or incorrectly found (unexpected with red
	  letters). The script should only report "expected" and not
	  "unexpected."

	  Finally, when the script terminates, it reports the total
	  number of microbenchmarks, the total number of
	  microbenchmarks that reports each type of error, and whether
	  the numbers are correct or not.

	  Therefore, you should see a total of 43 benchmarks, 16
	  benchmarks with catastrophic cancellation, 0 programs with
	  NaN computation, 2 programs with Inf,
        5 programs with branch flips, and 0 program
	  with integer conversion error.

  $ cd psan_fse_2021/correctness_suite
  $ python3 correctness_test.py
  $ cd ..


(2). Debugging (Section 5: (Debugging a previously unknown error in Cholesky from Polybench))

  To run Cholesky with gdb, compile runtime with -O0

  $ export SET_DEBUG=DEBUG //reset to debug mode
  $ cd runtime
  $ make clean
  $ make
  $ cd ../case_study/cholesky
  $ make clean
  $ make
  $ gdb ./cholesky_fp
  $ b handleReal.cpp:1416
    Make breakpoint pending on future shared library load? (y or [n]) y
  $ r
  $ call fpsan_trace(buf_id, res)

  This will show the error trace matching with Figure 7(a) in the paper.
  This trace shows the "Inst ID: Opcode: Op1_Inst_ID: Op2_Inst_ID: (real value: computed value: error)"
  As it can be seen in the trace, the Inf exception has occurred in instruction 189.
  If you trace back, then you would notice that error was propagated from instruction 450.
  However, instruction 450 is computed in a different function. To trace back the error in the instruction
  450 follow the below instructions.


  $ b handleReal.cpp:1367 if res->error >= 28
  $ r
    Start it from the beginning? (y or n) y
  $ call fpsan_trace(buf_id, res)

  You will get the trace matching Figure 7(b).
  This error trace would show that the addition of 1 and 2.70400000000000e7 has resulted in an error of 28 bits.
  The first instruction in the trace shows real computation as 2.70400010000000e7 and floating-point computation as 27040000, hence rounding error has occurred.
  $ quit

(3). Performance testing (Shorter Version) (Section 5: (Performance speedup with PFSan compared to FPSanitizer))

  This version will take around 

  $ cd /home/psan_fse_2021/performance
  $ ./run_perf_short.sh
  This script will run performance benchmarks and produce graphs similar to speedup.pdf(Figure 8)
  and slowdown.pdf(Figure 9) with only 128 bits of precision and 1, 8, 32, and 64 number of cores.
  Please note that we have disabled turbo-boost and hyper-threading, and our data is generated on a 64 core machine.

(4). Performance testing (Full Version) (Section 5: (Performance speedup with PFSan compared to FPSanitizer))

  $ cd psan_fse_2021/performance
  $ ./run_perf.sh
  This script will run performance benchmarks and produce graphs speedup.pdf(Figure 8)
  and slowdown.pdf(Figure 9).
  Please note that we have disabled turbo-boost and hyper-threading, and our data is
  generated on a 64 core machine.
