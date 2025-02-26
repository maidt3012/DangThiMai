Provide your solution here:
# Troubleshooting High Memory Usage on Ubuntu 24.04 VM Running NGINX Load Balancer

As a DevOps Engineer, troubleshooting high memory usage (99%) on a VM running **NGINX** as a load balancer requires a methodical approach. Below are the **steps, possible causes, impacts, and recovery solutions** for two potential production issues.

---

## **Issue 1: Memory Leak in NGINX Due to Misconfiguration or Traffic Spikes**

### **Troubleshooting Steps**
1. **Check memory usage per process:**
   ```bash
   ps aux --sort=-%mem | head -n 10
   ```
   - This will show the top 10 processes consuming the most memory.

2. **Check NGINX worker processes:**
   ```bash
   ps -eo pid,cmd,%mem --sort=-%mem | grep nginx
   ```
   - If multiple `nginx` worker processes are consuming excessive memory, there could be a misconfiguration.

3. **Check active connections and requests:**
   ```bash
   netstat -anp | grep nginx | wc -l
   ```
   - If the connection count is unusually high, it indicates a possible **traffic surge** or **DDoS attack**.

4. **Analyze NGINX logs:**
   ```bash
   tail -f /var/log/nginx/access.log
   tail -f /var/log/nginx/error.log
   ```
   - Look for **excessive connections**, slow responses, or unusual patterns.

5. **Check NGINX configuration:**
   ```bash
   cat /etc/nginx/nginx.conf
   ```
   - Look for settings like:
     ```nginx
     worker_processes auto;
     worker_connections 1024;
     client_max_body_size 10M;
     ```
   - If `worker_processes` is too high or `worker_connections` is misconfigured, it may cause memory exhaustion.

### **Possible Causes**
- High concurrent traffic overwhelming NGINX.
- Memory leaks due to misconfigured buffers, logs, or caching.
- DDoS or bot traffic causing resource exhaustion.

### **Impact**
- **Service degradation** or **downtime** due to memory exhaustion.
- Requests may start **failing** with **502 Bad Gateway** or **504 Gateway Timeout** errors.
- VM may become **unresponsive** due to excessive memory consumption.

### **Recovery Steps**
✅ **Reduce Memory Usage Temporarily**
```bash
systemctl restart nginx
```
- This frees up memory but does not solve the root cause.

✅ **Optimize NGINX Configuration**
- Adjust buffer and keepalive settings in `/etc/nginx/nginx.conf`:
  ```nginx
  worker_rlimit_nofile 65535;
  worker_processes auto;
  worker_connections 4096;
  keepalive_timeout 60;
  client_body_buffer_size 128k;
  ```
  - Increase `worker_connections` moderately.
  - Reduce `client_body_buffer_size` to prevent excessive memory usage.

✅ **Enable Rate Limiting to Mitigate Traffic Surges**
  ```nginx
  limit_req_zone $binary_remote_addr zone=limit:10m rate=10r/s;
  ```
  - Helps prevent memory exhaustion from excessive requests.

✅ **Implement Caching & Compression**
- Reduce memory footprint by enabling **gzip compression**:
  ```nginx
  gzip on;
  gzip_types text/plain text/css application/json;
  ```
- Utilize caching to minimize repeated processing of identical requests.

✅ **Scale Up or Load Balance**
- If high traffic is expected, add **more VMs** to share the load.

---

## **Issue 2: Memory Saturation Due to Swapping or Out-of-Control System Processes**

### **Troubleshooting Steps**
1. **Check system-wide memory usage:**
   ```bash
   free -m
   ```
   - If **swap usage is high**, the system may be over-committing memory.

2. **Check swap activity:**
   ```bash
   vmstat 2 5
   ```
   - Look at the **swap in (si) and swap out (so)** columns; high numbers indicate excessive swapping.

3. **Identify non-NGINX processes consuming memory:**
   ```bash
   top -o %MEM
   ```
   - Look for rogue processes consuming memory.

4. **Check system logs for errors:**
   ```bash
   dmesg | tail -20
   journalctl -xe
   ```
   - Identify **Out-of-Memory (OOM) kills** or **kernel warnings**.

5. **Check if a memory leak is causing issues:**
   ```bash
   sudo pmap -x <nginx_worker_pid>
   ```
   - Large memory allocation could indicate a leak.

### **Possible Causes**
- **Swap space misconfiguration** leading to system slowdowns.
- **Zombie processes** consuming memory in the background.
- **Background services** (e.g., monitoring agents, misconfigured cron jobs) eating RAM.

### **Impact**
- Excessive swapping can make the system **unresponsive**.
- NGINX may experience **timeouts** due to delayed response.
- Entire **VM may crash**, requiring a restart.

### **Recovery Steps**
✅ **Reduce Swapping by Increasing Swap Space**
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
- Prevents system freeze by adding swap.

✅ **Kill Non-Essential High-Memory Processes**
```bash
sudo kill -9 <pid>
```
- Frees up memory immediately.

✅ **Optimize Memory Usage in Kernel**
```bash
sudo sysctl -w vm.swappiness=10
```
- Reduces unnecessary swap usage.

✅ **Upgrade VM Resources or Scale Out**
- If the workload is growing, consider **vertical scaling** (increasing RAM) or **horizontal scaling** (adding more VMs).

---

## **Final Summary**
| **Issue** | **Cause** | **Impact** | **Recovery Steps** |
|-----------|----------|------------|---------------------|
| **NGINX Memory Leak or Traffic Surge** | High traffic, misconfiguration, DDoS attack | 502/504 errors, high memory, VM slowdown | Optimize NGINX, rate limiting, restart service, scale up |
| **Swap Saturation or Rogue Processes** | Background services, excessive swap, zombie processes | System unresponsiveness, slow response, OOM kills | Kill rogue processes, increase swap, optimize swappiness |

By following these troubleshooting steps, you can diagnose and resolve high memory usage issues efficiently while ensuring minimal downtime. 🚀
