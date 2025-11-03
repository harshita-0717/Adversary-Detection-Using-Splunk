This readme file focuses on the disk cleanup of the splunk.
Sometime, it happens that our splunk's disk space goes beyond the limit, so it stops working properly
For this problem the simple solution we have is:

✅ Stop Splunk
✅ Remove old logs, cache, and unused apps
✅ Lower the 10 GB disk threshold to 2 GB
✅ Restart Splunk cleanly

---

### 🧹 **Splunk Cleanup Script**

Copy and paste this whole block into your Ubuntu terminal:

```bash
#!/bin/bash
echo "=== Splunk Cleanup Script ==="

SPLUNK_HOME="/opt/splunk"

echo "Stopping Splunk..."
sudo $SPLUNK_HOME/bin/splunk stop

echo "Cleaning logs and temporary files..."
sudo rm -rf $SPLUNK_HOME/var/*
sudo rm -rf $SPLUNK_HOME/quarantined_files/*

echo "Removing large, optional apps..."
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk-visual-exporter
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_pipeline_builders
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_archiver
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_monitoring_console
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_app_for_splunk_o11y_cloud
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_secure_gateway
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_metrics_workspace
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk-dashboard-studio
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_instrumentation
sudo rm -rf $SPLUNK_HOME/etc/apps/splunk_rapid_diag
sudo rm -rf $SPLUNK_HOME/etc/apps/sample_app

echo "Lowering minimum free space requirement to 2 GB..."
sudo mkdir -p $SPLUNK_HOME/etc/system/local
sudo bash -c "cat > $SPLUNK_HOME/etc/system/local/server.conf <<EOF
[diskUsage]
minFreeSpace = 2000
EOF"

echo "Starting Splunk..."
sudo $SPLUNK_HOME/bin/splunk start

echo "=== Cleanup complete ==="
df -h $SPLUNK_HOME
sudo du -sh $SPLUNK_HOME
```

---

### 💡 How to use

1. Save it as a script:

   ```bash
   nano splunk_cleanup.sh
   ```

   (Paste the content, then press **Ctrl + O**, **Enter**, **Ctrl + X**)
2. Make it executable:

   ```bash
   chmod +x splunk_cleanup.sh
   ```
3. Run it:

   ```bash
   sudo ./splunk_cleanup.sh
   ```

---

If this does not works even though you've cleaned a lot, then follow other method:

Let’s fix that **once and for all**

---

## 🩺 Root cause

Splunk’s default config:

```ini
[diskUsage]
minFreeSpace = 10000
```

→ means: "Stop indexing if less than 10 GB free."


---

## 🛠 Fix: Lower the limit to 2 GB (safe for VMs)

1. Open the config file:

   ```bash
   sudo nano /opt/splunk/etc/system/local/server.conf
   ```

2. Add or update this section:

   ```ini
   [diskUsage]
   minFreeSpace = 2000
   ```

   *(You can go lower, e.g. 1000 = 1 GB, but 2000 MB is a good safety buffer.)*

3. Save and exit (Ctrl + O, Enter, Ctrl + X).

4. Restart Splunk:

   ```bash
   sudo /opt/splunk/bin/splunk restart
   ```

---

## ✅ After restart

Check that the new limit is applied:

```bash
sudo grep -A2 diskUsage /opt/splunk/etc/system/local/server.conf
```

And confirm free space:

```bash
df -h /opt/splunk
```

Splunk will now run normally even with ~5 GB free.

---

## 💡 Optional: remove old audit data

Since the warning specifically mentions `/opt/splunk/var/lib/splunk/audit/db`, you can safely clear that too (it just tracks internal user activity).

```bash
sudo /opt/splunk/bin/splunk stop
sudo rm -rf /opt/splunk/var/lib/splunk/audit/db/*
sudo /opt/splunk/bin/splunk start
```

---

After doing these changes:

* the red “Disk Space” alert will disappear,
* Splunk searches and indexing will resume,
* you’ll stop getting the flood of “MinFreeSpace=10000” warnings.

---

After it finishes:

* Splunk will start fresh with minimal data.
* The `/opt/splunk` directory should drop to around **2–3 GB total**.
* Your searches will run again, even with limited disk space.

---
