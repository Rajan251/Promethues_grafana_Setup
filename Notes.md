**complete list of commands** you can run on an **Ubuntu server** to detect and troubleshoot the hidden issues mentioned earlier.  

---

## **🛠️ Commands to Fix Hidden Problems (Ubuntu Server)**  

### **1. ⚡ Catch Tiny CPU/Memory/Disk Spikes**  
**Problem:** Short bursts of high CPU/Disk/Memory that normal tools miss.  

#### **Commands:**  
```bash
# Install sysstat (for 'sar' and 'iostat')
sudo apt update && sudo apt install sysstat -y

# Start sysstat (enables historical data collection)
sudo systemctl enable sysstat && sudo systemctl start sysstat

# Check CPU spikes (every 1 second, 10 times)
sar -u 1 10

# Check memory usage (every 1 second)
sar -r 1 5

# Check disk I/O (every 1 second)
iostat -dx 1 5

# Check process-level CPU usage (top alternative)
pidstat -u 1 5
```

---

### **2. 🔁 Detect Kernel-Level Traffic Jams (CPU Scheduler, Run Queue)**  
**Problem:** Too many tasks waiting for CPU (invisible in normal monitoring).  

#### **Commands:**  
```bash
# Check CPU run queue (how many tasks are waiting)
sar -q 1 5

# Check context switches (high = too many small tasks)
sar -w 1 5

# Advanced: Use 'perf' to see kernel scheduling delays
sudo apt install linux-tools-common linux-tools-$(uname -r) -y
sudo perf sched latency
```

---

### **3. 🐢 Detect Micro-Network Latency (Jitter, Packet Loss)**  
**Problem:** Tiny internet slowdowns causing apps to fail randomly.  

#### **Commands:**  
```bash
# Install mtr (combines ping + traceroute)
sudo apt install mtr -y

# Continuous ping test (check for packet loss)
ping -i 0.1 google.com

# MTR (shows network path + packet loss)
mtr --report google.com

# Check DNS resolution time (slow DNS = slow apps)
dig google.com | grep "Query time"
```

---

### **4. 📉 Detect Disk Burst Credit Depletion (AWS EBS Slowdowns)**  
**Problem:** Disk suddenly slows down because it ran out of "burst credits."  

#### **Commands:**  
```bash
# Check disk I/O latency (high = problem)
iostat -dxm 1 5

# Check AWS EBS burst balance (if on AWS)
# (Requires AWS CLI + CloudWatch)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EBS \
  --metric-name BurstBalance \
  --dimensions Name=VolumeId,Value=vol-1234567890 \
  --statistics Average \
  --period 60 \
  --start-time $(date -u +"%Y-%m-%dT%H:%M:%SZ" --date="-5 minutes") \
  --end-time $(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

---

### **5. � Check Docker/Container Throttling**  
**Problem:** Containers get slowed down, but host looks fine.  

#### **Commands:**  
```bash
# Install cAdvisor (Google’s container monitor)
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  gcr.io/cadvisor/cadvisor:v0.47.0

# Check container stats (CPU, memory, I/O)
docker stats

# Check if a container is being throttled
docker inspect <container_id> | grep -i throttled
```

---

### **6. � Detect App-Level Stalls (DB Waits, Lock Contention)**  
**Problem:** App is stuck waiting for a database, but CPU is low.  

#### **Commands:**  
```bash
# For MySQL/MariaDB: Check slow queries
sudo apt install mysql-client -y
mysql -u root -p -e "SHOW FULL PROCESSLIST;"

# For PostgreSQL: Check locks
sudo -u postgres psql -c "SELECT * FROM pg_locks;"

# General Linux: Check which processes are in 'D' state (uninterruptible sleep)
ps aux | awk '$8 == "D" { print $0 }'
```

---

### **7. 💥 Detect DNS Failures (Apps Failing Randomly)**  
**Problem:** DNS fails once, app crashes, no logs.  

#### **Commands:**  
```bash
# Test DNS resolution time
time nslookup google.com

# Check system DNS cache (if using systemd-resolved)
sudo systemd-resolve --statistics

# Force clear DNS cache (if issues)
sudo systemd-resolve --flush-caches
```

---

### **8. 🔒 Detect Security Issues (Failed Logins, Attacks)**  
**Problem:** Hackers try to break in, but logs are ignored.  

#### **Commands:**  
```bash
# Check failed SSH logins
sudo grep "Failed password" /var/log/auth.log

# Install & configure Fail2Ban (blocks brute-force attacks)
sudo apt install fail2ban -y
sudo systemctl enable fail2ban && sudo systemctl start fail2ban

# Check active bans
sudo fail2ban-client status sshd
```

---

## **🎯 Summary Cheat Sheet**  
| Problem | Command(s) |
|---------|------------|
| **CPU/Memory/Disk Spikes** | `sar -u 1 5`, `iostat -dx 1 5` |
| **Kernel Traffic Jams** | `sar -q 1 5`, `perf sched latency` |
| **Network Jitter** | `mtr --report google.com`, `ping -i 0.1 google.com` |
| **Disk Burst Credit Issues** | `iostat -dxm 1 5`, AWS `BurstBalance` metric |
| **Container Throttling** | `docker stats`, `cAdvisor` |
| **App Stalls (DB Locks)** | MySQL: `SHOW PROCESSLIST;`, Postgres: `SELECT * FROM pg_locks;` |
| **DNS Failures** | `nslookup google.com`, `systemd-resolve --flush-caches` |
| **Security (SSH Attacks)** | `grep "Failed password" /var/log/auth.log`, `fail2ban-client status` |

Now you can **find and fix hidden problems** like a pro! 🚀
