## What is csvkit and Why It's a Game Changer

**csvkit** is a suite of command-line tools for working with CSV files. Think of it as the Swiss Army knife for CSV data - it can convert, analyze, clean, and query CSV files without opening Excel or writing Python scripts.

**💡 Mentor Tip**: csvkit doesn't need jq! It's a complete toolkit on its own. However, csvkit + jq is a powerful combo when you need to convert CSV to JSON and do complex transformations.

## Installation

```bash
# Using pip (recommended)
pip install csvkit

# On Ubuntu/Debian
sudo apt-get install python3-csvkit

# On macOS with Homebrew
brew install csvkit

# Verify installation
csvcut --version
```

## Core Tools Overview

csvkit includes several specialized tools:
- **csvlook**: Pretty-print CSV files
- **csvcut**: Extract columns (like cut for CSV)
- **csvgrep**: Search CSV files (like grep for CSV)
- **csvjson**: Convert CSV to JSON
- **csvsql**: Run SQL queries on CSV files
- **csvstat**: Get statistics about CSV data
- **csvclean**: Fix common CSV problems
- **csvjoin**: Join CSV files (like SQL JOIN)
- **csvstack**: Stack CSV files vertically
- **csvformat**: Convert between delimiters

## Getting Started - The Basics

### View CSV Files Properly
```bash
# Instead of this mess:
cat data.csv

# Use csvlook for pretty printing:
csvlook data.csv

# Sample output:
# | name    | age | city |
# | ------- | --- | ---- |
# | Alice   |  25 | NYC  |
# | Bob     |  30 | LA   |
# | Charlie |  35 | CHI  |

# Limit rows for large files
csvlook data.csv | head -20
```

### Extract Specific Columns
```bash
# Get just the name and city columns
csvcut -c name,city data.csv

# Get columns by position (1-indexed)
csvcut -c 1,3 data.csv

# Get all columns except age
csvcut -C age data.csv

# See all column names
csvcut -n data.csv
```

### Filter Rows
```bash
# Find all people aged 30 or over
csvgrep -c age -m "30" data.csv

# Find all people from NYC (exact match)
csvgrep -c city -m "NYC" data.csv

# Find names starting with 'A' (regex)
csvgrep -c name -r "^A" data.csv

# Inverse match (NOT from NYC)
csvgrep -c city -m "NYC" -i data.csv
```

## Real-World Examples

### Data Analysis Workflow
```bash
# 1. Explore the data structure
csvcut -n sales.csv
#   1: date
#   2: product
#   3: quantity
#   4: price
#   5: customer_id

# 2. Get basic statistics
csvstat sales.csv

# 3. Find top products
csvcut -c product,quantity sales.csv | csvsort -c quantity -r | head -10

# 4. Filter by date range
csvgrep -c date -r "2023-11|2023-12" sales.csv > q4_sales.csv

# 5. Calculate total revenue
csvcut -c quantity,price sales.csv | \
  awk -F',' 'NR>1 {sum += $1 * $2} END {print "Total Revenue: $" sum}'
```

### Clean Messy Data
```bash
# Fix common issues automatically
csvclean dirty_data.csv

# This creates:
# - dirty_data_out.csv (cleaned data)
# - dirty_data_out_problems.csv (rows with issues)

# Convert different delimiters to standard CSV
csvformat -t data.tsv > data.csv  # Tab-separated to CSV
csvformat -d "|" data.txt > data.csv  # Pipe-delimited to CSV
```

### Database-Style Operations

#### SQL Queries on CSV
```bash
# Run SQL directly on CSV files!
csvsql --query "SELECT name, age FROM data WHERE age > 25" data.csv

# Join two CSV files
csvsql --query "SELECT * FROM sales JOIN customers ON sales.customer_id = customers.id" \
  sales.csv customers.csv

# Group by and aggregate
csvsql --query "SELECT product, SUM(quantity) as total FROM sales GROUP BY product" sales.csv

# Create an actual SQLite database
csvsql --db sqlite:///sales.db --insert sales.csv
sqlite3 sales.db "SELECT * FROM sales LIMIT 10"
```

#### Join Files Without SQL
```bash
# Join on customer_id
csvjoin -c customer_id sales.csv customers.csv > sales_with_customers.csv

# Join on different column names
csvjoin -c "id,customer_id" customers.csv sales.csv

# Left join (keep all rows from first file)
csvjoin --left -c customer_id sales.csv customers.csv
```

### Data Transformation

#### Convert to JSON
```bash
# Basic conversion
csvjson data.csv > data.json

# Pretty printed
csvjson -i 2 data.csv > data.json

# Streaming mode for large files
csvjson --stream data.csv

# Now you CAN use jq!
csvjson data.csv | jq '.[] | select(.age > 25)'
```

#### Reshape Data
```bash
# Stack multiple CSV files
csvstack north_sales.csv south_sales.csv east_sales.csv > all_sales.csv

# Add a grouping column
csvstack -g "North,South,East" -n "region" \
  north_sales.csv south_sales.csv east_sales.csv > regional_sales.csv

# Transpose rows and columns
csvlook data.csv | datamash transpose
```

## Advanced Patterns

### Pipeline Processing
```bash
# Complex data pipeline
cat raw_sales.csv | \
  csvgrep -c status -m "completed" | \           # Filter completed sales
  csvcut -c date,product,quantity,price | \      # Select columns
  csvsort -c date -r | \                          # Sort by date descending
  csvgrep -c date -r "2023" | \                  # Only 2023 data
  csvstat --mean -c price                         # Average price

# Generate report
csvcut -c product,quantity sales.csv | \
  csvsql --query "SELECT product, SUM(quantity) as total 
                  FROM stdin GROUP BY product 
                  ORDER BY total DESC" | \
  csvlook > product_report.txt
```

### Working with Large Files
```bash
# Process in chunks
csvcut -c important_columns huge_file.csv | gzip > processed.csv.gz

# Sample data
head -n 10000 huge_file.csv | csvstat

# Use streaming for JSON conversion
csvjson --stream huge_file.csv | \
  jq -c 'select(.important_field == "value")' > filtered.jsonl
```

### Data Validation
```bash
# Check for required columns
csvcut -n data.csv | grep -E "(customer_id|email|name)" || echo "Missing required columns"

# Find duplicate entries
csvcut -c email customers.csv | sort | uniq -d

# Validate data types
csvstat --type data.csv

# Find null/empty values
csvgrep -c email -r "^$" customers.csv > missing_emails.csv
```

## Common Patterns and Solutions

### Date Handling
```bash
# Filter by date range
csvgrep -c date -r "2023-0[1-3]" sales.csv  # Q1 2023

# Sort by date
csvsort -c date sales.csv

# Extract year/month
csvcut -c date sales.csv | \
  awk -F',' '{print substr($1,1,7)}' | \
  sort | uniq -c
```

### Numeric Operations
```bash
# Sum a column
csvcut -c amount data.csv | \
  awk '{sum+=$1} END {print sum}'

# Find min/max
csvstat --min -c price products.csv
csvstat --max -c price products.csv

# Calculate percentages
csvsql --query "SELECT category, 
                COUNT(*) * 100.0 / (SELECT COUNT(*) FROM products) as percentage 
                FROM products GROUP BY category" products.csv
```

### Text Processing
```bash
# Clean whitespace
csvcut -c name data.csv | \
  awk '{$1=$1};1' | \
  csvformat -T > cleaned_names.csv

# Extract email domains
csvcut -c email users.csv | \
  awk -F'@' 'NR>1 {print $2}' | \
  sort | uniq -c | sort -nr

# Combine columns
csvjoin --no-inference -c 1 \
  <(csvcut -c first_name,last_name names.csv) \
  <(csvcut -c first_name,email emails.csv)
```

## Integration with Other Tools

### With jq (JSON processing)
```bash
# CSV → JSON → Complex transformation → CSV
csvjson users.csv | \
  jq '[.[] | {name, email, age: (.age | tonumber + 1)}]' | \
  in2csv -f json > users_aged.csv
```

### With awk (when csvkit isn't enough)
```bash
# Complex calculations
csvcut -c quantity,price sales.csv | \
  awk -F',' 'NR>1 {revenue+=$1*$2; count++} 
             END {print "Average order value:", revenue/count}'
```

### With Python (for complex logic)
```bash
# Use csvkit for extraction, Python for processing
csvcut -c important_columns data.csv | \
  python3 -c "
import csv, sys
reader = csv.DictReader(sys.stdin)
for row in reader:
    # Complex processing here
    print(row)
"
```

## Performance Tips

### For Speed
```bash
# Use specific columns early
csvcut -c needed_columns huge.csv | csvgrep -c column -m "value"

# Not this:
csvgrep -c column -m "value" huge.csv | csvcut -c needed_columns
```

### For Memory
```bash
# Stream processing
csvjson --stream large.csv | process_line_by_line

# Use system tools when possible
csvcut -c 1 huge.csv | sort -u  # Faster than csvsql for unique values
```

### For Reliability
```bash
# Always check your data first
csvclean input.csv
csvstat input.csv

# Use --no-inference for predictable types
csvjson --no-inference data.csv
```

## Common Gotchas and Solutions

### Encoding Issues
```bash
# Specify encoding
csvlook data.csv --encoding latin1

# Convert encoding
iconv -f latin1 -t utf8 data.csv | csvlook
```

### Memory Errors
```bash
# Process in chunks
split -l 100000 huge.csv chunk_
for f in chunk_*; do
    csvgrep -c status -m "active" "$f"
done > filtered.csv
```

### Delimiter Detection
```bash
# Force delimiter
csvlook -d "|" data.txt
csvformat -d "|" data.txt > data.csv
```

## Quick Command Reference

```bash
# Viewing
csvlook file.csv                    # Pretty print
csvcut -n file.csv                  # Show column names/positions

# Selecting  
csvcut -c col1,col2 file.csv       # Select columns
csvcut -C col3 file.csv             # Exclude columns

# Filtering
csvgrep -c col -m "value" file.csv  # Match value
csvgrep -c col -r "regex" file.csv  # Match regex
csvgrep -c col -m "value" -i file.csv # Inverse match

# Transforming
csvjson file.csv                    # Convert to JSON
csvformat -T file.csv               # Convert to TSV
csvsort -c col file.csv             # Sort by column
csvjoin -c key left.csv right.csv   # Join files

# Analyzing
csvstat file.csv                    # Full statistics
csvstat --mean -c col file.csv      # Column mean
csvsql --query "SELECT..." file.csv # SQL queries

# Cleaning
csvclean file.csv                   # Fix common issues
csvformat file.csv                  # Normalize format
```

## Next Steps

1. **Practice with your own data**: Start with `csvlook` and `csvcut`
2. **Learn SQL basics**: Makes `csvsql` incredibly powerful
3. **Combine with other tools**: csvkit + jq + awk = data superpowers
4. **Automate reports**: Build pipelines for regular analysis

**💡 Final Tip**: csvkit shines for quick analysis and data cleaning. For complex transformations or huge files (GB+), consider pandas or dedicated databases. But for 90% of CSV tasks, csvkit is faster than writing code!

## Related Topics
- [[Linux/Quick Reference/Command Line Tools/Text Processing/🔧 jq Command Quick Reference Sheet 🔧]] - JSON processing
- [[Linux/Quick Reference/Command Line Tools/Text Processing/🔧 awk Command Quick Reference Sheet 🔧]] - Text processing
- [[SQL/Basic SQL (SQLITE3)/SQLite3 CLI Quick Reference Sheet]] - SQL basics
- [[Python/Python Overview]] - For complex data processing