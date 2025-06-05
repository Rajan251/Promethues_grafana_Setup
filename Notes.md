# **Detecting and Troubleshooting Microsecond Spikes on Servers**

## **Why Standard Monitoring Misses Microsecond Spikes**
Most monitoring tools (Prometheus, Zabbix, CloudWatch) use **1-60 second intervals**, missing:
- **CPU spikes lasting <100ms**
- **Disk I/O micro-stalls**
- **Network packet jitter**
- **Kernel scheduler delays**

**Impact on DB Servers:**
| Spike Type       | Database Impact                                                                 |
|------------------|--------------------------------------------------------------------------------|
| **100ms CPU**    | Query timeouts, replication lag, connection drops                              |
| **Disk I/O**     | Transaction stalls, write delays, InnoDB buffer pool issues                    |
| **Network**      | Replication failures, cluster split-brain risk                                 |
| **Kernel**       | MySQL thread scheduling delays, lock contention                                |

---

## **Troubleshooting Commands (Ubuntu Server)**

### **1. Detect CPU Spikes (<100ms)**
```bash
# Install necessary tools
sudo apt install linux-tools-common linux-tools-$(uname -r) sysstat -y

# Real-time CPU usage (millisecond precision)
perf stat -e cpu-clock -a sleep 1

# Check CPU run queue (tasks waiting)
sar -q 1 3  # Look for 'runq-sz' > CPU cores

# Identify processes causing spikes
pidstat -u 1 5 -l
```

### **2. Catch Disk I/O Micro-Stalls**
```bash
# Check I/O latency at µs level
sudo iostat -xmd 1 3 | grep -E 'Device|await'

# Kernel block layer tracing
sudo blktrace -d /dev/nvme0n1 -o - | blkparse -i -

# Alternative: bpftrace for disk latency
sudo bpftrace -e 'kprobe:blk_account_io_start { @start[tid] = nsecs; } kprobe:blk_account_io_done { @usecs = hist(nsecs - @start[tid]); delete(@start[tid]); }'
```

### **3. Network Jitter Analysis**
```bash
# Micro-latency detection
ping -A -i 0.001 google.com  # Look for >1ms spikes

# Advanced: tcptraceroute for TCP-level jitter
sudo apt install tcptraceroute
tcptraceroute -n -f 64 -m 64 example.com 3306
```

### **4. Kernel Scheduler Issues**
```bash
# Check for scheduler latency
sudo perf sched latency

# System-wide tracing
sudo trace-cmd record -e sched:sched_switch sleep 1
trace-cmd report
```

---

## **Specialized Monitoring Tools**

### **1. For µs-Level Spikes**
| Tool            | Best For                          | Install Command                     |
|-----------------|-----------------------------------|-------------------------------------|
| **eBPF/bpftrace** | Kernel-level tracing              | `sudo apt install bpftrace`         |
| **perf**        | CPU pipeline stalls               | Part of `linux-tools` package       |
| **nicstat**     | Micro-network drops               | `sudo apt install nicstat`          |

### **2. Persistent Monitoring**
```bash
# Netdata (real-time dashboard)
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# Vector (high-res metrics collection)
curl --proto '=https' --tlsv1.2 -sSf https://sh.vector.dev | sh
```

### **3. Database-Specific Tools**
```bash
# MySQL InnoDB metrics
SHOW ENGINE INNODB STATUS\G

# PostgreSQL wait events
SELECT * FROM pg_stat_activity WHERE wait_event_type IS NOT NULL;
```

---

## **Troubleshooting Workflow**
1. **Reproduce the Spike**:  
   ```bash
   stress-ng --cpu 1 --timeout 100ms  # Simulate 100ms CPU spike
   ```

2. **Correlate Metrics**:  
   ```bash
   # Run simultaneously:
   perf stat -e cpu-clock -a sleep 1 &  # CPU
   iostat -xmd 1 &                      # Disk
   ping -A -i 0.001 db-replica &        # Network
   ```

3. **Check Kernel Logs**:  
   ```bash
   dmesg -T | grep -E 'sched|oom|cpu'
   ```

4. **Profile Database**:  
   ```sql
   /* MySQL */ SET GLOBAL slow_query_log=1; SET long_query_time=0.1;
   /* PostgreSQL */ ALTER SYSTEM SET log_min_duration_statement=100;
   ```

---

## **Key Fixes for DB Servers**
- **CPU Spikes**: Adjust CFS scheduler (`/proc/sys/kernel/sched_latency_ns`)
- **Disk I/O**: Switch to **NOOP scheduler** for NVMe (`echo noop > /sys/block/nvme0n1/queue/scheduler`)
- **Network**: Enable **TCP BBR** (`echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf`)
- **Kernel**: Update to **Low-Latency Kernel** (`sudo apt install linux-lowlatency`)

> **Pro Tip**: For AWS/GCP, use **Enhanced Monitoring** (1-5s granularity) and pair with `perf` for µs-level analysis.
