How to compile Mali Midgard drivers from source

# **Resources Needed:**

- Midgard driver: version used for compiling in this doc, TX056-SW-99002-r32p0-01eac0.tgz
    https://developer.arm.com/downloads/-/Mali%20Midgard%20GPU%20Kernel%20Drivers
- Compiler: Clang v18
	Ubuntu clang version 18.1.3 (1ubuntu1)`
	Can also use prebuilt clang from google source corresponding to tag of kernel source used
	https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/
- Kernel source code for headers
	https://android.googlesource.com/kernel/common/
	branch used: android12-5.4.254_r00
	recommended to use 5.4 headers since it leads to lesser compilation errors
- Build essentials for kernel compilation
  https://source.android.com/docs/setup/start/requirements#install-packages

# Compilation

1. Clone the driver source code and kernel source
	`git clone https://github.com/FrosT2k5/midgard_driver_codeql ~/midgard_driver/`
	`git clone https://android.googlesource.com/kernel/common/ --depth=1 ~/kernel/`
	
2. Compile kernel headers
	- cd into kernel source
		`cd ~/kernel`
	- make defconfig
		`make CC=clang ARCH=arm64 LLVM=1 CROSS_COMPILE=aarch64-linux-gnu- gki_defconfig`
		args: use clang for compilation, arm64(android) arch and GCC as cross compiler, LLVM=1 is used for clang based compilation
	- build kernel headers
		`make CC=clang ARCH=arm64 LLVM=1 CROSS_COMPILE=aarch64-linux-gnu- prepare`

3. Compile driver with the kernel headers
	- cd into driver source
		`cd ~/midgard_driver/product/kernel/drivers/gpu/arm`
	- compile with codeql wrapper
		`codeql database create ~/midgarddb --language=cpp --command="make CC=clang ARCH=arm64 LLVM=1 CROSS_COMPILE=aarch64-linux-gnu- KDIR=~/kernel" --overwrite`
	- This will initialize the midgard codeql db at ~/midgarddb folder after successful compilation


Repo with source code and precompiled database: https://github.com/FrosT2k5/midgard_driver_codeql/