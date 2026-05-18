---
heroImage: '/awk-sed-advanced-text-processing.svg'
title: 'Command Line Magic: Advanced Text Processing with awk and sed'
description: 'Master the classic UNIX tools awk and sed to parse logs, transform data, and automate text editing directly from the terminal.'
pubDate: 'May 05 2026'
---

While Python or Ruby are excellent for complex text processing, nothing beats the speed and ubiquity of `awk` and `sed`. These tools are installed on virtually every Unix-like system. Mastering them allows you to parse gigabytes of log files and transform data on the fly.

## `sed`: The Stream Editor

`sed` is designed to parse and transform text. It reads text line by line, applies a set of rules, and outputs the result.

### 1. Simple Substitutions
The most common use of `sed` is finding and replacing text using regular expressions.
```bash
sed 's/ERROR/WARNING/g' server.log
```
The `s` command stands for substitute. `/ERROR/` is the pattern to find, `WARNING` is the replacement, and `g` stands for global (replace all occurrences on a line, not just the first).

### 2. Deleting Lines
You can easily delete lines matching a specific pattern. To delete all lines containing the word "debug":
```bash
sed '/debug/d' application.log
```

### 3. In-Place Editing
By default, `sed` outputs to standard out. To modify the file directly, use the `-i` (in-place) flag:
```bash
sed -i 's/127.0.0.1/0.0.0.0/g' /etc/nginx/nginx.conf
```
*Tip: Use `sed -i.bak` to automatically create a backup file before making changes.*

### 4. Printing Specific Line Ranges
To extract lines 100 through 150 from a massive log file without opening it in `less`:
```bash
sed -n '100,150p' huge_log.txt
```
(The `-n` flag suppresses default output, and `p` explicitly prints the matched lines).

## `awk`: The Data Extraction Language

While `sed` is great for altering text, `awk` is a full programming language specifically designed for processing columnar data and extracting fields.

By default, `awk` splits every line into fields based on whitespace.
- `$0` represents the entire line.
- `$1` represents the first column, `$2` the second, and so on.
- `NF` represents the Number of Fields (columns) in the line.

### 1. Extracting Columns
Print only the User (1st column) and Home Directory (6th column) from `/etc/passwd`. (We tell `awk` to use `:` as the field separator using `-F`):
```bash
awk -F':' '{print $1 " lives at " $6}' /etc/passwd
```

### 2. Filtering by Condition
Print lines where the 3rd column (e.g., HTTP status code) is greater than 400:
```bash
awk '$3 >= 400 {print $0}' access.log
```

### 3. Data Aggregation
`awk` really shines at aggregating data. Let's calculate the total sum of numbers in the 5th column (e.g., bytes transferred):
```bash
awk '{sum += $5} END {print "Total Bytes: " sum}' access.log
```
This script adds the 5th column of every line to the `sum` variable. The `END` block runs after all lines have been processed, printing the final total.

### 4. Finding Unique IP Addresses
Extract the first column (IP addresses), sort them, and count occurrences using a combination of tools:
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```
This classic one-liner prints a list of IP addresses ordered by how many times they appear in an Nginx access log.

## Combining Them

The true UNIX philosophy is piping simple tools together. For example, use `sed` to filter a block of time, and `awk` to average the response times within that block. Learning `sed` and `awk` transforms the terminal from a basic interface into a powerful data analytics engine.
