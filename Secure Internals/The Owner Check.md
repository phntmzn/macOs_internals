To implement the **Owner Check** on macOS using Python, we can simulate this behavior by checking if the current user is the owner of the resource (file or directory). If the user is the owner, we can grant them **ReadControl** and **WriteDac** access. These permissions allow the user to read the security attributes and modify the **Discretionary Access Control List (DACL)**, even if the DACL doesn't explicitly grant them any other access.

### Step 1: Owner Check Implementation

The **Owner Check** will compare the **owner** of the resource with the current user's effective user ID. If the user owns the resource, they will be granted **ReadControl** and **WriteDac** access.

Here is how we can implement the **Owner Check** in Python:

```python
import os
import pwd

def resolve_token_owner_access(context):
    """
    Resolve token owner access by checking if the current user owns the resource.
    If the user owns the resource, grant ReadControl and WriteDac access.
    """
    file_stat = os.stat(context['filepath'])
    file_owner_uid = file_stat.st_uid

    # Get the current user's UID
    current_user_uid = os.getuid()

    # Check if the current user is the owner of the resource
    if current_user_uid == file_owner_uid:
        print(f"User owns the resource {context['filepath']}, granting ReadControl and WriteDac.")
        context['remaining_access'].discard('ReadControl')
        context['remaining_access'].discard('WriteDac')
        return True
    else:
        file_owner_name = pwd.getpwuid(file_owner_uid).pw_name
        print(f"User does not own the resource {context['filepath']}. Resource owned by {file_owner_name}.")
        return False
```

### Step 2: Token Access Context

This function will be part of the larger token access check context, where we keep track of the requested access and privileges.

### Step 3: Putting It All Together

We can now simulate a full token access check that includes the **Owner Check** and the **Privilege Check**:

```python
def test_token_access(filepath, access_type):
    # Create the context for the token access check
    context = {
        'filepath': filepath,
        'access_type': access_type,
        'remaining_access': set(access_type),  # Track the remaining access
        'privileges': []  # Track which privileges were used to grant access
    }

    # Perform the owner check first (before any privilege checks)
    if resolve_token_owner_access(context):
        print(f"Access granted for {access_type} on {filepath} (Owner Access).")
    else:
        print(f"Access denied for {access_type} on {filepath} (Not the owner).")

if __name__ == "__main__":
    filepath = '/path/to/your/resource'  # Replace with your file path
    access_type = ['ReadControl', 'WriteDac']  # Example: the access types to check
    test_token_access(filepath, access_type)
```

### Explanation:

1. **Owner Check**: The function `resolve_token_owner_access` checks if the current user owns the file or directory. If the user owns the resource, it grants **ReadControl** and **WriteDac** access by removing those permissions from the list of remaining access restrictions.
   
2. **File Ownership**: We use the `os.stat` function to retrieve the file's metadata, including the owner's UID. This is then compared to the current user's UID (`os.getuid()`).

3. **Remaining Access**: The `remaining_access` set tracks which access rights are still required. If the owner has **ReadControl** or **WriteDac**, these permissions are removed from the list of remaining required access.

### Example Output:

If the current user owns the resource:

```
User owns the resource /path/to/your/resource, granting ReadControl and WriteDac.
Access granted for ['ReadControl', 'WriteDac'] on /path/to/your/resource (Owner Access).
```

If the current user does not own the resource:

```
User does not own the resource /path/to/your/resource. Resource owned by admin.
Access denied for ['ReadControl', 'WriteDac'] on /path/to/your/resource (Not the owner).
```

### Conclusion:

This Python implementation simulates the **Owner Check** by determining if the current user owns the file or resource. If the user is the owner, they are granted **ReadControl** and **WriteDac** access, even if the DACL would normally prevent access.

Would you like to integrate this with further privilege checks or implement more detailed access control checks?