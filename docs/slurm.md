# SLURM How-To

## Upstream documentation links

Slurm installed version: `22.05.9`   
   
[Slurm Documentation](https://slurm.schedmd.com/archive/slurm-22.05.9/)   
   
[Slurm Documentation - sbatch](https://slurm.schedmd.com/archive/slurm-22.05.9/sbatch.html)   

## Sugested usage

All/any sbatch scripts will have the following structure:

### Head of the script
will contain SLURM directives; these are identified by `#SBATCH` label at the beggining of line

```
#!/bin/bash
#SBATCH --job-name=MEANINGFUL_NAME       # Common name for the running jobs; can be customized with https://slurm.schedmd.com/sbatch.html#SECTION_%3CB%3Efilename-pattern%3C/B%3E
#SBATCH --mail-type=FAIL                   # Type of email notification- BEGIN,END,FAIL,ALL
#SBATCH --mail-user=EMAIL_ADDRES           # N.B. for a 1000 jobs started and UNTESTED, that fails, one will receive 1000 mails!!!!

# other tags are possible, see https://slurm.schedmd.com/sbatch.html#SECTION_FILENAME-PATTERN
#SBATCH --output=%x_%j.out                 # STDOUT file with format: [JOB_NAME]_[JOB_ID].out
#SBATCH --error=%x_%j.err                  # STDERR file with format: [JOB_NAME]_[JOB_ID].err

#SBATCH --ntasks=1                         # We are using only 1 task per job
#SBATCH --cpus-per-task=1                  # !!! if needed change, to 2 !!! ; ensuing job steps will require ncpus number of processors per task

## see https://slurm.schedmd.com/sbatch.html#OPT_hint
#SBATCH --hint=nomultithread               # [don't] use extra threads with in-core multi-threading;

## Possibility to specify nodes
## #SBATCH --nodelist=issaf-0-2.issaf,issaf-0-3.issaf,issaf-0-4.issaf,issaf-0-5.issaf    # select a given list of nodes for allocation
## #SBATCH --exclude=issaf-0-0.issaf,issaf-0-1.issaf                                     # exclude these nodes from allocation
```

#### Optional mechanics to be added before starting work

```
# Define and create a unique scratch directory for this job
# in the tail of the script we will copy back all content to SLURM_SUBMIT_DIR
export SCRATCH_DIRECTORY=/scratch/workdir_${USER}/${SLURM_JOBID}
mkdir -p ${SCRATCH_DIRECTORY}
cd ${SCRATCH_DIRECTORY}

# You can copy everything you need to the scratch directory
# ${SLURM_SUBMIT_DIR} points to the path where this script was submitted from
rsync -azWHAXS4 ${SLURM_SUBMIT_DIR}/ ${SCRATCH_DIRECTORY}/

# This is where the actual work is done.

# Save job info before starting the actual executable - useful for debbuging
scontrol show jobid -ddd  ${SLURM_JOB_ID} | sed 's/^[ \t]*//g' > job_info_${SLURM_JOB_ID}.txt
echo -e "\n##########\n" >> job_info_${SLURM_JOB_ID}.txt
env                      >> job_info_${SLURM_JOB_ID}.txt
echo -e "\nStarting job @ $(date +%Y%m%d_%H%M%S)"
```

### Body of the script

This part will contain everything related to the work to be done.   
Very usefull is to have an stand-alone analysis scritp that can be checked and tested   
on it's own. (this should contain everything related to job to be done)

### Tail of the script

#### Optional part if a work-directory was created
```
# After the job is done we copy our output back to $SLURM_SUBMIT_DIR
rsync -azWHAXS4 ${SCRATCH_DIRECTORY}/* ${SLURM_SUBMIT_DIR}/

# After everything is saved to the home directory, delete the work directory to save space on /scratch/workdir
cd ${SLURM_SUBMIT_DIR} && rm -rf ${SCRATCH_DIRECTORY}
```

Add this at the end of the script; it can be used to infer the succesful finish of the job.
```
echo "end of ${SLURM_JOB_NAME}"
```

