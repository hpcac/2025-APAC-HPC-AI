[toc]

# NWChem HPC Task

## 1. Task Workload and Input

Participants are required to optimize the following workload for improved performance.

- Application: [NWChem](https://nwchemgit.github.io/)
- Input dataset: `w12_b3lyp_cc-pvtz_energy.nw`

- Performance metric: 
  - Wall-clock time to convergence: Overall execution time for complete calculations
  - Strong scaling efficiency: Performance scaling across different node counts
- Node limit: Maximum 4 CPU nodes
- Time limit: 300 seconds
- Recommendation: Use 2 nodes for optimization development and testing

## 2. Task Rules

- The results will be executed on `up to 4 standard CPU servers`
  - For NCI Gadi users: Use **normalsr** queue (Intel Xeon Platinum 8470Q Sapphire Rapids)
  - For NSCC Singapore users: Use standard compute nodes (AMD EPYC 7713)
- Results with execution time beyond `300 seconds` are considered INVALID
- Grading will be based on the `optimization method`, `performance improvement`, and `technical understanding`
- Participants may use any `tagged version` of [NWChem official releases](https://github.com/nwchemgit/nwchem/releases) or the `master code` from the main branch
- Source code modifications are allowed but participants must demonstrate accuracy is preserved

## 3. General Rules

### Cluster Usage

- Computing platforms: The HPC task must be optimized on either the NCI Gadi or NSCC Singapore ASPIRE-2A supercomputer. Teams can choose which supercomputer to use for optimization. Additionally, teams may present optimization work done on other supercomputers as extra information.
- System maintenance: Teams do not need to worry about system maintenance. If any of the NCI, NSCC supercomputers is down for maintenance, teams can switch to another operational system to continue their optimization work, ensuring the competition runs smoothly.

### Evaluation Criteria

- Technical assessment: The judging panel will assess each team's methods for optimizing NWChem performance. Teams must clearly explain and demonstrate the performance improvements achieved through their optimization techniques, showing a solid understanding of the optimization methods.
- Innovation assessment: Teams are encouraged to adopt innovative optimization approaches, including novel compile and runtime techniques, advanced parallel strategies, and creative system-level optimizations.

### Performance Assessment

- Cross-platform comparison: The absolute performance metrics across different supercomputers will not be directly compared. These metrics will indicate the effectiveness of the optimization techniques and will be a significant factor in the judges' scoring. However, they will not be the only criteria for ranking teams.
- Comprehensive scoring: When performance metrics are close, the judges will consider how well teams understand and explain their optimization process, the sophistication of their approaches, and the depth of their HPC application tuning knowledge.

## 4. Optimization Guidelines

### Allowed Optimizations (including but not limited to)

- Compiler optimizations: Use of different compilers (Intel, GCC, AOCC) and optimization flags
- Mathematical libraries: Integration of optimized BLAS/LAPACK libraries (Intel MKL, OpenBLAS, AOCL)
- Parallel strategies: MPI configuration, hybrid parallelization, process placement and affinity, and thread management
- Communication optimization: Reducing communication overhead and latency
- Memory management: Optimization of memory allocation and data locality
- System-level tuning: CPU frequency scaling, NUMA topology optimization, file system and storage access
- Refer to: [Getting Started with NWChem for ISC22 SCC](https://hpcadvisorycouncil.atlassian.net/wiki/spaces/HPCWORKS/pages/2799534081/Getting+Started+with+NWChem+for+ISC22+SCC)

### Prohibited Modifications

- Algorithm alteration: Changing fundamental computational algorithms
- Precision reduction: Lowering calculation accuracy or data precision
- Input modification: Altering molecular geometries, basis sets, or convergence criteria

## 5. Submission Guidelines

### Required Deliverables

- Prepare presentation slides for your team's technical interview based on your application tuning work
- Include a comparison with the baseline performance in your presentation slides
- Include scaling analysis showing performance across different node configurations
- Submit your `build scripts`, `run scripts`, and `output text` files that support your presentation (e.g., nwchem.out, *.o{jobid}, etc.)
- (Optional) Provide performance profiling data demonstrating bottleneck identification and optimization strategies

### Submission Example

The following is a suggested submission structure. Teams may adapt the format based on their specific optimization approach:

```
team_name_nwchem/
├── build/
│   ├── build.sh              # Build script with optimization flags
│   ├── configure.log         # Configuration output
│   ├── make.log              # Compilation log
│   └── optimization_flags.txt # Detailed compiler options used
├── run/
│   └── submit.sh             # Job submission script
├── results/                  
│   └── logs/                 # Runtime output and error logs
├── analysis(Optional)/
│   ├── profiling_data/       # Performance profiling results(Optional)
│   └── optimization_report.pdf # Detailed optimization analysis(Optional)
└── presentation/
    └── team_presentation.ppt # Final presentation slides
```

## 6. Presentation Guidelines

- Duration: 15-20 minutes presentation + 10 minutes Q&A
- Technical depth: Demonstrate understanding of both bottleneck identification and optimization strategies
- Performance analysis: Show before/after comparisons with detailed metrics
- Innovation highlight: Emphasize optimization approaches and their effectiveness

## 7. Recommendations

### Optimization Methodology

- Start simple: Begin with compiler flags and library choices before complex modifications
- Profile first: Use performance analysis tools to identify bottlenecks before optimizing
- Focus on hotspots: Concentrate optimization efforts on the most time-consuming sections
- Systematic approach: Document optimization attempts and their results
- Consider memory access hierarchy: Optimize for cache, memory and networking performance

### Presentation Excellence

- Present a logical workflow: Structure your presentation as a HPC optimization problem-solving journey
- Show understanding: Demonstrate deep comprehension of HPC and system optimization
- Quantify impact: Use clear metrics and visualizations to show improvement

## 8. Technical Support

- Comprehensive reference guides will be provided at https://github.com/hpcac/2025-APAC-HPC-AI
- Slack Channel established for participating teams to communicate at https://join.slack.com/t/2025apachpcai-lcl5303/shared_invite/zt-34376r42p-itO0z_DwW~PJHpTbdjldxg

## 9. Input file

Run the application with the given input file and submit the results

The input file that follows (generated with `./make_nwchem_input.py w12 b3lyp cc-pvtz energy`) is the one that must be used. The only lines that you can change are noted as such.

```
echo

start w12_b3lyp_cc-pvtz_energy

memory stack 8000 mb heap 100 mb global 8000 mb noverify	# you can change this

permanent_dir .		# you can change this
scratch_dir /tmp	# you can change this

geometry units angstrom
  O       1.79799517    -2.87189360    -0.91374020
  O       0.96730604    -2.75911220     1.62798799
  O       1.65380168    -0.07006642    -1.01974524
  O       1.02235809     0.07175530     1.65572815
  O      -1.02223258     0.06714841    -1.65231025
  O      -1.65303657    -0.06953734     1.02328922
  O      -1.79714075    -2.87225082     0.91489225
  O      -0.96714604    -2.76268933    -1.62695366
  O      -0.91046484     2.86984708    -1.80205865
  O       1.62976535     2.75963881    -0.96492508
  O       0.90962365     2.87584732     1.79706100
  O      -1.63064745     2.76057720     0.96075175
  H       1.58187452    -2.90533857     0.05372043
  H       2.45880812    -3.55319457    -1.06640225
  H      -1.58013630    -2.90956053    -0.05228132
  H      -2.45728026    -3.55363054     1.07010099
  H       0.05632428     2.90394112    -1.58314254
  H      -1.06231290     3.55393270    -2.46019459
  H      -1.56680088     2.89107809    -0.00131897
  H      -1.92210192     1.83983329     1.06475175
  H       1.34757079     0.06744042     0.72858318
  H       1.14627218     0.98941003     1.95482520
  H      -1.14676057     0.98295160    -1.95675717
  H      -1.34650335     0.06810117    -0.72482129
  H       0.72810518    -0.06186014    -1.34872142
  H       1.95057642    -0.98833069    -1.14558354
  H      -0.72703573    -0.06575415     1.35156867
  H      -1.95270073    -0.98745311     1.14464678
  H       1.56681459     2.89291882    -0.00316599
  H       1.92433921     1.83958829    -1.06611724
  H      -0.00511757    -2.89477284    -1.56679971
  H      -1.07087869    -1.84139535    -1.91693517
  H      -0.05748186     2.90789148     1.57927484
  H       1.06103881     3.56065162     2.45452965
  H       0.00532585    -2.89179177     1.56768173
  H       1.07022772    -1.83898762     1.92167783
end

basis "ao basis" spherical noprint
  * library cc-pvtz
end

# you can remove this entire block if you want
scf
  direct   # you can change this
  singlet
  rhf
  thresh 1e-7
  maxiter 100
  vectors input atomic output w12_scf_cc-pvtz.movecs
  noprint "final vectors analysis" "final vector symmetries"
end

task scf energy ignore
# /end thing you can remove

dft
  direct   # you can change this
  xc b3lyp
  grid fine
  iterations 100
  vectors input w12_scf_cc-pvtz.movecs   # remove this if you remove the block above
  noprint "final vectors analysis" "final vector symmetries"
end

task dft energy
```

