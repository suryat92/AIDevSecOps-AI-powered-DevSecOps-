# 🚀 Learning Linux & Shell Scripting For DevOps - Online Environment

# What is Linux? (Definition)

**Linux** is a free, open-source operating system based on Unix. It is the foundation of most servers, cloud platforms, and DevOps tools worldwide. Linux is known for its stability, security, and flexibility, allowing multiple users and multitasking, and treating everything (devices, processes, data) as files in a hierarchical directory structure.

## Why Learn Linux for DevOps?

Linux powers the majority of IT infrastructure and cloud environments. DevOps professionals rely on Linux for automation, server management, and application deployment. Mastering Linux and shell scripting is essential for anyone pursuing a career in DevOps, cloud computing, or system administration.

## Course Overview

**Target Audience:** Computer Science Students  
**Prerequisites:** Basic computer knowledge, Internet connection  
**Environment:** Online Linux Terminal & Playground  
**Duration:** 40+ hours of hands-on learning  

This comprehensive course uses **online Linux environments** to provide immediate access to Linux systems without any installation requirements. Perfect for beginners who want to start learning immediately!

---

## 🌐 Environment Setup - Online Linux Playground

### Why Choose Online Environment?

✅ **Instant Access** - Start learning in under 60 seconds  
✅ **Zero Installation** - No downloads, no setup hassles  
✅ **Universal Compatibility** - Works on Windows, Mac, Linux, tablets  
✅ **Safe Learning** - Cannot damage your computer  
✅ **Pre-configured** - All tools already installed  
✅ **Resource Efficient** - Uses no local CPU/memory  
✅ **Collaborative** - Share sessions with classmates  

### 🏆 Recommended Online Platforms

#### Option 1: LabEx Linux Playground ⭐ (PRIMARY RECOMMENDATION)

**Website:** https://labex.io/tutorials/linux-online-linux-playground-372915

**Features:**
- Ubuntu 22.04 LTS environment
- Three interfaces: VS Code, Desktop, Web Terminal
- Pre-installed development tools (git, vim, nano, curl, wget)
- Interactive tutorials and challenges
- Real-time practice environment
- 50+ built-in Linux labs

**Quick Start:**
1. Visit https://labex.io and create free account
2. Navigate to "Linux Playground" 
3. Click "Start Lab"
4. Choose "Web Terminal" interface
5. Wait 30-60 seconds for environment to load
6. Start practicing immediately!
---

## 🔧 Initial Setup (Using LabEx - Recommended)

### Step 1: Account Creation and Access

```bash
# 1. Visit https://labex.io
# 2. Click "Sign Up" (use your gmail account)
# 3. Verify email and login
# 4. Navigate to "Linux Playground"
# 5. Click "Start Lab" → Choose "Web Terminal"
```

### Step 2: Environment Verification

```bash
# Check your current location
pwd

# Check Linux distribution and version
cat /etc/os-release

# Check available disk space
df -h

# Check memory information
free -h

# Check CPU information
lscpu | head -10

# Check current user
whoami

# Check network connectivity
ping -c 3 google.com
```

Expected output should show Ubuntu 22.04 or similar Linux distribution.

### Step 3: Create Course Directory Structure

```bash
# Create main lab directory
mkdir -p ~/linux-devops-lab

# Create subdirectories for different modules
cd ~/linux-devops-lab
mkdir -p {scripts,exercises,projects,backups,config,logs}

# Create module-specific directories
mkdir -p exercises/{module2-fundamentals,module3-commands,module4-scripting,module5-advanced,module6-devops,module7-projects}

# Verify directory structure
tree . || find . -type d
```

### Step 4: Install Additional Tools (if needed)

```bash
# Update package list (if you have sudo access)
sudo apt update 2>/dev/null || echo "Note: Limited sudo access in online environment"

# Check available tools
which git vim nano curl wget tree

# If tree is not available, create simple alternative
if ! which tree >/dev/null; then
    alias tree='find . -type d | sed -e "s/[^-][^\/]*\//  |/g" -e "s/|\([^ ]\)/|-\1/"'
fi
```

### Step 5: Configure Environment

```bash
# Set up basic bash configuration
cat >> ~/.bashrc << 'EOF'

# Custom aliases for DevOps course
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias cls='clear'
alias h='history'

# Useful functions
mkcd() { mkdir -p "$1" && cd "$1"; }
backup() { cp "$1" "$1.backup.$(date +%Y%m%d_%H%M%S)"; }

# DevOps environment indicator
export PS1='\[\033[01;32m\]devops-student@linux-lab\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# Course directory shortcut
export COURSE_HOME="$HOME/linux-devops-lab"
alias course='cd $COURSE_HOME'

echo "DevOps Linux Lab Environment Ready!"
EOF

# Reload configuration
source ~/.bashrc
```

---

## 📚 Module 2: Linux Fundamentals

### Understanding Linux Operating System

Linux is a free, open-source operating system that powers most servers, cloud platforms, and DevOps tools worldwide.

**Key Characteristics:**
- **Multi-user:** Multiple users can work simultaneously
- **Multi-tasking:** Runs multiple processes concurrently  
- **Case-sensitive:** `File.txt` and `file.txt` are different
- **Everything is a file:** Devices, processes, and data are treated as files
- **Hierarchical structure:** Organized in a tree-like directory structure

### Linux File System Hierarchy

```bash
# Explore the root directory
ls -la /

# Understanding key directories:
echo "=== Key Linux Directories ==="
echo "/bin    - Essential command binaries"
echo "/boot   - Boot loader files"
echo "/dev    - Device files"  
echo "/etc    - System configuration files"
echo "/home   - User home directories"
echo "/lib    - Essential shared libraries"
echo "/tmp    - Temporary files"
echo "/usr    - User programs and data"
echo "/var    - Variable data (logs, databases)"
```

### Practice Exercise 2.1: File System Exploration

```bash
# Navigate to course directory
cd ~/linux-devops-lab/exercises/module2-fundamentals

# Create practice directory
mkdir filesystem-exploration
cd filesystem-exploration

# Explore different directories
echo "Current location: $(pwd)"

# List root directory contents
ls -la / > root_contents.txt

# Check your home directory
ls -la ~ > home_contents.txt

# Explore /etc directory (configuration files)
ls -la /etc | head -20 > etc_sample.txt

# Check system information
uname -a > system_info.txt
cat /etc/os-release > os_info.txt

# Display results
echo "=== File System Exploration Results ==="
echo "Files created:"
ls -la *.txt

echo -e "\n=== System Information ==="
cat system_info.txt
cat os_info.txt
```

---

## 💻 Module 3: Essential Linux Commands

### File and Directory Operations

```bash
# Navigate to exercises directory
cd ~/linux-devops-lab/exercises/module3-commands
mkdir file-operations && cd file-operations

echo "=== FILE AND DIRECTORY OPERATIONS ==="

# Creating directories
echo "1. Creating directories..."
mkdir documents pictures scripts
mkdir -p projects/{web,mobile,desktop}

# Creating files
echo "2. Creating files..."
touch documents/report1.txt documents/report2.txt
touch pictures/image1.jpg pictures/image2.png
echo "#!/bin/bash" > scripts/backup.sh
echo "echo 'Hello World'" > scripts/hello.sh

# Listing files and directories
echo "3. Directory structure:"
tree . || find . -type d

# Copying files
echo "4. Copying files..."
cp documents/report1.txt documents/report1_backup.txt
cp -r documents backup_documents

# Moving and renaming
echo "5. Moving and renaming..."
mv documents/report2.txt documents/reports/
mkdir -p documents/reports
mv documents/report2.txt documents/reports/

# File permissions
echo "6. Setting permissions..."
chmod +x scripts/*.sh
ls -la scripts/

echo "File operations completed!"
```

### Text Processing Commands

```bash
# Navigate to text processing exercise
cd ~/linux-devops-lab/exercises/module3-commands
mkdir text-processing && cd text-processing

# Create sample data files
cat > employees.txt << 'EOF'
John Doe,25,Developer,50000
Jane Smith,30,Manager,65000
Bob Johnson,28,Designer,45000
Alice Brown,35,Developer,55000
Charlie Wilson,22,Intern,25000
Diana Davis,29,Developer,52000
Eve Martinez,31,Manager,70000
Frank Lee,27,Designer,48000
EOF

cat > server.log << 'EOF'
2024-08-06 10:15:30 INFO Server started successfully
2024-08-06 10:16:45 INFO User john.doe logged in
2024-08-06 10:17:20 ERROR Database connection failed
2024-08-06 10:18:10 INFO Database connection restored
2024-08-06 10:19:55 WARN High memory usage detected
2024-08-06 10:20:30 INFO User jane.smith logged in
2024-08-06 10:21:15 ERROR Authentication failed for user test
2024-08-06 10:22:00 INFO Backup process started
EOF

echo "=== TEXT PROCESSING EXERCISES ==="

# 1. Display file contents
echo "1. Viewing files:"
echo "First 5 lines of employees.txt:"
head -5 employees.txt

echo -e "\nLast 3 lines of server.log:"
tail -3 server.log

# 2. Searching with grep
echo -e "\n2. Searching with grep:"
echo "All developers:"
grep "Developer" employees.txt

echo -e "\nAll error messages:"
grep "ERROR" server.log

echo -e "\nLines containing 'Manager' or 'Developer':"
grep -E "(Manager|Developer)" employees.txt

# 3. Text manipulation with cut and awk
echo -e "\n3. Text manipulation:"
echo "Employee names only:"
cut -d',' -f1 employees.txt

echo -e "\nEmployee names and salaries:"
awk -F',' '{print $1 ": $" $4}' employees.txt

# 4. Sorting and counting
echo -e "\n4. Sorting and counting:"
echo "Employees sorted by age:"
sort -t',' -k2 -n employees.txt

echo -e "\nJob title distribution:"
cut -d',' -f3 employees.txt | sort | uniq -c

# 5. Advanced text processing
echo -e "\n5. Statistics:"
echo "Average salary:"
cut -d',' -f4 employees.txt | awk '{sum+=$1; count++} END {printf "%.2f\n", sum/count}'

echo "Error count in logs:"
grep -c "ERROR" server.log

echo "Text processing exercises completed!"
```

---

## 📝 Module 4: Shell Scripting Fundamentals

### Your First Shell Script

```bash
# Navigate to scripting exercises
cd ~/linux-devops-lab/exercises/module4-scripting
mkdir basic-scripts && cd basic-scripts

# Create your first script
cat > hello_world.sh << 'EOF'
#!/bin/bash
# My First Shell Script
# Purpose: Introduction to shell scripting

echo "================================="
echo "    Welcome to Shell Scripting!"
echo "================================="
echo
echo "Script Information:"
echo "- Script name: $0"
echo "- Current user: $(whoami)"
echo "- Current directory: $(pwd)"
echo "- Current date: $(date)"
echo "- System uptime: $(uptime -p 2>/dev/null || uptime)"
echo
echo "System Details:"
echo "- Operating System: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'=' -f2 | tr -d '\"')"
echo "- Kernel Version: $(uname -r)"
echo "- Architecture: $(uname -m)"
echo
echo "Your first shell script executed successfully!"
echo "================================="
EOF

# Make script executable and run
chmod +x hello_world.sh
echo "Running your first shell script:"
./hello_world.sh
```

### Variables and User Input

```bash
# Create variables demonstration script
cat > variables_demo.sh << 'EOF'
#!/bin/bash
# Variables and User Input Demo

# String variables
COURSE_NAME="Linux & Shell Scripting for DevOps"
VERSION="1.0"
INSTRUCTOR="DevOps Academy"

# Numeric variables
TOTAL_MODULES=7
CURRENT_MODULE=4

# Command substitution
CURRENT_DATE=$(date +"%Y-%m-%d %H:%M:%S")
DISK_USAGE=$(df -h / | awk 'NR==2{print $5}')

# Display information
echo "Course: $COURSE_NAME"
echo "Version: $VERSION" 
echo "Instructor: $INSTRUCTOR"
echo "Module: $CURRENT_MODULE of $TOTAL_MODULES"
echo "Generated on: $CURRENT_DATE"
echo "System disk usage: $DISK_USAGE"
echo

# Get user input
echo "Let's personalize this script..."
read -p "Enter your name: " STUDENT_NAME
read -p "Enter your age: " STUDENT_AGE
read -p "What's your goal with DevOps? " GOAL

echo
echo "Personal Information:"
echo "- Name: $STUDENT_NAME"
echo "- Age: $STUDENT_AGE years old"
echo "- Goal: $GOAL"

# Conditional based on age
if [ $STUDENT_AGE -ge 18 ]; then
    echo "- Status: Adult learner"
else
    echo "- Status: Young learner"
fi

echo
echo "Welcome to the course, $STUDENT_NAME!"
EOF

chmod +x variables_demo.sh
echo -e "\nRunning variables demo script:"
# Note: This script requires interactive input, so we'll show how to run it
echo "To run this script interactively: ./variables_demo.sh"
```

### Control Structures and Loops

```bash
# Create control structures demonstration
cat > control_structures.sh << 'EOF'
#!/bin/bash
# Control Structures and Loops Demo

echo "=== CONTROL STRUCTURES DEMO ==="

# 1. If-elif-else statement
echo "1. Grade Calculator:"
SCORE=85

if [ $SCORE -ge 90 ]; then
    GRADE="A"
elif [ $SCORE -ge 80 ]; then
    GRADE="B"
elif [ $SCORE -ge 70 ]; then
    GRADE="C"
elif [ $SCORE -ge 60 ]; then
    GRADE="D"
else
    GRADE="F"
fi

echo "Score: $SCORE -> Grade: $GRADE"

# 2. For loop with range
echo -e "\n2. Countdown:"
for i in {5..1}; do
    echo "Countdown: $i"
    sleep 1
done
echo "Liftoff! 🚀"

# 3. For loop with list
echo -e "\n3. DevOps Tools:"
TOOLS="Git Docker Kubernetes Jenkins Ansible Terraform"
for tool in $TOOLS; do
    echo "- $tool"
done

# 4. While loop
echo -e "\n4. Process Monitor (sample):"
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Check #$COUNT: System OK"
    COUNT=$((COUNT + 1))
    sleep 1
done

# 5. Case statement
echo -e "\n5. Environment Detector:"
ENV="production"

case $ENV in
    "development"|"dev")
        echo "Development environment detected"
        ;;
    "testing"|"test")
        echo "Testing environment detected"
        ;;
    "production"|"prod")
        echo "Production environment detected - Extra caution required!"
        ;;
    *)
        echo "Unknown environment: $ENV"
        ;;
esac

echo -e "\nControl structures demo completed!"
EOF

chmod +x control_structures.sh
echo -e "\nRunning control structures demo:"
./control_structures.sh
```

---

## 🚀 Module 5: Advanced Shell Scripting

### Functions and Error Handling

```bash
# Navigate to advanced scripting
cd ~/linux-devops-lab/exercises/module5-advanced
mkdir advanced-scripts && cd advanced-scripts

# Create advanced script with functions and error handling
cat > system_toolkit.sh << 'EOF'
#!/bin/bash
# System Administration Toolkit
# Purpose: Demonstrate functions, error handling, and advanced scripting

# Global variables
SCRIPT_NAME=$(basename "$0")
LOG_FILE="system_toolkit.log"
ERROR_COUNT=0

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Logging function
log_message() {
    local level=$1
    local message=$2
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message" >> "$LOG_FILE"
    
    case $level in
        "ERROR")
            echo -e "${RED}[ERROR]${NC} $message" >&2
            ((ERROR_COUNT++))
            ;;
        "WARN")
            echo -e "${YELLOW}[WARN]${NC} $message"
            ;;
        "INFO")
            echo -e "${BLUE}[INFO]${NC} $message"
            ;;
        "SUCCESS")
            echo -e "${GREEN}[SUCCESS]${NC} $message"
            ;;
        *)
            echo "$message"
            ;;
    esac
}

# Function to check if command exists
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Function to get system information
get_system_info() {
    log_message "INFO" "Gathering system information..."
    
    echo "=== SYSTEM INFORMATION ==="
    echo "Hostname: $(hostname)"
    echo "Operating System: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'=' -f2 | tr -d '\"')"
    echo "Kernel: $(uname -r)"
    echo "Architecture: $(uname -m)"
    echo "Uptime: $(uptime -p 2>/dev/null || uptime)"
    
    log_message "SUCCESS" "System information gathered"
}

# Function to check disk usage
check_disk_usage() {
    log_message "INFO" "Checking disk usage..."
    
    echo -e "\n=== DISK USAGE ==="
    df -h | head -1
    df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{print $1 "\t" $2 "\t" $3 "\t" $4 "\t" $5 "\t" $6}'
    
    # Check for high disk usage
    local usage=$(df / | awk 'NR==2{print $5}' | sed 's/%//')
    if [ "$usage" -gt 80 ]; then
        log_message "WARN" "High disk usage detected: ${usage}%"
    else
        log_message "SUCCESS" "Disk usage is normal: ${usage}%"
    fi
}

# Function to check memory usage
check_memory() {
    log_message "INFO" "Checking memory usage..."
    
    echo -e "\n=== MEMORY USAGE ==="
    free -h
    
    # Calculate memory usage percentage
    local mem_usage=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100.0}')
    if [ "$mem_usage" -gt 80 ]; then
        log_message "WARN" "High memory usage: ${mem_usage}%"
    else
        log_message "SUCCESS" "Memory usage is normal: ${mem_usage}%"
    fi
}

# Function to check running processes
check_processes() {
    log_message "INFO" "Checking running processes..."
    
    echo -e "\n=== TOP PROCESSES ==="
    echo "PID     %CPU %MEM COMMAND"
    ps aux --sort=-%cpu | head -6 | tail -5 | awk '{printf "%-8s%-5s%-5s%s\n", $2, $3, $4, $11}'
    
    local process_count=$(ps aux | wc -l)
    log_message "INFO" "Total running processes: $process_count"
}

# Function to create system report
create_report() {
    local report_file="system_report_$(date +%Y%m%d_%H%M%S).txt"
    
    log_message "INFO" "Creating system report: $report_file"
    
    {
        echo "SYSTEM HEALTH REPORT"
        echo "===================="
        echo "Generated on: $(date)"
        echo "Generated by: $SCRIPT_NAME"
        echo
        
        get_system_info
        check_disk_usage  
        check_memory
        check_processes
        
        echo -e "\n=== REPORT SUMMARY ==="
        echo "Total errors encountered: $ERROR_COUNT"
        if [ $ERROR_COUNT -eq 0 ]; then
            echo "System Status: HEALTHY ✅"
        else
            echo "System Status: NEEDS ATTENTION ⚠️"
        fi
        
    } > "$report_file"
    
    log_message "SUCCESS" "Report created: $report_file"
    echo -e "\nFull report saved to: $report_file"
}

# Function to display help
show_help() {
    echo "Usage: $SCRIPT_NAME [OPTION]"
    echo "System Administration Toolkit"
    echo
    echo "Options:"
    echo "  -s, --system     Show system information"
    echo "  -d, --disk       Check disk usage" 
    echo "  -m, --memory     Check memory usage"
    echo "  -p, --processes  Show top processes"
    echo "  -r, --report     Generate full system report"
    echo "  -h, --help       Show this help message"
    echo
    echo "Examples:"
    echo "  $SCRIPT_NAME --system"
    echo "  $SCRIPT_NAME --report"
}

# Main function
main() {
    # Initialize log
    log_message "INFO" "Starting $SCRIPT_NAME"
    
    case "${1:-}" in
        -s|--system)
            get_system_info
            ;;
        -d|--disk)
            check_disk_usage
            ;;
        -m|--memory)
            check_memory
            ;;
        -p|--processes)
            check_processes
            ;;
        -r|--report)
            create_report
            ;;
        -h|--help|"")
            show_help
            ;;
        *)
            log_message "ERROR" "Unknown option: $1"
            show_help
            exit 1
            ;;
    esac
    
    log_message "INFO" "$SCRIPT_NAME completed"
}

# Execute main function with all arguments
main "$@"
EOF

chmod +x system_toolkit.sh

echo "Advanced script created! Try these commands:"
echo "./system_toolkit.sh --help"
echo "./system_toolkit.sh --system"
echo "./system_toolkit.sh --report"

# Demonstrate the script
echo -e "\nDemonstrating system information:"
./system_toolkit.sh --system
```

---

## 🔧 Module 6: DevOps Applications

### Automated Backup Script

```bash
# Navigate to DevOps projects
cd ~/linux-devops-lab/exercises/module6-devops
mkdir automation-scripts && cd automation-scripts

# Create professional backup script
cat > backup_automation.sh << 'EOF'
#!/bin/bash
# Automated Backup Script for DevOps
# Purpose: Demonstrate backup automation with rotation and monitoring

# Configuration
BACKUP_SOURCE="$HOME/linux-devops-lab"
BACKUP_DEST="$HOME/backups"
BACKUP_NAME="devops-lab-backup"
RETENTION_DAYS=7
LOG_FILE="backup.log"

# Colors for output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Logging function
log() {
    local message="$1"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] $message" | tee -a "$LOG_FILE"
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1" | tee -a "$LOG_FILE"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1" | tee -a "$LOG_FILE"
}

log_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1" | tee -a "$LOG_FILE"
}

# Function to check prerequisites
check_prerequisites() {
    log "Checking prerequisites..."
    
    if [ ! -d "$BACKUP_SOURCE" ]; then
        log_error "Source directory does not exist: $BACKUP_SOURCE"
        return 1
    fi
    
    # Create backup destination if it doesn't exist
    mkdir -p "$BACKUP_DEST"
    
    # Check available disk space (need at least 100MB)
    available_space=$(df "$BACKUP_DEST" | awk 'NR==2{print $4}')
    if [ "$available_space" -lt 102400 ]; then  # 100MB in KB
        log_warning "Low disk space available: ${available_space}KB"
    fi
    
    log_success "Prerequisites check completed"
    return 0
}

# Function to create backup
create_backup() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="$BACKUP_DEST/${BACKUP_NAME}_${timestamp}.tar.gz"
    
    log "Creating backup: $(basename $backup_file)"
    
    # Create compressed backup
    tar -czf "$backup_file" -C "$(dirname $BACKUP_SOURCE)" "$(basename $BACKUP_SOURCE)" 2>/dev/null
    
    if [ $? -eq 0 ]; then
        local size=$(du -h "$backup_file" | cut -f1)
        log_success "Backup created successfully: $(basename $backup_file) ($size)"
        echo "$backup_file"
        return 0
    else
        log_error "Failed to create backup"
        return 1
    fi
}

# Function to verify backup
verify_backup() {
    local backup_file="$1"
    
    log "Verifying backup integrity..."
    
    if tar -tzf "$backup_file" >/dev/null 2>&1; then
        log_success "Backup integrity verified"
        return 0
    else
        log_error "Backup integrity check failed"
        return 1
    fi
}

# Function to cleanup old backups
cleanup_old_backups() {
    log "Cleaning up backups older than $RETENTION_DAYS days..."
    
    local deleted_count=0
    find "$BACKUP_DEST" -name "${BACKUP_NAME}_*.tar.gz" -mtime +$RETENTION_DAYS | while read old_backup; do
        if [ -f "$old_backup" ]; then
            rm "$old_backup"
            log "Deleted old backup: $(basename $old_backup)"
            ((deleted_count++))
        fi
    done
    
    local remaining=$(find "$BACKUP_DEST" -name "${BACKUP_NAME}_*.tar.gz" | wc -l)
    log_success "Cleanup completed. $remaining backup(s) remaining"
}

# Function to show backup status
show_backup_status() {
    log "Current backup status:"
    echo
    echo "=== BACKUP SUMMARY ==="
    echo "Source: $BACKUP_SOURCE"
    echo "Destination: $BACKUP_DEST"
    echo "Retention: $RETENTION_DAYS days"
    echo
    
    echo "Available backups:"
    if ls "$BACKUP_DEST"/${BACKUP_NAME}_*.tar.gz 2>/dev/null; then
        ls -lh "$BACKUP_DEST"/${BACKUP_NAME}_*.tar.gz | while read perm links owner group size date time file; do
            echo "  $(basename $file): $size (created $date $time)"
        done
    else
        echo "  No backups found"
    fi
    echo
}

# Function to restore backup
restore_backup() {
    local backup_file="$1"
    local restore_dir="$2"
    
    if [ -z "$backup_file" ] || [ ! -f "$backup_file" ]; then
        log_error "Backup file not found: $backup_file"
        return 1
    fi
    
    log "Restoring backup: $(basename $backup_file)"
    
    mkdir -p "$restore_dir"
    tar -xzf "$backup_file" -C "$restore_dir"
    
    if [ $? -eq 0 ]; then
        log_success "Backup restored to: $restore_dir"
        return 0
    else
        log_error "Failed to restore backup"
        return 1
    fi
}

# Main execution
main() {
    echo "DevOps Backup Automation Script"
    echo "==============================="
    log "Starting backup process"
    
    # Check prerequisites
    if ! check_prerequisites; then
        exit 1
    fi
    
    # Create backup
    backup_file=$(create_backup)
    if [ $? -ne 0 ]; then
        exit 1
    fi
    
    # Verify backup
    if ! verify_backup "$backup_file"; then
        exit 1
    fi
    
    # Cleanup old backups
    cleanup_old_backups
    
    # Show status
    show_backup_status
    
    log_success "Backup process completed successfully"
}

# Handle command line arguments
case "${1:-backup}" in
    backup)
        main
        ;;
    status)
        show_backup_status
        ;;
    restore)
        if [ -z "$2" ] || [ -z "$3" ]; then
            echo "Usage: $0 restore <backup_file> <restore_directory>"
            exit 1
        fi
        restore_backup "$2" "$3"
        ;;
    *)
        echo "Usage: $0 [backup|status|restore <file> <dir>]"
        exit 1
        ;;
esac
EOF

chmod +x backup_automation.sh

# Test the backup script
echo "Testing backup automation script:"
./backup_automation.sh
echo -e "\nCheck backup status:"
./backup_automation.sh status
```

### Log Analysis Script

```bash
# Create log analysis script
cat > log_analyzer.sh << 'EOF'
#!/bin/bash
# Log Analysis Script for DevOps
# Purpose: Analyze system logs and generate insights

# Configuration
LOG_FILES=("/var/log/syslog" "/var/log/auth.log" "backup.log" "system_toolkit.log")
OUTPUT_DIR="log_analysis_$(date +%Y%m%d_%H%M%S)"
REPORT_FILE="$OUTPUT_DIR/analysis_report.txt"

# Create output directory
mkdir -p "$OUTPUT_DIR"

# Function to analyze log file
analyze_log() {
    local log_file="$1"
    local output_file="$OUTPUT_DIR/$(basename $log_file)_analysis.txt"
    
    if [ ! -f "$log_file" ]; then
        echo "Log file not found: $log_file" >> "$REPORT_FILE"
        return 1
    fi
    
    echo "Analyzing: $log_file" >> "$REPORT_FILE"
    echo "=========================" >> "$REPORT_FILE"
    
    # Basic statistics
    local total_lines=$(wc -l < "$log_file")
    local error_lines=$(grep -i "error" "$log_file" | wc -l)
    local warning_lines=$(grep -i "warn" "$log_file" | wc -l)
    
    echo "Total lines: $total_lines" >> "$REPORT_FILE"
    echo "Error lines: $error_lines" >> "$REPORT_FILE"
    echo "Warning lines: $warning_lines" >> "$REPORT_FILE"
    
    # Top error patterns
    if [ $error_lines -gt 0 ]; then
        echo -e "\nTop error patterns:" >> "$REPORT_FILE"
        grep -i "error" "$log_file" | cut -d' ' -f4- | sort | uniq -c | sort -nr | head -5 >> "$REPORT_FILE"
    fi
    
    echo -e "\n" >> "$REPORT_FILE"
}

# Main analysis
echo "DevOps Log Analysis Report" > "$REPORT_FILE"
echo "Generated on: $(date)" >> "$REPORT_FILE"
echo "=========================================" >> "$REPORT_FILE"
echo >> "$REPORT_FILE"

for log_file in "${LOG_FILES[@]}"; do
    analyze_log "$log_file"
done

echo "Log analysis completed!"
echo "Report saved to: $REPORT_FILE"
cat "$REPORT_FILE"
EOF

chmod +x log_analyzer.sh
echo -e "\nRunning log analysis:"
./log_analyzer.sh
```

---

## 🧪 Module 7: Lab Exercises and Projects

### Lab Exercise 1: Command Line Mastery

```bash
# Navigate to final projects
cd ~/linux-devops-lab/exercises/module7-projects
mkdir command-mastery && cd command-mastery

echo "=== COMMAND LINE MASTERY CHALLENGE ==="

# Create test environment
mkdir -p {logs,configs,scripts,data}

# Generate sample data
seq 1 1000 > data/numbers.txt
echo -e "apple\nbanana\ncherry\napple\ndate\nbanana\nfig" > data/fruits.txt

# Create sample log entries
for i in {1..50}; do
    level=$(shuf -n1 -e "INFO" "WARN" "ERROR")
    echo "$(date +%Y-%m-%d\ %H:%M:%S) [$level] Sample log message $i"
done > logs/application.log

# Challenge tasks
echo "Challenge 1: Find all ERROR messages in logs"
grep "ERROR" logs/application.log | wc -l

echo -e "\nChallenge 2: Count unique fruits"
sort data/fruits.txt | uniq | wc -l

echo -e "\nChallenge 3: Sum all numbers divisible by 5"
cat data/numbers.txt | awk '$1 % 5 == 0 {sum += $1} END {print "Sum:", sum}'

echo -e "\nChallenge 4: Create backup with timestamp"
tar -czf "backup_$(date +%Y%m%d_%H%M%S).tar.gz" data/ logs/

echo -e "\nChallenge 5: Top 5 largest files in current directory"
find . -type f -exec ls -la {} \; | sort -k5 -nr | head -5

echo "Command mastery challenge completed!"
```

### Lab Exercise 2: Shell Scripting Project

```bash
# Create final project - System Health Monitor
cat > system_health_monitor.sh << 'EOF'
#!/bin/bash
# Final Project: Comprehensive System Health Monitor
# Student: [Your Name]
# Course: Linux & Shell Scripting for DevOps

# Configuration
HEALTH_CHECK_INTERVAL=5
MAX_CPU_USAGE=80
MAX_MEMORY_USAGE=85
MAX_DISK_USAGE=90
ALERT_LOG="health_alerts.log"
REPORT_DIR="health_reports"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Initialize
mkdir -p "$REPORT_DIR"

# Functions
log_alert() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ALERT: $1" | tee -a "$ALERT_LOG"
}

check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print int(100 - $1)}')
    
    if [ "$cpu_usage" -gt "$MAX_CPU_USAGE" ]; then
        echo -e "${RED}CPU: ${cpu_usage}%${NC} (HIGH)"
        log_alert "High CPU usage: ${cpu_usage}%"
        return 1
    else
        echo -e "${GREEN}CPU: ${cpu_usage}%${NC} (OK)"
        return 0
    fi
}

check_memory() {
    local mem_usage=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100.0}')
    
    if [ "$mem_usage" -gt "$MAX_MEMORY_USAGE" ]; then
        echo -e "${RED}Memory: ${mem_usage}%${NC} (HIGH)"
        log_alert "High memory usage: ${mem_usage}%"
        return 1
    else
        echo -e "${GREEN}Memory: ${mem_usage}%${NC} (OK)"
        return 0
    fi
}

check_disk() {
    local disk_usage=$(df / | awk 'NR==2{print $5}' | sed 's/%//')
    
    if [ "$disk_usage" -gt "$MAX_DISK_USAGE" ]; then
        echo -e "${RED}Disk: ${disk_usage}%${NC} (HIGH)"
        log_alert "High disk usage: ${disk_usage}%"
        return 1
    else
        echo -e "${GREEN}Disk: ${disk_usage}%${NC} (OK)"
        return 0
    fi
}

generate_report() {
    local report_file="$REPORT_DIR/health_report_$(date +%Y%m%d_%H%M%S).txt"
    
    {
        echo "SYSTEM HEALTH REPORT"
        echo "===================="
        echo "Generated: $(date)"
        echo "Hostname: $(hostname)"
        echo
        echo "Resource Usage:"
        echo "- CPU: $(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print int(100 - $1)}'')%"
        echo "- Memory: $(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100.0}')%"
        echo "- Disk: $(df / | awk 'NR==2{print $5}')"
        echo
        echo "System Information:"
        echo "- Uptime: $(uptime -p 2>/dev/null || uptime)"
        echo "- Load Average: $(uptime | awk -F'load average:' '{print $2}')"
        echo "- Processes: $(ps aux | wc -l) running"
        echo
        echo "Recent Alerts:"
        tail -5 "$ALERT_LOG" 2>/dev/null || echo "No recent alerts"
    } > "$report_file"
    
    echo "Report saved: $report_file"
}

# Main monitoring loop
main() {
    echo "System Health Monitor Starting..."
    echo "Press Ctrl+C to stop monitoring"
    echo
    
    while true; do
        clear
        echo -e "${BLUE}=== SYSTEM HEALTH MONITOR ===${NC}"
        echo "Time: $(date)"
        echo "Uptime: $(uptime -p 2>/dev/null || uptime | cut -d',' -f1)"
        echo
        
        # Check all metrics
        local issues=0
        check_cpu || ((issues++))
        check_memory || ((issues++))
        check_disk || ((issues++))
        
        echo
        if [ $issues -eq 0 ]; then
            echo -e "${GREEN}Overall Status: HEALTHY${NC}"
        else
            echo -e "${YELLOW}Overall Status: $issues ISSUE(S) DETECTED${NC}"
        fi
        
        echo
        echo "Next check in $HEALTH_CHECK_INTERVAL seconds..."
        sleep $HEALTH_CHECK_INTERVAL
    done
}

# Handle command line options
case "${1:-monitor}" in
    monitor)
        main
        ;;
    report)
        generate_report
        ;;
    alerts)
        echo "Recent alerts:"
        tail -10 "$ALERT_LOG" 2>/dev/null || echo "No alerts found"
        ;;
    *)
        echo "Usage: $0 [monitor|report|alerts]"
        ;;
esac
EOF

chmod +x system_health_monitor.sh

echo "Final project created! System Health Monitor"
echo "Usage:"
echo "  ./system_health_monitor.sh         # Start monitoring"
echo "  ./system_health_monitor.sh report  # Generate report"
echo "  ./system_health_monitor.sh alerts  # View alerts"
```

---

## 🎯 Course Summary and Achievements

### What You've Accomplished

Congratulations! You have successfully completed the **Linux & Shell Scripting for DevOps** course using online environments. You have:

✅ **Mastered Linux Fundamentals**
- File system navigation and hierarchy
- Essential commands for daily operations
- File permissions and security concepts

✅ **Developed Shell Scripting Skills**
- Variables, loops, and conditional statements
- Functions and error handling
- Advanced text processing techniques

✅ **Built DevOps Automation Tools**
- System monitoring scripts
- Automated backup solutions
- Log analysis and management tools

✅ **Created Real-World Projects**
- System health monitoring application
- Professional backup automation
- Log analysis and reporting tools

### Skills Acquired

**Technical Skills:**
- Linux command line proficiency
- Shell scripting and automation
- System monitoring and maintenance
- DevOps tooling and practices

**Professional Skills:**
- Problem-solving and troubleshooting
- Documentation and code organization
- Automation thinking and design
- System administration concepts

### Your Learning Portfolio

All your work is organized in: `~/linux-devops-lab/`

```bash
# Review your complete learning portfolio
cd ~/linux-devops-lab
find . -name "*.sh" -type f | wc -l  # Count your scripts
find . -name "*.txt" -type f | wc -l # Count your documentation
du -sh .                             # Total size of your work
```

### Next Steps and Career Path

1. **Version Control with Git**
   - Learn Git fundamentals
   - Practice with GitHub repositories
   - Implement GitOps workflows

2. **Containerization Technologies**
   - Docker fundamentals and containers
   - Docker Compose for multi-container apps
   - Container orchestration basics

3. **Infrastructure as Code**
   - Ansible for configuration management
   - Terraform for infrastructure provisioning
   - Infrastructure automation practices

4. **Cloud Platforms**
   - AWS/Azure/GCP fundamentals
   - Cloud-native DevOps tools
   - Serverless and microservices

5. **CI/CD Pipelines**
   - Jenkins automation
   - GitLab CI or GitHub Actions
   - Deployment pipeline design

### Continue Learning Resources

**Online Practice Platforms:**
- LabEx: https://labex.io (Continue with advanced Linux labs)
- KodeKloud: https://kodekloud.com (DevOps engineer path)
- Linux Academy: Advanced system administration

**Communities to Join:**
- r/devops (Reddit community)
- DevOps.com forums
- Linux User Groups (LUG)
- Local DevOps meetups

**Certification Paths:**
- CompTIA Linux+ 
- Red Hat Certified System Administrator (RHCSA)
- AWS Certified DevOps Engineer
- Kubernetes certifications (CKA/CKAD)

### Final Challenge

Create your own GitHub repository with all your course work:

```bash
# Prepare your portfolio for GitHub
cd ~/linux-devops-lab

# Create a comprehensive README
cat > README.md << 'EOF'
# My DevOps Learning Journey

This repository contains my work from the "Linux & Shell Scripting for DevOps" course.

## What I've Learned
- Linux system administration
- Shell scripting and automation  
- DevOps tools and practices
- System monitoring and maintenance

## Projects Included
- System Health Monitor
- Automated Backup Solutions
- Log Analysis Tools
- Various automation scripts

## Next Steps
- Advancing to containerization (Docker/Kubernetes)
- Learning infrastructure as code (Terraform/Ansible)
- Building CI/CD pipelines
EOF

echo "Your DevOps learning journey is complete!"
echo "Portfolio ready for GitHub at: $(pwd)"
```

---

## 🏆 Congratulations!

You have successfully completed the **Linux & Shell Scripting for DevOps** course using online environments!

**Course Statistics:**
- **Modules Completed:** 7/7 ✅
- **Scripts Created:** 15+ automation scripts
- **Lab Exercises:** 10+ hands-on challenges  
- **Learning Hours:** 40+ intensive hours
- **Environment:** Online Linux playground

**You are now equipped with:**
- Solid Linux foundation for DevOps
- Shell scripting automation skills
- System administration knowledge
- Real-world project experience
- Professional development practices

**Keep Learning, Keep Building, Keep Automating!** 🚀

---

*Course completed using online Linux environments - No local installation required!*