# Chapter 2: Navigating the Linux File System

Mastering Linux begins with knowing how to efficiently navigate and manipulate its file system from the command line. Unlike graphical user interfaces that rely on point-and-click interactions, the Linux shell provides precise, scriptable control over system files and directories.

In this chapter, you will learn the core mechanics of command parsing, how to locate documentation using system tools, the structure of the Filesystem Hierarchy Standard (FHS), and how to execute powerful file management operations using wildcards.

---

## 2.1 Command Line Basics & Variable Manipulation

### Shell Architecture & Command Parsing
When you type a command into a Linux terminal, the **shell** (typically `bash` on Debian systems) reads, parses, and executes the input. Every command follows a standard anatomy:

$$\text{Command} \quad [\text{Options}] \quad [\text{Arguments}]$$

* **Command:** The executable program or built-in operation (e.g., `ls`, `cd`).
* **Options:** Flags that modify how the command behaves, usually preceded by a single hyphen (`-`) for short flags or a double hyphen (`--`) for long flags (e.g., `-l`, `--all`).
* **Arguments:** The targets of the command, such as file paths, directory names, or user accounts.

Commands in Linux fall into two primary categories:
1. **Shell Built-in Commands:** Commands integrated directly into the shell executable itself (e.g., `cd`, `echo`, `pwd`). They execute instantly without spawning a separate system process.
2. **External Executable Binaries:** Independent program files located within the file system (e.g., `/usr/bin/ls`, `/usr/bin/top`).

You can inspect the type and location of any command using `type` and `which`:
```bash
type cd
type ls
which ls
