##Linux essential areas for DevOps Engineer
**1. Process Management: Your Applications Under the Hood**
When your application crashes, consumes too much CPU, or becomes unresponsive, you need to understand processes. Think of processes as individual workers in a factory — each doing a specific job.
The Commands You’ll Actually Use
*See what’s running:*
```
# The holy trinity of process monitoring
ps aux                    # Snapshot of all processes
top                       # Real-time view (like Task Manager)
htop                      # Enhanced version (install it everywhere)
```
When I run ps aux on a production server, I’m looking for:

> High CPU usage processes (%CPU column)
> Memory hogs (%MEM column)
> Processes that shouldn’t be running
Example scenario: Your web server is slow. Run top and you see a rogue process eating 90% CPU. Now you know exactly what to kill.

**Kill misbehaving processes:**
```
# Find the troublemaker first
ps aux | grep nginx
# Kill it gently
kill 1234
# Kill it with force (when gentle doesn’t work)
kill -9 1234
# Nuclear option - kill all instances
killall nginx
```
Pro tip: Always try kill before kill -9. The gentle kill allows the process to clean up properly.
*Process Hierarchy Matters*
```
# See the family tree of processes
pstree                    # Visual process tree
pstree -p                 # Include process IDs
```
This shows you parent-child relationships. Kill a parent process, and its children die too. Essential for understanding how applications spawn sub-processes.

**2. Networking: How Your Services Talk**
In a microservices world, everything is connected. Applications talk to databases, APIs call other APIs, load balancers distribute traffic. When networking breaks, everything breaks.

**Network Interface Management**
```
# Modern way to check network setup
ip addr show              # Show all network interfaces
ip link show              # Show network devices
# The old way (still works)
ifconfig                  # Shows interface configuration
```
Real scenario: Your application can’t reach the database. First thing I check: ip addr show to ensure the network interface is up and has the right IP address.
**Port Monitoring — The Detective Work**
```
# Who’s using what port?
netstat -tulpn            # Show all listening ports with processes
ss -tulpn                 # Modern replacement (faster)
# Specific port investigation
lsof -i :80               # What’s using port 80?
lsof -i :3306             # Check if MySQL is running
```
Example workflow: Application won’t start, claiming “port already in use.” Run lsof -i :8080 and discover another process is squatting on your port. Kill it or change your app’s port.
**Network Troubleshooting Arsenal**
```
# Is the server reachable?
ping google.com           # Basic connectivity test
ping -c 4 server.com      # Send only 4 packets
# Trace the network path
traceroute google.com     # Show every router hop
# DNS investigation
dig google.com            # Detailed DNS lookup
nslookup google.com       # Simple DNS check
```
Production debugging story: API calls were timing out randomly. traceroute revealed packet loss at a specific router. Network team fixed the routing issue.
**Firewall Management:**
```
# Ubuntu/Debian firewall (UFW - User Friendly Firewall)
sudo ufw status           # Check firewall status
sudo ufw enable           # Turn on firewall
sudo ufw allow 22         # Allow SSH
sudo ufw allow 80/tcp     # Allow HTTP traffic
sudo ufw deny 8080        # Block specific port
# Traditional iptables (more complex but powerful)
sudo iptables -L          # List all rules
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT  # Allow HTTPS
```
##3. File System Navigation##
You’ll spend countless hours navigating server file systems. Efficiency here saves hours daily.

**Navigation Mastery:**
```
# Basic movement
pwd                       # Where am I?
cd /var/log              # Go to logs directory
cd ..                    # Go up one level
cd ~                     # Go home
cd -                     # Go to previous directory
```
Pro workflow: I always start with pwd to know my location, then use ls -la to see what’s available.
**File and Directory Operations**
```
# Creating things
mkdir -p /path/to/deep/directory    # Create nested directories
touch config.yaml                   # Create empty file
# Copying and moving
cp app.py app.py.backup            # Backup before changes
cp -r /source/dir /destination/     # Copy entire directories
mv oldname.txt newname.txt          # Rename files
# Removing (be careful!)
rm filename                         # Delete file
rm -rf directory                    # Delete directory and contents (DANGEROUS!)
```
Safety tip: Always use ls to verify what you’re about to delete. I’ve seen entire projects wiped because someone was in the wrong directory.
**File Viewing Techniques**
```
# Quick file inspection
cat config.yaml           # Show entire file
head -20 app.log          # First 20 lines
tail -50 error.log        # Last 50 lines
tail -f application.log   # Follow log in real-time (lifesaver for debugging)
# Paginated viewing
less large-file.txt       # Navigate large files
```
Real scenario: Application is throwing errors. I run tail -f /var/log/app.log in one terminal while reproducing the issue in another. I can see errors appear in real-time.

##4. File Permissions: The Security Foundation##

Nothing is more frustrating than “Permission denied” errors. Understanding permissions prevents security issues and deployment failures.

**Understanding the Permission Matrix**
```
rwx rwx rwx
│   │   └── Others (everyone else)
│   └────── Group (file’s group)  
└────────── Owner (file creator)
r = read (4)    - Can view file content
w = write (2)   - Can modify file
x = execute (1) - Can run file as program
```
**Permission Management in Practice**
```
# Symbolic notation (readable)
chmod u+x script.sh       # Give owner execute permission
chmod g-w file.txt        # Remove group write permission
chmod o+r document.pdf    # Give others read permission
chmod a+x binary          # Give everyone execute permission
# Numeric notation (faster once you learn it)
chmod 755 script.sh       # rwxr-xr-x (common for scripts)
chmod 644 config.txt      # rw-r--r-- (common for config files)
chmod 600 private.key     # rw------- (private keys)
chmod 777 file.txt        # rwxrwxrwx (AVOID THIS - security risk!)
```
Production example: Deploy a script to production, but it won’t run. Check with ls -la script.sh and see it lacks execute permission. Fix with chmod +x script.sh.
**Ownership Management**
```
# Change file ownership
sudo chown user:group file.txt      # Change both user and group
sudo chown jenkins config.yaml      # Change only owner
sudo chown :docker script.sh        # Change only group
# Recursive ownership changes
sudo chown -R nginx:nginx /var/www/html   # Change entire directory tree
```
##5. File System Usage: Avoiding the Disk Space Disaster##
Running out of disk space crashes applications. Monitoring disk usage prevents 90% of storage-related incidents.

**Disk Space Monitoring**
```
# Check available space
df -h                     # Human-readable disk usage
df -i                     # Check inode usage (can run out even with space)
# Find what’s eating your disk
du -h /var/log            # Directory size
du -sh *                  # Size of each item in current directory
du -h --max-depth=1 /     # Top-level directory sizes
```
Emergency scenario: Server alerts show 95% disk usage. Run `du -sh /*` to find the culprit directory, then dive deeper with `du -h --max-depth=1 /var/log` to find massive log files.

**Finding Space Hogs**
```
# Find large files
find / -size +100M -type f 2>/dev/null    # Files larger than 100MB
find /var/log -name “*.log” -size +50M    # Large log files
# Find old files (potential cleanup candidates)
find /tmp -type f -mtime +30               # Files older than 30 days
```
##6. Search and Filter: Finding Needles in Haystacks##
Production environments generate massive amounts of data. Efficient searching separates good DevOps engineers from frustrated ones.

**File Finding Mastery**
```
# Find files by name
find /var/log -name “*.log”           # All log files
find /etc -name “*nginx*”             # Anything with nginx in name
find / -type f -name “config.yaml”    # Specific filename anywhere
# Find by size and date
find /var/log -size +100M             # Large log files
find /tmp -mtime +7                   # Files older than 7 days
find /etc -type f -perm 777           # World-writable files (security risk)
```
**Text Processing Powerhouse**
```
# grep - your text searching best friend
grep “ERROR” /var/log/app.log         # Find errors in logs
grep -r “database” /etc/              # Recursively search for “database”
grep -i “warning” *.log               # Case-insensitive search
grep -n “failed” app.log              # Show line numbers
grep -v “DEBUG” app.log               # Exclude debug messages
# Real-world log analysis
grep “500” /var/log/nginx/access.log | wc -l     # Count 500 errors
grep “$(date ‘+%Y-%m-%d’)” /var/log/app.log      # Today’s logs only
```
Debugging workflow: Application is slow. I run `grep -i “slow\|timeout\|error” /var/log/app.log | tail -20` to see recent issues.
**Advanced Text Processing**
```
# awk - column extraction magic
awk ‘{print $1}’ /var/log/nginx/access.log        # Extract IP addresses
awk -F: ‘{print $1}’ /etc/passwd                  # Extract usernames
awk ‘$9 == 404 {print $1}’ /var/log/nginx/access.log  # IPs with 404 errors
# sed - stream editing
sed ‘s/old/new/g’ config.txt          # Replace all occurrences
sed -n ‘10,20p’ large-file.txt        # Print lines 10-20
# cut - simple column extraction
cut -d: -f1 /etc/passwd               # First field using : delimiter
cut -c1-10 filename                   # Characters 1-10 of each line
# sort and unique analysis
sort /var/log/ips.txt | uniq -c | sort -nr    # Count and sort unique IPs
```
##7. Package Management: Installing and Managing Software##
Different Linux distributions use different package managers. Knowing them prevents “software not found” headaches.

**APT (Ubuntu/Debian) — The Most Common**

```
# Update package database first (always!)
sudo apt update                       # Refresh package lists
sudo apt upgrade                      # Upgrade installed packages

# Install software
sudo apt install nginx                # Install web server
sudo apt install htop git curl       # Multiple packages at once

# Remove software
sudo apt remove nginx                 # Remove package
sudo apt purge nginx                  # Remove package and configs
sudo apt autoremove                   # Clean up orphaned dependencies

# Search for packages
apt search docker                     # Find docker-related packages
apt list --installed | grep python   # List installed Python packages
```
**YUM/DNF (RedHat/CentOS/Fedora)**
```
# YUM (older RHEL/CentOS systems)
sudo yum update                       # Update all packages
sudo yum install docker               # Install Docker
sudo yum remove docker                # Remove Docker

# DNF (newer Fedora/RHEL systems)
sudo dnf update                       # Update packages
sudo dnf install podman               # Install container runtime
sudo dnf search kubernetes            # Search for packages
```
Production tip: Always run apt update before apt install to ensure you get the latest package versions.

##8. System Configuration: Making Your Environment Work##
Understanding system configuration prevents mysterious failures and enables automation.

**System Information Commands**
```
# Know your system
uname -a                              # Complete system info
lscpu                                 # CPU information
free -h                               # Memory usage
lsblk                                 # Block devices (disks)
df -h                                 # Disk usage

# OS and version info
cat /etc/os-release                   # OS version details
hostnamectl                           # Hostname and system info
```
**Shell Configuration Mastery**
```
# Profile files (loaded in order)
/etc/profile                          # System-wide login profile
~/.bash_profile                       # User login profile
~/.bashrc                             # User interactive shell config

# Create useful aliases
alias ll=’ls -la’                     # Long listing
alias la=’ls -la’                     # All files with details
alias grep=’grep --color=auto’        # Colored grep output
alias k=’kubectl’                     # Kubernetes shortcut

# Make aliases permanent
echo “alias ll=’ls -la’” >> ~/.bashrc
source ~/.bashrc                      # Reload configuration
```
Pro productivity tip: I create aliases for commonly used commands. My .bashrc has aliases for kubectl, docker, and complex find commands I use regularly.

**Environment Variables**
```
# View environment
env                                   # All environment variables
echo $PATH                            # Show PATH variable
echo $HOME                            # Home directory
# Set temporary variables
export API_KEY=”your-key-here”        # For current session
# Set permanent variables
echo ‘export API_KEY=”your-key-here”’ >> ~/.bashrc
source ~/.bashrc                      # Reload to activate
```
##9. System Monitoring: Keeping Your Finger on the Pulse##
Monitoring prevents problems. When you can see resource usage patterns, you can predict and prevent failures.
**Load and Performance Monitoring**
```
# System overview
uptime                                # Load average and uptime
w                                     # Who’s logged in and doing what
top                                   # Real-time process monitoring
htop                                  # Enhanced process viewer
# Memory analysis
free -h                               # Memory usage summary
cat /proc/meminfo                     # Detailed memory info
vmstat 1 5                            # Virtual memory stats (1 sec intervals, 5 times)
# I/O performance
iostat 1 5                            # Disk I/O statistics
sar 1 5                               # System activity report
```
Monitoring workflow: When a server feels slow, I run top to see CPU usage, free -h for memory, and iostat to check if disk I/O is the bottleneck.
**Swap Usage Analysis**
```
# Check swap status
swapon --show                         # Active swap partitions
cat /proc/swaps                       # Swap usage details
free -h                               # Memory and swap summary
```
Performance tip: If swap usage is high, your system needs more RAM or has a memory leak.

##10. Service Management: Controlling Your Applications##
Modern Linux uses systemd for service management. Master this, and you can control any application in production.

**Systemd Service Control**
```
# Basic service operations
sudo systemctl start nginx            # Start web server
sudo systemctl stop nginx             # Stop web server  
sudo systemctl restart nginx          # Restart (stop then start)
sudo systemctl reload nginx           # Reload config without restart
# Service status and health
systemctl status nginx                # Detailed service status
systemctl is-active nginx             # Quick active check
systemctl is-enabled nginx            # Check if starts at boot
# Enable/disable services
sudo systemctl enable nginx           # Start automatically at boot
sudo systemctl disable nginx          # Don’t start at boot
```
Real scenario: Deploy a new application version. Instead of killing processes manually, I run sudo systemctl restart myapp for a clean restart that respects the service configuration.

**Service Discovery and Troubleshooting**
```
# List services
systemctl list-units --type=service   # All services
systemctl list-units --failed         # Only failed services
systemctl list-unit-files --type=service | grep enabled  # Boot-enabled services
# Log investigation
journalctl -u nginx                    # Service-specific logs
journalctl -u nginx --since “1 hour ago”  # Recent logs
journalctl -f -u nginx                 # Follow logs in real-time
```
**Legacy Service Management**
```
# Traditional service command (still works)
sudo service nginx start              # Start service
sudo service nginx status             # Check status
sudo /etc/init.d/nginx restart        # Direct init script
```
##The Commands You’ll Use Every Day##

Here are the commands I use most frequently in production:
**Daily Operations**
```
# Process monitoring
ps aux | grep python                  # Find Python processes
top -p $(pgrep nginx)                 # Monitor specific processes
htop                                  # Interactive process manager

# Network debugging
ss -tulpn | grep :80                  # Check web server port
ping -c 3 database-server             # Test connectivity
curl -I https://api.example.com       # Check API health

# File operations
tail -f /var/log/app.log              # Follow application logs
find /var/log -name “*.log” -mtime -1 # Today’s log files
grep -r “ERROR” /var/log/ | tail -10  # Recent errors

# System health
df -h                                 # Disk space check
free -h                               # Memory usage
systemctl status docker               # Service status
```
**Emergency Debugging**
```
# High CPU investigation
top -n 1 -b | grep -A 5 “^top”       # Quick CPU snapshot
ps aux --sort=-%cpu | head -10        # Top CPU users

# Memory issues
ps aux --sort=-%mem | head -10        # Top memory users
cat /proc/meminfo | grep Available    # Available memory

# Disk space emergency
du -sh /* | sort -hr | head -10       # Largest directories
find /var/log -name “*.log” -size +100M  # Large log files
```
**Monitor logs in real-time during deployments**
```
# One terminal for deployment 
sudo systemctl restart myapp 
# Another terminal for monitoring 
tail -f /var/log/myapp.log
```
**The real power comes from combining them:**
```
# Complex real-world command combining multiple concepts
ps aux | grep nginx | grep -v grep | awk ‘{print $2}’ | xargs sudo kill -9

#This command:
Lists all processes (ps aux)
Finds nginx processes (grep nginx)
Excludes the grep command itself (grep -v grep)
Extracts process IDs (awk ‘{print $2}’)
Kills all those processes (xargs sudo kill -9)
```
Once you understand each piece, you can create powerful one-liners that solve complex problems.
Thanks for reading, see you on the next one.
