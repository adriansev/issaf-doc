# ISS-AF howto and usage documentation

ISS Analisys Facility is a (small) cluster dedicated to local research activities (no GRID)


## Cluster information

### Compute resources

Particular nodes specifications:
```
HOSTNAMES    PARTITION    S:C:T CPUS   MEMORY WEIGHT
issaf-0-0    CLUSTER     2:32:2  128   515000    100
issaf-0-1    CLUSTER     2:32:2  128   515000    100
issaf-0-2    CLUSTER     1:64:2  128   515000     50
issaf-0-3    CLUSTER     1:64:2  128   515000     50
issaf-0-4    CLUSTER     1:64:2  128   515000     50
issaf-0-5    CLUSTER     1:64:2  128   515000     50
issaf-0-6    CLUSTER    2:128:2  512  2097152     10
issaf-0-7    CLUSTER    2:144:2  576  2310755     10
```

Total resources:
```
     Partition     #Nodes     #CPU_cores     Job_Nodes MaxJobTime Cores Mem/Node
     Name State     Total          Total     Min   Max  Day-hr:mn /node     (GB)
CLUSTER:*    up         8           1856       1          2-00:00   128     515+
      MPI    up         8           1856       1 infin    1-12:00   128     515+
      GPU    up         1            512       1 infin    1-12:00   512    2097

```

The `issaf-0-6` node have two H200 NVidia GPUs and is has associated the `GPU` queue:
```
HOSTNAMES    PARTITION    S:C:T CPUS   MEMORY WEIGHT
issaf-0-6    GPU        2:128:2  512  2097152     10
```

```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 595.71.05              Driver Version: 595.71.05      CUDA Version: 13.2     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA H200 NVL                Off |   00000000:D1:00.0 Off |                    0 |
| N/A   31C    P0             73W /  600W |       0MiB / 143771MiB |      0%      Default |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA H200 NVL                Off |   00000000:F1:00.0 Off |                    0 |
| N/A   32C    P0             74W /  600W |       0MiB / 143771MiB |      0%      Default |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+
```

```
        GPU0    GPU1    CPU Affinity    NUMA Affinity   GPU NUMA ID
GPU0     X      NV18    224-231,480-487 28              N/A
GPU1    NV18     X      128-135,384-391 16              N/A
```



### Storage resources

```
Filesystem               Size  Mounted on
/dev/md126p1             1.8T  /export      <-- /home, RAID1 NVME
/dev/md123               1.8T  /software    <-- RAID1 HDD
/dev/md120p1              66T  /data        <-- RAID6 8xHDD
/dev/md121p1             200T  /data2       <-- RAID6 12xHDD
/dev/md119p1             200T  /data3       <-- RAID6 12xHDD
/dev/md124p1             200T  /data4       <-- RAID6 12xHDD
10.99.99.200:/storage01  200T  /data5       <-- RAID6 12xHDD
10.99.99.200:/storage02  200T  /data6       <-- RAID6 12xHDD
10.99.99.200:/storage03  200T  /data7       <-- RAID6 12xHDD
```

## Resource usage



### Compute resources

!!! warning

    No compute loads, or long running processing should be run on the `front-end` node.   
    The `front-end` is both `login` and `service` node and this can impact overall usage.   


Slurm scheduling is for the moment of type `"first come, first served"`.   
To be converted to `fair share`.


### Storage resources

* `/home` should be used for small personal files, small analysis codes
* `/software` is dedicated for software frameworks (locally compiled)
* each user have an symbolic link `$HOME/data` pointing to `/data/data_${USER}` pointing to a much larger storage area
* for requirements over 1TB, a request should be sent to administrator for creation of a personal directory in `/data{2..7}`

    !!! note "Institute management approval required"

        For data usage for more than 1 TB a request to institute management is required.
        First step is to send an email to institute director with cc to `issaf AT spacescience DOT ro`
        then more information will follow in subsequent e-mails.

