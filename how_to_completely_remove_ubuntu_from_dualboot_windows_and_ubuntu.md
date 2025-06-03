

```md
# DiskPart Command Session

## Command: `diskpart`

```cmd
C:\Windows\System32>diskpart
```

Microsoft DiskPart version 10.0.26100.1150  
Copyright (C) Microsoft Corporation.  
On computer: TATA  

## List Disks

```cmd
DISKPART> list disk
```

| Disk ### | Status  | Size  | Free  | Dyn | Gpt |
|----------|---------|-------|-------|-----|-----|
| Disk 0   | Online  | 931 GB  | 2048 KB  |     | * |
| Disk 1   | Online  | 465 GB  | 106 GB  |     | * |

## Select Disk 1

```cmd
DISKPART> select disk 1
```

**Output:**  
Disk 1 is now the selected disk.  

## List Partitions

```cmd
DISKPART> list partition
```

| Partition ### | Type      | Size    | Offset   |
|--------------|----------|--------|---------|
| Partition 1  | System   | 100 MB  | 1024 KB |
| Partition 2  | Reserved | 16 MB   | 101 MB  |
| Partition 3  | Primary  | 357 GB  | 117 MB  |
| Partition 4  | Recovery | 921 MB  | 357 GB  |
| Partition 5  | System   | 476 MB  | 358 GB  |
| Partition 8  | Recovery | 739 MB  | 465 GB  |

## Select Partition 5

```cmd
DISKPART> select partition 5
```

**Output:**  
Partition 5 is now the selected partition.  

## Assign Drive Letter

```cmd
DISKPART> assign letter=z
```

**Output:**  
DiskPart successfully assigned the drive letter or mount point.  

## Delete Partition Override

```cmd
DISKPART> delete partition override
```

**Output:**  
DiskPart successfully deleted the selected partition.  

**End of Session**
```

