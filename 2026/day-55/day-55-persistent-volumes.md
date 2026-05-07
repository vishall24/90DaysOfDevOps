# Day 55 – Persistent Volumes (PV) and Persistent Volume Claims (PVC)

## Task
Containers are ephemeral — when a Pod dies, everything inside it disappears. That is a serious problem for databases and anything that needs to survive a restart. Today you fix this with Persistent Volumes and Persistent Volume Claims.

---

## Challenge Tasks

### Task 1: See the Problem — Data Lost on Pod Deletion
1. Write a Pod manifest that uses an `emptyDir` volume and writes a timestamped message to `/data/message.txt`
2. Apply it, verify the data exists with `kubectl exec`
3. Delete the Pod, recreate it, check the file again — the old message is gone

**Verify:** Is the timestamp the same or different after recreation?

---

Pod → PVC → PV → Actual Disk

created emptydir-pod and verified the data inside the pod:

<img width="1101" height="596" alt="image" src="https://github.com/user-attachments/assets/75eaa06a-584e-4f26-bd4d-3a76860a5960" />

emptyDir?:

Temporary storage.
Lives only while Pod exists.
Pod dies = data dies.


recrated the pod and the date changed :
Timestamp changed.

OLD DATA LOST.

<img width="1099" height="134" alt="image" src="https://github.com/user-attachments/assets/961d1784-23e1-4960-8496-e6c49bdf8dc8" />

data is not persistant

---

### Task 2: Create a PersistentVolume (Static Provisioning)
1. Write a PV manifest with `capacity: 1Gi`, `accessModes: ReadWriteOnce`, `persistentVolumeReclaimPolicy: Retain`, and `hostPath` pointing to `/tmp/k8s-pv-data`
2. Apply it and check `kubectl get pv` — status should be `Available`

Access modes to know:
- `ReadWriteOnce (RWO)` — read-write by a single node
- `ReadOnlyMany (ROX)` — read-only by many nodes
- `ReadWriteMany (RWX)` — read-write by many nodes

`hostPath` is fine for learning, not for production.

**Verify:** What is the STATUS of the PV?

---

created pv :


<img width="1366" height="490" alt="image" src="https://github.com/user-attachments/assets/8d9bb53e-d41e-44c0-82cc-94e4609b4c43" />

---

*) reclaim policy:

- Retain

*) When PVC deleted:

= KEEP data

*) Other policy

- Delete

Delete storage automatically.

*) hostPath

- Actual location on node.

---

### Task 3: Create a PersistentVolumeClaim
1. Write a PVC manifest requesting `500Mi` of storage with `ReadWriteOnce` access
2. Apply it and check both `kubectl get pvc` and `kubectl get pv`
3. Both should show `Bound` — Kubernetes matched them by capacity and access mode

**Verify:** What does the VOLUME column in `kubectl get pvc` show?

---

Created pvc with size 500Mi:

<img width="824" height="329" alt="image" src="https://github.com/user-attachments/assets/7d624fc7-61f7-47af-b49e-683c1607facb" />

<img width="1379" height="70" alt="image" src="https://github.com/user-attachments/assets/8b78c3e7-7300-4d01-9775-838a14777a19" />

the status is pending 
it will bound only when. the pod is created with it

---

### Task 4: Use the PVC in a Pod — Data That Survives
1. Write a Pod manifest that mounts the PVC at `/data` using `persistentVolumeClaim.claimName`
2. Write data to `/data/message.txt`, then delete and recreate the Pod
3. Check the file — it should contain data from both Pods

**Verify:** Does the file contain data from both the first and second Pod?

---

Created pod and bounded :

<img width="1432" height="118" alt="image" src="https://github.com/user-attachments/assets/3ac38332-2929-4eed-99ab-a673d8f2b43d" />

<img width="735" height="572" alt="image" src="https://github.com/user-attachments/assets/159fcd0b-a1b2-4be1-b521-fc443d6b6707" />


Check and verified the data is still there:

<img width="1072" height="199" alt="image" src="https://github.com/user-attachments/assets/07975d60-e667-44fe-bd08-0863b69794fd" />


---

### Task 5: StorageClasses and Dynamic Provisioning
1. Run `kubectl get storageclass` and `kubectl describe storageclass`
2. Note the provisioner, reclaim policy, and volume binding mode
3. With dynamic provisioning, developers only create PVCs — the StorageClass handles PV creation automatically

**Verify:** What is the default StorageClass in your cluster?

---

check the storage class:

<img width="1428" height="346" alt="image" src="https://github.com/user-attachments/assets/23f63b34-4d33-43dc-9b6f-09d88bf2954b" />

Observe:

Field	              |  Meaning
provisioner	        |  rancher.io/local-path
reclaim policy	    |  delete
binding mode	when  |  WaitForFirstConsumer

Default storage class : standard

---

### Task 6: Dynamic Provisioning
1. Write a PVC manifest that includes `storageClassName: standard` (or your cluster's default)
2. Apply it — a PV should appear automatically in `kubectl get pv`
3. Use this PVC in a Pod, write data, verify it works

**Verify:** How many PVs exist now? Which was manual, which was dynamic?

---

<img width="1433" height="262" alt="image" src="https://github.com/user-attachments/assets/80d5f927-2138-4ae7-b44f-4e70f7931908" />

After applying the dynamic PVC, Kubernetes automatically created a new PV using the default StorageClass.

Total PVs:
- 1 manually created PV (my-pv)
- 1 dynamically provisioned PV (auto-created pvc-xxxxx)

Static provisioning requires manually creating PVs.
Dynamic provisioning automatically creates PVs when a PVC is requested.

Verified:

<img width="962" height="45" alt="image" src="https://github.com/user-attachments/assets/b58bc566-979c-430f-9c45-b05f37f94667" />

BIG FINAL LEARNING:
Static Provisioning	            |   Dynamic Provisioning
Admin creates PV manually	    	|   Kubernetes creates PV automatically
More manual work	             	|   Easier and scalable
Used less in modern clusters		|   Used in real production
PVC binds existing PV		        |   PVC triggers PV creation

---

### Task 7: Clean Up
1. Delete all pods first
2. Delete PVCs — check `kubectl get pv` to see what happened
3. The dynamic PV is gone (Delete reclaim policy). The manual PV shows `Released` (Retain policy).
4. Delete the remaining PV manually

**Verify:** Which PV was auto-deleted and which was retained? Why?

---

<img width="1253" height="164" alt="image" src="https://github.com/user-attachments/assets/d948abe4-e5cf-4c2a-8818-3aa5350d260a" />

---

<img width="765" height="385" alt="image" src="https://github.com/user-attachments/assets/4822b8c9-cb85-485c-9089-7518cec4c961" />

my-pv was not deleted because of retain policy and others got deleted because the policy was delete.



FINAL UNDERSTANDING

emptyDir
Temporary storage.

Pod dies = data dies.

PV
Actual storage resource.

PVC
Request for storage.

StorageClass
Automatic PV creator.

Static provisioning
Admin manually creates PV.

Dynamic provisioning
StorageClass auto-creates PV.

MOST IMPORTANT REAL-WORLD FLOW
   
    Developer creates PVC
    ↓
    StorageClass provisions PV
    ↓
    Pod mounts PVC
    ↓
    Data survives Pod restart

    
