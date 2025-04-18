## Installation 📥
```bash
# Linux
apt install jq         # Debian/Ubuntu
yum install jq         # CentOS/RHEL
pacman -S jq           # Arch Linux

# macOS
brew install jq        # Using Homebrew

# Windows
choco install jq       # Using Chocolatey
winget install jq      # Using winget
```

## Basic Usage
```bash
jq 'filter' file.json  # Apply filter to JSON file
cat file.json | jq '.' # Pipe JSON to jq for pretty-printing
```

## Common Options 🔍
```bash
jq -r                  # Raw output (no quotes)
jq -c                  # Compact output (one line)
jq -s                  # Slurp (read all inputs as array)
jq -f file.jq          # Read filters from file
jq --arg name value    # Pass variable to filter
```

## Basic Filters 🔎
```bash
jq '.'                 # Identity: pretty-print JSON
jq '.property'         # Access object property
jq '.[0]'              # Access array element
jq '.items[]'          # Iterate over array
jq '.items[2:5]'       # Array slice
jq '.items | length'   # Get array length
```

## Advanced Filters 💡
```bash
jq 'map(.property)'    # Transform each item in array
jq 'select(.prop > 5)' # Filter items
jq 'del(.property)'    # Delete property
jq '.property?'        # Optional access (no error if missing)
jq '.[] | .name, .id'  # Multiple outputs per input
jq 'keys'              # Get object keys as array
```

## Common Examples 🚀
```bash
# Extract specific fields from array
jq '.items[] | {name: .name, id: .id}' file.json

# Filter array by condition
jq '.[] | select(.status == "active")' users.json

# Count items matching condition
jq '[.[] | select(.type == "error")] | length' logs.json

# Group by property
jq 'group_by(.category)' items.json

# Sort array
jq 'sort_by(.date)' events.json
```

## Data Transformation 🔄
```bash
# Convert array to object with key from property
jq 'map({key: .id, value: .}) | from_entries' array.json

# Flatten nested arrays
jq 'flatten' nested.json

# Combine objects
jq '. + {"new": "property"}' object.json

# Convert CSV to JSON
jq -R -s 'split("\n") | map(split(",")) | .[0] as $header | .[1:-1] | map(. as $row | $header | to_entries | map({key: .value, value: $row[.key]}) | from_entries)' file.csv
```

## Further Reading 📚
- Official Documentation: [jq Manual](https://stedolan.github.io/jq/manual/)
- Man Page: `man jq` in your terminal
- Additional Resources:
  - [jq Playground](https://jqplay.org/) - Interactive jq editor
  - [jq Cookbook](https://github.com/stedolan/jq/wiki/Cookbook)
  - [Learn jq Interactively](https://jqterm.com/learn)
  - [jq Cheat Sheet](https://lzone.de/cheat-sheet/jq)

---