# C++ Security Guide

A practical guide to modern C++ for secure software development, cybersecurity tooling, systems programming, vulnerability research, and reverse engineering.

> **Purpose:** Learn how C++ interacts with memory, operating systems, networks, and executable files—and how to build security tools without introducing unnecessary vulnerabilities.
>
> **Scope:** Educational use, defensive engineering, authorized security testing, and isolated research environments.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Development Environment](#development-environment)
3. [C++ Security Fundamentals](#c-security-fundamentals)
4. [Memory Management and Memory Safety](#memory-management-and-memory-safety)
5. [Secure Network Programming](#secure-network-programming)
6. [System Programming](#system-programming)
7. [Cryptography](#cryptography)
8. [Reverse Engineering](#reverse-engineering)
9. [Exploit Development Concepts](#exploit-development-concepts)
10. [Building Defensive Security Tools](#building-defensive-security-tools)
11. [Debugging and Analysis](#debugging-and-analysis)
12. [Compiler and Build Security](#compiler-and-build-security)
13. [Practical Projects](#practical-projects)
14. [Learning Roadmap](#learning-roadmap)
15. [References](#references)

---

## Introduction

C++ is useful in cybersecurity because it provides access to low-level system functionality while retaining higher-level programming abstractions.

It is commonly used in:

- Security research and vulnerability analysis
- Operating-system and systems programming
- Network monitoring and protocol analysis
- Reverse engineering and executable-file analysis
- Embedded security and firmware development
- Cryptographic applications
- Defensive security tools
- Performance-sensitive applications

### Why learn C++ for cybersecurity?

C++ helps bridge the gap between high-level programming and the underlying behavior of a computer.

For example, understanding pointers, stack frames, heap allocation, and machine instructions makes it easier to understand memory corruption vulnerabilities and how operating systems manage processes.

C++ is also useful for developing software that interacts directly with Windows APIs, Linux system calls, network sockets, and hardware.

**Important:** C++ does not automatically make software secure. Its low-level capabilities also introduce risks such as buffer overflows, use-after-free errors, integer overflow, and data races.

---

## Development Environment

### Recommended tools

| Tool | Purpose |
|---|---|
| GCC | C++ compiler on Linux |
| Clang | Alternative compiler and analysis tooling |
| MSVC | Microsoft C++ compiler |
| CMake | Cross-platform build configuration |
| GDB | Linux debugging |
| LLDB | Debugging with LLVM |
| WinDbg | Windows debugging |
| Valgrind | Memory-error analysis |
| AddressSanitizer | Runtime memory-error detection |
| Ghidra | Reverse engineering |
| IDA Free | Disassembly and analysis |
| Wireshark | Network protocol analysis |

### Debian / Ubuntu installation

```bash
sudo apt update

sudo apt install -y \
    build-essential \
    cmake \
    gdb \
    clang \
    lldb \
    valgrind \
    git \
    pkg-config \
    libssl-dev
```

Verify the compiler:

```bash
g++ --version
clang++ --version
cmake --version
gdb --version
```

### Create a project

```bash
mkdir cpp-security-lab
cd cpp-security-lab

mkdir src include build
```

Example structure:

```text
cpp-security-lab/
├── CMakeLists.txt
├── include/
├── src/
│   └── main.cpp
└── build/
```

### First C++ program

```cpp
#include <iostream>
#include <string>

int main() {
    std::string message = "C++ Security Lab";

    std::cout << message << '\n';

    return 0;
}
```

Compile:

```bash
g++ -std=c++20 -Wall -Wextra -Wpedantic \
    -g src/main.cpp -o build/security_lab
```

Run:

```bash
./build/security_lab
```

---

## C++ Security Fundamentals

### 1. Memory layout

Understanding process memory is essential for systems programming and vulnerability research.

A simplified process memory layout includes:

| Region | Description |
|---|---|
| Text | Executable machine instructions |
| Data | Initialized global and static variables |
| BSS | Zero-initialized global and static variables |
| Heap | Dynamically allocated memory |
| Stack | Function calls, local variables, and return information |
| Memory mappings | Shared libraries, mapped files, and other mapped regions |

Actual layouts depend on the operating system, executable format, compiler, and runtime.

### 2. Pointers

Pointers hold memory addresses.

```cpp
#include <iostream>

int main() {
    int value = 42;

    int* pointer = &value;

    std::cout << "Value: " << value << '\n';
    std::cout << "Address: " << pointer << '\n';
    std::cout << "Dereferenced: " << *pointer << '\n';
}
```

Important concepts:

- Address-of operator: `&`
- Dereference operator: `*`
- Null pointers: `nullptr`
- Pointer arithmetic
- Object lifetime
- Dangling pointers
- Ownership and aliasing

A pointer does not automatically manage the lifetime of the object it references.

### 3. References

References provide another way to access an existing object.

```cpp
#include <iostream>

void update_value(int& value) {
    value = 100;
}

int main() {
    int number = 10;

    update_value(number);

    std::cout << number << '\n';
}
```

References are useful when passing objects to functions without copying them.

### 4. RAII

Resource Acquisition Is Initialization (RAII) connects resource ownership to object lifetime.

When an object leaves scope, its destructor releases the resource it owns.

Common examples:

- `std::unique_ptr`
- `std::shared_ptr`
- `std::vector`
- `std::string`
- `std::fstream`
- `std::lock_guard`

Prefer RAII over manually managing resources whenever practical.

### 5. Input validation

Never assume external input is trustworthy.

```cpp
#include <iostream>
#include <string>

bool valid_username(const std::string& username) {
    constexpr std::size_t max_length = 32;

    if (username.empty() || username.size() > max_length) {
        return false;
    }

    for (unsigned char c : username) {
        if (!(std::isalnum(c) || c == '_')) {
            return false;
        }
    }

    return true;
}

int main() {
    std::string username;

    std::cout << "Username: ";
    std::getline(std::cin, username);

    if (valid_username(username)) {
        std::cout << "Input accepted\n";
    } else {
        std::cout << "Invalid input\n";
    }
}
```

Input validation must be appropriate to the application's actual requirements. A blacklist of suspicious strings is not a reliable security boundary.

---

## Memory Management and Memory Safety

Memory errors are among the most important subjects in C++ security research.

### Common memory vulnerabilities

| Vulnerability | Description |
|---|---|
| Buffer overflow | Writing beyond an object's allocated bounds |
| Use-after-free | Accessing an object after its lifetime ends |
| Double free | Releasing the same allocation more than once |
| Memory leak | Losing access to allocated memory without releasing it |
| Out-of-bounds read | Reading outside an object's valid range |
| Integer overflow | Arithmetic produces an unintended value |
| Uninitialized memory | Reading a value before it has been properly initialized |
| Data race | Unsynchronized concurrent access to shared data |

### Safer containers

Prefer standard containers instead of manually managed arrays.

```cpp
#include <array>
#include <iostream>
#include <vector>

int main() {
    std::array<int, 4> fixed_values{10, 20, 30, 40};

    std::vector<int> dynamic_values{1, 2, 3, 4};

    for (int value : fixed_values) {
        std::cout << value << '\n';
    }

    for (int value : dynamic_values) {
        std::cout << value << '\n';
    }
}
```

### Smart pointers

Use smart pointers to express ownership.

```cpp
#include <iostream>
#include <memory>

int main() {
    auto number = std::make_unique<int>(42);

    std::cout << *number << '\n';
}
```

- `std::unique_ptr`: Exclusive ownership.
- `std::shared_ptr`: Shared ownership through reference counting.
- `std::weak_ptr`: Non-owning reference to a shared object.

Do not use `shared_ptr` everywhere. Prefer `unique_ptr` unless shared ownership is genuinely necessary.

### Buffer overflow research

A buffer overflow occurs when a program accesses memory outside the bounds of an object.

Important concepts to study:

- Stack-based buffer overflows
- Heap-based memory corruption
- Stack canaries
- Address Space Layout Randomization (ASLR)
- Data Execution Prevention (DEP/NX)
- Control Flow Guard
- Control-flow integrity
- Bounds checking

Use deliberately vulnerable training programs only inside isolated labs. For production code, use bounded interfaces and standard containers.

### AddressSanitizer

AddressSanitizer can detect several memory errors during testing.

Compile with GCC or Clang:

```bash
g++ -std=c++20 \
    -g -O1 \
    -fsanitize=address,undefined \
    -fno-omit-frame-pointer \
    src/main.cpp \
    -o build/security_lab
```

Run:

```bash
./build/security_lab
```

This is a development diagnostic tool, not a replacement for secure coding or production hardening.

---

## Secure Network Programming

C++ supports network programming through operating-system socket APIs and libraries.

### Networking concepts

Understand:

- TCP and UDP
- IPv4 and IPv6
- Ports and sockets
- DNS resolution
- Client-server architecture
- HTTP and HTTPS
- TLS certificates
- Connection timeouts
- Input framing and message boundaries
- Network error handling

### Socket programming

Linux and Windows expose different socket APIs.

On Linux, commonly used headers include:

```cpp
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
```

On Windows:

```cpp
#include <winsock2.h>
#include <ws2tcpip.h>
```

Windows applications must initialize and clean up Winsock appropriately.

### Secure socket design

When building a network application:

1. Validate addresses, ports, and protocol input.
2. Set appropriate connection and read timeouts.
3. Handle partial reads and writes.
4. Define message framing explicitly.
5. Avoid assuming one `recv()` call returns a complete message.
6. Use TLS when confidentiality and peer authentication are required.
7. Validate certificates and hostnames.
8. Limit input sizes and concurrent connections.
9. Log errors without exposing secrets.

### Useful libraries

| Library | Purpose |
|---|---|
| Boost.Asio | Asynchronous networking |
| libcurl | HTTP and URL transfers |
| OpenSSL | TLS and cryptographic operations |
| standalone Asio | Cross-platform networking |
| nlohmann/json | JSON processing |

### Network security project

Build a local network-service inventory tool that:

- Accepts a manually supplied lab address.
- Connects only to explicitly configured test targets.
- Uses bounded concurrency and timeouts.
- Reports connection results.
- Identifies common service ports without claiming that a port number proves a service identity.
- Saves results in JSON or CSV.

Do not implement traffic flooding or source-address spoofing as a beginner networking exercise.

---

## System Programming

System programming involves interacting with operating-system facilities such as processes, files, memory, permissions, and system information.

### Linux system programming

Useful interfaces include:

| Interface | Purpose |
|---|---|
| `open()` | Open files |
| `read()` | Read file descriptors |
| `write()` | Write file descriptors |
| `close()` | Close file descriptors |
| `stat()` | Inspect file metadata |
| `fork()` | Create a process |
| `exec()` | Replace a process image |
| `waitpid()` | Wait for child processes |
| `mmap()` | Map memory |
| `ptrace()` | Process tracing and debugging |

Learn the distinction between C++ standard-library facilities and operating-system-specific APIs.

### Windows system programming

Useful Windows APIs and concepts include:

- Process and thread management
- Windows handles
- File-system access
- Windows services
- Registry access
- Event logging
- Virtual memory
- Access tokens and security descriptors
- Windows debugging interfaces

### Process inspection

Security analysts often need to understand:

- Process IDs and parent process IDs
- Executable paths
- Loaded modules
- Process command lines
- User identities
- File and network handles
- Process permissions

For learning, build a read-only process inventory utility using documented operating-system APIs.

### Defensive analysis of suspicious behavior

Process injection, unauthorized keylogging, credential extraction, and covert remote command execution are dangerous capabilities. This guide does not provide implementation instructions for those activities.

Instead, study their security implications and how defenders identify them:

| Behavior | Defensive investigation |
|---|---|
| Unexpected remote memory modification | Examine process access and memory-operation telemetry |
| Unusual executable memory | Investigate executable private memory regions |
| Suspicious keyboard hooks | Review unexpected input-monitoring behavior |
| Unusual process relationships | Inspect parent-child process chains |
| Unexpected outbound connections | Correlate network telemetry with process activity |
| Unauthorized credential access | Review endpoint protection and credential-access alerts |

---

## Cryptography

Cryptography is a critical part of secure software development. It is also an area where incorrect implementations can create serious vulnerabilities.

### Core concepts

| Concept | Purpose |
|---|---|
| Encryption | Protect confidentiality |
| Decryption | Recover encrypted information |
| Hashing | Produce a fixed-length digest |
| HMAC | Provide message authentication |
| Digital signatures | Authenticate data and its origin |
| Key exchange | Establish shared cryptographic secrets |
| Random number generation | Produce unpredictable values |

### Cryptographic algorithms

| Algorithm | Appropriate use |
|---|---|
| AES-256-GCM | Authenticated symmetric encryption |
| ChaCha20-Poly1305 | Authenticated symmetric encryption |
| SHA-256 | General-purpose cryptographic hashing |
| SHA-3 | General-purpose cryptographic hashing |
| Ed25519 | Digital signatures |
| X25519 | Elliptic-curve key exchange |
| Argon2id | Password hashing |

Avoid MD5 and SHA-1 for security-sensitive integrity or authentication applications.

**Password storage is different from ordinary hashing.** Use a password-hashing algorithm such as Argon2id, with a unique salt and appropriate parameters. Do not store passwords as plain SHA-256 or MD5 digests.

### OpenSSL

OpenSSL provides established cryptographic functionality.

```cpp
#include <openssl/evp.h>
#include <openssl/rand.h>
```

Prefer OpenSSL's high-level EVP interface instead of low-level cipher-specific interfaces.

### Authenticated encryption

Authenticated encryption protects both confidentiality and integrity.

For example, AES-GCM provides:

- Encryption of plaintext
- An authentication tag
- Detection of unauthorized modifications
- Associated authenticated data when required

Important implementation requirements:

- Generate keys with a cryptographically secure random generator.
- Never reuse a nonce with the same AES-GCM key.
- Store or transmit the nonce alongside the ciphertext.
- Verify the authentication tag before accepting decrypted data.
- Handle all cryptographic API errors.
- Protect keys separately from encrypted data.

Do not invent your own encryption algorithm or authentication scheme.

### Secure random generation

For cryptographic randomness, use an operating-system cryptographic random source or a trusted library such as OpenSSL's `RAND_bytes()`.

Do not use `rand()` for cryptographic keys, nonces, tokens, or passwords.

---

## Reverse Engineering

Reverse engineering is the process of examining software to understand its structure, behavior, and implementation.

It is particularly useful for:

- Malware analysis
- Vulnerability research
- Software auditing
- Firmware analysis
- Binary exploitation education
- Understanding undocumented file formats
- Investigating suspicious executables

### Essential concepts

| Concept | Description |
|---|---|
| Assembly | Human-readable representation of machine instructions |
| Disassembly | Converting machine code into assembly |
| Decompilation | Reconstructing higher-level representations |
| Registers | Small storage locations used by the CPU |
| Stack frame | Function-related stack storage |
| Calling convention | Rules for passing arguments and returning values |
| ELF | Common Linux executable format |
| PE | Windows executable format |
| Symbols | Names and metadata associated with program elements |
| Import table | External functions and libraries referenced by an executable |

### Linux analysis tools

```bash
file ./program
```

```bash
readelf -h ./program
```

```bash
readelf -S ./program
```

```bash
objdump -d ./program
```

```bash
strings ./program
```

### Debugging with GDB

Start GDB:

```bash
gdb ./program
```

Useful commands:

```gdb
break main
run
info registers
info functions
info frame
disassemble main
x/16gx $rsp
stepi
nexti
continue
```

These commands help you inspect execution, registers, memory, and assembly instructions.

### Windows PE analysis

Study:

- DOS headers
- NT headers
- Section tables
- Import and export tables
- Relative Virtual Addresses (RVAs)
- Virtual addresses
- Relocations
- Debug directories
- Digital signatures

A PE parser must validate every offset and length before accessing file data. Checking only the DOS and NT signatures is not sufficient to establish that a file is structurally valid.

### Recommended tools

- Ghidra
- IDA Free
- x64dbg
- WinDbg
- GDB
- LLDB
- Detect It Easy
- PE-bear
- `readelf`
- `objdump`

Use isolated sample binaries and intentionally vulnerable training programs.

---

## Exploit Development Concepts

Understanding exploitation is useful for vulnerability research and defensive engineering.

### Core concepts

- Stack memory corruption
- Heap memory corruption
- Integer overflow
- Return-oriented programming (ROP)
- Calling conventions
- Memory permissions
- ASLR
- DEP/NX
- Stack canaries
- Control-flow integrity
- Address disclosure
- Use-after-free vulnerabilities

### ROP

Return-oriented programming is an exploitation technique involving existing instruction sequences in a process.

A gadget is a short sequence of instructions that ends in a control-flow transfer, commonly a return instruction.

ROP research requires understanding:

- CPU architecture
- Register usage
- Stack layout
- Instruction encoding
- Binary loading
- Memory addresses
- Runtime mitigations

Hardcoded gadget addresses are not portable. Addresses change with binary versions, compilation options, ASLR, and runtime environment.

### Shellcode

Shellcode is a term for machine-code payloads used in exploitation research.

When studying shellcode, concentrate on:

- Instruction encoding
- Calling conventions
- System-call interfaces
- Memory permissions
- Architecture differences
- Static analysis
- Detection and prevention

Use disassembly and non-executing analysis exercises rather than deploying payloads against real systems.

### Mitigations to understand

| Mitigation | Security purpose |
|---|---|
| ASLR | Randomizes memory locations |
| DEP/NX | Prevents execution from non-executable memory pages |
| Stack canaries | Detects certain stack-buffer overwrites |
| CFG | Restricts certain indirect control-flow transfers |
| CET | Provides hardware-assisted control-flow protections |
| PIE | Supports relocation of executable code |
| RELRO | Protects parts of ELF relocation structures |

No single mitigation prevents every exploitation technique.

---

## Building Defensive Security Tools

C++ can be used to build practical security utilities.

### Useful project ideas

| Project | Skills developed |
|---|---|
| File integrity monitor | Hashing, file I/O, metadata |
| Log analyzer | Parsing, filtering, reporting |
| Process inventory tool | Operating-system APIs |
| PE metadata reader | Binary formats |
| ELF metadata reader | Executable analysis |
| Network service inventory | Sockets, timeouts, concurrency |
| Packet metadata analyzer | Networking and protocol structures |
| Configuration auditor | Input validation and reporting |
| Firmware metadata parser | Binary parsing and embedded systems |
| Local vulnerability-lab reporter | Structured findings and JSON |

### Example: File integrity monitor

A file integrity monitor records file hashes and compares later results to identify unexpected changes.

A practical implementation should:

1. Accept an explicitly configured directory.
2. Enumerate files safely.
3. Calculate SHA-256 using a maintained cryptographic library.
4. Store a baseline manifest.
5. Recalculate hashes during subsequent checks.
6. Report added, removed, and modified files.
7. Protect the baseline from unauthorized modification.

A hash comparison can detect changes, but it does not prove that a file is trustworthy. Protect the baseline and consider signing it.

### Example: Log analyzer

A log analyzer can:

- Read structured log files.
- Parse timestamps.
- Identify repeated error patterns.
- Count authentication failures.
- Group events by source.
- Export results as JSON or CSV.

Avoid treating a log entry as proof of malicious activity without additional context.

---

## Debugging and Analysis

Debugging is essential for understanding C++ behavior.

### GDB commands

| Command | Purpose |
|---|---|
| `break main` | Set a breakpoint |
| `run` | Start execution |
| `next` | Execute the next source line |
| `step` | Step into a function |
| `nexti` | Execute the next machine instruction |
| `stepi` | Step into the next machine instruction |
| `info registers` | Display CPU registers |
| `info frame` | Display stack-frame information |
| `backtrace` | Display the call stack |
| `disassemble` | Display assembly |
| `x` | Examine memory |
| `continue` | Resume execution |

### GDB memory examination

Example:

```gdb
x/16xb $rsp
```

This examines 16 bytes starting at the stack pointer.

```gdb
x/8gx $rsp
```

This examines eight 8-byte values starting at the stack pointer.

```gdb
info registers
```

This displays the current register state.

Understanding memory inspection is particularly important when studying stack frames, pointers, and function calls.

### Static and dynamic analysis

| Technique | Description |
|---|---|
| Static analysis | Examines code or binaries without executing them |
| Dynamic analysis | Examines behavior while a program executes |
| Fuzzing | Tests software with generated or mutated inputs |
| Symbolic execution | Explores possible execution paths using symbolic values |
| Sanitizers | Detect certain classes of runtime errors |
| Debugging | Examines execution state and program behavior |

---

## Compiler and Build Security

A secure C++ application should use appropriate compiler protections, testing, and dependency management.

### GCC / Clang

A useful development command:

```bash
g++ -std=c++20 \
    -Wall -Wextra -Wpedantic \
    -Wconversion -Wsign-conversion \
    -Wformat=2 \
    -Wshadow \
    -fstack-protector-strong \
    -D_FORTIFY_SOURCE=3 \
    -g -O1 \
    -fsanitize=address,undefined \
    src/main.cpp \
    -o build/security_lab
```

Some options depend on compiler version, optimization level, and system library support. Treat this as a development configuration, not a universal production command.

For a production build, use the toolchain's supported hardening features and test the resulting binary.

### Windows MSVC

Common protections include:

| Option | Purpose |
|---|---|
| `/GS` | Stack buffer security checks |
| `/sdl` | Additional security development checks |
| `/DYNAMICBASE` | ASLR compatibility |
| `/NXCOMPAT` | DEP compatibility |
| `/guard:cf` | Control Flow Guard |
| `/analyze` | Static analysis |

These options add protection but do not replace correct memory management.

### CMake example

```cmake
cmake_minimum_required(VERSION 3.20)

project(CppSecurityLab LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(security_lab
    src/main.cpp
)

if(MSVC)
    target_compile_options(security_lab PRIVATE
        /W4
        /GS
        /sdl
        /guard:cf
    )

    target_link_options(security_lab PRIVATE
        /DYNAMICBASE
        /NXCOMPAT
        /guard:cf
    )
else()
    target_compile_options(security_lab PRIVATE
        -Wall
        -Wextra
        -Wpedantic
        -fstack-protector-strong
    )
endif()
```

Build:

```bash
cmake -S . -B build
cmake --build build
```

For production, review compiler and linker support for additional platform-specific hardening options.

---

## Practical Projects

These exercises provide a progression from basic C++ to systems security.

### Beginner

- [ ] Build a command-line file hash calculator.
- [ ] Write a safe file reader with size limits.
- [ ] Create a log parser.
- [ ] Practice pointers, references, and smart pointers.
- [ ] Compile and debug small programs with GDB.

### Intermediate

- [ ] Build a file integrity monitor.
- [ ] Parse ELF executable headers.
- [ ] Create a PE metadata reader.
- [ ] Build a local TCP service inventory utility.
- [ ] Write a JSON-based security report generator.
- [ ] Test programs with AddressSanitizer and UndefinedBehaviorSanitizer.

### Advanced

- [ ] Analyze a deliberately vulnerable C++ program in an isolated VM.
- [ ] Investigate a use-after-free bug.
- [ ] Study stack canaries and ASLR using controlled examples.
- [ ] Build a read-only process inventory tool.
- [ ] Analyze a suspicious executable using Ghidra.
- [ ] Create a fuzz-testing harness for a binary parser.
- [ ] Develop a firmware metadata extraction tool.

---

## Learning Roadmap

A useful order for learning C++ security is:

1. **C++ fundamentals:** Variables, functions, classes, references, pointers, and the standard library.
2. **Memory management:** Stack, heap, allocation, object lifetime, RAII, and smart pointers.
3. **Operating systems:** Processes, threads, file descriptors, handles, permissions, and virtual memory.
4. **Assembly:** Registers, instructions, calling conventions, and stack frames.
5. **Debugging:** GDB, WinDbg, disassembly, and memory inspection.
6. **Networking:** TCP/IP, sockets, DNS, TLS, and protocol parsing.
7. **Cryptography:** Hashing, authenticated encryption, key management, and password storage.
8. **Reverse engineering:** ELF, PE, Ghidra, and binary analysis.
9. **Vulnerability research:** Memory corruption, fuzzing, sanitizers, and exploit mitigations.
10. **Security engineering:** Build secure applications and document findings with reproducible tests.

### Recommended practice environments

- OverTheWire
- pwn.college
- TryHackMe
- Hack The Box
- OWASP Juice Shop
- Local vulnerable-programming exercises

Use systems you own or have explicit authorization to test.

---

## References

### C++ and secure coding

- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
- [cppreference](https://en.cppreference.com/)
- [Microsoft: Build Reliable and Secure C++ Programs](https://learn.microsoft.com/en-us/cpp/code-quality/build-reliable-secure-programs)
- [Microsoft: Security Best Practices for C++](https://learn.microsoft.com/en-us/cpp/security/security-best-practices-for-cpp)

### Compilers and debugging

- [GCC Documentation](https://gcc.gnu.org/onlinedocs/)
- [Clang Documentation](https://clang.llvm.org/docs/)
- [GDB Documentation](https://sourceware.org/gdb/documentation/)
- [CMake Documentation](https://cmake.org/documentation/)

### Cryptography

- [OpenSSL Documentation](https://docs.openssl.org/)
- [libsodium Documentation](https://doc.libsodium.org/)
- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)

### Reverse engineering

- [Ghidra](https://ghidra-sre.org/)
- [IDA](https://hex-rays.com/ida-free/)
- [Microsoft PE Format Documentation](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)
- [System V ABI](https://gitlab.com/x86-psABIs/x86-64-ABI)

### Security research

- [MITRE ATT&CK](https://attack.mitre.org/)
- [OWASP](https://owasp.org/)
- [CWE](https://cwe.mitre.org/)
- [NIST Secure Software Development Framework](https://csrc.nist.gov/Projects/ssdf)

---

## Disclaimer

This repository is intended for education, secure software development, and authorized security research.

- Only test systems you own or have explicit permission to assess.
- Use isolated virtual machines for vulnerability research.
- Do not access, collect, or disclose other people's private information.
- Do not deploy unauthorized monitoring, credential collection, or remote-access software.
- Follow responsible disclosure practices.
- Document vulnerabilities clearly and include remediation guidance.

**The goal is to understand how software works, identify weaknesses, and build more secure systems.**
