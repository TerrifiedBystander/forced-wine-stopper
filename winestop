#!/bin/bash

# Define colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Function to print info messages
info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

# Function to print success messages
success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

# Function to print warning messages
warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1" >&2
}

# Function to print error messages
error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

# Function to check if a process is actually running
is_process_running() {
    ps -p $1 > /dev/null 2>&1
    return $?
}

# Function to get live process list from original PIDs
get_live_pids() {
    local pid_list=()
    for pid in $1; do
        if is_process_running $pid; then
            pid_list+=($pid)
        fi
    done
    echo "${pid_list[@]}"
}

# Function to terminate processes with increasing force
terminate_processes() {
    local original_pids=$1
    local force_kill=$2
    local retry_count=0
    local max_retries=3
    
    if [ "$force_kill" = true ]; then
        warning "Force killing processes with SIGKILL (-9)..."
        for pid in $original_pids; do
            if is_process_running $pid; then
                kill -9 $pid 2>/dev/null
            fi
        done
    else
        # First try SIGTERM
        info "Attempting graceful termination with SIGTERM..."
        for pid in $original_pids; do
            if is_process_running $pid; then
                kill $pid 2>/dev/null
            fi
        done
        sleep 1
        
        # Check if processes still exist and retry with increasing force
        while [ $retry_count -lt $max_retries ]; do
            local live_pids=$(get_live_pids "$original_pids")
            
            if [ -n "$live_pids" ]; then
                retry_count=$((retry_count + 1))
                warning "Some processes still running, attempting stronger termination (attempt $retry_count of $max_retries)..."
                for pid in $live_pids; do
                    kill -9 $pid 2>/dev/null
                done
                sleep 1
            else
                break
            fi
        done
    fi
}

wine_cellar="${HOME}/.wine"
info "Starting Wine process management..."
info "Using Wine cellar: ${wine_cellar}"

if (($#)); then
    if [[ -e "${wine_cellar}/$1" ]]; then
        WINEPREFIX="${wine_cellar}/$1"
        info "Using Wine prefix: ${WINEPREFIX}"
        shift
    elif [[ "${1:0:1}" != "-" ]]; then
        error "Didn't understand argument '$1'"
        exit 1
    fi
fi

# First, try to kill wineserver which should handle cleanup
info "Attempting to terminate wineserver first..."
wineserver -k
sleep 1

# Then look for remaining processes
if ((${#WINEPREFIX})); then
    info "Searching for Wine processes in prefix ${WINEPREFIX}..."
    pids=$(
        grep -l "WINEPREFIX=${WINEPREFIX}$" $(
            ls -l /proc/*/exe 2>/dev/null |
            grep -E 'wine(64)?-preloader|wineserver|wine|\.exe' |
            perl -pe 's;^.*/proc/(\d+)/exe.*$;/proc/$1/environ;g;'
        ) 2> /dev/null |
        perl -pe 's;^/proc/(\d+)/environ.*$;$1;g;'
    )
else
    info "Searching for all Wine processes..."
    pids=$(ls -l /proc/*/exe 2>/dev/null |
        grep -E 'wine(64)?-preloader|wineserver|wine|\.exe' |
        perl -pe 's;^.*/proc/(\d+)/exe.*$;$1;g;'
    )
fi

if ((${#pids})); then
    # Get initial count of actually running processes
    live_pids=$(get_live_pids "$pids")
    pid_count=$(echo $live_pids | wc -w)
    
    if [ $pid_count -gt 0 ]; then
        warning "Found ${pid_count} Wine process(es) to terminate"
        
        # Display process details before killing
        echo -e "\n${YELLOW}Processes to be terminated:${NC}"
        for pid in $live_pids; do
            if is_process_running $pid; then
                command=$(ps -p $pid -o comm= 2>/dev/null)
                echo -e "${YELLOW}PID: ${pid}${NC} - ${command}"
            fi
        done
        
        echo # Empty line for readability
        
        # Check if -9 was specified
        force_kill=false
        if [[ "$*" == *"-9"* ]]; then
            force_kill=true
        fi
        
        # Attempt to terminate processes
        terminate_processes "$live_pids" "$force_kill"
        
        # Final check
        final_pids=$(get_live_pids "$pids")
        if [ -n "$final_pids" ]; then
            error "Some processes could not be terminated. You may need to reboot your system."
            warning "Remaining processes:"
            for pid in $final_pids; do
                command=$(ps -p $pid -o comm= 2>/dev/null)
                echo -e "${RED}PID: ${pid}${NC} - ${command}"
            done
        else
            success "All Wine processes have been terminated successfully"
        fi
    else
        info "No running Wine processes found"
    fi
else
    info "No Wine processes found"
fi
