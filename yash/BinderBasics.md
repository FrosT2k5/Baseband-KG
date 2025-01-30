![[Binder.png]]

Binder is the InterProcess Communication method of Android
- Low Level kernel API (/dev/binder is the interface)

A user space program mainly interacts with Binder driver using three system calls: `open`, `mmap` and `ioctl`. A process calls `open` to register itself as a user of Binder driver. `mmap` is used to create a kernel data buffer and reserve a range of user space virtual address for it. After `open` and `mmap` a process can interact with the driver using `ioctl`. The most important `ioctl` command is `BINDER_WRITE_READ` with which a process can write and read data payload from the Binder driver

Sources:
https://null-android-pentesting.netlify.app/src/android-internals#android-architecture-and-binder
