<h1>Day 13 – Linux Volume Management (LVM)</h1>  

## Task
  Learn LVM to manage storage flexibly – create, extend, and mount volumes.


Challenge Tasks

## Task 1: Check Current Storage
  Run: lsblk, pvs, vgs, lvs, df -h

## Task 2: Create Physical Volume
  pvcreate /dev/sdb   # or your loop device
  pvs

## Task 3: Create Volume Group
  vgcreate devops-vg /dev/sdb
  vgs
  
## Task 4: Create Logical Volume
  lvcreate -L 500M -n app-data devops-vg
  lvs
  
## Task 5: Format and Mount
  mkfs.ext4 /dev/devops-vg/app-data
  mkdir -p /mnt/app-data
  mount /dev/devops-vg/app-data /mnt/app-data
  df -h /mnt/app-data

My answer ( Task - 1,2,3,4 & 5):

<img width="852" height="899" alt="Screenshot 2026-03-25 at 11 06 52 PM" src="https://github.com/user-attachments/assets/bf6cd57d-2ba5-47d5-b908-26268025f555" />

  Marked my commands in red and their response in green for more visibility.

## Task 6: Extend the Volume
  
  lvextend -L +200M /dev/devops-vg/app-data
  resize2fs /dev/devops-vg/app-data
  df -h /mnt/app-data

My answer:

<img width="878" height="353" alt="Screenshot 2026-03-25 at 11 21 40 PM" src="https://github.com/user-attachments/assets/4e2cab5c-aaa1-4e87-a567-c0bcbff7139d" />

Seems too messy ??

here is the flow ::

## Flow

1. Added a 4GB EBS volume to EC2 instance

2. Verified disk:
lsblk

3. Created Physical Volume:
sudo pvcreate /dev/xvdf
pvs

4. Created Volume Group:
sudo vgcreate devops-vg /dev/xvdf  --> this will create Volume group named devops-vg under /dev/
vgs

5. Created Logical Volume:
sudo lvcreate -L 2G -n app-data devops-vg  --> this will create logical volume named " app-data" under /dev/devops-vg/
lvs

6. Formatted the volume:
sudo mkfs.ext4 /dev/devops-vg/app-data 

7. Mounted the volume:
sudo mkdir -p /mnt/app-data
sudo mount /dev/devops-vg/app-data /mnt/app-data

8. Verified mount:
df -h

9. Extended Logical Volume:
sudo lvextend -L +1G /dev/devops-vg/app-data
sudo resize2fs /dev/devops-vg/app-data

10. Verified updated size:
df -h


## Explanation::


Step 0: AWS Disk

    attached a new 4GB disk by creating EBS on AWS and attaching on to the EC2 instance getting used.
    Linux sees it as /dev/nvme1n1

lsblk

    * Shows all disks
    * You confirm new disk exists


Step 1: pvcreate
  
    pvcreate /dev/nvme1n1
    
    Converts raw disk → Physical Volume (PV)

Think:

    “Make this disk usable for LVM”

pvs

    *Shows all PVs


Step 2: vgcreate
  
    vgcreate devops-vg /dev/nvme1n1
    
    Creates Volume Group (VG)

Think:

    “Create a storage pool from disk”
  
    vgs
  
    Shows available storage pools


Step 3: lvcreate
  
    lvcreate -L 2G -n app-data devops-vg
    
    Creates Logical Volume (LV)
  
    -L 2G → size   (here -L means size -L means give me exact size of 2GB)
  
    -n app-data → name
  
    devops-vg → from which pool

Think:

    “Create a partition from pool”

lvs

    Shows logical volumes


Step 4: mkfs.ext4
  
    mkfs.ext4 /dev/devops-vg/app-data
  
    Formats volume

Think:

    “Prepare it like a new drive”
  
    Without this we cannot store files


Step 5: Mount
 
    mkdir -p /mnt/app-data
    mount /dev/devops-vg/app-data /mnt/app-data
  
    Makes storage usable

Think:

    “Attach disk to folder”

Now:
  
     /mnt/app-data = usable storage


Step 6: Verify
  
    df -h
  
    Shows:
      mounted disks
      sizes
      usage


Step 7: Extend volume (main power)

    lvextend -L +1G /dev/devops-vg/app-data
    
    Increase size by 1GB
  
    BUT filesystem still old to overcome ::


Step 8: Resize filesystem
  
    resize2fs /dev/devops-vg/app-data
    
    Expands filesystem to use new space
  
    Kind of reboot to show new changes.


Final check
  
    df -h
    Now size increased ✅


## FULL FLOW ::

    AWS Disk (/dev/nvme1n1)
            ↓
    Physical Volume (pvcreate)
            ↓
    Volume Group (vgcreate)
            ↓
    Logical Volume (lvcreate)
            ↓
    Format (mkfs)
            ↓
    Mount (mount)
            ↓
    Extend (lvextend + resize2fs)

