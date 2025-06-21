On macOS, modifying a mapped memory section or file can be achieved using the `mmap` module in Python. While macOS doesn't have direct equivalents to PowerShell commands like `Add-NtSection` or `Write-NtVirtualMemory`, the concept of memory-mapped files can still be applied.

Here’s an example of how to map a file into memory, modify its contents, and display the memory contents similar to how you would use `Write-NtVirtualMemory` and `Out-HexDump` in Windows.

### Python Example for Mapping and Modifying Memory (Hex Dump Style)

```python
import mmap
import os
import random

# Helper function to display memory as a hex dump
def hex_dump(buffer, length):
    print("Address                 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F")
    print("---------------------------------------------------------------------")
    for i in range(0, length, 16):
        # Print the address and 16 bytes of memory in hex
        hex_values = ' '.join(f"{x:02X}" for x in buffer[i:i + 16])
        address = hex(i).rjust(22)
        print(f"{address}: {hex_values}")

# Create a temporary file and map it into memory
file_path = "/tmp/mappedfile"
with open(file_path, "wb") as f:
    f.write(b"\x00" * 4096)  # Create a 4KB file with zeroed content

# Open the file and memory-map it with read-write access
with open(file_path, "r+b") as f:
    mm = mmap.mmap(f.fileno(), 4096, access=mmap.ACCESS_WRITE)

    # Modify the mapped memory with random bytes (similar to fuzzing)
    random_bytes = bytearray(random.getrandbits(8) for _ in range(4096))
    mm.seek(0)
    mm.write(random_bytes)

    # Display the modified memory in hex dump format
    hex_dump(random_bytes, 16)

    # Close the memory map
    mm.close()

# Optionally, clean up by deleting the file
os.remove(file_path)
```

### Explanation:
1. **Memory Mapping**: The `mmap` module maps the file into memory, allowing it to be modified as if it were loaded directly into memory.
2. **Random Byte Modification**: We generate a random byte array (similar to fuzzing) and write it to the memory-mapped file.
3. **Hex Dump**: The `hex_dump` function prints the memory contents in a format similar to `Out-HexDump` in PowerShell, showing both the address and the byte values in hexadecimal.

### Equivalent Functionality:
- **Mapping a File**: The `mmap` module serves as the equivalent to `Add-NtSection`, mapping the file into memory.
- **Writing to Memory**: The `write` method of `mmap` is analogous to `Write-NtVirtualMemory`, allowing modification of the memory-mapped file.
- **Hex Dump**: The custom `hex_dump` function mimics `Out-HexDump` to display the contents of the mapped memory region.

### Reference:
Forshaw, James. *Windows Security Internals*. Apple Books, https://books.apple.com/us/book/windows-security-internals/id6455259366. [Insert Heading], p. [Insert Page Number].

This Python script simulates the behavior of mapping and modifying a section of memory, adapted for macOS, using `mmap` and a custom hex dump function to provide similar functionality to PowerShell commands used in Windows.