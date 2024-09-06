# Git Commit User Check Script

This script checks the current Git user by inspecting the SSH configuration file and extracting the email associated with GitHub's `IdentityFile`. The script is designed to help users verify their current SSH identity before committing changes to a Git repository.

## Features
- **SSH Config Parsing**: It looks for the `Host github.com` section in the user's `~/.ssh/config` file.
- **Email Extraction**: Extracts the email from the `IdentityFile` associated with GitHub.
- **Commit Notification**: Displays the user's GitHub email before committing, or prompts them to fix the configuration if not found.
- **Revert Suggestion**: Offers the user the option to revert their commit with `git reset --soft HEAD~1` if needed.

## Prerequisites

- The script requires a correctly configured `~/.ssh/config` file that includes the `Host github.com` section with an associated `IdentityFile`. You can use [this script](https://github.com/thecist/gitSwitch) to manage your ssh profiles

### Example `~/.ssh/config`:

```bash
Host github.com
  IdentityFile ~/.ssh/your-email@example.com/id_ed25519
  User git
```

## Usage

1. Save the script in the `.git/hooks` directory in your repository:

   ```bash
   .git/hooks/check-user
   ```

2. Make the script executable:

   ```bash
   chmod +x .git/hooks/check-user
   ```
3. Execute the script in your `precommit` file
   ```bash
   #!/bin/sh

   source "$(dirname "$0")/check-user"
   ```

3. The script will automatically run before every commit, verifying the email address associated with your SSH key.

## Script Breakdown

### 1. **Parsing the SSH Config**
The script uses `awk` to search for the `Host github.com` section in `~/.ssh/config` and extract all relevant lines until a non-indented line is encountered:

```sh
matchedGroup=$(awk '
BEGIN { flag = 0 }
/^\s*[Hh]ost\s+github\.com/ {
  flag = 1
  print
  next
}
flag && /^\s/ {
  print
  next
}
flag && /^[^\s]/ {
  flag = 0
}
' ~/.ssh/config)
```

### 2. **Email Extraction**
Once the relevant `Host` section is found, the script looks for the `IdentityFile` and extracts the email part from its path:

```sh
file_path=$(echo "$matchedGroup" | grep "IdentityFile")
if [[ -n "$file_path" ]]; then
  email=$(echo "$file_path" | grep -oP '(?<=\.ssh\\)[^\\]+')
```

### 3. **Notifications**
- If the email is found, the user is notified with the following message:
  
  ```sh
  echo -e "\nYou are currently committing as: $email.\nType 'git reset --soft HEAD~1' to revert.\n"
  ```

- If no `IdentityFile` or email is found, or the SSH config is missing, appropriate messages are displayed.

### 4. **Reverting a Commit**
The script suggests using `git reset --soft HEAD~1` if the commit needs to be reverted without losing changes.

## Error Handling

- **SSH Config Not Found**: If the `Host github.com` section is missing in `~/.ssh/config`, the script will notify the user.
- **IdentityFile Not Found**: If no `IdentityFile` is specified under the `Host github.com` section, the user is alerted.

## Customization

If the email is not located in the `IdentityFile` path or the structure of your SSH config differs, you can modify the `grep` and `sed` commands within the script to suit your configuration needs.
