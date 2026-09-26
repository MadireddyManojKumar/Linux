# 04 · Text Processing & Filters

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Slice logs, configs and command output fast — the skill that separates a 5-minute RCA from a 2-hour one.

---

## Table of Contents

1. [The Unix Philosophy — Pipes](#1-the-unix-philosophy--pipes)
2. [grep — Search Text](#2-grep--search-text)
3. [Regular Expressions Quick Guide](#3-regular-expressions-quick-guide)
4. [cut — Extract Columns](#4-cut--extract-columns)
5. [sort — Order Lines](#5-sort--order-lines)
6. [uniq — Deduplicate & Count](#6-uniq--deduplicate--count)
7. [tr — Translate Characters](#7-tr--translate-characters)
8. [sed — Stream Editor](#8-sed--stream-editor)
9. [awk — Column Processing Language](#9-awk--column-processing-language)
10. [Other Filters: tee, paste, join, column, fold, rev, comm, split](#10-other-filters)
11. [JSON & YAML: jq, yq](#11-json--yaml-jq-yq)
12. [Compressed Logs: zcat, zgrep, zless](#12-compressed-logs-zcat-zgrep-zless)
13. [journalctl — Filtering systemd Logs](#13-journalctl--filtering-systemd-logs)
14. [Real-World One-Liners (Log Analysis)](#14-real-world-one-liners-log-analysis)
15. [Interview Questions](#15-interview-questions)
16. [Cheat Sheet](#16-cheat-sheet)

---

## 1. The Unix Philosophy — Pipes

Small tools, each doing one thing, chained with `|`.

```bash
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head
#   read file     get IP column     group  count     top first  top 10
```

> Tip: skip "useless cat" — `awk '{print $1}' access.log` is the same and faster.

---

## 2. grep — Search Text

```bash
grep "ERROR" app.log
grep -i "error" app.log                 # Case-insensitive
grep -v "DEBUG" app.log                 # Exclude lines
grep -n "timeout" app.log               # Show line numbers
grep -c "500" access.log                # Count matches
grep -r "db_password" /etc/app/         # Recursive
grep -rl "8080" /etc/                   # Only filenames
grep -w "fail" log                      # Whole word only
grep -A 5 -B 2 "Exception" app.log      # Context: 2 before, 5 after
grep -C 3 "OOM" /var/log/messages       # 3 lines each side
grep -E "ERROR|WARN|FATAL" app.log      # Multiple patterns (regex)
grep -o "user=[a-z]*" auth.log          # Print only the match
grep -F "a.b*c" file                    # Literal string (no regex)
grep -e "-v" file                       # Pattern starting with dash
grep -f patterns.txt app.log            # Patterns from a file
grep -rn --include="*.yaml" "image:" .  # Only YAML files
grep -rn --exclude-dir=.git "TODO" .
grep -P '\d{3}\.\d+' file               # Perl regex (\d, lookahead)
grep -m 1 "started" app.log             # Stop after first match
grep -q "ready" status && echo OK       # Quiet: exit code only (scripts)
grep -v '^\s*#' nginx.conf | grep -v '^$'   # Strip comments & blanks
```

| Option | What it does |
|--------|--------------|
| `-i` | Ignore uppercase/lowercase differences |
| `-v` | Invert: show non-matching lines |
| `-n` | Prefix each line with number |
| `-c` | Count matching lines only |
| `-r` / `-R` | Search directories recursively (R follows links) |
| `-l` | Print only matching file names |
| `-L` | Print files with no match |
| `-w` | Match whole words only |
| `-x` | Match whole line exactly |
| `-o` | Print only the matched part |
| `-E` | Extended regex (`\|`, `+`, `?`) |
| `-F` | Fixed string, no regex |
| `-P` | Perl-compatible regex (`\d`, `\s`) |
| `-A N` | Show N lines after match |
| `-B N` | Show N lines before match |
| `-C N` | Show N lines around match |
| `-m N` | Stop after N matches |
| `-q` | Quiet, only return exit code |
| `-s` | Suppress file error messages |
| `-h` / `-H` | Hide / show filename per line |
| `--color=auto` | Highlight matches in color |
| `--include=GLOB` | Only search matching file names |
| `--exclude-dir=DIR` | Skip this directory while searching |

> Faster alternative: **`rg` (ripgrep)** — recursive by default, respects `.gitignore`.

---

## 3. Regular Expressions Quick Guide

| Pattern | Matches |
|---------|---------|
| `.` | Any single character |
| `^` | Start of line |
| `$` | End of line |
| `*` | 0 or more of previous |
| `+` | 1 or more (ERE `-E`) |
| `?` | 0 or 1 (ERE) |
| `{n,m}` | Between n and m times (ERE) |
| `[abc]` | One of a, b, c |
| `[^abc]` | Not a, b, c |
| `[0-9]` / `[a-z]` | Digit / lowercase letter |
| `\|` | OR (ERE) |
| `()` | Group (ERE) |
| `\b` | Word boundary |
| `\s` / `\d` / `\w` | Space / digit / word char (`-P`) |

```bash
grep -E '^[0-9]{1,3}(\.[0-9]{1,3}){3}' file          # Lines starting with IPv4
grep -E ' (5[0-9]{2}) ' access.log                   # HTTP 5xx
grep -Eo '[[:alnum:]._%+-]+@[[:alnum:].-]+' users    # Emails
```

---

## 4. cut — Extract Columns

```bash
cut -d: -f1 /etc/passwd              # Usernames
cut -d: -f1,3,7 /etc/passwd          # User, UID, shell
cut -d, -f2- data.csv                # Field 2 to end
cut -c1-10 file                      # Characters 1-10
```

| Option | What it does |
|--------|--------------|
| `-d` | Set field delimiter character |
| `-f` | Select these field numbers |
| `-c` | Select these character positions |
| `--complement` | Show everything except selected fields |
| `--output-delimiter` | Use different delimiter in output |

> `cut` only handles **single-character** delimiters. Multiple spaces? Use `awk`.

---

## 5. sort — Order Lines

```bash
sort names.txt
sort -r file                          # Reverse
sort -n numbers.txt                   # Numeric
sort -rn                              # Numeric, largest first
sort -h sizes.txt                     # Human sizes (1K, 2M, 3G)
sort -k3 -n file                      # By 3rd column numerically
sort -t: -k3 -n /etc/passwd           # By UID
sort -u file                          # Sort + unique
sort -k2,2 -k1,1 file                 # Multi-key sort
du -sh * | sort -hr                   # Biggest dirs first
sort -V versions.txt                  # Version sort (1.2 < 1.10)
```

| Option | What it does |
|--------|--------------|
| `-n` | Sort by numeric value |
| `-r` | Reverse, descending order |
| `-h` | Compare human sizes like 2G |
| `-k N` | Sort using column N |
| `-t` | Field separator for columns |
| `-u` | Output unique lines only |
| `-f` | Ignore case while sorting |
| `-V` | Natural version number sort |
| `-M` | Sort by month name |
| `-o file` | Write result to file |
| `-c` | Check if already sorted |

---

## 6. uniq — Deduplicate & Count

> ⚠️ `uniq` only removes **adjacent** duplicates → always `sort` first.

```bash
sort ips.txt | uniq                   # Unique
sort ips.txt | uniq -c                # Count each
sort ips.txt | uniq -c | sort -rn     # Top talkers ← classic
sort file | uniq -d                   # Only duplicated lines
sort file | uniq -u                   # Only lines seen once
```

| Option | What it does |
|--------|--------------|
| `-c` | Prefix lines with occurrence count |
| `-d` | Print only duplicated lines |
| `-u` | Print only non-repeated lines |
| `-i` | Ignore case when comparing |
| `-f N` | Skip first N fields comparing |

---

## 7. tr — Translate Characters

```bash
echo "hello" | tr 'a-z' 'A-Z'          # HELLO
tr -d '\r' < win.sh > unix.sh          # Remove Windows CR
echo "a  b   c" | tr -s ' '            # Squeeze spaces
echo $PATH | tr ':' '\n'               # PATH one per line
tr -dc 'A-Za-z0-9' </dev/urandom | head -c 20   # Random password
```

| Option | What it does |
|--------|--------------|
| `-d` | Delete listed characters entirely |
| `-s` | Squeeze repeated characters into one |
| `-c` | Use complement of the set |

---

## 8. sed — Stream Editor

### Substitute (most common)

```bash
sed 's/old/new/' file                  # First match per line
sed 's/old/new/g' file                 # All matches
sed -i 's/8080/9090/g' app.conf        # Edit file in place ⚠️
sed -i.bak 's/8080/9090/g' app.conf    # In place, keep backup ← safer
sed 's/old/new/gI' file                # Case-insensitive (GNU)
sed 's#/usr/local#/opt#g' file         # Different delimiter for paths
sed -E 's/(v)[0-9.]+/\12.0.0/' file    # Regex groups
sed "s/__ENV__/${ENV}/g" tpl > conf    # Template with variable (double quotes)
```

### Print / delete lines

```bash
sed -n '10,20p' file                   # Print lines 10-20
sed -n '/START/,/END/p' file           # Print between patterns
sed -n '$p' file                       # Last line
sed '5d' file                          # Delete line 5
sed '/^#/d' file                       # Delete comment lines
sed '/^$/d' file                       # Delete blank lines
sed '/^\s*#/d;/^$/d' nginx.conf        # Clean config view
```

### Insert / append / change

```bash
sed '3i\new line above 3' file
sed '/\[mysqld\]/a max_connections=500' my.cnf    # After a match
sed '/^PasswordAuthentication/c\PasswordAuthentication no' sshd_config
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
```

| Option / Cmd | What it does |
|--------------|--------------|
| `-i` | Edit the file in place |
| `-i.bak` | In-place edit, keep backup copy |
| `-n` | Suppress auto-print; use with `p` |
| `-E` / `-r` | Use extended regular expressions |
| `-e` | Add multiple edit expressions |
| `s/a/b/` | Substitute first a with b |
| `g` flag | Replace every match on line |
| `I` flag | Case-insensitive match (GNU sed) |
| `p` | Print the pattern space |
| `d` | Delete the matching line |
| `i\` / `a\` | Insert before / append after line |
| `c\` | Replace whole line with text |
| `N,Mp` | Print only lines N to M |

---

## 9. awk — Column Processing Language

Default separator = whitespace. `$1` first field, `$0` whole line, `NF` number of fields, `NR` line number.

```bash
awk '{print $1}' access.log                       # 1st column (IP)
awk '{print $1, $9}' access.log                   # IP + status
awk -F: '{print $1, $7}' /etc/passwd              # User + shell
awk -F: '$3 >= 1000 {print $1}' /etc/passwd       # Human users
awk '$9 == 500' access.log                        # Status 500 lines
awk '$9 ~ /^5/' access.log                        # Any 5xx
awk '/ERROR/ {c++} END {print c}' app.log         # Count ERRORs
awk '{print $NF}' file                            # Last column
awk 'NR==5' file                                  # Line 5
awk 'NR>1' file.csv                               # Skip header
awk '{sum += $10} END {print sum/1024/1024 " MB"}' access.log   # Total bytes
awk '{s+=$NF; n++} END {print "avg:", s/n}' latency.log          # Average
awk '{a[$1]++} END {for (ip in a) print a[ip], ip}' access.log | sort -rn | head
awk 'length($0) > 200' file                       # Long lines
awk -v t=90 '$5+0 > t {print $6, $5}' <(df -h)    # Mounts over 90%
awk 'BEGIN {OFS=","} {print $1,$2}' file          # CSV output
awk '!seen[$0]++' file                            # Dedupe keep order (no sort)
```

| Option / Var | What it does |
|--------------|--------------|
| `-F` | Set input field separator |
| `-v var=val` | Pass shell variable into awk |
| `$0` | The entire current line |
| `$1..$N` | Field number 1 to N |
| `NF` | Number of fields in line |
| `$NF` | Last field of the line |
| `NR` | Current line (record) number |
| `FS` / `OFS` | Input / output field separator |
| `BEGIN {}` | Runs once before reading input |
| `END {}` | Runs once after all input |
| `~` / `!~` | Matches / doesn't match regex |

---

## 10. Other Filters

```bash
cmd | tee out.log                     # Screen + file
cmd | tee -a out.log                  # Append
paste -d, names.txt ages.txt          # Merge files side by side
paste -sd, list.txt                   # Lines → a,b,c
join -t, a.csv b.csv                  # SQL-like join on sorted key
mount | column -t                     # Align into table
column -t -s, data.csv                # Pretty print CSV
fold -w 80 long.txt                   # Wrap at 80 chars
rev file                              # Reverse each line
comm -23 <(sort a) <(sort b)          # Lines only in a
split -l 100000 big.log part_         # Split by lines
split -b 100M big.tar chunk_          # Split by size
nl -ba file                           # Number all lines
expand / unexpand                     # Tabs ↔ spaces
fmt -w 72 file                        # Reformat paragraphs
seq 1 5                               # 1 2 3 4 5
shuf -n 1 hosts.txt                   # Random line
```

| Option | What it does |
|--------|--------------|
| `tee -a` | Append instead of overwrite file |
| `paste -s` | Join all lines into one |
| `paste -d` | Delimiter used when joining |
| `column -t` | Align input into neat table |
| `column -s` | Input separator for column |
| `comm -12` | Show lines common to both |
| `comm -23` | Lines only in first file |
| `split -l` | Split every N lines |
| `split -b` | Split every N bytes |

---

## 11. JSON & YAML: jq, yq

```bash
curl -s api/health | jq .                               # Pretty print
jq '.status' resp.json
jq -r '.items[].metadata.name' pods.json                # Raw strings
kubectl get pods -o json | jq -r '.items[] | select(.status.phase!="Running") | .metadata.name'
aws ec2 describe-instances | jq -r '.Reservations[].Instances[] | [.InstanceId,.State.Name] | @tsv'
jq '.replicas = 3' deploy.json > new.json
jq -c '.[]' arr.json                                     # One compact object per line
yq '.spec.replicas' deploy.yaml
yq -i '.image.tag = "v2.1"' values.yaml                  # Edit YAML in place
```

| Option | What it does |
|--------|--------------|
| `-r` | Raw output, no JSON quotes |
| `-c` | Compact output, one line each |
| `-e` | Exit non-zero if null/false |
| `-s` | Slurp all inputs into array |
| `--arg k v` | Pass shell variable into jq |
| `select()` | Filter items by a condition |
| `@tsv` / `@csv` | Format array as TSV/CSV |

---

## 12. Compressed Logs: zcat, zgrep, zless

```bash
zcat app.log.2.gz | grep ERROR
zgrep -i "timeout" /var/log/app/*.gz
zless syslog.3.gz
zgrep -h "ERROR" app.log* | wc -l     # Search rotated + current
xzcat file.xz | head ; bzcat file.bz2 | head
```

---

## 13. journalctl — Filtering systemd Logs

```bash
journalctl -u nginx                    # One service
journalctl -u nginx -f                 # Follow live
journalctl -u nginx -n 100 --no-pager
journalctl -u app --since "1 hour ago"
journalctl --since "2026-09-26 10:00" --until "2026-09-26 11:00"
journalctl -p err -b                   # Errors since this boot
journalctl -b -1                       # Previous boot (after crash)
journalctl -k                          # Kernel messages only
journalctl _PID=1234
journalctl -u app -o json-pretty
journalctl -u app -o cat | grep -i exception
```

| Option | What it does |
|--------|--------------|
| `-u` | Filter logs by systemd unit |
| `-f` | Follow new log entries live |
| `-n N` | Show last N log lines |
| `-p err` | Only priority error and above |
| `-b` / `-b -1` | This boot / previous boot |
| `-k` | Kernel messages only, like dmesg |
| `--since` / `--until` | Time window for the logs |
| `-o json` / `-o cat` | Output format: JSON / message only |
| `-r` | Reverse, newest entries first |
| `--no-pager` | Print directly, don't use less |
| `-x` | Add explanatory help text |

---

## 14. Real-World One-Liners (Log Analysis)

*Nginx combined log format: `$1`=IP · `$4`=time · `$7`=URL · `$9`=status · `$10`=bytes*

```bash
# Top 10 client IPs
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head

# HTTP status code breakdown
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Top URLs returning 5xx
awk '$9 ~ /^5/ {print $7}' access.log | sort | uniq -c | sort -rn | head

# Requests per minute (spot spikes)
awk '{print substr($4,2,17)}' access.log | sort | uniq -c | tail -20

# Errors in the last deployment window
sed -n '/2026-09-26 10:00/,/2026-09-26 10:30/p' app.log | grep -c ERROR

# Most frequent exception types
grep -oE '[A-Za-z.]+Exception' app.log | sort | uniq -c | sort -rn | head

# Failed SSH logins by IP (brute force check)
grep "Failed password" /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head

# Slow requests (> 2s, request_time is last field)
awk '$NF > 2.0 {print $NF, $7}' access.log | sort -rn | head

# Users with bash shell
awk -F: '$7 ~ /bash/ {print $1}' /etc/passwd

# Show effective (non-comment) config
grep -Ev '^\s*(#|;|$)' /etc/ssh/sshd_config

# Replace a value across many files (preview then apply)
grep -rl "old.db.host" /etc/app/ | xargs sed -i.bak 's/old.db.host/new.db.host/g'

# Watch errors live, highlighted
tail -F app.log | grep --line-buffered -iE --color "error|fatal|exception"
```

---

## 15. Interview Questions

1. **Why `sort` before `uniq`?** `uniq` only collapses adjacent duplicates.
2. **`grep -E` vs `grep -F`?** `-E` extended regex; `-F` literal, faster.
3. **Top 10 IPs from access log?** `awk '{print $1}' log | sort | uniq -c | sort -rn | head`
4. **Replace text in a file without opening it?** `sed -i.bak 's/a/b/g' file`
5. **`cut` vs `awk`?** `cut` = single-char delimiter; `awk` handles whitespace, logic, math.
6. **Search rotated `.gz` logs?** `zgrep`
7. **Show 5 lines after each error?** `grep -A 5 ERROR log`
8. **Parse JSON from an API in bash?** `jq`

---

## 16. Cheat Sheet

```bash
grep -rinw -E -v -c -o -l -A -B -C -q --include
cut -d: -f1,3 | sort -rn -h -k -t -u -V | uniq -c -d
tr -d -s 'a-z' 'A-Z'
sed -i.bak 's/a/b/g' | sed -n '10,20p' | sed '/^#/d'
awk -F: '{print $1}' | '$9~/^5/' | '{s+=$10}END{print s}' | '!seen[$0]++'
tee -a | paste -sd, | column -t | comm | split
jq -r '.items[].name' | yq '.spec'
zcat zgrep zless | journalctl -u -f -p err --since
```
