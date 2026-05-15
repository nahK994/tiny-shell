# 🐚 tiny-shell — A Minimal Unix-like Shell in C

**tiny-shell** is a lightweight Unix-like shell built in C, designed purely for learning and exploring operating system internals.
This project focuses on understanding how shells interact with the OS at the syscall level — no abstractions, no runtime, just you and the kernel.

> ⚙️ Built for curiosity. Powered by syscalls. Fueled by "why does this work like that?"

---

## 🎯 Goals

* Understand how a shell works at the syscall level
* Explore process creation with real `fork()` and `exec()`
* Learn file descriptor manipulation with `dup2()` and `pipe()`
* Handle signals and process lifecycle manually
* Build intuition for OS-level abstractions from scratch

---

## 🛠️ Build & Run

```bash
make
./tiny-shell
```

---

## 🗺️ Roadmap

---

### 🔹 Milestone 1: Basic Shell Loop

> Build the beating heart of the shell.

* [ ] Print a prompt and read a line of input
* [ ] Tokenize the input (split by spaces)
* [ ] Fork a child process
* [ ] Exec the command in the child (`execvp`)
* [ ] Wait for the child to finish (`waitpid`)
* [ ] Handle "command not found" error

**Syscalls to use**

```c
fork()
execvp()
waitpid()
write()
read()
```

---

### 🔹 Milestone 2: Built-in Commands

> Not everything deserves a new process.

* [ ] Detect and run built-in commands before forking
* [ ] Implement `cd` — change working directory
* [ ] Implement `exit` — terminate the shell
* [ ] Implement `echo` — print arguments to stdout
* [ ] Implement `pwd` — print current working directory

**Syscalls to use**

```c
chdir()
getcwd()
getenv()
```

**Key question to answer:** Why can't `cd` be an external process?

---

### 🔹 Milestone 3: Command Parsing

> Turn raw text into structured data.

* [ ] Write a lexer that tokenizes input character by character
* [ ] Handle single-quoted strings (`'hello world'`)
* [ ] Handle double-quoted strings (`"hello world"`)
* [ ] Handle backslash escaping (`hello\ world`)
* [ ] Represent a parsed command as a struct

**Struct to design yourself:**

```c
// hint: what does a command need to know?
typedef struct { ... } Command;
```

**No library functions allowed:** no `strtok`, no `sscanf`. Parse manually.

---

### 🔹 Milestone 4: Pipes (`|`)

> Let processes talk to each other through file descriptors.

* [ ] Detect `|` in input and split into multiple commands
* [ ] Create a pipe with `pipe()`
* [ ] Fork two processes
* [ ] Connect stdout of the left command to the write end of the pipe
* [ ] Connect stdin of the right command to the read end of the pipe
* [ ] Close unused file descriptor ends
* [ ] Support chaining: `cmd1 | cmd2 | cmd3`

**Syscalls to use**

```c
pipe()
dup2()
close()
fork()
execvp()
```

**Key question to answer:** What happens if you forget to close the unused pipe ends?

---

### 🔹 Milestone 5: I/O Redirection

> Bend input and output to your will.

* [ ] Detect `>` and redirect stdout to a file (overwrite)
* [ ] Detect `>>` and redirect stdout to a file (append)
* [ ] Detect `<` and redirect stdin from a file
* [ ] Combine redirection with pipes
* [ ] Handle errors (file not found, permission denied)

**Syscalls to use**

```c
open()
close()
dup2()
```

**Flags to understand:**

```c
O_WRONLY | O_CREAT | O_TRUNC   // for >
O_WRONLY | O_CREAT | O_APPEND  // for >>
O_RDONLY                        // for <
```

---

### 🔹 Milestone 6: Background Jobs (`&`)

> Let processes run without blocking the shell.

* [ ] Detect trailing `&` in input
* [ ] Fork the child but do not call `waitpid` immediately
* [ ] Print the background process PID
* [ ] Keep the shell prompt responsive
* [ ] Store background PIDs in a job table

**Syscalls to use**

```c
fork()
getpid()
waitpid()  // with WNOHANG
```

**Key question to answer:** What is a zombie process and when does it appear here?

---

### 🔹 Milestone 7: Signal Handling

> Tame the chaos of interrupts.

* [ ] Catch `SIGINT` (Ctrl+C) — do not kill the shell, only the foreground child
* [ ] Catch `SIGCHLD` — reap zombie background processes automatically
* [ ] Catch `SIGTSTP` (Ctrl+Z) — stop the foreground process
* [ ] Restore default signal behavior in child processes before `exec`

**Syscalls to use**

```c
signal()       // or sigaction() — prefer sigaction
sigaction()
kill()
waitpid()      // with WNOHANG inside SIGCHLD handler
```

**Key question to answer:** Why must child processes reset their signal handlers before calling `exec`?

---

### 🔹 Milestone 8: Job Control

> Bring suspended and background jobs under control.

* [ ] Implement `jobs` — list all active background/stopped jobs
* [ ] Implement `fg %n` — bring job n to foreground
* [ ] Implement `bg %n` — resume stopped job n in background
* [ ] Assign each job a terminal process group (`setpgid`)
* [ ] Transfer terminal control with `tcsetpgrp`

**Syscalls to use**

```c
setpgid()
getpgrp()
tcsetpgrp()
tcgetpgrp()
kill()         // send SIGCONT
waitpid()
```

---

### 🔹 Milestone 9: AST-based Parsing

> Your shell starts thinking like a compiler.

* [ ] Define an AST node type for commands, pipes, and redirections
* [ ] Write a recursive descent parser that builds the AST from tokens
* [ ] Walk the AST to execute commands
* [ ] Support nested pipes and combined redirections in a single pass

**Struct to design yourself:**

```c
typedef enum { NODE_CMD, NODE_PIPE, NODE_REDIR } NodeType;

typedef struct ASTNode {
    NodeType type;
    // design the rest yourself
} ASTNode;
```

---

### 🔹 Milestone 10: Extensibility (Optional)

> Make your shell programmable.

* [ ] Add a built-in `source` command to execute a script file line by line
* [ ] Support shell variables (`VAR=value`, `$VAR`)
* [ ] Support a simple `if` / `then` / `fi` construct
* [ ] Add command history (store lines, recall with Up arrow via `termios`)

**Syscalls / APIs to explore**

```c
tcgetattr()
tcsetattr()    // raw terminal mode for arrow keys
```

---

## 📁 Project Structure

```
tiny-shell/
├── Makefile
├── README.md
├── src/
│   ├── main.c         ← entry point, REPL loop
│   ├── lexer.c        ← tokenizer
│   ├── lexer.h
│   ├── parser.c       ← command / AST parser
│   ├── parser.h
│   ├── executor.c     ← fork, exec, pipe, redirect
│   ├── executor.h
│   ├── builtins.c     ← cd, echo, exit, pwd, jobs, fg, bg
│   ├── builtins.h
│   ├── jobs.c         ← job table management
│   ├── jobs.h
│   ├── signals.c      ← signal handlers
│   └── signals.h
└── tests/
    └── ...
```

---

## 📌 Rules

* No use of `system()` — ever.
* No use of `popen()`.
* No use of `strtok()` in the parser — tokenize manually.
* Every file descriptor you open, you close.
* Every process you fork, you reap.
