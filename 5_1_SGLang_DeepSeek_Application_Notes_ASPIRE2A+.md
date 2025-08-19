[toc]

# Set up SGLang Framework

## Install Miniforge and Python Environment

The following commands

1. Download and install Miniforge3 for conda package management
2. Create a dedicated Python 3.12 environment for SGLang
3. Install the SGLang framework with all dependencies

```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh -O ${HOME}/scratch/Miniforge3-Linux-x86_64.sh

time bash ${HOME}/scratch/Miniforge3-Linux-x86_64.sh -b -p ~/scratch/miniforge
# real	0m24.953s

${HOME}/scratch/miniforge/bin/conda init

bash

time conda create -p ${HOME}/scratch/py312 python=3.12 -y
# real    0m57.389s

time ${HOME}/scratch/py312/bin/pip install --upgrade pip
# real	0m3.906s
```

## Install SGLang Package

Install the latest SGLang framework with all optional dependencies:

```bash
${HOME}/scratch/py312/bin/pip install "sglang[all]==9999"

time ${HOME}/scratch/py312/bin/pip install "sglang[all]>=0.5.0rc2"
# real	28m23.786s
```

Verify the installation by checking available packages and testing the benchmark tool:

```bash
${HOME}/scratch/py312/bin/pip list

time ${HOME}/scratch/py312/bin/python3 -m sglang.bench_offline_throughput --help
# real	0m17.581s
```

## Download ShareGPT Dataset

Download the ShareGPT dataset for benchmarking:

```bash
cd ${HOME}/scratch
time wget https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered/resolve/main/ShareGPT_V3_unfiltered_cleaned_split.json
# real	0m2.629s
```

# Run Distributed SGLang Benchmark

## Create PBS Job Script

Create a script file `${HOME}/sglang.sh` with the following contents:

```bash
tee ${HOME}/sglang.sh << 'EOF'
#!/bin/bash
#PBS -P 51401008
#PBS -l walltime=420
#PBS -l select=2:ncpus=112:ngpus=8:mpiprocs=2:mem=1880gb
#PBS -j oe
#PBS -M 393958790@qq.com
#PBS -m abe
##PBS -l other=hyperthread

module load cuda

time /usr/mpi/gcc/openmpi-4.1.7a1/bin/mpirun \
-hostfile ${PBS_NODEFILE} \
-map-by ppr:1:node:PE=112 -oversubscribe -use-hwthread-cpus \
-bind-to none --report-bindings -display-map \
-tag-output -output-filename ${HOME}/run/sglang.${PBS_JOBID} \
-x PATH \
-x NCCL_DEBUG=INFO \
-x DIST_INIT_ADDR=$(head -n 1 $PBS_NODEFILE) \
bash -c 'time ${HOME}/scratch/py312/bin/python3 \
-m sglang.bench_offline_throughput \
--model-path deepseek-ai/DeepSeek-R1 \
--dataset-path ${HOME}/scratch/ShareGPT_V3_unfiltered_cleaned_split.json \
--num-prompts 2000 --load-format dummy --seed 2025 --dtype bfloat16 \
--tp 16 --nnodes 2 --trust-remote-code \
--dist-init-addr ${DIST_INIT_ADDR}:5000 --node-rank ${OMPI_COMM_WORLD_RANK}' \
2>&1 | tee ${HOME}/run/stdout.sglang.${PBS_JOBID}
EOF
```

## Submit the Job

Submit the benchmark job to the PBS scheduler:

```bash
cd ${HOME}/run
qsub ${HOME}/sglang.sh
```

## Monitor Job Status

Check the job status and monitor the output:

```bash
qstat -u ${USER}
tail -f ${HOME}/run/stdout.sglang.*
```

# Read the Benchmark Results

## Extract Performance Results

Use the following command to extract the benchmark results:

```bash
cat ${HOME}/sglang.sh.o74975 | grep "Offline Throughput Benchmark Result" -A 11
```

**Example Output:**

```bash
[1,0]<stdout>:====== Offline Throughput Benchmark Result =======
[1,0]<stdout>:Backend:                                 engine    
[1,0]<stdout>:Successful requests:                     2000      
[1,0]<stdout>:Benchmark duration (s):                  172.37    
[1,0]<stdout>:Total input tokens:                      626729    
[1,0]<stdout>:Total generated tokens:                  388685    
[1,0]<stdout>:Last generation throughput (tok/s):      33.36     
[1,0]<stdout>:Request throughput (req/s):              11.60     
[1,0]<stdout>:Input token throughput (tok/s):          3635.89   
[1,0]<stdout>:Output token throughput (tok/s):         2254.91   
[1,0]<stdout>:Total token throughput (tok/s):          5890.80   
[1,0]<stdout>:==================================================
```

## Understanding SGLang Performance Metrics

Refer to [Achieving Faster Open-Source Llama3 Serving with SGLang Runtime (vs. TensorRT-LLM, vLLM) | LMSYS Org](https://lmsys.org/blog/2024-07-25-sglang-llama3/#benchmark-setup)

The SGLang offline throughput benchmark provides metrics that measure inference performance. The key metrics include:

**Performance Indicators:**

- **Total token throughput (tok/s)**: The most important metric - combines input and output token processing speed
- **Request throughput (req/s)**: Number of requests processed per second
- **Input token throughput (tok/s)**: Speed of processing input tokens
- **Output token throughput (tok/s)**: Speed of generating output tokens

**Higher values indicate better performance.**
