# Chapter 3: The Power of the Command Line
The Linux command line provides powerful text processing tools, stream redirection capabilities, and process management controls. Master administrators leverage these tools to chain simple commands together, transformation-filtering stream data dynamically to execute complex system administration tasks efficiently.
In this chapter, you will master the Linux I/O redirection model, standard POSIX text processing pipelines, regular expression pattern matching, and file viewing utilities on Debian Stable.
## 3.1 Standard I/O Redirection and Pipelines
### The Standard Streams Architecture
In UNIX-like operating systems, every process opened by the shell is automatically provided with three standard I/O data streams, represented by integer file descriptors (FD):
```
                       ┌───────────────────────┐
                       │                       │ ──► Standard Output (FD 1) ──► Screen / File
  Keyboard ──────────► │    Linux Process      │
  (or Input File)      │   (e.g., grep/sed)    │
  Standard Input (FD 0)│                       │ ──► Standard Error  (FD 2) ──► Screen / Log File
                       └───────────────────────┘

```
 * **Standard Input (stdin / FD 0):** The default input stream, normally fed by user keyboard input.
 * **Standard Output (stdout / FD 1):** The default output stream for successful command execution, normally routed to the terminal display.
 * **Standard Error (stderr / FD 2):** The diagnostic stream used to display error messages and logs independently of standard output, normally routed to the terminal display.
### Stream Redirection Operators
You can override standard I/O routes using shell redirection operators:
 * **Redirect stdout to a file (Overwrite):** command > file.txt
 * **Append stdout to a file:** command >> file.txt
 * **Redirect stderr to a file:** command 2> error.log
 * **Append stderr to a file:** command 2>> error.log
 * **Redirect both stdout and stderr to a single file:** command > combined.log 2>&1 or command &> combined.log
 * **Discard output silently:** command > /dev/null 2>&1
 * **Redirect file content into stdin:** command < input.txt
### Pipelines (|)
The pipe operator (|) connects the Standard Output (stdout, FD 1) of one process directly into the Standard Input (stdin, FD 0) of the next process. Pipelines allow you to construct complex data-processing chains from simple tools:
To pipe both stdout and stderr combined into the next command, use |& (or 2>&1 |).
### Hands-On Lab 3.1: Constructing Pipelines and Redirecting Error Streams
In this lab, you will isolate success and error outputs from system commands and build a multi-stage pipeline.
**Step 1: Separate standard output from standard error streams.**
```bash
ls -l /etc/passwd /nonexistent_path > output.log 2> error.log
cat output.log
cat error.log

```
**Step 2: Merge standard error into standard output and discard unwanted data.**
```bash
ls -l /etc/passwd /nonexistent_path > combined.log 2>&1
cat combined.log

# Discard all error output to /dev/null
ls -l /etc/passwd /nonexistent_path 2> /dev/null

```
**Step 3: Construct a multi-stage processing pipeline.**
List all files in /etc, count the total matching entries, and sort them:
```bash
ls /etc | grep "^p" | wc -l

```
## 3.2 Advanced Text Processing Tools
### Standard Text Operations Ecosystem
Debian Linux includes lightweight, highly optimized GNU utilities for transforming text files:
 * **cat:** Concatenates and displays full file contents (cat -n displays line numbers).
 * **tac:** Displays file contents in reverse line order (bottom to top).
 * **head:** Outputs the first lines of a stream (head -n 20 shows the first 20 lines).
 * **tail:** Outputs the last lines of a stream (tail -n 20 shows the last 20 lines; tail -f monitors log growth in real time).
 * **cut:** Extracts designated delimited fields or character positions from file lines (cut -d':' -f1 /etc/passwd).
 * **paste:** Merges lines of files side by side using custom delimiters.
 * **sort:** Sorts lines of text numerically or alphabetically (sort -n numerical; sort -k sort by specific column; sort -r reverse).
 * **uniq:** Filters or reports adjacent duplicate lines (uniq -c counts occurrences; input must be pre-sorted via sort).
 * **wc:** Counts lines, words, and byte/character counts (wc -l counts lines; wc -w counts words).
 * **tr:** Translates or deletes specific target characters from stdin (tr 'a-z' 'A-Z' converts text to uppercase).
 * **tee:** Reads from standard input and writes simultaneously to standard output *and* target files (command | tee output.log).
### Hands-On Lab 3.2: Filtering and Summarizing System Log Data
In this lab, you will extract user accounts, sort attributes, and count system service occurrences.
**Step 1: Extract all username accounts defined on the system.**
```bash
cut -d':' -f1 /etc/passwd | head -n 10

```
**Step 2: Extract usernames alongside their default logins and sort them alphabetically.**
```bash
cut -d':' -f1,7 /etc/passwd | sort

```
**Step 3: Count unique login shells configured across system user accounts.**
```bash
cut -d':' -f7 /etc/passwd | sort | uniq -c | sort -nr

```
**Step 4: Stream and filter output using tee.**
```bash
getent passwd | cut -d':' -f1,3 | sort -t':' -k2 -n | tee active_uids.txt | head -n 5

```
## 3.3 Regular Expressions and Dynamic Pattern Matching
### POSIX Regular Expressions Overview
Regular expressions (Regex) define structured search patterns used by tools like grep, sed, and awk to inspect and transform text data.
#### 1. Basic Regular Expressions (BRE)
Supported by standard grep and sed by default:
| Symbol | Description |
|---|---|
| ^ | Anchors match to the absolute beginning of a line. |
| $ | Anchors match to the end of a line. |
| . | Matches any single character except a newline. |
| * | Matches zero or more occurrences of the preceding character or group. |
| [...] | Matches any single character within the bracket set. |
| [^...] | Matches any single character *not* within the bracket set. |
#### 2. Extended Regular Expressions (ERE)
Enabled via grep -E (or egrep):
| Symbol | Description |
|---|---|
| + | Matches one or more occurrences of the preceding character. |
| ? | Matches zero or one occurrence of the preceding character. |
| | | Logical OR operation (e.g., pattern1|pattern2). |
| () | Groups sub-patterns together for quantifier evaluation. |
| {n,m} | Matches between n and m repetitions of the preceding element. |
### Global Regular Expression Print (grep)
The grep utility searches input files or streams for matching line expressions:
 * grep -i : Performs case-insensitive matching.
 * grep -v : Inverts the match (prints non-matching lines).
 * grep -c : Prints the count of matching lines instead of the actual lines.
 * grep -n : Displays line numbers along with matching lines.
 * grep -r / grep -R : Recursively searches all files inside a directory tree.
 * grep -E : Treats the pattern as an Extended Regular Expression (ERE).
### Hands-On Lab 3.3: Inspecting Configuration Files with grep
In this lab, you will strip comment lines, isolate IP configurations, and run structural regex searches.
**Step 1: Filter out comment lines and blank lines from configuration files.**
```bash
grep -v "^#" /etc/adduser.conf | grep -v "^$"

```
*ERE alternative:*
```bash
grep -E -v "^(#|$)" /etc/adduser.conf

```
**Step 2: Locate all instances of active IP addresses inside system files.**
```bash
grep -E -r "([0-9]{1,3}\.){3}[0-9]{1,3}" /etc/ 2> /dev/null

```
**Step 3: Perform case-insensitive recursive matching.**
```bash
grep -i -rn "debian" /etc/apt/

```
## 3.4 File Viewers, Pagers, and Text Editors
### Interactive Terminal File Viewers
When reading large text files or log streams without loading the entire document into memory, administrators use interactive pagers:
 * **less:** The standard Linux terminal pager. Supports fast navigation, forward/backward keyword searching (/pattern search forward, ?pattern search backward, n next match, N previous match), and efficient memory usage on large files.
 * **more:** Legacy pagination tool; moves forward only through text files.
### Command Line Text Editors (nano & vim)
System administrators rely on command-line text editors to update system configuration files over remote SSH sessions:
```
┌─────────────────────────────────────────────────────────────┐
│                 Vim Operational Modal States                │
│                                                             │
│       ┌──────────────┐   i / a / o    ┌──────────────┐      │
│       │              │ ─────────────► │              │      │
│       │ Command Mode │                │ Insert Mode  │      │
│       │  (Default)   │ ◄───────────── │              │      │
│       └──────┬───────┘      Esc       └──────────────┘      │
│              │                                              │
│              │ :                                            │
│              ▼                                              │
│       ┌──────────────┐                                      │
│       │ Ex / Command │                                      │
│       │  Line Mode   │ (:w, :q, :q!)                        │
│       └──────────────┘                                      │
└─────────────────────────────────────────────────────────────┘

```
#### 1. nano
An intuitive, beginner-friendly editor displaying shortcut keys along the bottom of the screen (e.g., Ctrl+O to save, Ctrl+X to exit).
#### 2. vim (Vi IMproved)
A powerful modal text editor used across enterprise Linux distributions:
 * **Command Mode (Default):** Used for navigation and text manipulation commands (h, j, k, l navigation; dd delete line; yy copy line; p paste).
 * **Insert Mode:** Type i, a, or o to enter insert mode for typing text. Press Esc to return to Command Mode.
 * **Command-Line (Ex) Mode:** Type : from Command Mode to execute file commands:
   * :w : Write (save) the file.
   * :q : Quit the editor.
   * :q! : Quit discarding unsaved changes.
   * :wq or :x : Save changes and quit.
### Hands-On Lab 3.4: Log Navigation and Basic Vim Editing
In this lab, you will search system log archives using less and update configuration properties inside vim.
**Step 1: Inspect dynamic log files using less.**
```bash
less /var/log/dpkg.log

```
*Navigation:* Press G to jump to the end of the file, g to jump to the beginning, type /installed and press Enter to search forward, press q to quit.
**Step 2: Create and modify a file using vim.**
```bash
vim ~/vim_practice.txt

```
 1. Press i to enter **Insert Mode**.
 2. Type: Enterprise Linux Systems Administration - Chapter 3.
 3. Press Esc to return to **Command Mode**.
 4. Type dd to delete the current line.
 5. Type u to undo the previous action.
 6. Type :wq and press Enter to save and exit.
**Step 3: Verify file content creation.**
```bash
cat ~/vim_practice.txt

```
## Chapter 3 Summary Checklist
Before moving on to **Chapter 4: The Linux Operating System Architecture**, make sure you can complete these tasks:
 * [ ] Can you differentiate between standard input (stdin), standard output (stdout), and standard error (stderr) file descriptors?
 * [ ] Do you know how to redirect error streams (2>) separately from standard output (>) and append data (>>)?
 * [ ] Can you chain individual text processing tools (cut, sort, uniq, tr, wc, tee) into pipelines?
 * [ ] Do you understand the difference between BRE and ERE syntax in grep?
 * [ ] Can you perform line filtering, navigate log files with less, and save/quit files using vim?
