Here’s how you can implement a similar concept of Section objects and memory-mapped files using Python on macOS. We will use the `mmap` module to map files into memory and perform operations like reading and writing to the mapped memory.

### Python Example for Memory-Mapped Files on macOS

```python
import mmap
import os

# Create a file to map into memory
filename = "/tmp/testfile"
with open(filename, "wb") as f:
    f.write(b"\x00" * 4096)  # Create a 4KB file

# Open the file and memory-map it
with open(filename, "r+b") as f:
    # Memory-map the file, equivalent to creating a Section object and mapping it
    mm = mmap.mmap(f.fileno(), 4096, access=mmap.ACCESS_WRITE)

    # 1. Write data to the memory (similar to Write-NtVirtualMemory)
    mm.write(b"Hello world!")

    # 2. Read data from the memory (similar to Read-NtVirtualMemory)
    mm.seek(0)
    print(f"Memory Content: {mm.read(12)}")  # Output: Hello world!

    # 3. Change protection (you can't change protection directly in Python, but you can remap)
    mm.close()  # Unmap the memory (similar to Remove-NtSection)

# Reopen with read-only access to simulate changing protection
with open(filename, "r+b") as f:
    mm = mmap.mmap(f.fileno(), 4096, access=mmap.ACCESS_READ)

    # Read content from the read-only memory
    mm.seek(0)
    print(f"Read-Only Memory Content: {mm.read(12)}")  # Output: Hello world!

    mm.close()
```

### Explanation:
1. **File-backed memory mapping**: We map a file into memory, which mimics the Section object in Windows where the file contents are loaded into memory.
2. **Memory operations**: We perform reading and writing operations to the mapped memory, similar to `Read-NtVirtualMemory` and `Write-NtVirtualMemory`.
3. **Changing access**: Although Python doesn’t have direct memory protection change functions like `Set-NtVirtualMemory`, closing and reopening the memory-mapped file with different access modes (`ACCESS_READ` instead of `ACCESS_WRITE`) simulates changing the memory protection.

### Reference:
Forshaw, James. *Windows Security Internals*. Apple Books, https://books.apple.com/us/book/windows-security-internals/id6455259366. [Insert Heading], p. [Insert Page Number].

This example demonstrates how to work with memory-mapped files in Python on macOS, simulating the behavior of Section objects and related operations discussed in the excerpt.