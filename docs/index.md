# ISS-AF howto and usage documentation

ISS Analisys Facility is a (small) cluster dedicated to local research activities (no GRID)


## Cluster information

### Compute resources

Particular nodes specifications:
```
HOSTNAMES    PARTITION   S:C:T CPUS  MEMORY
issaf-0-0    CLUSTER    2:32:2  128  515000
issaf-0-1    CLUSTER    2:32:2  128  515000
issaf-0-2    CLUSTER    1:64:2  128  515000
issaf-0-3    CLUSTER    1:64:2  128  515000
issaf-0-4    CLUSTER    1:64:2  128  515000
issaf-0-5    CLUSTER    1:64:2  128  515000
```

Total resources:
```
Partition          #Nodes       #CPU_cores   Cores_pending   Job_Nodes  MaxJobTime Cores   Mem/Node
Name         State Total  Idle  Total   Idle Resorc  Other   Min   Max  Day-hr:mn  /node   (GB)
CLUSTER:*    up     6     6     768     768      0      0     1         20-00:00   128     515
```

### Storage resources

```
Filesystem               Size  Mounted on
/dev/md126p1             1.8T  /export     <-- /home, RAID1 NVME
/dev/md123               1.8T  /software   <-- RAID1 HDD
/dev/md120p1              66T  /data       <-- RAID6 8xHDD
/dev/md121p1             200T  /data2      <-- RAID6 12xHDD
/dev/md119p1             200T  /data3      <-- RAID6 12xHDD
/dev/md124p1             200T  /data4      <-- RAID6 12xHDD
10.99.99.200:/storage01  200T  /data5      <-- RAID6 12xHDD
10.99.99.200:/storage02  200T  /data6      <-- RAID6 12xHDD
10.99.99.200:/storage03  200T  /data7      <-- RAID6 12xHDD
```

## Resource usage



### Compute resources
For the moment, it's `"first come, first served"`.   
To be converted to `fair share`.


### Storage resources

* `/home` should be used for small personal files, small analysis codes
* each user have an symbolic link `$HOME/data` pointing to `/data/data_${USER}` pointing to a much larger storage area
* for requirements over 1TB, a request should be sent to administrator for creation of a personal directory in `/data{2..7}`
* `/software` is dedicated for software frameworks (locally compiled)
