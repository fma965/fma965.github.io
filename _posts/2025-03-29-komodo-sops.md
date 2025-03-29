---
layout: post
title: "Using SOPS Encryption with Komodo"
date: 2025-03-29 18:41:00 0000
categories: docker gitops
tags: homelab docker gitops komodo
image:
 path: /assets/img/thumbnails/komodosops.webp
---

Notes
This guide assumes you already have a GitOps system setup with Komodo, if you don't please follow [FoxxMD's guide here](https://blog.foxxmd.dev/posts/migrating-to-komodo/#what-is-komodo0)

We will cover using the sops binary and a age key to safely and securely store your secrets in a public git repo encrypted with sops.

Lets first set our `$PERIPHERY_ROOT_DIRECTORY` variable to be the same as what is set in our Komodo compose file.
```bash
export PERIPHERY_ROOT_DIRECTORY=/path/used/in/komodo/compose
```
for me this is as follows since i use Unraid
```bash
export PERIPHERY_ROOT_DIRECTORY=/mnt/user/appdata/komodo
```

Run the following commands
```bash
curl -LO https://github.com/getsops/sops/releases/download/v3.9.4/sops-v3.9.4.linux.amd64
mv sops-v3.9.4.linux.amd64 $PERIPHERY_ROOT_DIRECTORY/sops
chmod +x $PERIPHERY_ROOT_DIRECTORY/sops
```

Now ensure you have `age` installed, use your standard package manager to install this, apt, apk, etc

Once `age` is installed run
```bash
age-keygen -o $PERIPHERY_ROOT_DIRECTORY/.sops.key
```

Now all you need to do is use the following script in your GitOps repo
```bash
#!/bin/bash

# Check if required env var is set
if [ -z "$PERIPHERY_ROOT_DIRECTORY" ]; then
    echo "Error: PERIPHERY_ROOT_DIRECTORY is not set." >&2
    exit 1
fi

export SOPS_AGE_KEY_FILE="$PERIPHERY_ROOT_DIRECTORY/.sops.key"

sops="$PERIPHERY_ROOT_DIRECTORY/sops"

# Only proceed if secret.enc.env exists
if [ -f "secret.enc.env" ]; then
    # Ensure sops exists and is executable
    if [ -f "$sops" ] && [ ! -x "$sops" ]; then
        chmod +x "$sops"
    fi
    # Decrypt and write to .env
    "$sops" -d secret.enc.env > .env
fi
```
Save this as something useful, for me i used `decrypt-secrets.sh`

Finally depending on where you have your stacks and where you save the `decrypt-secrets.sh` add the following to your komodo toml for each stack you want to decrypt secrets on
```toml
pre_deploy.command = """
bash ../../decrypt-secrets.sh
"""
```
I'm using ../../ as my stacks are at `docker/stacks/stackname` but my `decrypt-secrets.sh` file is 2 directory above at `docker/`

Remove any enviroment variables you have set in the stack, and redeploy and it should work!

Make sure any secrets are called secret.env

Optionally use this githook to ensure no unecrypted secrets get imported
`pre-commit`
```bash
#!/bin/sh

# Debug: Print environment and basic info
echo "=== Starting encryption script ==="

# Define where your age public key for sops is stored
AGE_KEY_FILE=".sops/age/public.key"
if [ ! -f "$AGE_KEY_FILE" ]; then
  echo "Age key file not found: $AGE_KEY_FILE"
  exit 1
fi
PUBLIC_KEY=$(cat "$AGE_KEY_FILE")

# Function to compute SHA-256 hash of file content
compute_hash() {
  sha256sum "$1" | awk '{print $1}'
}

# Function to encrypt file with sops
encrypt_file() {
  local file=$1
  local ext=${file##*.}
  local enc_file="${file%.$ext}.enc.$ext"
  local sops_args="--encrypt --age $PUBLIC_KEY"

  # Add YAML-specific regex if needed
  if [ "$ext" = "yaml" ] || [ "$ext" = "yml" ]; then
    sops_args="$sops_args --encrypted-regex '^(data|stringData)$'"
  fi

  echo "Encrypting $file to $enc_file"
  if eval "sops $sops_args '$file'" > "$enc_file"; then
    local hash=$(compute_hash "$file")
    echo "# Hash: $hash" >> "$enc_file"
    echo "Successfully encrypted: $enc_file"
    git add "$enc_file"
    return 0
  else
    echo "Failed to encrypt $file"
    return 1
  fi
}

# Get list of staged files that match our patterns
echo "=== Checking staged files ==="
MATCHING_FILES=$(git diff --cached --name-only | grep -E '(.*secret\.yaml$|.*secret\.env$|.*\.auto\.tfvars$)')
echo "Matched files:"
echo "$MATCHING_FILES" | while read line; do
  echo "- $line"
done

# Process matching files
echo "=== Processing files ==="
for file in $MATCHING_FILES; do
  echo "Processing file: $file"
  
  # Skip if file is already encrypted
  if [[ "$file" == *.enc.* ]]; then
    echo "Skipping already encrypted file: $file"
    continue
  fi

  # Verify file exists
  if [ ! -f "$file" ]; then
    echo "Warning: File $file is staged but doesn't exist in working directory"
    continue
  fi

  current_hash=$(compute_hash "$file")
  ext=${file##*.}
  
  # Determine encrypted filename
  if [[ "$file" == *.auto.tfvars ]]; then
    enc_file="${file%.auto.tfvars}.enc.auto.tfvars"
  else
    enc_file="${file%.$ext}.enc.$ext"
  fi

  if [ -f "$enc_file" ]; then
    previous_hash=$(tail -n 1 "$enc_file" | grep '# Hash:' | cut -d' ' -f3)
    if [ "$current_hash" != "$previous_hash" ]; then
      echo "File has changed, re-encrypting: $file"
      if ! encrypt_file "$file"; then
        echo "Failed to encrypt $file"
        continue
      fi
    else
      echo "No changes detected in $file. Skipping encryption."
    fi
  else
    echo "No existing encrypted file found, encrypting: $file"
    if ! encrypt_file "$file"; then
      echo "Failed to encrypt $file"
      continue
    fi
  fi

  # Unstage the plaintext file
  git reset HEAD -- "$file"
  echo "Unstaged plaintext file: $file"
done

echo "=== Script completed ==="
exit 0
```

Note you will have to use interpolation for these secrets to be exposed to the container
https://github.com/FoxxMD/compose-env-interpolation-example

For more information check out my repo here
https://github.com/fma965/f9-homelab