[toc]

# DeepSeek AI Task

## 1. Task Workload and Input

Participants are required to optimize SGLang-based DeepSeek inference performance.

- Framework: [SGLang](https://github.com/sgl-project/sglang) - Structured Generation Language for Large Language Models
- Model: [deepseek-ai/DeepSeek-R1](https://huggingface.co/deepseek-ai/DeepSeek-R1) (671B total parameters, 37B active parameters)
- Data type precision for model weights and activations: BF16
- Input dataset: [ShareGPT_V3_unfiltered_cleaned_split.json](https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered/blob/main/ShareGPT_V3_unfiltered_cleaned_split.json) - Conversational AI benchmark dataset

- Performance metric:
  - Throughput (tokens/second) for offline inference
- Number of nodes: 2 GPU nodes (16 H100 GPUs in total)
- Benchmark prompts: 1000
- Load format: Dummy
- Random seed: 2025
- Time limit: 420 seconds, including model loading, warm-up, and benchmarking

## 2. Task Rules

- The results will be executed on `2 GPU servers` with `8 GPUs per node` (16 H100 GPUs in total).
- Results with execution time beyond `420 seconds` are considered INVALID.
- Grading will be based on the `optimization method`, `performance improvement`, and `technical understanding`.
- Participants may use any `tagged version` of [SGLang](https://github.com/sgl-project/sglang) or the `master code` from the main branch.
- Source code modifications are allowed but participants must `design and provide` justification and validation to demonstrate output quality is preserved.

## 3. General Rules

### Cluster Usage

- Computing platforms: The AI task must be optimized on NSCC Singapore ASPIRE-2A+ supercomputer. Teams may present optimization work done on other GPU clusters as extra information.

### Evaluation Criteria

- Technical assessment: The judging panel will assess each team's methods for optimizing SGLang DeepSeek inference performance. Teams must clearly explain and demonstrate the performance improvements achieved through their optimization techniques, showing a solid understanding of the optimization methods.
- Innovation assessment: Teams are encouraged to adopt innovative optimization approaches, including novel compute and communication techniques, advanced parallelization strategies, and creative system-level optimizations for distributed Large Language Model inference workloads.

### Performance Assessment

- Cross-platform comparison: The absolute performance metrics across different GPU clusters will not be directly compared. These metrics will indicate the effectiveness of the optimization techniques and will be a significant factor in the judges' scoring. However, they will not be the only criteria for ranking teams.
- Comprehensive scoring: When performance metrics are close, the judges will consider how well teams understand and explain their optimization process, the sophistication of their approaches, and the depth of their AI application tuning  knowledge.

## 4. Optimization Guidelines

### Allowed Optimizations (including but not limited to)

- Parallel strategies: Tensor parallelism, pipeline parallelism, data parallelism configurations
- Communication optimization: NCCL tuning, inter-node communication reduction
- Framework optimization: Custom inference engines, runtime optimizations
- Memory management: KV-cache optimization, memory pooling, batch processing
- Inference optimization: Kernel optimization, operator optimization, custom CUDA kernels
- Software Environment: CUDA version, Python version, and other software dependencies

### Prohibited Modifications

- Model architecture changes: Altering the fundamental model structure or parameters
- Reducing computation: Quantization, LoRA, pruning, and knowledge distillation
- Output quality degradation: Modifications that significantly impact generation quality
- Input modification: Changing the benchmark dataset content or evaluation criteria

### Additional Restrictions

- Experimental features: Experimental features are not allowed. As of August 1st, torch.compile and torchao fall into this category
- Competition resource limits: Double sparsity attention, multimodal capabilities, and custom logit processors are not allowed due to competition task design or resource limitations

## 5. Submission Guidelines

### Required Deliverables

- Prepare presentation slides for your team's technical interview based on your AI inference optimization work.
- Include a comparison with the baseline performance in your presentation slides.
- Include scaling analysis showing performance across different GPU configurations.
- Submit your `build scripts`, `run scripts`, and `output logs` that support your presentation (e.g., sglang.out, *.o{jobid}, etc.)
- (Optional) Provide performance profiling data demonstrating bottleneck identification and optimization strategies.
- If you modified the source code, design and submit output quality verification reports demonstrating that your changes maintain generation quality.

### Submission Example

The following is a suggested submission structure. Teams may adapt the format based on their specific optimization approach:

```
team_name_deepseek/
├── build/
│   ├── install.sh            # Installation scripts
│   ├── requirements.txt      # Python dependencies
│   ├── setup.log             # Installation log
│   └── optimization_config.txt # Detailed optimization settings used(if have)
├── run/
│   └── submit.sh            # Job submission script
├── results/                  
│   └── logs/                # Runtime output and error logs
├── analysis(Optional)/
│   ├── quality_verification.txt # Output quality report (if code modified)
│   ├── profiling_data/      # Performance profiling results (Optional)
│   └── optimization_report.pdf # Detailed optimization analysis (Optional)
└── presentation/
    └── team_presentation.ppt # Final presentation slides
```

## 6. Presentation Guidelines

- Duration: 15-20 minutes presentation + 10 minutes Q&A
- Technical depth: Demonstrate understanding of both AI inference bottlenecks and optimization strategies
- Performance analysis: Show before/after comparisons with detailed metrics
- Innovation highlight: Emphasize optimization approaches and their effectiveness

## 7. Recommendations

### Optimization Methodology

- Start with profiling: Use profiling tools to identify bottlenecks
- Consider parallelization: Optimize tensor parallelism and pipeline parallelism configurations
- Systematic approach: Document optimization attempts and their impact on performance

### Presentation Excellence

- Present a logical workflow: Structure your presentation as an AI optimization problem-solving journey
- Show understanding: Demonstrate deep comprehension of LLM inference and system optimization
- Quantify impact: Use clear metrics and visualizations to show throughput improvements

## 8. Technical Support

- Comprehensive reference guides will be provided at https://github.com/hpcac/2025-APAC-HPC-AI
- Slack Channel established for participating teams to communicate at https://join.slack.com/t/2025apachpcai-lcl5303/shared_invite/zt-34376r42p-itO0z_DwW~PJHpTbdjldxg
