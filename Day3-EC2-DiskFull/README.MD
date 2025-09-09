# 🚀 Scenario 3: EC2 Disk Full

## 📝 Problem Statement
EC2 instance root volume filled due to uncontrolled log growth → “No space left on device” errors.  
Impact: applications crash, updates fail, potential downtime.

---

## ⚙️ Environment Setup
- **Region:** ap-south-1 (Mumbai)  
- **Instance Type:** t2.micro  
- **Key Pair:** <key_pair.pem>  
- **Folder:** Day3-EC2-DiskFull  

---

## 🔄 Steps to Reproduce

### 1️⃣ Baseline Check

df -h

lsblk

Explanation: df -h → human-readable disk usage; lsblk → shows partitions 
Expected Output**: Current disk usage & partitions

Screenshots**: disk_baseline_dfh.png, disk_baseline_lsblk.png

---

### 2️⃣ Simulate Disk Full

sudo dd if=/dev/zero of=/var/log/fillfile bs=1M count=5000
sudo dd if=/dev/zero of=/var/log/fillfile2 bs=1M count=3000

Explanation: Fills disk with dummy files to trigger “No space left”
Expected Error: dd: error writing '/var/log/fillfile2': No space left on device

Screenshot: disk_after_fill_dfh.png

---

### 3️⃣ Troubleshoot Disk Space

**Step 3A — Identify Large Files** 

sudo du -sh /var/log/*
sudo du -sh /var/*

Screenshot: disk_top_consumers.png, disk_varlog_sizes.png

**Step 3B — Cleanup Logs**

sudo truncate -s 0 /var/log/messages
sudo truncate -s 0 /var/log/secure

Screenshot: disk_after_cleanup_dfh.png

---

### 4️⃣ Expand EBS Volume (Optional)

Before: ebs_volume_before.png → 8 GB root volume

Modify via console: ebs_modify_volume_console.png → 12 GB

After: ebs_volume_after.png → confirmed 12 GB

Verify disk:

df -h

Screenshot: disk_after_expand_dfh.png

---

⚠️ Special Note:

The **root volume of this EC2 instance** uses the **XFS filesystem** (not ext4).  
- To expand the filesystem: `sudo xfs_growfs -d /`  
- Do **not** use `resize2fs`, as it works only for ext4.

❌ Error Observed

- No space left on device

🔍 Investigation

- df -h → confirmed 100% usage
- Checked /var/log → largest files

🛑 Root Cause

- Log files + filler files exhausted root volume

✅ Solution

- Truncated & cleaned up logs
- Expanded EBS volume (8 → 12 GB)
- Verified disk space with df -h

📊 Impact

- Restored instance functionality
- Practiced disk troubleshooting workflow
- Gained confidence in log management & EBS scaling