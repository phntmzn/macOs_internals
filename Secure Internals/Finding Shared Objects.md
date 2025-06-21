On macOS, there is no direct equivalent to the `Get-NtHandle` command from PowerShell. However, you can achieve similar functionality using system commands and Python to search for open file descriptors and matching filenames. You can use `lsof` (list open files) in macOS, which lists information about files that are open by processes, analogous to querying open handles in Windows.

### Python Example for Finding Open Files by Name on macOS

```python
import subprocess

# Function to find open files matching a specific name
def find_open_files_by_name(name_filter):
    try:
        # Run the lsof command to list all open files
        result = subprocess.run(['lsof'], stdout=subprocess.PIPE, text=True)

        # Filter lines that match the name filter (e.g., "Windows" in the original example)
        matching_lines = [line for line in result.stdout.splitlines() if name_filter in line]

        # Print matching lines with relevant information
        for line in matching_lines:
            columns = line.split()
            process_id = columns[1]
            file_descriptor = columns[3]
            file_name = columns[-1]
            print(f"Process ID: {process_id}, File Descriptor: {file_descriptor}, File: {file_name}")

    except Exception as e:
        print(f"Error finding open files: {e}")

# Example usage: find open files containing "System" in their file path
find_open_files_by_name("System")
```

### Explanation:
1. **`lsof` Command**: This command lists open files and their associated processes in macOS. It can show open files, sockets, directories, and more. We filter the output to find files matching a specific name.
2. **Filtering by Name**: Similar to the Windows PowerShell example, this script filters the output to match filenames that contain a specific string (e.g., "System").
3. **Output**: The process ID, file descriptor, and file path are extracted and printed, similar to how handles and file paths are shown in the original Windows example.

### Equivalent Functionality:
- **`lsof`**: This command acts as a rough equivalent to `Get-NtHandle` by listing all open files and the processes that have them open.
- **Filtering**: Similar to PowerShell's `Where-Object Name -Match`, we use Python to filter out lines containing the desired file name.

### Reference:
Forshaw, James. *Windows Security Internals*. Apple Books, https://books.apple.com/us/book/windows-security-internals/id6455259366. [Insert Heading], p. [Insert Page Number].

This Python script uses macOS's `lsof` command to list and filter open files by name, similar to how you would query file object handles in PowerShell on Windows, providing an efficient way to identify open files matching a specific name.