# Python Parsing Syntax Cheatsheet

**Python exam (1 hour)**

https://www.geeksforgeeks.org/python/python-execute-and-parse-linux-commands/

https://www.geeksforgeeks.org/python/pathlib-module-in-python/

TIPS

- to execute a command through python you do : `subprocess.check_output(["command", "arg1,arg2,arg3"], text=True)`
- to add something to a set in python, you do `[name of set].add([whatever])`
- if you want to skip the header of an output command do : `result.splitlines()[1:]`
- if you want the first 3 elements of something `[:3]` but if it’s a string you should use `.endswith("[whatever you want]")` instead
- `parts = line.split(None, 3)` where `line` happens in a for loop
- how to increment a Counter() in python? let’s say you have `c = Counter()` , then do :  `c[key] += 1`
- you can force ASCII or unicode :
    - `pstree -A | head   # force ASCII`
    - `pstree -U | head   # force Unicode`
- This is the exact shape you get on Debian 12 when you force ASCII, show PIDs, and show args : `pstree -A -p -a`

<aside>
⚠️

you cannot use `.splitlines()` with files !! so not possible with `sys.stdin`

</aside>

<aside>
⚠️

for IPv4 regex with ‘^ and $’, it matches **only if the entire string** is exactly an IPv4 address!! 

</aside>

- Exercises parsing
    
    parse this and extract fias much info as possible : 
    
    - how many users are there?
    - how many python files are there? how many .txt files?
    - how many directories are there?
    - return a list of which files have read and write access? (files, not directories)
    - what is the heaviest item (can be file or directory)? what is the lightest one?
    
    ```bash
    total 120
    drwxr-xr-x@ 19 raphael  staff   608B Mar 13 18:13 .
    drwx------@ 56 raphael  staff   1.8K Mar 13 18:13 ..
    -rw-r--r--@  1 raphael  staff    10K Mar 13 18:12 .DS_Store
    -rw-r--r--@  1 raphael  staff    87B Mar  7 19:17 input.txt
    -rw-r--r--@  1 raphael  staff    66B Mar 11 20:42 input2.txt
    drwxr-xr-x@  5 raphael  staff   160B Mar  6 19:39 out1
    drwxr-xr-x@  5 raphael  staff   160B Mar 13 18:13 out2
    -rw-r--r--@  1 raphael  staff   560B Mar 11 21:02 parse.py
    -rw-r--r--@  1 raphael  staff   822B Mar  6 19:39 task1.py
    -rw-r--r--@  1 raphael  staff   1.3K Mar  8 14:20 task2_pstree.py
    -rw-r--r--@  1 raphael  staff   1.5K Mar  7 19:56 task2.py
    -rw-r--r--@  1 raphael  staff   1.4K Mar  7 19:55 test.py
    -rw-r--r--@  1 raphael  staff   940B Mar 11 17:15 test2.py
    -rw-r--r--@  1 raphael  staff   811B Mar 11 17:35 test3.py
    -rw-r--r--@  1 raphael  staff   883B Mar 11 18:38 test4.py
    -rw-r--r--@  1 raphael  staff   617B Mar 13 17:01 test5.py
    -rw-r--r--@  1 raphael  staff   668B Mar 13 18:12 test6.py
    drwxr-xr-x@ 10 raphael  staff   320B Mar  6 19:34 testdir
    drwxr-xr-x@  6 raphael  staff   192B Mar 13 18:06 testdire
    ```
    
    - answer
        
        ```python
        import re
        from collections import Counter
        
        s = """PASTE THE ls -l OUTPUT HERE"""
        
        users = set()
        py = 0
        txt = 0
        dirs = 0
        rw_files = []
        sizes = {}  # name -> bytes
        
        def to_bytes(x):
            m = re.fullmatch(r"(\d+(?:\.\d+)?)([BK])", x)
            if not m:
                return None
            n = float(m.group(1))
            u = m.group(2)
            return int(n * (1024 if u == "K" else 1))
        
        for line in s.splitlines()[1:]:  # skip "total ..."
            if not line.strip():
                continue
            parts = line.split(None, 8)
            if len(parts) < 9:
                continue
        
            perm, _, user, _, size, _, _, _, name = parts
            users.add(user)
        
            b = to_bytes(size)
            if b is not None:
                sizes[name] = b
        
            if perm[0] == "d":
                dirs += 1
                continue
        
            if name.endswith(".py"):
                py += 1
            if name.endswith(".txt"):
                txt += 1
        
            if perm[1:3] == "rw":   # owner read+write
                rw_files.append(name)
        
        print("users:", len(users))
        print("python_files:", py)
        print("txt_files:", txt)
        print("directories:", dirs)
        print("rw_files:", rw_files)
        
        heaviest = max(sizes, key=sizes.get)
        lightest = min(sizes, key=sizes.get)
        print("heaviest:", heaviest, sizes[heaviest], "bytes")
        print("lightest:", lightest, sizes[lightest], "bytes")
        ```
        
    
    ## **Exercises: parse these inputs in Python (no solutions)**
    
    Rules for all exercises:
    
    - Input is a **single string** (like text = """...""").
    - You must parse it into Python structures (dict/list/sets), then compute the requested outputs.
    - Use only: splitlines(), split(...), maxsplit, re, shlex, Counter, defaultdict, set, dict.
    
    ---
    
    ## **Exercise 1 — ps style with free-form tail (maxsplit)**
    
    **Input**
    
    ```
    44173 44172 raphael -/bin/zsh
    65180 65179 root ps -axwwo user,pid,ppid,pgid,command
    1 0 root /sbin/init splash
    ```
    
    **Task**
    
    Parse each line into (pid:int, ppid:int, user:str, cmd:str).
    
    Compute:
    
    1. number of processes
    2. unique users
    3. top 2 most common executables (cmd.split()[0])
    
    ---
    
    ## **Exercise 2 — ps with missing cmd**
    
    **Input**
    
    ```
    100 1 root
    101 1 daemon /usr/sbin/cron -f
    102 101 daemon
    ```
    
    **Task**
    
    Handle lines that may have only 2 or 3 fields.
    
    Compute:
    
    1. how many lines have an empty cmd
    2. unique users
    3. count per executable (use "<empty>" when cmd missing)
    
    ---
    
    ## **Exercise 3 — CSV-like but cmd contains commas (hard)**
    
    **Input**
    
    ```
    2001,1999,alice,/usr/bin/python3 script.py --arg "a,b,c"
    2002,1999,alice,/bin/bash -lc echo,hello
    2003,2001,bob,/usr/bin/ssh bob@host "echo hi,there"
    ```
    
    **Task**
    
    You cannot .split(",") naively.
    
    Parse into 4 fields: pid,ppid,user,cmd.
    
    Compute:
    
    1. unique command lines
    2. for each user, how many processes
    3. edges by PPID: ppid -> [pid...]
    
    ---
    
    ## **Exercise 4 — key=value logs (unordered fields)**
    
    **Input**
    
    ```
    ts=1710000001 pid=400 user=alice cmd="/bin/bash -lc 'echo hi'"
    pid=401 cmd="/usr/bin/python3 a.py" ts=1710000002 user=alice
    user=bob ts=1710000003 cmd="/bin/sh -c whoami" pid=402
    ```
    
    **Task**
    
    Parse into dicts with keys ts,pid,user,cmd (types: ts/pid int).
    
    Compute:
    
    1. earliest timestamp
    2. unique commands
    3. list of pids for user=alice
    
    ---
    
    ## **Exercise 5 — quoted fields (use shlex)**
    
    **Input**
    
    ```
    pid=500 user="alice smith" cmd="python3 script.py --name 'A B'"
    pid=501 user="bob" cmd="bash -lc \"echo hello world\""
    ```
    
    **Task**
    
    Parse key/value pairs correctly so user can contain spaces.
    
    Compute:
    
    1. unique users
    2. executable counts (cmd first token)
    
    ---
    
    ## **Exercise 6 — indentation-based tree (spaces)**
    
    **Input**
    
    ```
    /oon.sh user1
      /lin.sh user2
        /child.sh user2
      /script.sh user1
    ```
    
    **Task**
    
    Parse into parent→children edges.
    
    Compute:
    
    1. total nodes
    2. unique scripts
    3. direct children of /oon.sh
    
    ---
    
    ## **Exercise 7 — Linux pstree -A ASCII connectors**
    
    **Input**
    
    ```
    |-gdm3,1754
    |  |-gdm-session-wor,2824
    |  |  `-gnome-session-b,2923 --session=ubuntu
    |  `-{gdm3},1757
    `-cron,1723 -f -P
    ```
    
    **Task**
    
    Parse depth from the prefix (|   blocks and  `-).
    
    Extract:
    
    - name (before comma)
    - pid (after comma)
    - args (anything after PID)
    
    Compute:
    
    1. edges parent_pid -> child_pid
    2. count of unique names (ignore {...} threads or include them—your choice, but be consistent)
    
    ---
    
    ## **Exercise 8 — Linux Unicode pstree box drawing**
    
    **Input**
    
    ```
    systemd─┬─NetworkManager───{NetworkManager}
            └─cron
    ```
    
    **Task**
    
    Parse nodes separated by ─ and depth from leading spaces.
    
    Compute:
    
    1. edges parent -> child by name
    2. unique names
    
    ---
    
    ## **Exercise 9 — PPID tree reconstruction (classic)**
    
    **Input**
    
    ```
    pid=10 ppid=1 user=root cmd=/sbin/daemon
    pid=11 ppid=10 user=root cmd=/usr/bin/python3 a.py
    pid=12 ppid=10 user=alice cmd=/bin/bash -lc echo hi
    pid=13 ppid=12 user=alice cmd=/bin/sleep 5
    ```
    
    **Task**
    
    Build children[ppid].
    
    Compute:
    
    1. subtree size under pid=10 (including all descendants)
    2. list descendants of pid=10 in DFS order
    3. unique users in that subtree
    
    ---
    
    ## **Exercise 10 — Mixed whitespace + tabs (robust splitting)**
    
    **Input**
    
    ```
    700\t1\troot\t/usr/sbin/sshd -D
    701    700   root    sshd: root@pts/0
    702\t700\talice\tsshd: alice@pts/1
    ```
    
    (Those \t are literal tab characters.)
    
    **Task**
    
    Parse into (pid,ppid,user,cmd) reliably.
    
    Compute:
    
    1. how many children does pid=700 have
    2. unique users
    
    ---
    
    ## **Exercise 11 — Extract scripts only (regex)**
    
    **Input**
    
    ```
    800 1 alice /bin/bash /home/alice/run.sh --fast
    801 1 alice python3 /home/alice/run.py
    802 1 bob /usr/bin/env bash -lc /opt/tools/deploy.sh stage
    803 1 bob /bin/zsh
    ```
    
    **Task**
    
    Extract only .sh script paths (not .py).
    
    Compute:
    
    1. unique .sh scripts
    2. users per script
    
    ---
    
    ## **Exercise 12 — “pstree FreeBSD-like” fixed fields after connector**
    
    **Input**
    
    ```
    |-+= 11335 raphael /Applications/Ghostty.app/Contents/MacOS/ghostty
    | \--= 44173 raphael -/bin/zsh
    \--- 67641 root ps -axwwo user,pid,ppid,pgid,command
    ```
    
    **Task**
    
    Parse:
    
    - depth from connector prefix
    - pid (int)
    - user (str)
    - cmd (rest)
    
    Compute:
    
    1. unique users
    2. unique executables (cmd.split()[0])
    3. edges parent_cmd -> child_cmd
    
    ---
    
    If you want a structured progression: pick **3 exercises** you want to start with (easy/medium/hard), and paste your attempt for each; I’ll review only the parsing logic and point out exact failure cases without giving the final solution unless you ask.
    
- Exercises regex
    
    ## **Regex parsing exercises (Python re) — with specific inputs, no solutions**
    
    Use: re.search, re.findall, re.finditer, re.sub, re.compile, groups (...), named groups (?P<name>...), anchors ^ $, boundaries \b, flags.
    
    For each exercise: write a regex, apply it, and produce the requested output.
    
    ---
    
    ## **Exercise 1 — Extract IPv4 candidates**
    
    **Input**
    
    ```
    ok 192.168.1.1
    bad 999.1.2.3
    edge 0.0.0.0
    dns 8.8.8.8 and 1.1.1.1
    ```
    
    **Task**
    
    1. Extract all IPv4-like strings (d{1,3} dotted quad).
    2. Then filter to valid 0–255 per octet (do this either by stricter regex or by post-validate in Python).
    
    ---
    
    ## **Exercise 2 — Parse Apache/Nginx combined log (named groups)**
    
    **Input**
    
    ```
    127.0.0.1 - - [10/Mar/2026:12:00:01 +0000] "GET /index.html HTTP/1.1" 200 1024
    10.0.0.5 - john [10/Mar/2026:12:00:02 +0000] "POST /login HTTP/1.1" 403 531
    ```
    
    **Task**
    
    Extract fields using **named groups**:
    
    - ip
    - user (the authenticated user field, - if missing)
    - timestamp (the bracket content)
    - method
    - path
    - status (int)
    - bytes (int)
    
    ---
    
    ## **Exercise 3 — Extract URLs and domains**
    
    **Input**
    
    ```
    Visit https://example.com/a?x=1 and http://sub.test.co.uk:8080/path.
    Also mailto:test@example.com should NOT count as http URL.
    ```
    
    **Task**
    
    1. Extract all http/https URLs.
    2. From those URLs, extract only the domain (host) part (no port).
    
    ---
    
    ## **Exercise 4 — Capture PID, user, command from “FreeBSD-like pstree” lines**
    
    **Input**
    
    ```
    |-+= 11335 raphael /Applications/Ghostty.app/Contents/MacOS/ghostty
    | \--= 44173 raphael -/bin/zsh
    \--- 67641 root ps -axwwo user,pid,ppid,pgid,command
    ```
    
    **Task**
    
    Write one regex that extracts:
    
    - pid (digits)
    - user (non-space)
    - cmd (rest of line)
    
    Ignore the connector prefix.
    
    ---
    
    ## **Exercise 5 — Validate a simple username rule**
    
    **Input**
    
    ```
    valid: alice
    valid: alice_123
    valid: bob.smith
    invalid: -alice
    invalid: alice-
    invalid: ALICE
    invalid: a
    ```
    
    **Task**
    
    Match usernames that:
    
    - are 2–20 chars
    - lowercase letters, digits, underscore, dot
    - cannot start or end with _ or .
        
        Return only valid usernames.
        
    
    ---
    
    ## **Exercise 6 — Extract key=value pairs (quoted + unquoted)**
    
    **Input**
    
    ```
    pid=500 user="alice smith" cmd="python3 a.py --x=1"
    pid=501 user=bob cmd=/bin/sh
    ```
    
    **Task**
    
    Extract key/value pairs where value may be:
    
    - "quoted string" (may contain spaces)
    - or unquoted token (no spaces)
    
    Return a dict per line.
    
    ---
    
    ## **Exercise 7 — Replace secrets (redaction with re.sub)**
    
    **Input**
    
    ```
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.abcd.efgh
    api_key=sk_live_1234567890abcdef
    password = "hunter2"
    ```
    
    **Task**
    
    Redact:
    
    - Bearer tokens → Bearer [REDACTED]
    - api_key=... → api_key=[REDACTED]
    - password = ... → password=[REDACTED] (keep key, replace value)
    
    ---
    
    ## **Exercise 8 — Greedy vs non-greedy (HTML-ish tags)**
    
    **Input**
    
    ```
    <div>one</div><div>two</div><div>three</div>
    ```
    
    **Task**
    
    Extract the inner text of each <div>...</div> as separate matches.
    
    You must avoid one big greedy match.
    
    ---
    
    ## **Exercise 9 — Extract dates in multiple formats**
    
    **Input**
    
    ```
    2026-03-11
    11/03/2026
    Mar 11, 2026
    bad: 2026-13-99
    ```
    
    **Task**
    
    Extract all date strings and normalize them to YYYY-MM-DD (regex + Python logic).
    
    ---
    
    ## **Exercise 10 — Parse ps line with cmd containing spaces (regex-only)**
    
    **Input**
    
    ```
    65180 65179 root ps -axwwo user,pid,ppid,pgid,command
    44173 44172 raphael -/bin/zsh
    ```
    
    **Task**
    
    Use a regex (not .split) to capture:
    
    - pid (group 1)
    - ppid (group 2)
    - user (group 3)
    - cmd (group 4, rest of line)
    
    ---
    
    ## **Exercise 11 — Find suspicious file paths**
    
    **Input**
    
    ```
    ok /usr/bin/python3
    ok /home/alice/script.sh
    sus /tmp/.hidden/run.sh
    sus /var/tmp/../../etc/passwd
    sus /dev/fd/3
    ```
    
    **Task**
    
    Match lines with “suspicious” paths:
    
    - contain ..
    - or start with /tmp/ or /var/tmp/
    - or start with /dev/fd/
        
        Return the matched paths and the reason (you can use multiple regexes).
        
    
    ---
    
    ## **Exercise 12 — Balanced-ish: extract JSON objects embedded in text (best-effort)**
    
    **Input**
    
    ```
    INFO x=1 payload={"a":1,"b":2} done
    WARN payload={"nested":{"x":1}} ignore
    ```
    
    **Task**
    
    Extract the JSON substring after payload= on each line.
    
    (Assume no spaces inside the JSON; do not attempt full JSON parsing with regex.)
    
    ---
    
    If you want, pick **3 exercise numbers** and paste your regex attempts; I’ll debug them by pointing out exactly what they match/miss and why (no final regex unless you ask).
    

REGEX :

- every string starts with `r’^` and ends with `$` if you want to match the exact regex you specify
- `[…]+` means one or more characters (so at least one)
- to use a normal dot `.` you must escape it first : `\.`
- `|` pipes means OR
- `?` means multiple things such as optional (when used like this : `[…]?`), changes meaning inside () special group modifiers, etc
- A character class like `[0-255]` does not mean 0–255; it means “one character that is 0, 1, 2, 5, or -”, because character classes match **single characters**, not numbers.

`email_pattern = re.compile(r'^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$')`

date_month_year = re.compile(r’^[1-2][0

**LEARN PYTHON PARSING** 

- basics
    
    You’re being tested on writing **Python 3 scripts that behave like shell scripts**:
    
    •	Traverse directories, read/write files, create output files
    
    •	Filter with **regex**
    
    •	Parse semi-structured text (like process trees / command outputs)
    
    •	Produce counts and summaries
    
    •	Be robust: handle missing files, weird lines, whitespace, etc.
    
    •	Use stdin/stdout like Unix tools (optional, but often helpful)
    
    In practice, you want to be comfortable with:
    
    •	`pathlib` (file traversal, paths)
    
    •	`re` (regex)
    
    •	`argparse` (optional but nice)
    
    •	`sys.stdin.read()` / `sys.argv`
    
    •	collections (`defaultdict`, `Counter`)
    
    •	basic data structures (dict/set/list)
    
    **File/dir operations**
    
    •	`Path(".").rglob("*.txt")` → performs a recursive search, making it easy to scan entire directory trees for specific file types.
    
    •	`read_text()`, `write_text()`, `open()`
    
    •	`mkdir(exist_ok=True)`
    
    •	`Path.cwd()` → returns a new path object which represents current working directory. Will give us path from where your Python script is executed.
    
    •	sorting paths deterministically
    
    read and writing to a file
    
    ```python
    # Reading from a file
    with (p / 'example.txt').open('r') as file:
        content = file.read()
        print(content)
    
    # Writing to a file
    with (p / 'output.txt').open('w') as file:
        file.write("Hello, GFG!")
    ```
    
    .read() is a method of a **file object**, not of a Path.
    
    - p is a Path. Doing p.read() fails because Path has no .read() method.
    - open(...) returns a **file handle** (a file object). That object *does* have .read().
    
    **Regex**
    
    •	`re.compile(r"...")` 
    
    •	`re.search()` vs `re.match()`
    
    •	`re.findall()` 
    
    •	anchors ^ $, word boundary `\b`
    
    •	escaping dot \.
    
    **Parsing**
    
    •	`.shlex.split()` → tokenizes a line of text into a list of arguments while respecting shell quoting rules
    
    •	`.splitlines()` → splits a string into a list (in bytes!!). The splitting is done at line breaks. (’hello\n how are you’ becomes {”hello”, “how are you”})
    
    •	`.strip()` → removes any leading, and trailing whitespaces
    
    •	`.lstrip()` → removes any leading characters, space is the default leading character to remove
    
    •	`.rstrip()` → removes any trailing characters (characters at the end a string), space is the default trailing character to remove.
    
    •	`.split(separator, maxsplit)` robust tokenization (tips : use `None` instead of `“ “` if you don’t know how many whitespaces there are)
    
    •	indentation-based parent tracking with a stack
    
    **Counting**
    
    •	`set()` for unique (you populate it by doing [name of the set].add())
    
    •	`Counter` for frequencies
    
    •	`defaultdict`(set/list)
    

You have to write your scripts with nano in the terminal.

You are not given files

You don’t have the actual files you could test.

You are allowed during the exam to open an online IDE.

input = open file

input =" jhsjhrjwherjhj ejrhiwe\n hdjqhdjhjd bhjwe\n"

# assumption : I tested with this input, my assumption is that the input from open file resembles my test data

**First task (30min)**

When you filter files, you use regex,

Read and write in a file, in a folder,

In a folder, you have different IPs and some match. Take those that matches and concatenate in one file of each category

A.txt

A.txt

A.txt

B.txt

B

C

C

C

C

C

Take all .txt to concatenate into one file A, one file B, and one file C.

- Script
    
    ```python
    #!/usr/bin/env python3
    from pathlib import Path
    from collections import defaultdict
    import re, sys
    
    IP = re.compile(r"((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)") # we want it to work with substring, it may match within x192.168.0.1y
    
    root = Path(sys.argv[1]) if len(sys.argv) > 1 else Path(".") # if the script was called with a first argument, use it as the directory path: sys.argv[1]. Otherwise default to the current directory "."
    out  = Path(sys.argv[2]) if len(sys.argv) > 2 else Path("out") # Sets output directory. If a second argument is provided, use it. Else default to out in the current working directory
    out.mkdir(exist_ok=True) # creates the output directory if it doesn’t exist.
    
    G = defaultdict(list) # creates a dictionary where keys will be group labels (like "A", "B", "C"), values default to empty lists automatically
    
    for f in root.rglob("*.txt"): # recursively walks root and all subdirectories, yielding every file path f whose filename matches *.txt.
        content = f.read_text(errors="replace") # reads the entire file f as text into string t.
        if IP.search(content): # checks whether the regex matches anywhere in the file content.
            G[f.name[0].upper()].append(content)   # 1
    
    # 1
    # if the file contains an IP, group it by the first character of its fil
    # f.name is the filename only (e.g., "A123.txt")
    # [0] takes the first character (e.g., "A")
    # .upper() forces uppercase
    # .append(t) stores the file’s content in that group’s list
    # So all matching files starting with A... get their contents appended to G["A"], etc.
    
    for k, list_file_contents in G.items(): # iterates over each group. k is the group key ("A", "B", …). texts is the list of file contents for that group
        with (out / f"{k}.txt").open("w") as w: # 2
            w.write("\n".join(i.rstrip("\n") for i in list_file_contents) + "\n") # 3
    
    # 2
    # opens an output file for that group, in write mode:
    # out / f"{k}.txt" builds a path like out/A.txt (you have to write f"" otherwise it'll just print {k}.txt literally)
    # .open("w") opens for writing and truncates existing content
    # with ... as w: ensures it closes properly
    
    # 3
    # writes all collected file contents into the group output file, separated cleanly by newlines.
    # s.rstrip("\n") removes trailing newline characters from each file’s content so you don’t get double blank lines
    # "\n".join(...) concatenates (links together) all contents with exactly one newline between them
    # + "\n" ensures the final output file ends with a newline
    ```
    
    ```bash
    # no parenthesis around the printfs !!!
    mkdir -p testdir
    printf "hello 1.2.3.4\n" > testdir/A.txt
    printf "no ip here\n" > testdir/A2.txt
    printf "ip 8.8.8.8\n" > testdir/B.txt
    python3 task1.py testdir 
    ```
    

IP regex :  `^((25[0-5] | 2[0-4][0-9] | [01]? [0-9] [0-9]? )\.){3}    (25[0-5] | 2[0-4][0-9] | [01]?[0-9][0-9]?)`

`25[0-5]`→ matches 250 …. 255

`2[0-4][0-9]`→ matches 200 …. 249

`[01]? [0-9] [0-9]?` → matches 0 …. 99 and 100 …. 199.        [optional leading 0 or 1], [required digit], [optional digit]
`\.` → we have to escape the dot 

`{3}` → means repeat three times an octet following by a dot

same thing for the fourth octet, except it’s not followed by a dot so it has to have its own regex 

- Example
    
    Input: 192.168.0.1
    
    ### **Start anchor ^**
    
    We’re at the beginning of the string.
    
    ### **First repeated block (...\.){3}**
    
    **1st octet: 192**
    
    Try alternations in order:
    
    - 25[0-5]? No (doesn’t start with 25)
    - 2[0-4][0-9]? No (doesn’t start with 2)
    - [01]?[0-9][0-9]?? Yes:
        - [01]? matches 1
        - [0-9] matches 9
        - [0-9]? matches 2
            
            So octet1 = 192
            
    
    Then \. matches the dot after it.
    
    So we’ve matched: 192.
    
    ### **2nd octet: 168**
    
    Same check:
    
    - 25[0-5]? No
    - 2[0-4][0-9]? No
    - [01]?[0-9][0-9]?? Yes:
        - optional [01]? matches 1
        - required digit matches 6
        - optional digit matches 8
    
    Match 168.
    
    ### **3rd octet: 0**
    
    - 25[0-5]? No
    - 2[0-4][0-9]? No
    - [01]?[0-9][0-9]?? Yes:
        - optional [01]? can match nothing
        - required [0-9] matches 0
        - optional [0-9]? matches nothing
    
    Match 0.
    
    At this point the {3} repetition is satisfied. We matched: 192.168.0.
    
    ### **Final octet: 1**
    
    Same logic as above:
    
    - [01]?[0-9][0-9]? matches 1
    
    ### **End anchor $**
    
    We are at end of string → match succeeds.
    
    ---
    
    ## **Why it rejects invalid example: 256.1.2.3**
    
    Try first octet 256:
    
    - 25[0-5] would match 25 then expects [0-5]; last digit is 6 → fails
    - 2[0-4][0-9] expects second digit 0..4; here second digit is 5 → fails
    - [01]?[0-9][0-9]? can only make 0–199 or 000–199 with leading zeros; it can’t match 256 → fails
    
    No alternative matches → whole regex fails immediately.
    

**Second task (30min)**

if the input is just a .txt file with a ‘tree-like’ structure :

- script 1
    
    input :
    
    ```bash
    python3 task2.py << 'EOF'
    /boot.sh root
    /setup.sh root
        /net.sh root
            /dhcp.sh root
            /dns.sh root
        /login.sh alice
            /shell.sh alice
                /scan.sh alice
                /report.sh alice
    /cron.sh root
        /backup.sh root
        /cleanup.sh root
    /oon.sh bob
        /lin.sh carol
        /script.sh bob
    EOF
    ```
    
    ```python
    import sys
    from collections import defaultdict, Counter
    
    E = defaultdict(set)            # stores scripts invocation relationships {parent : {child}}
    U = defaultdict(set)            # maps scripts to set of users than ran them
    scripts = Counter()             # counts how many times each script appears in the input
    users = Counter()               # counts how many lines each user appears on
    stack = []                      # a stack that stores (indent_level, script_name)
    
    for r in sys.stdin:             # reads input line by line from stdin
    if not r.strip():       # skip empty lines (or lines with only whitespace)
    continue
        indent = len(r) - len(r.lstrip(" \\t") # number of leading spaces/tabs (tree depth)
        p = r.split()           # splits the line into tokens like ["/test.sh", "user1"]
        s = p[0]                # takes first token as the script path/name
        u = p[-1]               # takes the last token as the user (can do p[1] but [-1] means last token)
    
        scripts[s] +=1          # increments count for this script occurrence
        users[u] +=1            # increments count for this user occurrence
        U[s].add(u)             # records that user u runs script s (set prevents duplicates)
    
        while stack and stack[-1][0] >= indent: # moves up the tree until top stack is a parent of the current line
                stack.pop()
        if stack:
                E[stack[-1][1]].add(s) # we add the value 'child script' (s) to the key 'parent script' (stack[-1][0]) in the E dictionary
        stack.append((indent, s))      # append the tuple {indentation level, script name}
        
    print("running:", sum(scripts.values()))
    print("unique_scripts:", len(scripts))
    print("unique_users:", len(users))
    print("users_per_script:", {k: len(v) for k, v in U.items()})
    print("invokes:", {k: sorted(v) for k, v in E.items()})
    
    # example : stack = [(0,"/foo.sh"), (4,"/lin.sh")]
    
    # E = { "\foo.sh": {"/lin.sh"}}
    ```
    
    In python, keep in mind that [-1] means last element, however in a stack it means the top of the stack (because most recent element is the last one pushed)
    
    stack[-1][0] means : for the top element (tuple) of the stack, select the first ([0]) element, which is the indentation level (first element of the tuple)
    
    stack[-1][1] means : for the top element (tuple) of the stack, select the second element ([1]), which is the script name (second element of the tuple)
    

if the input is an actual pstree that you need to do yourself, then : 

- script 2
    
    **MacOS version** : run this as `pstree | python3 parse.py` (pstree in linux gives unicode tree structure, not ascii) 
    
    ```python
    import sys, re
    from collections import defaultdict
    
    E = defaultdict(list)
    stack = []
    SEP = re.compile(r"─+")
    
    for line in sys.stdin:
        if not line.strip(): 
            continue
        depth = line.count("│")
        parts = [p for p in SEP.split(line.strip()) if p]
        for i, raw in enumerate(parts):
            name = raw.strip()
            if not name: 
                continue
            if depth and len(stack) >= depth:
                E[stack[depth-1]].append(name)
            if i == 0:
                stack = stack[:depth] + [name]
    
    for p, cs in E.items():
        print(p, "->", cs)
    ```
    
    **Linux pstree** : a command like `pstree -p -a` can answer which script invokes which, how many are running, how many different ones run but it doesn’t answer how many users run the script because it doesn’t have a user field.
    
    run it as `pstree -p -a | python3 parse_pstree.py` 
    
    ```python
    import sys
    from collections import Counter, defaultdict
    
    E = defaultdict(set)
    names = Counter()
    n = 0
    stack = []
    
    for line in sys.stdin:
    
        if "," not in line: 
            continue
            
        d = line.count("│")
        name = line.split(",", 1)[0].strip(" │├└─┬\n")
        
        n += 1
        names[name] += 1
        
        while stack and stack[-1][0] >= d: 
    			stack.pop()
        
        if stack: 
    	    E[stack[-1][1]].add(name)
    	    
        stack.append((d, name))
    
    print("nodes:", n)
    print("unique_names:", len(names))
    print("top10:", names.most_common(10))
    print("invokes:", {p: sorted(c) for p, c in E.items
    ```
    
    `ps -eo pid,ppid,user,args --no-headers`
    
    ```python
    import subprocess
    from collections import Counter
    
    scripts = Counter()   # or cmdlines
    users = Counter()
    pids = set()
    
    out = subprocess.check_output(
        ["ps", "-eo", "pid,user,args", "--no-headers"],
        text=True,
        errors="replace",
    )
    
    for line in out.splitlines():
        parts = line.split(None, 2)
        if len(parts) < 2:
            continue
        pid = parts[0]
        user = parts[1]
        cmd = parts[2] if len(parts) > 2 else ""
    
        pids.add(pid)
        users[user] += 1
        scripts[cmd] += 1
    
    print("how many instances are running? ", len(pids))
    print("how many commands are running? ", len(scripts))
    print("how many unique users are there? ", len(users))
    ```
    
    the command `ps -eo pid,ppid,user,args --no-headers` can answer all the questions, but its not a tree.
    
    ```python
    import subprocess, re
    from collections import Counter, defaultdict
    
    out = subprocess.check_output(
        ["ps", "-eo", "pid,user,args", "--no-headers"], # argv list for the command
        text=True, # return output as str, not bytes
        errors="replace", # if decoding hits invalid bytes, replace them instead of crashing
    )
    SCRIPT = re.compile(r"(/\S+\.sh)\b") # compiles a regex used to locate a shell-script path in each process’s command line.
    # \S+ means one or more non-whitespace haracters
    # \b word boundary to avoid matching .shXYZ as .sh
    # adjust if scripts aren't .sh
    
    pids = set() # empty set to store pids
    sc = Counter()
    uc = Counter()
    U = defaultdict(set) # how many different users run each script. it maps script → set of users.
    
    for line in out.splitlines(): # splitlines() returns a list of lines without the trailing newline characters.
        pid, user, *args = line.split(None, 2) # splits into tokens by splitting on any whitespace + perform at most 2 splits
        m = SCRIPT.search(args[0] if args else "") # searches the command string for a .sh path
        if not m:
            continue # if not script found in that line, skip the line 
        s = m.group(1) # extract the captured part of the line from the SCRIPT regex match above
        pids.add(pid)
        sc[s] += 1
        uc[user] += 1
        U[s].add(user) # adds the user to the set for that script
    
    print("running_instances:", len(pids))
    print("unique_scripts:", len(sc))
    print("unique_users:", len(uc))
    print("users_per_script:", {k: len(v) for k, v in U.items()})
    ```
    

« Create a program that has this … input (like the picture below) but not the file

/ lin.sh	user 1

/oon.sh 	user 1

/lin.sh	user 2

/script.sh	user 1

Oon invokes /lin.sh and /script.sh.

<aside>
⚠️

Unicode box drawing (─, ├, └) (linux default pstree structure) is not the same as the classic ASCII |- / \_ 

</aside>

![06000.png](Python%20Parsing%20Syntax%20Cheatsheet/06000.png)

Which script invokes which scripts. You have to count how many scripts there are, how many are running, how many different ones run, how many users run the script, etc etc .

Let’s say -csh invokes script 1 and script 2, print the subscripts (scrip1 invoked script 2 and 3, etc)

Parse data …

Parse command de pid voir combien de script différent run dans le user program

```bash
ps -eo pid,ppid,user,cmd --no-headers
      1       0 root     /sbin/init
      2       0 root     [kthreadd]
      3       2 root     [pool_workqueue_release]
      4       2 root     [kworker/R-rcu_g]
      5       2 root     [kworker/R-rcu_p]
      6       2 root     [kworker/R-slub_]
      7       2 root     [kworker/R-netns]
      8       2 root     [kworker/0:0-cgroup_destroy]
      9       2 root     [kworker/0:0H-kblockd]
     10       2 root     [kworker/0:1-cgroup_destroy]
     11       2 root     [kworker/u4:0-flush-8:0]
     12       2 root     [kworker/R-mm_pe]
     13       2 root     [rcu_tasks_kthread]
     14       2 root     [rcu_tasks_rude_kthread]
     15       2 root     [rcu_tasks_trace_kthread]
     16       2 root     [ksoftirqd/0]
     17       2 root     [rcu_preempt]
     18       2 root     [migration/0]
     19       2 root     [idle_inject/0]
     20       2 root     [cpuhp/0]
     21       2 root     [cpuhp/1]
     22       2 root     [idle_inject/1]
     23       2 root     [migration/1]
     24       2 root     [ksoftirqd/1]
     25       2 root     [kworker/1:0-cgroup_destroy]
     26       2 root     [kworker/1:0H-kblockd]
     27       2 root     [kdevtmpfs]
     28       2 root     [kworker/R-inet_]
     29       2 root     [kworker/u4:1-events_unbound]
     30       2 root     [kauditd]
     31       2 root     [khungtaskd]
     32       2 root     [oom_reaper]
     33       2 root     [kworker/R-write]
     34       2 root     [kcompactd0]
     35       2 root     [ksmd]
     36       2 root     [khugepaged]
     37       2 root     [kworker/R-kinte]
     38       2 root     [kworker/R-kbloc]
     39       2 root     [kworker/R-blkcg]
     40       2 root     [kworker/R-tpm_d]
     41       2 root     [kworker/R-ata_s]
     42       2 root     [kworker/R-md]
     43       2 root     [kworker/R-md_bi]
     44       2 root     [kworker/R-edac-]
     45       2 root     [kworker/R-devfr]
     46       2 root     [watchdogd]
     47       2 root     [kworker/0:1H-kblockd]
     48       2 root     [kworker/1:1-events]
     49       2 root     [kswapd0]
     50       2 root     [ecryptfs-kthread]
     51       2 root     [kworker/R-kthro]
     52       2 root     [irq/12-ACPI:Ged]
     53       2 root     [kworker/R-acpi_]
     54       2 root     [kworker/u4:2-flush-8:0]
     55       2 root     [kworker/u4:3-ext4-rsv-conversion]
     56       2 root     [kworker/R-mld]
     57       2 root     [kworker/R-ipv6_]
     64       2 root     [kworker/R-kstrp]
     66       2 root     [kworker/u5:0]
     68       2 root     [kworker/0:2-virtio_vsock]
     70       2 root     [kworker/R-charg]
    110       2 root     [kworker/1:1H]
    116       2 root     [scsi_eh_0]
    117       2 root     [kworker/R-scsi_]
    118       2 root     [scsi_eh_1]
    119       2 root     [kworker/R-scsi_]
    120       2 root     [scsi_eh_2]
    122       2 root     [kworker/R-scsi_]
    124       2 root     [scsi_eh_3]
    125       2 root     [kworker/R-scsi_]
    130       2 root     [kworker/1:2-events]
    132       2 root     [scsi_eh_4]
    133       2 root     [kworker/R-scsi_]
    134       2 root     [scsi_eh_5]
    138       2 root     [kworker/R-scsi_]
    141       2 root     [kworker/u4:4-flush-8:0]
    142       2 root     [kworker/u4:5-flush-8:0]
    143       2 root     [kworker/u4:6-ext4-rsv-conversion]
    144       2 root     [kworker/u4:7-events_power_efficient]
    400       2 root     [kworker/0:3-events]
    435       2 root     [kworker/R-raid5]
    478       2 root     [jbd2/sda2-8]
    479       2 root     [kworker/R-ext4-]
    548       1 root     /usr/lib/systemd/systemd-journald
    571       2 root     [kworker/1:3-virtio_vsock]
    572       2 root     [kworker/R-kmpat]
    573       2 root     [kworker/R-kmpat]
    600       1 root     /sbin/multipathd -d -s
    621       1 root     /usr/lib/systemd/systemd-udevd
    626       2 root     [psimon]
    679       2 root     [kworker/0:2H-kblockd]
   1005       1 systemd+ /usr/lib/systemd/systemd-oomd
   1007       1 systemd+ /usr/lib/systemd/systemd-resolved
   1033       2 root     [kworker/0:4-events]
   1145       1 avahi    avahi-daemon: running [ubuntu-linux-2404.local]
   1146       1 message+ @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
   1150       1 gnome-r+ /usr/libexec/gnome-remote-desktop-daemon --system
   1155       1 polkitd  /usr/lib/polkit-1/polkitd --no-debug
   1158       1 root     /usr/libexec/power-profiles-daemon
   1163       1 root     /usr/bin/qrtr-ns -f 1
   1169       1 root     /usr/lib/snapd/snapd
   1172       1 root     /usr/libexec/accounts-daemon
   1176       1 root     /usr/libexec/switcheroo-control
   1187       1 root     /usr/lib/systemd/systemd-logind
   1191       1 root     /usr/libexec/udisks2/udisksd
   1216       1 root     /usr/sbin/NetworkManager --no-daemon
   1217    1145 avahi    avahi-daemon: chroot helper
   1218       1 root     /usr/sbin/wpa_supplicant -u -s -O DIR=/run/wpa_supplicant GROUP=netdev
   1245       1 root     /usr/bin/prltoolsd -p /var/run/prltoolsd.pid
   1312       1 syslog   /usr/sbin/rsyslogd -n -iNONE
   1315    1245 root     prlshprint
   1317    1245 root     prltimesync
   1350       1 root     /usr/sbin/ModemManager
   1514       1 root     /usr/bin/prl_fsd /media/psf/RosettaLinux -orw,nosuid,nodev,noatime,big_writes,fsname=RosettaLinux,subtype=prl_fsd --sh
   1518    1514 root     fusermount -o rw,nosuid,nodev,noatime,allow_other,default_permissions,fsname=RosettaLinux,auto_unmount,subtype=prl_fsd
   1624       1 root     /media/psf/RosettaLinux/rosettad daemon /var/cache/prlrosettad
   1698       1 root     /usr/sbin/cupsd -l
   1700       1 root     /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
   1707       1 root     /usr/bin/containerd
   1711       1 colord   /usr/libexec/colord
   1723       1 root     /usr/sbin/cron -f -P
   1730       1 kernoops /usr/sbin/kerneloops --test
   1745       1 kernoops /usr/sbin/kerneloops
   1754       1 root     /usr/sbin/gdm3
   1784       1 root     /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
   1788       1 rtkit    /usr/libexec/rtkit-daemon
   1795       1 cups-br+ /usr/sbin/cups-browsed
   1895       1 root     /usr/bin/prl_fsd /media/psf/iCloud -orw,nosuid,nodev,noatime,big_writes,fsname=iCloud,subtype=prl_fsd --share --sf=iCl
   1898    1895 root     fusermount -o rw,nosuid,nodev,noatime,allow_other,default_permissions,fsname=iCloud,auto_unmount,subtype=prl_fsd -- /m
   1923       1 root     /usr/bin/prl_fsd /media/psf/SD_Card -orw,nosuid,nodev,noatime,big_writes,fsname=SD_Card,subtype=prl_fsd --share --sf=S
   1928    1923 root     fusermount -o rw,nosuid,nodev,noatime,allow_other,default_permissions,fsname=SD_Card,auto_unmount,subtype=prl_fsd -- /
   2030       2 root     [kworker/u4:8]
   2349       1 root     /usr/libexec/packagekitd
   2353       1 root     /usr/libexec/upowerd
   2589       2 root     [psimon]
   2672    1698 lp       /usr/lib/cups/notifier/dbus dbus://
   2824    1754 root     gdm-session-worker [pam/gdm-password]
   2861       1 paralle+ /usr/lib/systemd/systemd --user
   2864    2861 paralle+ (sd-pam)
   2876    2861 paralle+ /usr/bin/pipewire
   2877    2861 paralle+ /usr/bin/pipewire -c filter-chain.conf
   2878    2861 paralle+ /usr/bin/wireplumber
   2881    2861 paralle+ /usr/bin/pipewire-pulse
   2885    2861 paralle+ /usr/bin/gnome-keyring-daemon --foreground --components=pkcs11,secrets --control-directory=/run/user/1000/keyring
   2888    2861 paralle+ /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
   2918    2824 paralle+ /usr/libexec/gdm-wayland-session env GNOME_SHELL_SESSION_MODE=ubuntu /usr/bin/gnome-session --session=ubuntu
   2923    2918 paralle+ /usr/libexec/gnome-session-binary --session=ubuntu
   2996    2861 paralle+ /usr/libexec/gcr-ssh-agent --base-dir /run/user/1000/gcr
   2997    2861 paralle+ /usr/libexec/gnome-session-ctl --monitor
   3013    2861 paralle+ /usr/libexec/gvfsd
   3019    2861 paralle+ /usr/libexec/gvfsd-fuse /run/user/1000/gvfs -f
   3021    2861 paralle+ /usr/libexec/gnome-session-binary --systemd-service --session=ubuntu
   3055    3021 paralle+ /usr/libexec/at-spi-bus-launcher --launch-immediately
   3064    2861 paralle+ /usr/bin/gnome-shell
   3069    3055 paralle+ /usr/bin/dbus-daemon --config-file=/usr/share/defaults/at-spi2/accessibility.conf --nofork --print-address 11 --addres
   3130    2861 paralle+ /usr/libexec/at-spi2-registryd --use-gnome-session
   3142    2861 paralle+ /usr/libexec/xdg-permission-store
   3144    2861 paralle+ /usr/libexec/gnome-shell-calendar-server
   3150    2861 paralle+ /usr/libexec/dconf-service
   3161    2861 paralle+ /usr/libexec/evolution-source-registry
   3173    2861 paralle+ /usr/bin/gjs -m /usr/share/gnome-shell/org.gnome.Shell.Notifications
   3180    2861 paralle+ /usr/bin/ibus-daemon --panel disable
   3186    2861 paralle+ /usr/libexec/gsd-a11y-settings
   3195    2861 paralle+ /usr/libexec/gsd-color
   3197    2861 paralle+ /usr/libexec/gsd-datetime
   3199    2861 paralle+ /usr/libexec/gsd-housekeeping
   3202    2861 paralle+ /usr/libexec/gsd-keyboard
   3206    3021 paralle+ /usr/libexec/gsd-disk-utility-notify
   3207    2861 paralle+ /usr/libexec/gsd-media-keys
   3208    2861 paralle+ /usr/libexec/gsd-power
   3209    2861 paralle+ /usr/libexec/gsd-print-notifications
   3210    2861 paralle+ /usr/libexec/gsd-rfkill
   3214    2861 paralle+ /usr/libexec/gsd-screensaver-proxy
   3216    2861 paralle+ /usr/libexec/gsd-sharing
   3218    2861 paralle+ /usr/libexec/gsd-smartcard
   3225    2861 paralle+ /usr/libexec/gsd-sound
   3226    2861 paralle+ /usr/libexec/gsd-wacom
   3227    3021 paralle+ /usr/libexec/evolution-data-server/evolution-alarm-notify
   3235    3021 paralle+ /usr/bin/prlcc
   3254    3235 paralle+ /usr/bin/prldnd
   3255    3235 paralle+ /usr/bin/prlcp
   3256    3235 paralle+ /usr/bin/prlshprof
   3342    2861 paralle+ /usr/libexec/goa-daemon
   3360    2861 paralle+ /usr/libexec/gvfs-udisks2-volume-monitor
   3385    2861 paralle+ /usr/libexec/gsd-printer
   3396    3180 paralle+ /usr/libexec/ibus-memconf
   3398    3180 paralle+ /usr/libexec/ibus-extension-gtk3
   3425    2861 paralle+ /usr/libexec/evolution-calendar-factory
   3439    2861 paralle+ /usr/libexec/ibus-portal
   3458    2861 paralle+ /usr/libexec/goa-identity-service
   3474    2861 paralle+ /usr/libexec/evolution-addressbook-factory
   3481    2861 paralle+ /usr/libexec/gvfs-gphoto2-volume-monitor
   3488    2861 paralle+ /usr/libexec/gvfs-afc-volume-monitor
   3498    2861 paralle+ /usr/libexec/gvfs-mtp-volume-monitor
   3503    2861 paralle+ /usr/libexec/gvfs-goa-volume-monitor
   3515    3180 paralle+ /usr/libexec/ibus-engine-simple
   3553    3013 paralle+ /usr/libexec/gvfsd-trash --spawner :1.17 /org/gtk/gvfs/exec_spaw/0
   3579    2861 paralle+ /usr/libexec/xdg-desktop-portal
   3589    2861 paralle+ /usr/libexec/xdg-document-portal
   3599    3589 root     fusermount3 -o rw,nosuid,nodev,fsname=portal,auto_unmount,subtype=portal -- /run/user/1000/doc
   3607    2861 paralle+ /usr/libexec/tracker-miner-fs-3
   3609    2861 paralle+ /usr/libexec/xdg-desktop-portal-gnome
   3620    2861 paralle+ /usr/bin/gjs -m /usr/share/gnome-shell/org.gnome.ScreenSaver
   3639    2861 paralle+ /usr/libexec/xdg-desktop-portal-gtk
   3683    3064 paralle+ gjs /usr/share/gnome-shell/extensions/ding@rastersoft.com/app/ding.js -E -P /usr/share/gnome-shell/extensions/ding@ras
   3715    2861 paralle+ /usr/libexec/gvfsd-metadata
   3764    2861 paralle+ /usr/libexec/gnome-terminal-server
   3771    3764 paralle+ bash
   3987    3021 paralle+ /usr/bin/update-notifier
   4157    2861 paralle+ /usr/bin/python3 /usr/bin/update-manager --no-update --no-focus-on-map
   4407    3021 paralle+ /usr/libexec/deja-dup/deja-dup-monitor
   4541    3771 paralle+ ps -eo pid,ppid,user,cmd --no-headers
```
