![[Binder.png]]

Binder is the InterProcess Communication method of Android
- Low Level kernel API (/dev/binder is the interface)

A user space program mainly interacts with Binder driver using three system calls: `open`, `mmap` and `ioctl`. A process calls `open` to register itself as a user of Binder driver. `mmap` is used to create a kernel data buffer and reserve a range of user space virtual address for it. After `open` and `mmap` a process can interact with the driver using `ioctl`. The most important `ioctl` command is `BINDER_WRITE_READ` with which a process can write and read data payload from the Binder driver

Binder Internals:
![[binderschematic.png]]

The Apps/Services have proxy and stub written using AIDL, to perform binder transactions.
The Android Framework contains several abstraction layers on top of the binder device. Usually when developers implement new services they describe interfaces exposed in a high level language. In the case of framework application, descriptions are written with the [**AIDL**](https://developer.Android.com/guide/components/aidl) language, while hardware services developed by vendors, have interface descriptions written in [**HIDL**](https://source.Android.com/devices/architecture/hidl#grammar) language. Theses descriptions are compiled into Java/C++ files where parameters are de/serialized using **Parcel** component. Generated code contains two classes, a **Binder Proxy** and a **Binder Stub**. The proxy class is used to request a distant service and the stub to receive incoming call as described on the following diagram.

A parcel is a serialised package(can contain data, objects which's class implement parcelable)) is sent and received via the binder module in the kernel. 

Sources and Further readings:
https://null-android-pentesting.netlify.app/src/android-internals#android-architecture-and-binder
https://www.synacktiv.com/publications/binder-transactions-in-the-bowels-of-the-linux-kernel
https://dispatchersdotplayground.hashnode.dev/interprocess-communication-and-the-binder-interface

