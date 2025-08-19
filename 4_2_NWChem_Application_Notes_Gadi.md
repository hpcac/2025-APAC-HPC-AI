[toc]

# Get Familiar with NCI Gadi PBS allocation

Run the following 1-second test job with different qsub resource definition parameters to understand the PBS allocation behavior and check the output files.

The test script performs the following operations:

1. Sets up PBS job parameters including project allocation, walltime, and notification settings
2. Purges existing modules and loads required OpenMPI and libfabric modules
3. Displays environment variables and CPU information
4. Executes a simple MPI hostname command with process binding information

```bash
mkdir -p ${HOME}/scratch/${USER}/qsub

tee ${HOME}/scratch/${USER}/qsub/hostname.sh << 'EOF'
#!/bin/bash
#PBS -P pc08
#PBS -l walltime=1
#PBS -j oe
#PBS -M 393958790@qq.com
#PBS -m abe
##PBS -l other=hyperthread

module purge
module load nwchem/7.0.0
module list

env
lscpu
free -g

which mpirun
mpirun --report-bindings \
-oversubscribe -use-hwthread-cpus \
hostname
EOF

cd ${HOME}/scratch/${USER}/qsub
qsub -l ncpus=1 hostname.sh
qsub -l ncpus=$((48*2)),mem=$((48*1*2))gb hostname.sh
```

The above command requests 2 vnodes (virtual nodes) with 127 CPU cores each, allowing you to observe the job allocation and execution behavior on the ASPIRE-2A system.

In PBS terminology, a vnode represents a collection of resources that can be allocated as a unit.

---

# Create the Input File

Create the required NWChem input file for the benchmark calculation. The input file specification is provided by the competition rules and must be used exactly as given, with only specific lines allowed to be modified for optimization.

Use the following command to create the input file `w12_b3lyp_cc-pvtz_energy.nw`:

```bash
mkdir -p ${HOME}/scratch/${USER}/nwchem/input

tee ${HOME}/scratch/${USER}/nwchem/input/w12_b3lyp_cc-pvtz_energy.nw << 'EOF'
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
  H       1.92433921     1.83983329    -1.06611724
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

scf
  direct
  singlet
  rhf
  thresh 1e-7
  maxiter 100
  vectors input atomic output w12_scf_cc-pvtz.movecs
  noprint "final vectors analysis" "final vector symmetries"
end

task scf energy ignore

dft
  direct
  xc b3lyp
  grid fine
  iterations 100
  vectors input w12_scf_cc-pvtz.movecs
  noprint "final vectors analysis" "final vector symmetries"
end

task dft energy
EOF
```

## Modifiable Parameters

According to the competition rules, only the following lines can be modified for performance optimization:

1. **Memory allocation**: `memory stack 8000 mb heap 100 mb global 8000 mb noverify`
2. **Permanent directory**: `permanent_dir .`
3. **Scratch directory**: `scratch_dir /tmp`

## Optimization Considerations

- **Memory settings**: Adjust based on available node memory and job requirements
- **Scratch directory**: Consider using faster local storage
- **Permanent directory**: Set to current working directory or a location with sufficient space

---

# Run the Task workload

## Create PBS bash script

Create a shell script file, `${HOME}/run/nwchem.sh`:

```bash
mkdir -p ${HOME}/scratch/${USER}/run

tee ${HOME}/scratch/${USER}/run/nwchem.sh << 'EOF'
#!/bin/bash
#PBS -P pc08
#PBS -l walltime=00:08:00
#PBS -l ncpus=48,mem=96gb
#PBS -j oe
#PBS -M 393958790@qq.com
#PBS -m abe
##PBS -l other=hyperthread

# Load required modules
module purge
module load nwchem/7.0.0

env
env | grep -E "(PBS|OMP)" | sort
module list

export OMP_NUM_THREADS=1

# Change to working directory
mkdir -p ${HOME}/scratch/${USER}/nwchem/run/${PBS_JOBID}
cd       ${HOME}/scratch/${USER}/nwchem/run/${PBS_JOBID}

OUTPUT_FILE=${HOME}/run/job.${PBS_JOBNAME}.stdout

time mpirun -np ${NCPUS:-48} \
    nwchem \
    ${HOME}/scratch/${USER}/nwchem/input/w12_b3lyp_cc-pvtz_energy.nw \
    2>&1 | tee ${OUTPUT_FILE}
EOF
```

## Submit the job script to PBS

The following command submits the PBS job script to the queue:

```bash
mkdir -p ${HOME}/run
cd       ${HOME}/run

qsub -N nwchem.w12.nodes1.ncpus48 ${HOME}/scratch/${USER}/run/nwchem.sh
```

## Alternative submission with custom parameters

For more flexibility, you can customize the job parameters:

```bash
mkdir -p ${HOME}/run
cd       ${HOME}/run

# Define job parameters
nodes=2 \
ncpus=48 \
walltime=00:08:00 \
bash -c \
	'qsub -V \
    -l walltime=${walltime},ncpus=$((ncpus*1*nodes)),mem=$((ncpus*2*nodes))gb \
    -N nwchem.w12.nodes${nodes}.ncpus${ncpus} \
    ${HOME}/scratch/${USER}/run/nwchem.sh'
```

---

# Monitoring and Output

## Monitor job status

Check the job status and output:

```bash
cd       ${HOME}/run

# Check job queue status
qstat -u $USER

# Check completed job output
ls -la ${HOME}/run/nwchem.w12.*.o*

# Monitor job output (replace JOBID with actual job ID)
tail -f job.nwchem.w12.*.stdout

# Historical job data
qstat -x | grep nwchem
```

## Expected output files

After successful completion, you should find:

- **Job output file**: `nwchem.w12.nodes[X].ncpus[Y].o[JOBID]` - Stdout and stderr that contains calculation results and timing information
- **Vector files**: `w12_scf_cc-pvtz.movecs` - Molecular orbital coefficients at `${HOME}/scratch/${USER}/nwchem/run`

The benchmark timing, "Total times", will be reported in the job output file for performance evaluation.

## Read the results

The performance results of NWChem are measured in “Total times”. The lower the value, the better.

```bash
[pz7344@gadi-login-02 run]$ grep "Total times" *
job.nwchem.w12.nodes1.ncpus48.stdout: Total times  cpu:      350.9s     wall:      353.5s
nwchem.w12.nodes1.ncpus48.o147376500: Total times  cpu:      350.9s     wall:      353.5s
```

---

# Build NWChem from Source Code

## Prepare NWChem files

The following commands

1. Created a workspace directory for NWChem under `scratch` directory
2. Download the latest NWChem v7.2.3 release source and non-source tarballs from GitHub
3. Extract the non-source components to the `noncode` directory
4. Create the build directory structure for HPC-X with Intel OneAPI configuration
5. Set up symbolic links and extract source code to the designated code directory

```bash
# Create directory structure for NWChem installation
mkdir -p ${HOME}/scratch/${USER}/nwchem/noncode
cd ${HOME}/scratch/${USER}/nwchem

# Download NWChem 7.2.3 release tarballs
wget -c https://github.com/nwchemgit/nwchem/releases/download/v7.2.3-release/nwchem-7.2.3-release.revision-d690e065-nonsrconly.2024-08-27.tar.bz2
wget -c https://github.com/nwchemgit/nwchem/releases/download/v7.2.3-release/nwchem-7.2.3-release.revision-d690e065-srconly.2024-08-27.tar.bz2

# Extract non-source components
tar xf nwchem-7.2.3-release.revision-d690e065-nonsrconly.2024-08-27.tar.bz2 -C noncode

# Setup source directory structure and extract source code
mkdir -p ${HOME}/scratch/${USER}/nwchem/softwarestack/code
cd ${HOME}/scratch/${USER}/nwchem/softwarestack
ln -s ../nwchem-7.2.3-release.revision-d690e065-src* ./
time tar xf nwchem-7.2.3-release.revision-d690e065-srconly.2024-08-27.tar.bz2 -C code
# real	0m5.874s
```





