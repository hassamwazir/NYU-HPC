# NYU-HPC

To run a jupyter notebook for PyTorch, use the following .sbatch file

### run-jupyter.sbatch
Run this file and then follow the instructions in the file to get a jupyter notebook link
```
#!/bin/bash

#SBATCH --job-name=jupyter
#SBATCH --nodes=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=2GB
#SBATCH --time=04:00:00

module purge

port=$(shuf -i 10000-65500 -n 1)

/usr/bin/ssh -N -f -R $port:localhost:$port log-1
/usr/bin/ssh -N -f -R $port:localhost:$port log-2
/usr/bin/ssh -N -f -R $port:localhost:$port log-3

cat<<EOF

Jupyter server is running on: $(hostname)
Job starts at: $(date)

Step 1 :

If you are working in NYU campus, please open an iTerm window, run command

ssh -L $port:localhost:$port $USER@greene.hpc.nyu.edu

If you are working off campus, you should already have ssh tunneling setup through HPC bastion host,
that you can directly login to greene with command

ssh $USER@greene

Please open an iTerm window, run command

ssh -L $port:localhost:$port $USER@greene

Step 2:

Keep the iTerm windows in the previouse step open. Now open browser, find the line with

The Jupyter Notebook is running at: $(hostname)

the URL is something: http://localhost:${port}/?token=XXXXXXXX (see your token below)

you should be able to connect to jupyter notebook running remotly on greene compute node with above url

EOF

unset XDG_RUNTIME_DIR
if [ "$SLURM_JOBTMP" != "" ]; then
    export XDG_RUNTIME_DIR=$SLURM_JOBTMP
fi

if [[ $(hostname -s) =~ ^g ]]; then nv="--nv"; fi

singularity exec $nv \
            --overlay /scratch/work/public/singularity/pytorch1.7.1-cuda11.0.ext3:ro \
            /scratch/work/public/singularity/cuda11.0-cudnn8-devel-ubuntu18.04.sif \
            /bin/bash -c "
source /ext3/env.sh
jupyter notebook --no-browser --port $port --notebook-dir=$(pwd)
"
```

### Install a package in singularity
First open the container. Here as an example, a "my_pytorch.ext3" container is used. It has a 'cuda11.2.2-cudnn8-devel-ubuntu20.04' installed
```
singularity exec --overlay my_pytorch.ext3 /scratch/work/public/singularity/cuda11.2.2-cudnn8-devel-ubuntu20.04.sif /bin/bash
```

Then install a package using pip

```
pip install <package-name>
```

After you're done installing all pakcages, exit from the singularity container
```
exit
```
### Submit, Check, and Cancel jobs
To submit a job, use "sbatch"
```
sbatch example.SBATCH
```
To check the status of the submitted job in the queue, use "squeue"
```
squeue -u <NetID>
```
To cancel a submitetd job, use "scancel"
```
scancel -u <NetID>
```
