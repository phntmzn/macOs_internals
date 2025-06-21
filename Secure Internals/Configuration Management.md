On macOS, there is no direct equivalent to the Windows registry system. However, macOS uses property lists (plist files), environment variables, and other configuration files for system and application settings. The most common way to interact with these is through `defaults`, `plist` files, or other file-based configurations.

Below is an example of how you can access and modify system or user preferences using the **`defaults`** command in Python, which is a rough equivalent to interacting with the Windows registry.

### Python Example for Configuration Management on macOS

You can interact with macOS settings using the `defaults` command, which allows you to read, write, and delete values from plist files where macOS stores configuration data.

```python
import subprocess

def read_defaults(domain, key):
    try:
        result = subprocess.run(['defaults', 'read', domain, key], 
                                stdout=subprocess.PIPE, text=True)
        print(f"Value of '{key}' in domain '{domain}': {result.stdout.strip()}")
    except Exception as e:
        print(f"Error reading defaults: {e}")

def write_defaults(domain, key, value):
    try:
        subprocess.run(['defaults', 'write', domain, key, value])
        print(f"Set '{key}' to '{value}' in domain '{domain}'")
    except Exception as e:
        print(f"Error writing defaults: {e}")

def delete_defaults(domain, key):
    try:
        subprocess.run(['defaults', 'delete', domain, key])
        print(f"Deleted '{key}' from domain '{domain}'")
    except Exception as e:
        print(f"Error deleting defaults: {e}")

# Example usage
domain = "com.apple.finder"
key = "AppleShowAllFiles"

# Read a value from the Finder's configuration
read_defaults(domain, key)

# Set a value in the Finder's configuration
write_defaults(domain, key, "TRUE")

# Delete a key from the Finder's configuration
delete_defaults(domain, key)
```

### Explanation:
1. **`defaults read`**: Reads the value of a key from the specified domain (analogous to querying a registry key in Windows).
2. **`defaults write`**: Writes a value to a specified key (similar to setting a registry key value).
3. **`defaults delete`**: Deletes a key from the specified domain (similar to removing a key or value from the registry).

### Equivalent Functionality:
- **Windows Registry Key**: In macOS, the closest analogy would be reading from or writing to property lists or application preferences using `defaults`.
- **Registry Values**: These are analogous to keys and values in plist files or the user preferences domains accessible by `defaults`.

### Reference:
Forshaw, James. *Windows Security Internals*. Apple Books, https://books.apple.com/us/book/windows-security-internals/id6455259366. [Insert Heading], p. [Insert Page Number].

This Python script uses macOS's `defaults` command to perform configuration management tasks similar to the Windows registry, providing a basic interface for reading, writing, and deleting preference values.