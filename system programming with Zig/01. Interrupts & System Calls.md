## Processor (CPU): 
The central element in a computer system, responsible for executing instructions fetched from memory. It operates in a **fetch-decode-execute cycle** and uses *registers* for temporary storage.

# Interrupts:
Interrupts are a mechanism to notify the CPU when an event occurs, allowing the CPU to suspend its current task and address the event (interrupt).
( A signal sent to the processor by hardware or software indicating an event that needs immediate attention. )

- Hardware interrupts are sent by external devices like the keyboard, disk drives, etc.
- Software interrupts are triggered by programs through system calls.

### Sources of interrupts include:
- **Program**: E.g., a division by zero.
- **Timer**: E.g., updating the system clock.
- **I/O**: E.g., completion of disk read operations.
- **Hardware Failure**: E.g., power failure.
#### Interrupts improve CPU utilization by allowing the CPU to perform other tasks while waiting for slower operations (like I/O) to complete.

#### Interrupts vs. Polling:
- Interrupts allow the CPU to do other tasks and respond to events only when needed.
- Polling constantly checks for an event, which can waste CPU time.

#### Priority:
Some interrupts have higher priority, and they can interrupt lower-priority interrupt handlers.

# System Calls
**Definition**: A system call is a request from a program to the kernel to perform privileged operations on its behalf.
- Example operations: file I/O, process management, and communication.

### Mechanism:
- When a program issues a system call, it triggers a software interrupt.
- The CPU switches from user mode to kernel mode to execute the system call.
- Once the system call completes, the CPU returns to user mode.

- A system call is a way for programs to interact with the operating system, often used to request services such as reading from disk or sending data to a printer.
- System calls use the **trap mechanism**, which switches the CPU from user mode to kernel mode, allowing the OS to execute privileged instructions.

### Common System Calls:
- **File I/O**: `open()`, `read()`, `write()`, `close()`
- **Process Management**: `fork()`, `exec()`, `exit()`, `wait()`
- **Memory Management**: `mmap()`, `brk()`
- **Networking**: `socket()`, `connect()`, `send()`, `recv()`

## Dual-Mode Operation:
- Modern systems run in either user mode or kernel mode.
- User Mode: Restricted access; cannot directly interact with hardware or system resources.
- Kernel Mode: Full access; used to manage resources, access hardware, and execute system calls.
- System calls trigger a switch to kernel mode, allowing privileged operations like I/O.

## System Call Interface (SCI):
- Acts as a bridge between user programs and the kernel.
- Converts function calls into the corresponding system call numbers and transfers control to the kernel.

## Handling Interrupts:
When an interrupt occurs, 
1. The CPU suspends the current process
2. saves its state
3. runs an interrupt handler.

#### The handler:
1. resolves the interrupt,
2. The CPU restores the original process’s state and resumes execution.

### Context Switch:
- During a system call, the CPU performs a context switch from the user space to kernel space.
- The system saves the current state of the user program and loads the state of the kernel to execute the system call.


## System Call Example:

When a system call like read is invoked, the OS takes over in kernel mode, performs the requested operation (e.g., reading from disk), and then returns control to the user program.

#### Example in Zig (System Call read)
Here's a Zig version of a system call that reads from a file:
```zig
const std = @import("std");

pub fn main() !void {
    const allocator = std.heap.page_allocator;
    const args = try std.process.argsAlloc(allocator);
    defer std.process.argsFree(allocator, args);

    if (args.len != 2) {
        std.debug.print("Usage: {} <filename>\n", .{args[0]});
        return;
    }

    var file = try std.fs.cwd().openFile(args[1], .{});
    defer file.close();

    try readFile(file);
}

fn readFile(file: std.fs.File) !void {
    var buffer: [1024]u8 = undefined;
    var read_len = try file.read(&buffer);
    while (read_len > 0) {
        try std.io.getStdOut().writeAll(&buffer[0..read_len]);
        read_len = try file.read(&buffer);
    }
}
```
#### In this Zig example:
- We open the file passed as a command-line argument.
- The readFile function reads the file in chunks (buffer) and prints it to standard output.

## Additional Notes:
**Trap**: This is the mechanism that allows user programs to request services from the OS. In both Zig and C, system calls like `read` involve traps that transition the CPU from user mode to kernel mode.

**Mode Switching**: When a trap occurs, the CPU switches from `user mode` to `kernel mode`, performs the requested operation (e.g., file read), and then switches back to user mode.

**Efficiency**: Although the overhead of switching between user and kernel mode exists, it's necessary to maintain the system’s integrity and security.

## Key Differences between Interrupts & System Calls
### Initiation:
- Interrupts are initiated by hardware or software events.
- System calls are initiated by user programs to request kernel services.
### Purpose:
- Interrupts notify the processor of events (e.g., I/O operations).
- System calls allow a program to request system resources or services.
### Example of System Call Flow:
1. User Program: Makes a system call (e.g., read()).
2. CPU: Triggers a software interrupt (usually a trap).
3. Kernel: Handles the request by executing the appropriate system call handler.
4. Return: After the system call completes, control is returned to the user program.

```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    const file = try std.fs.cwd().openFile("file.txt", .{});
    var buffer: [1024]u8 = undefined;

    // Perform a read system call
    const bytesRead = try file.read(buffer[0..]);
    try stdout.write(buffer[0..bytesRead]);
}
```

