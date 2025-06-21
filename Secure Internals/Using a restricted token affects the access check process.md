On macOS, we do not have a direct equivalent of **restricted tokens** and **lowbox tokens** like in Windows, but we can simulate a restricted environment by imposing limitations based on file permissions, sandboxing, or even restricting groups and roles that the user belongs to. We can mimic some of the functionality of **restricted tokens** by using macOS security features such as user and group permissions, sandboxing, or checking the membership of the user in specific restricted groups.

In the context of Python, we can simulate this behavior by adding additional checks to ensure that access is only granted if the user belongs to certain groups or has certain permissions. We can extend the existing **discretionary access check** that we previously discussed to introduce a **restricted access** check, similar to the **restricted token** concept in Windows.

### Step 1: Modify the Owner Access Check to Simulate Restricted Access

We can simulate the **restricted owner access** check by adding additional checks for user group membership. This can be done by ensuring that the user belongs to both the allowed group and a restricted group before granting access.

Here’s how we can modify the **owner access check** to simulate restricted access:

```python
import os
import pwd
import grp

class RestrictedAccessError(Exception):
    pass

def resolve_token_owner_access(context, restricted_group=None):
    """
    Resolve token owner access by checking if the current user owns the resource
    and if the user belongs to a restricted group.
    """
    file_stat = os.stat(context['filepath'])
    file_owner_uid = file_stat.st_uid

    # Get the current user's UID and group memberships
    current_user_uid = os.getuid()
    current_user_groups = [g.gr_gid for g in grp.getgrall() if current_user_uid in g.gr_mem]
    current_user_name = pwd.getpwuid(current_user_uid).pw_name

    # Check if the current user is the owner of the resource
    if current_user_uid == file_owner_uid:
        print(f"User owns the resource {context['filepath']}, granting ReadControl and WriteDac.")

        # If restricted_group is specified, check if the user is in the restricted group
        if restricted_group and restricted_group not in current_user_groups:
            raise RestrictedAccessError(f"Access denied. User is not part of the restricted group: {restricted_group}")
        
        # Grant access
        context['remaining_access'].discard('ReadControl')
        context['remaining_access'].discard('WriteDac')
        return True
    else:
        file_owner_name = pwd.getpwuid(file_owner_uid).pw_name
        print(f"User does not own the resource {context['filepath']}. Resource owned by {file_owner_name}.")
        return False
```

### Explanation:
- **Owner Check**: We first check whether the current user is the owner of the resource. If so, they are granted **ReadControl** and **WriteDac** access.
- **Restricted Group Check**: If a `restricted_group` is provided, we check if the user is a member of that group. If the user is not part of the restricted group, we raise a `RestrictedAccessError`, denying access.

### Step 2: Modify the Discretionary Access Check

We can now extend the **discretionary access check** to simulate the restricted SID behavior by adding another layer of restrictions based on group membership. Here’s how we can modify the discretionary access check to support restricted access:

```python
def get_discretionary_access(filepath, access_type, restricted_group=None):
    """
    Simulate the Discretionary Access Check by comparing file permissions,
    ownership, and group membership with optional restricted group check.
    """
    file_stat = os.stat(filepath)
    file_mode = file_stat.st_mode
    file_owner_uid = file_stat.st_uid
    file_group_gid = file_stat.st_gid

    # Get the current user's UID and GID
    current_user_uid = os.getuid()
    current_user_gid = os.getgid()

    # Get current user's groups
    current_user_groups = [g.gr_gid for g in grp.getgrall() if current_user_uid in g.gr_mem]

    # Determine the access the current user should have based on permissions
    permissions = {
        'read': False,
        'write': False,
        'execute': False
    }

    # Check if the current user is the owner of the file
    if current_user_uid == file_owner_uid:
        if file_mode & stat.S_IRUSR:  # Owner read
            permissions['read'] = True
        if file_mode & stat.S_IWUSR:  # Owner write
            permissions['write'] = True
        if file_mode & stat.S_IXUSR:  # Owner execute
            permissions['execute'] = True
    elif current_user_gid == file_group_gid or current_user_gid in current_user_groups:
        # Check if the user is part of the group that owns the file
        if file_mode & stat.S_IRGRP:  # Group read
            permissions['read'] = True
        if file_mode & stat.S_IWGRP:  # Group write
            permissions['write'] = True
        if file_mode & stat.S_IXGRP:  # Group execute
            permissions['execute'] = True
    else:
        # Check permissions for others
        if file_mode & stat.S_IROTH:  # Others read
            permissions['read'] = True
        if file_mode & stat.S_IWOTH:  # Others write
            permissions['write'] = True
        if file_mode & stat.S_IXOTH:  # Others execute
            permissions['execute'] = True

    # If restricted_group is specified, check if the user is in the restricted group
    if restricted_group and restricted_group not in current_user_groups:
        raise RestrictedAccessError(f"Access denied. User is not part of the restricted group: {restricted_group}")

    # Determine whether access is granted based on the requested access type
    access_granted = True
    for access in access_type:
        if not permissions.get(access, False):
            access_granted = False
            break

    return access_granted, permissions
```

### Step 3: Test the Modified Discretionary Access Check

Now, we can test the **discretionary access check** with a restricted group to see if access is correctly granted or denied based on both permissions and group membership.

```python
def test_discretionary_access(filepath, access_type, restricted_group=None):
    try:
        access_granted, permissions = get_discretionary_access(filepath, access_type, restricted_group)
        print(f"Requested access: {access_type}")
        print(f"Permissions: {permissions}")
        if access_granted:
            print(f"Access granted for {access_type} on {filepath}")
        else:
            print(f"Access denied for {access_type} on {filepath}")
    except RestrictedAccessError as e:
        print(e)

if __name__ == "__main__":
    filepath = '/path/to/your/resource'  # Replace with your file path
    access_type = ['read', 'write']  # Example: the access types to check
    restricted_group = 'restricted_group_name'  # Replace with the restricted group name
    test_discretionary_access(filepath, access_type, restricted_group)
```

### Example Output:

If the current user belongs to the restricted group:

```
Requested access: ['read', 'write']
Permissions: {'read': True, 'write': True, 'execute': False}
Access granted for ['read', 'write'] on /path/to/your/resource
```

If the current user does not belong to the restricted group:

```
Access denied. User is not part of the restricted group: restricted_group_name
```

### Conclusion:

This Python implementation simulates the **restricted token** behavior by adding an additional check for restricted groups. The access check now depends not only on file permissions but also on the user’s membership in a restricted group, similar to the behavior of **restricted tokens** in Windows. You can extend this to incorporate more complex access control checks or integrate with macOS's security and sandboxing features.

Would you like to explore more advanced sandboxing or privilege restriction techniques on macOS?