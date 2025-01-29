# Wine Process Manager

A bash script that finds and terminates lingering Wine processes. 

Wine doesn't always clean up after itself - processes can remain after closing programs, crashes, or other scenarios. 
This script handles cleanup for any Wine containers including Lutris, Steam (Proton), and Heroic Launcher, using graceful termination with fallback mechanisms.

## Features

- Intelligent process detection and termination
- Support for specific Wine prefix targeting
- Colored output for better visibility
- Graceful shutdown with escalating force levels
- Detailed process information display
- Force kill option with `-9` flag

## Prerequisites

- Bash shell
- Wine installed
- Basic Unix utilities (ps, grep, perl)

## Installation

1. Download the script to your preferred location
2. Make it executable:
   ```bash
   chmod +x winestop
   ```

## Usage

### Basic Usage

```bash
./winestop
```

This will search for and terminate all Wine processes for the default Wine prefix.

### With Specific Wine Prefix

```bash
./wine-process-manager.sh prefix_name
```

Replace `prefix_name` with the name of your Wine prefix directory (relative to `~/.wine/`).

### Force Kill

```bash
./wine-process-manager.sh -9
```

Add the `-9` flag to immediately use SIGKILL instead of attempting graceful termination.

## How It Works

1. First attempts to terminate the wineserver gracefully
2. Searches for Wine-related processes:
   - wine-preloader
   - wine64-preloader
   - wineserver
   - wine
   - .exe processes
3. Displays found processes with their PIDs and names
4. Attempts termination in stages:
   - Initial SIGTERM signal
   - Multiple retry attempts with SIGKILL if needed
   - Final verification of process termination

## Output Colors

- 🔵 Blue: Information messages
- 🟢 Green: Success messages
- 🟡 Yellow: Warning messages
- 🔴 Red: Error messages

## Error Handling

- Validates Wine prefix existence
- Verifies process existence before termination
- Provides feedback on termination success/failure
- Reports remaining processes if termination fails

## Examples

### Terminate All Wine Processes
```bash
./wine-process-manager.sh
```

### Terminate Processes in Specific Prefix
```bash
./wine-process-manager.sh gaming_prefix
```

### Force Kill All Wine Processes
```bash
./wine-process-manager.sh -9
```

## Troubleshooting

If processes persist after running the script:

1. Try running with the `-9` flag
2. Check for root-owned Wine processes
3. As a last resort, system reboot may be required

## Notes

- The script uses `~/.wine` as the default Wine cellar location
- Process termination is attempted gracefully before force killing
- Multiple retry attempts are made before giving up
- System reboot recommendation is provided if processes cannot be terminated

