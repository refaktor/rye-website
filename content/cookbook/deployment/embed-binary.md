---
title: "Embedded binaries"
weight: 320
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Embedding Rye scripts into standalone executables"
mygroup: true
---

## Standalone executables

You can embed your Rye scripts into a single standalone binary. This lets you distribute your whole Rye program as one executable - no additional script files needed (and it can't invoke custom scripts).

Benefits:
- **Simpler deployment**: Single file to distribute
- **Security**: AppArmor rules can be set per binary
- **No dependencies**: Scripts are bundled in the executable

## Setup

You need:

1. Rye source code on your computer
2. Go(lang) installed
3. `RYE_HOME` environment variable set

```bash
# Define RYE_HOME environment variable pointing to Rye source
export RYE_HOME=/home/user/rye

# Get help and setup instructions
$RYE_HOME/bin/l.rye help
$RYE_HOME/bin/l.rye setup
```

## Embedding a single file

For a simple project with just main.rye:

```bash
# Build binary with main.rye embedded
$RYE_HOME/bin/l.rye build embed_main

# The resulting binary is called 'main' and can be run directly
./main
```

The script is embedded into the binary. You can move it anywhere or delete the source .rye files and it will still work.

## Embedding multiple files

For projects with multiple Rye files (main.rye plus libraries), create a `lrye.files` file listing all files to embed:

```
%main.rye
%lib.rye
%utils.rye
```

Each file path should be prefixed with `%`. When you build with `embed_main`, all listed files will be 
embedded into the binary.

Your main.rye can then import the embedded files normally:

```rye
; main.rye - imports embedded lib.rye
Import %lib.rye
hello-lib   ; call function from lib.rye

Import %utils.rye
do-something
```

## Combining with modules (lrye.mod)

You can combine embedding with custom modules. Create both `lrye.mod` and `lrye.files`:

```bash
# lrye.mod - modules to include
cat > lrye.mod << EOF
http
sqlite
contrib
EOF

# lrye.files - files to embed
cat > lrye.files << EOF
%main.rye
%lib.rye
EOF

# Build with modules AND embedded files
$RYE_HOME/bin/l.rye build embed_main
```

## Reading local files at runtime

Even with embedded scripts, you can still read non-embedded local files at runtime:

```rye
; This file is embedded
Import %lib.rye

; But we can still read external files
config: Read %config.txt
data: Read %data.json
```

This is useful for configuration files or data that  to be changeable without rebuilding.

## Complete workflow example

```bash
# 1. Create your project files
echo 'print "Hello from embedded Rye!"' > main.rye
echo 'hello-lib: fn { } { print "Hello from lib!" }' > lib.rye

# 2. Create lrye.mod with desired modules (can be empty)
echo 'http' > lrye.mod

# 3. Create lrye.files listing files to embed
cat > lrye.files << EOF
%main.rye
%lib.rye
EOF

# 4. Build the embedded binary
$RYE_HOME/bin/l.rye build embed_main

# 5. Run the standalone binary
./main

# 6. Test it works without source files
rm main.rye lib.rye
./main  # Still works!
```

## GUI applications (Rye-Fyne)

You can create embedded binaries with Rye-Fyne also. This gives you an executable that directly opens the GUI without needing script files.

*Support for Rye-Fyne, Rye-Gio, and other extensions is being added.*

## Building apk from Rye app

*Documentation coming soon for building Android APKs.*

## Security benefits

Embedded binaries have some security advantages:

1. **AppArmor/SELinux**: Rules can be set per binary. A generic Rye binary that runs any script makes rules harder to define.
2. **Tamper resistance**: Users can't easily modify the embedded scripts
3. **Distribution**: No risk of scripts being modified in transit

## See also

- [Local project binary](../local-binary/) - Creating project-specific Rye binaries
- [Rye binary options](../../binary_console/executable/) - Command-line options and flags
