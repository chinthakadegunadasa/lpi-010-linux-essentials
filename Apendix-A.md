Here is a complete, lightweight, interactive terminal-based exam simulator written in Python 3. It runs natively on Debian 12 (Bookworm) with zero external dependencies.
This script parses a structured test bank covering all core competencies from **Chapters 1 through 6** (FHS, standard I/O pipelines, kernel architecture, POSIX permissions/ACLs, and APT package management). It features dynamic question shuffling, an interactive command-line interface, detailed answer explanations, and automated pass/fail scoring.
### Prerequisites & Setup on Debian
Ensure Python 3 is installed on your Debian system and save the code to a file:
```bash
# Verify Python installation
python3 --version

# Save the code to exam_simulator.py
nano exam_simulator.py

```
### Python Exam Simulator Script (exam_simulator.py)
```python
#!/usr/bin/env python3
"""
Enterprise Linux Technical Certification Exam Simulator
Target OS: Debian 12 (Bookworm) Stable
Description: Interactive terminal-based practice simulator for Enterprise Linux Administration.
"""

import sys
import os
import random
import time

# Questions Bank: Chapters 1-6 Coverage
EXAM_BANK = [
    {
        "id": 1,
        "chapter": "Chapter 1: Introduction to Enterprise Linux",
        "question": "Which software license model requires derivative works to be released under the exact same license terms when modified and distributed?",
        "options": [
            "Permissive License (e.g., MIT)",
            "Strong Copyleft (e.g., GNU GPLv2)",
            "Weak Copyleft (e.g., LGPL)",
            "Public Domain"
        ],
        "answer": 2,
        "explanation": "Strong Copyleft licenses like the GNU GPLv2/v3 enforce that any modified code distributed publicly must also make its source code available under the same license terms."
    },
    {
        "id": 2,
        "chapter": "Chapter 2: Navigating the Linux File System",
        "question": "Which single command copies all files ending in '.txt' from the current directory into /tmp/backup/",
        "options": [
            "mv *.txt /tmp/backup/",
            "cp *.txt /tmp/backup/",
            "cp /tmp/backup/ *.txt",
            "locate *.txt /tmp/backup/"
        ],
        "answer": 2,
        "explanation": "The 'cp' utility copies files. Using the wildcard 'asterisk' (*.txt) matches all filenames ending in .txt."
    },
    {
        "id": 3,
        "chapter": "Chapter 3: The Power of the Command Line",
        "question": "How do you redirect standard error (stderr / FD 2) to a log file while discarding standard output (stdout / FD 1) completely?",
        "options": [
            "command > /dev/null 2> error.log",
            "command 2> /dev/null > error.log",
            "command &> error.log",
            "command | error.log 2> /dev/null"
        ],
        "answer": 1,
        "explanation": "'> /dev/null' redirects standard output (FD 1) to the bit bucket, while '2> error.log' routes standard error (FD 2) to the specified log file."
    },
    {
        "id": 4,
        "chapter": "Chapter 4: Operating System Architecture",
        "question": "What is the Process ID (PID) assigned to the initial user-space process ('systemd') executed by the Linux kernel upon boot?",
        "options": [
            "PID 0",
            "PID 1",
            "PID 100",
            "PID -1"
        ],
        "answer": 2,
        "explanation": "PID 1 is reserved for the initial init system ('systemd') spawned by the kernel to manage system services and state initialization."
    },
    {
        "id": 5,
        "chapter": "Chapter 5: Security and File Permissions",
        "question": "Which special permission bit ensures that files created inside a shared directory automatically inherit the group ownership of that directory?",
        "options": [
            "SUID (Set User ID)",
            "SGID (Set Group ID)",
            "Sticky Bit",
            "POSIX Default ACL"
        ],
        "answer": 2,
        "explanation": "When applied to a directory, SGID (chmod g+s or 2770) ensures that all newly created files inherit the directory's group ownership rather than the primary group of the creating user."
    },
    {
        "id": 6,
        "chapter": "Chapter 6: Software Package Management",
        "question": "Which low-level Debian packaging command lists all files installed onto the file system by a specific package?",
        "options": [
            "dpkg -S <file>",
            "dpkg -L <package>",
            "apt search <package>",
            "dpkg -i <package>"
        ],
        "answer": 2,
        "explanation": "'dpkg -L <package>' queries the local package database to display every file deployed to the system by that package. ('dpkg -S' works in reverse)."
    },
    {
        "id": 7,
        "chapter": "Chapter 3: The Power of the Command Line",
        "question": "Which command combination extracts the 1st and 7th delimited fields from /etc/passwd using ':' as the field separator?",
        "options": [
            "cut -d':' -f1,7 /etc/passwd",
            "grep -d':' -f1,7 /etc/passwd",
            "tr -d':' -f1,7 /etc/passwd",
            "sed -d':' -f1,7 /etc/passwd"
        ],
        "answer": 1,
        "explanation": "The 'cut' tool extracts specific columns or fields. '-d' defines the delimiter character and '-f' specifies target field indexes."
    },
    {
        "id": 8,
        "chapter": "Chapter 5: Security and File Permissions",
        "question": "What is the resulting default permissions mode for a newly created file if the system umask is set to 027?",
        "options": [
            "750 (rwxr-x---)",
            "640 (rw-r-----)",
            "644 (rw-r--r--)",
            "777 (rwxrwxrwx)"
        ],
        "answer": 2,
        "explanation": "Standard files start with a base mode of 666 (rw-rw-rw-). Applying umask 027 subtracts permissions (666 - 027 = 640 or rw-r-----)."
    }
]

def clear_screen():
    """Clear terminal screen for clean presentation."""
    os.system('clear' if os.name == 'posix' else 'cls')

def display_banner():
    """Print the exam header."""
    print("=" * 72)
    print("      ENTERPRISE LINUX ADMINISTRATION CERTIFICATION EXAM SIMULATOR")
    print("                   Debian 12 (Bookworm) Stable Edition")
    print("=" * 72)
    print()

def run_exam():
    clear_screen()
    display_banner()
    
    print("Instructions:")
    print(" - Answer all multiple-choice questions.")
    print(" - Passing score is set at 75%.")
    print(" - Detailed explanations will be displayed post-exam.")
    print("\nPress ENTER to start the exam...")
    input()

    questions = EXAM_BANK.copy()
    random.shuffle(questions)  # Randomize question sequence
    
    score = 0
    total_questions = len(questions)
    user_results = []

    start_time = time.time()

    for idx, q in enumerate(questions, start=1):
        clear_screen()
        display_banner()
        
        print(f"Question {idx} of {total_questions} | [{q['chapter']}]")
        print("-" * 72)
        print(f"\n{q['question']}\n")
        
        for option_idx, option in enumerate(q['options'], start=1):
            print(f"  [{option_idx}] {option}")
            
        print("\n" + "-" * 72)
        
        # User input loop with validation
        while True:
            try:
                choice = input("Select an answer [1-4] and press ENTER: ").strip()
                choice_num = int(choice)
                if 1 <= choice_num <= len(q['options']):
                    break
                else:
                    print("Invalid selection. Please choose a number between 1 and 4.")
            except ValueError:
                print("Invalid input. Please enter a numerical option.")

        is_correct = (choice_num == q['answer'])
        if is_correct:
            score += 1
            
        user_results.append({
            "num": idx,
            "question": q['question'],
            "user_choice": choice_num,
            "correct_choice": q['answer'],
            "is_correct": is_correct,
            "options": q['options'],
            "explanation": q['explanation']
        })

    elapsed_time = round(time.time() - start_time, 1)
    
    # Render Exam Results
    clear_screen()
    display_banner()
    
    percentage = (score / total_questions) * 100
    passed = percentage >= 75.0

    print("EXAM PERFORMANCE SUMMARY")
    print("-" * 72)
    print(f"Time Taken       : {elapsed_time} seconds")
    print(f"Total Questions  : {total_questions}")
    print(f"Correct Answers  : {score}")
    print(f"Final Score      : {percentage:.1f}%")
    print("-" * 72)

    if passed:
        print("\n>>> FINAL VERDICT: PASS <<<")
        print("Congratulations! You have demonstrated core technical competence in Enterprise Linux Administration.")
    else:
        print("\n>>> FINAL VERDICT: FAIL <<<")
        print("Result below 75% threshold. Review the detailed chapter feedback below.")

    print("\nPress ENTER to review detailed question feedback...")
    input()

    # Detailed Question Review Loop
    clear_screen()
    display_banner()
    print("DETAILED QUESTION REVIEW & FEEDBACK")
    print("=" * 72)

    for res in user_results:
        status_str = "[CORRECT]" if res['is_correct'] else "[INCORRECT]"
        print(f"\nQ{res['num']}: {res['question']}  --> {status_str}")
        print(f"    Your Answer   : [{res['user_choice']}] {res['options'][res['user_choice']-1]}")
        print(f"    Correct Answer: [{res['correct_choice']}] {res['options'][res['correct_choice']-1]}")
        print(f"    Explanation   : {res['explanation']}")
        print("-" * 72)

    print("\nSimulator Session Complete.")

if __name__ == "__main__":
    try:
        run_exam()
    except KeyboardInterrupt:
        print("\n\nExam session aborted by user. Exiting cleanly.")
        sys.exit(0)

```
### Execution Instructions
 1. Make the script executable:
   ```bash
   chmod +x exam_simulator.py
   
   ```
 2. Launch the simulator:
   ```bash
   ./exam_simulator.py
   
   ```
 3. **Verification:**
   * After answering all questions, the script calculates your score percentage.
   * A passing grade (\ge 75\%) outputs the PASS decision.
   * Interactive review displays every question alongside your response, the correct choice, and the technical explanation.
