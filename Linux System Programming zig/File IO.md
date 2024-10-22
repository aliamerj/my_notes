# File I/O in Unix (in Zig)
File I/O is an essential part of interacting with a Unix system, as many components in Unix are treated as files, such as devices, pipes, and sockets. In this section, we'll go over the basic system calls for file operations in Unix, focusing on the Zig programming language.

## Concepts of File Descriptors

A file descriptor is an integer that the kernel uses to keep track of open files. Each process has a table of file descriptors, and the kernel manages the state and access permissions for each open file. File descriptors are nonnegative integers, and in Unix, every process starts with at least three standard file descriptors:

  1. `0` (stdin): Standard input
  2. `1` (stdout): Standard output
  3. `2` (stderr): Standard error

In Zig, file descriptors are used similarly, and Zig’s standard library provides abstractions over system calls like open, close, read, and write to work with files.

## Opening Files in Zig

Before interacting with a file, it must first be opened. The Zig standard library provides a simple way to open a file:
```zig
const std = @import("std");

pub fn main() !void {
    // Open a file for reading
    var file = try std.fs.cwd().openFile("test.txt", .{ .read = true });

    // Do something with the file (e.g., read, write)
    
    // Always close the file when done
    try file.close();
}
```
Here, openFile takes a path to the file and a struct that specifies the desired access mode. In this case, .read = true opens the file in read-only mode.
System Call open in Unix (Equivalent in Zig)

The Unix open system call allows a file to be opened and associated with a file descriptor. In Zig, we use the std.fs module to open files, which abstracts this system call. Let's look at the Unix concept first:

In Unix, the open system call works like this:

```c
int open(const char *name, int flags);
```

It takes the file's name and a set of flags to specify how the file should be opened. Flags can include:
- O_RDONLY: Open for reading
- O_WRONLY: Open for writing
- O_RDWR: Open for both reading and writing

For example:
```c
int fd = open("/home/user/file.txt", O_RDONLY);
```
In Zig, the same can be achieved using the openFile method:
```zig
const std = @import("std");

pub fn main() !void {
    var file = try std.fs.cwd().openFile("test.txt", .{ .read = true });
    defer file.close();
}
```
Zig handles file opening and flag manipulation internally, so you don't need to directly manage file descriptors unless working at a lower level.
Closing Files in Zig

Once you're done with a file, you must close it to free the file descriptor. In Zig, you simply call the close method on the File object:
```zig
try file.close();
```
This ensures that system resources are released properly, similar to the Unix close system call.

### File Permissions in Zig

When creating new files, you may need to specify the file's permissions. Unix uses a bitset for file permissions (e.g., 0644), which defines the owner, group, and others' access to the file. In Zig, you can specify permissions when creating a file using openFile.

For example, if you wanted to create a file with read/write permissions for the owner and read-only permissions for others, you'd do something like this:
```zig
const std = @import("std");

pub fn main() !void {
    const mode = std.os.FileFlags{ .read = true, .write = true };
    var file = try std.fs.cwd().createFile("newfile.txt", mode, 0644);
    defer file.close();
}
```
Here, 0644 represents the permissions, where the owner can read and write, but others can only read.

### File Operations: Read/Write in Zig

Once a file is opened, you can read from or write to it using system calls. In Zig, you would use the read or write methods on the File object to perform these actions.

For example, to read from a file:
```zig
const std = @import("std");

pub fn main() !void {
    var file = try std.fs.cwd().openFile("test.txt", .{ .read = true });
    defer file.close();

    var buffer: [1024]u8 = undefined;
    const read_bytes = try file.read(buffer[0..]);
    std.debug.print("Read {} bytes: {}\n", .{read_bytes, buffer[0..read_bytes]});
}
```
This example reads the contents of test.txt into a buffer and prints the number of bytes read.
### Summary

This first section of file I/O covered the basic concepts of interacting with files in Unix and how to apply them using Zig. We went over file descriptors, how to open and close files, and how to perform basic read and write operations. The goal is to get comfortable working with files, which is foundational for more advanced topics like asynchronous I/O, direct I/O, and file manipulation.
