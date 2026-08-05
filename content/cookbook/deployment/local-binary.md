---
title: "Local project binary"
weight: 310
date: 2024-09-29T19:30:08+10:00
draft: false
group: true
summary: "Creating local per-project Rye binaries with custom modules"
mygroup: true
---

## Per project binary

Current idea is, or at least I do it like this, that I have one general `rye` installed that I can invoke at any time and at any location. 

But if I tackle a concrete and specific project, I build a project-specific binary and keep it local to the project. 

This approach lets you:
- Include only the modules you need
- Add contrib modules when required
- Have a reproducible build per project

## Setup

The `l.rye` script (local Rye) manages building project-specific binaries. You need:

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

Add `RYE_HOME` to your shell configuration file (~/.bashrc or ~/.zshrc) for persistence.

## Defining modules (lrye.mod)

Create a `lrye.mod` file in your project directory to define which modules to include in your build. 
List the module names - each will be prefixed with `b_` automatically to form build tags.

### Available modules

Common modules include:

- `http` - HTTP client and server functionality
- `sqlite` - SQLite database support
- `bleve` - Full-text search engine
- `contrib` - Community contributed builtins
- `mpv` - MPV media player integration
- `tiny` - Minimal build with reduced features

Contrib modules (require `contrib` flag):
- Various community contributed builtins from the `contrib/` folder

### Example lrye.mod files

For a project using contrib and mpv:
```
mpv
contrib
```

For a web application:
```
http
sqlite
bleve
```

An empty `lrye.mod` file is also valid if you just need the base modules.

## Building the local binary

```bash
# Build with modules from lrye.mod
$RYE_HOME/bin/l.rye build

# The resulting binary is called 'lrye'
# Run local rye binary with local main.rye
./lrye .

# Or run any script
./lrye myscript.rye
```

## Complete workflow example

```bash
# 1. Set up a new project
mkdir myproject && cd myproject

# 2. Create lrye.mod with desired modules
cat > lrye.mod << EOF
http
sqlite
EOF

# 3. Build the local binary
$RYE_HOME/bin/l.rye build

# 4. Create your main script
echo 'print "Hello from local Rye!"' > main.rye

# 5. Run with local binary
./lrye .
```

## Benefits

- **Smaller binary**: Include only what you need
- **Reproducible builds**: Same modules every time
- **Project isolation**: Each project has its own configured binary
- **Easy updates**: Rebuild when Rye updates

## See also

- [Embedded binaries](../embed-binary/) - Embed scripts into standalone executables
- [Rye binary options](../../binary_console/executable/) - Command-line options and flags
