## Introduction 

- **Binder** is the primary **inter-process communication (IPC)** channel on Android. It supports a variety of features such as passing file descriptors and objects containing pointers across process boundaries

 - All untrusted apps on Android are sandboxed and inter-process communication mostly occurs through Binder. 
  - untrusted apps can use binder using  char device `/dev/binder` . Binder is implemented as a kernel drivers .
  - Binder Works similar to io-uring in some aspects such as that userspace needs to allocate a buffer through which the driver and userpsace  can communicate regarding the commands .
-  we  can  communicate with the binder interface using ioctl calls .

```
  static const char * const binder_command_strings[] = {
	"BC_TRANSACTION",
	"BC_REPLY",
	"BC_ACQUIRE_RESULT",
	"BC_FREE_BUFFER",
	"BC_INCREFS",
	"BC_ACQUIRE",
	"BC_RELEASE",
	"BC_DECREFS",
	"BC_INCREFS_DONE",
	"BC_ACQUIRE_DONE",
	"BC_ATTEMPT_ACQUIRE",
	"BC_REGISTER_LOOPER",
	"BC_ENTER_LOOPER",
	"BC_EXIT_LOOPER",
	"BC_REQUEST_DEATH_NOTIFICATION",
	"BC_CLEAR_DEATH_NOTIFICATION",
	"BC_DEAD_BINDER_DONE",
	"BC_TRANSACTION_SG",
	"BC_REPLY_SG",
	"BC_REQUEST_FREEZE_NOTIFICATION",
	"BC_CLEAR_FREEZE_NOTIFICATION",
	"BC_FREEZE_NOTIFICATION_DONE",
};

```

To understand the high level implementation of the /dev/binder refer to  [this](https://medium.com/swlh/binder-architecture-and-core-components-38089933bba):  


##  Vulnerabilties

###  CVE-2019-2215

CVE-2019-2215 is Use-After-Free vulnerability in the binder kernel driver . driver. The `binder_thread` struct, defined in `drivers/android/binder.c`, has the member `wait` of the `wait_queue_head_t` struct type. `wait` is still referenced by a pointer in `epoll`, even after the `binder_thread` struct containing it is freed.

### Understanding the vulnerability 

So essential the vulnerability is that even after calling `BINDER_THREAD_EXIT`  which frees `struct binder_thread` associated with the proc . There exists a reference to the `wait_queue_head_t wait` in the poll data structure . So when we cleanup / call EPOLL_CTL_DEL  we call  `__remove_wait_queue` on the wait queue which leads to a UAF .


```C
static inline void

__remove_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)

{

list_del(&wq_entry->entry);

}
```
 
### Summary of  Exploit Strategy     
-  Exploit described in the project zero blog [post](https://googleprojectzero.blogspot.com/2019/11/bad-binder-android-in-wild-exploit.html) uses struct iovecs of size 0x10 to overlap our struct `wait_queue_head_t`  which can be to used to leak kernel memory and using the recvmsg() on a socket pair gain arbitary write into kernel memory.
- pipes are used in order to allow the exploit to block threads and control the race-condition while making it reliable.
- when leaking the `struct task_struct *` we use pipes in order to block threads and continue execution from a ideal state (we use writev and readv on the pipfds).
