# Bitcoin addresses bruteforce tool via OpenCL

This is fork of https://github.com/svtrostov/oclexplorer repo adapted for Visual Studio 2017.

## How it works

This software is designed as a tool for studying the safety aspects of using elliptic curves in practice.
The program searches (generate) the private keys, calculates the public key and on its basis the ripemd160 hash, looks for the calculated hash in the list of hashes of existing bitcoin addresses.
If a hash match is found, this means that a private key was found to one of the specified bitcoin addresses.

1. First you need to create a file with a list of ripemd160 hashes of the bitcoin addresses of interest in binary format.
This is a file in which the hashes of the addresses of interest in binary format (20 bytes per hash) are written one by one.
Attached to this project is a file with hashes of bitcoin addresses (bitcoin.bin file).

2. Next, you should run the program oclexplorer.exe with the hash file created as a parameter (see the corresponding section for launch parameters).

3. If the initial private key is not specified, then it will be randomly generated. The public key is calculated from the private key, and the ripemd160 hash is calculated from it (for more details on calculating the bitcoin address from the private key, see here: https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses), the received hash is searched in the list of hashes of interest (if a match is found, then the private key is found, information about the find will be displayed on the screen and written to the found.txt file in the program folder), then the private key is incremented by one and the process repeats.

##  Program parameters

	-b		A required parameter that tells the program the path to the file with the list of ripemd160 hashes in binary format.
			Example: oclexplorer.exe -b bitcoin.bin says that you should use the hash list from the bitcoin.bin file

	-k		The parameter, you can specify the private key (32-byte number in HEX format), from which the calculation of hashes will begin.
			If the private key is not specified, then its value will be randomly generated.
			Example: oclexplorer.exe -b bitcoin.bin -k 1234567890ABCDEF1234567890ABCDEF1234567890ABCDEF1234567890ABCDEF

	-u		if this parameter is specified, then the private key will increment to infinity,
			otherwise, each time after incrementing 4 bytes (4,294,967,296), the value of the private key will be regenerated and set randomly.

	-p		Platform ID. By default, set to 0. Normally, this parameter is never used if your hardware got only one device - one GPU or One CPU with OpenCL support.
			The OpenCL platform consists of a host connected to devices that support OpenCL.

	-d		Device ID in the selected platform to use for calculations. Set to 0 by default.
			This parameter is used if several devices supporting OpenCL are installed on the host, and it is required to specify on which device to perform calculations.
			Example: oclexplorer.exe -b bitcoin.bin -d 0

	-g		Parameter sets the size of the computational matrix (cols) x (rows) (numbers in the DEC format). These values are calculated automatically based on the GPU data.
			You can experiment with these values to improve performance.
			Example: oclexplorer.exe -b bitcoin.bin -g 1024x512

	-i		Parameter sets the size of the queue of modular inversion of numbers (a number in DEC format, always must be a power of 2).
			This value is calculated automatically based on the GPU data.
			You can experiment with this value to improve performance.
			Example: oclexplorer.exe -b bitcoin.bin -g 1024x512 -i 256

	-f		The file name of the OpenCL script, if the parameter is not specified, then the default file is "gpu.cl" (attached to this project)
			It is convenient to use if you want to experiment with modified OpenCL scripts.
			Example: oclexplorer.exe -f test.cl 


## Tests

Synthetic test, which will more clearly show the operation of the program.

Example wallet with follow parameters:

Address: 1testBq2oGSeE3WVfi1MUbJZbUSWwEktC

Private key: 6AA3789CFE067047480EED275D4A017B812D19AE6A4B82105E0B7DCEAF64A1B5

Public key: 0446C360263B1794E429E7D672A878B5083C37D6BA177BFB68405EED3DB01804A210BDFD925C8E2CD16C054C3919C6F0889376E96EDC6B2BAF5D06D8F139601268

RIPEMD-160: 09C4D5193EE73DE349FB8237CC83D14ED47DE115

Save upper the RIPEMD-160 hashe in binary format in the file "test1.bin"

Start the program with the following parameters:

oclexplorer.exe -b test1.bin -k 6AA3789CFE067047480EED275D4A017B812D19AE6A4B82105E0B7DCEAF000000 -u

After some time, the program will find a calculated hash with one of the hashes stored in the file "test1.bin"

The search result will be displayed and saved in the file "found.txt"

    ++++++++++++++++++++++++++++++++++++++++++++++++++
    
    TIME: 2018-01-25 16:03:42
    
    PRIV: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf64a1b5
    
    PUBL: 0446c360263b1794e429e7d672a878b5083c37d6ba177bfb68405eed3db01804a210bdfd925c8e2cd16c054c3919c6f0889376e96edc6b2baf5d06d8f139601268
    
    HASH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    
    ADDR: 1testBq2oGSeE3WVfi1MUbJZbUSWwEktC
    
    SALT: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf400000
    
    OFST: 2400692
    
    GPUH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    
    ++++++++++++++++++++++++++++++++++++++++++++++++++


Description of the result:

++++++++++++++++++++++++++++++++++++++++++++++++++

TIME: Date and time

PRIV: Private Key Found

PUBL: Public key

HASH: Hash bitcoin addresses in RIPEMD-160 format

ADDR: Bitcoin address

SALT: Private key from which the calculation began

OFST: Delta between found and initial private keys

GPUH: RIPEMD-160 bitcoin hash calculated on the GPU must always be HASH

++++++++++++++++++++++++++++++++++++++++++++++++++



## Install and run

Performance was tested on OS Windows 10.
Tested on video cards: Nvidia GTX 1050 (OpenCL 1.2 CUDA), Intel(R) HD Graphics 630 (OpenCL 2.1 NEO), Intel(R) Gen9 HD Graphics NEO (OpenCL 1.2 NEO).
Tested on CPU: Intel(R) Core(TM) i7-7700HQ (OpenCL 2.1), Intel(R) Pentium(R) Silver J5005 CPU @ 1.50GHz (OpenCL 2.1).
Work requires installed OpenCL driver (Nvidia and/or Intel CPU or GPU) and OpenSSL libraries (1.0.2 series, latter version incompatible by API) with elliptic curve support enabled.


## Denial of responsibility

The author is not responsible for the consequences caused by your use of this software.
Remember that finding the private key of a bitcoin wallet and appropriating the funds located on it, depending on your jurisdiction, may be considered a theft and therefore illegal.

## Program runing examples, outputs and perf counters

The test platform has the following configuration OpenCL devices:

    Starting OpenCL device query:
    Number of OpenCL capable platforms available: 2
             Platform ID: 0
             ----------------
    
                     [Platform 0] CL_PLATFORM_NAME: Intel(R) OpenCL
                     [Platform 0] CL_PLATFORM_VENDOR: Intel(R) Corporation
                     [Platform 0] CL_PLATFORM_VERSION: OpenCL 2.1
                     [Platform 0] CL_PLATFORM_PROFILE: FULL_PROFILE
                     [Platform 0] CL_PLATFORM_EXTENSIONS: cl_khr_3d_image_writes cl_khr_byte_addressable_store cl_khr_depth_images cl_khr_fp64 cl_khr_global_int32_base_atomics cl_khr_global_int32_extended_atomics cl_khr_icd cl_khr_image2d_from_buffer cl_khr_local_int32_base_atomics cl_khr_local_int32_extended_atomics cl_khr_spir
    
                     [Platform 0] Number of devices available: 2
                     ---------------------------------------------
    
                             [Platform 0] Device ID: 0
                             ---------------------------
    
                                             [Platform 0] [Device 0] CL_DEVICE_NAME: Intel(R) HD Graphics 630
                                             [Platform 0] [Device 0] CL_DEVICE_VERSION: OpenCL 2.1 NEO
                                             [Platform 0] [Device 0] CL_DRIVER_VERSION: 26.20.100.7463
                                             [Platform 0] [Device 0] CL_DEVICE_OPENCL_C_VERSION: OpenCL C 2.0
                                             [Platform 0] [Device 0] CL_DEVICE_MAX_CLOCK_FREQUENCY: 1100 MHz
                                             [Platform 0] [Device 0] CL_DEVICE_MAX_COMPUTE_UNITS: 24
                                             [Platform 0] [Device 0] CL_DEVICE_GLOBAL_MEM_SIZE: 3229 MB
                                             [Platform 0] [Device 0] CL_DEVICE_MAX_MEM_ALLOC_SIZE: 1614 MB
                                             [Platform 0] [Device 0] CL_DEVICE_LOCAL_MEM_SIZE: 64 KB
                                             [Platform 0] [Device 0] CL_DEVICE_MAX_WORK_GROUP_SIZE: 256
                                             [Platform 0] [Device 0] CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS: 3
                                             [Platform 0] [Device 0] CL_DEVICE_MAX_WORK_ITEM_SIZES: 256 256 256
                                             [Platform 0] [Device 0] CL_DEVICE_IMAGE_SUPPORT: 1 (Available)
                                             [Platform 0] [Device 0] CL_DEVICE_IMAGE2D_MAX_WIDTH: 16384
                                             [Platform 0] [Device 0] CL_DEVICE_IMAGE2D_MAX_HEIGHT: 16384
                                             [Platform 0] [Device 0] CL_DEVICE_IMAGE3D_MAX_WIDTH: 16384
                                             [Platform 0] [Device 0] CL_DEVICE_IMAGE3D_MAX_HEIGHT: 16384
                                             [Platform 0] [Device 0] CL_DEVICE_IMAGE3D_MAX_DEPTH: 2048
    
    
                             [Platform 0] Device ID: 1
                             ---------------------------
    
                                             [Platform 0] [Device 1] CL_DEVICE_NAME: Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz
                                             [Platform 0] [Device 1] CL_DEVICE_VERSION: OpenCL 2.1 (Build 0)
                                             [Platform 0] [Device 1] CL_DRIVER_VERSION: 7.6.0.0814
                                             [Platform 0] [Device 1] CL_DEVICE_OPENCL_C_VERSION: OpenCL C 2.0
                                             [Platform 0] [Device 1] CL_DEVICE_MAX_CLOCK_FREQUENCY: 2800 MHz
                                             [Platform 0] [Device 1] CL_DEVICE_MAX_COMPUTE_UNITS: 8
                                             [Platform 0] [Device 1] CL_DEVICE_GLOBAL_MEM_SIZE: 8073 MB
                                             [Platform 0] [Device 1] CL_DEVICE_MAX_MEM_ALLOC_SIZE: 2018 MB
                                             [Platform 0] [Device 1] CL_DEVICE_LOCAL_MEM_SIZE: 32 KB
                                             [Platform 0] [Device 1] CL_DEVICE_MAX_WORK_GROUP_SIZE: 8192
                                             [Platform 0] [Device 1] CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS: 3
                                             [Platform 0] [Device 1] CL_DEVICE_MAX_WORK_ITEM_SIZES: 8192 8192 8192
                                             [Platform 0] [Device 1] CL_DEVICE_IMAGE_SUPPORT: 1 (Available)
                                             [Platform 0] [Device 1] CL_DEVICE_IMAGE2D_MAX_WIDTH: 16384
                                             [Platform 0] [Device 1] CL_DEVICE_IMAGE2D_MAX_HEIGHT: 16384
                                             [Platform 0] [Device 1] CL_DEVICE_IMAGE3D_MAX_WIDTH: 2048
                                             [Platform 0] [Device 1] CL_DEVICE_IMAGE3D_MAX_HEIGHT: 2048
                                             [Platform 0] [Device 1] CL_DEVICE_IMAGE3D_MAX_DEPTH: 2048
    
    
             Platform ID: 1
             ----------------
    
                     [Platform 1] CL_PLATFORM_NAME: NVIDIA CUDA
                     [Platform 1] CL_PLATFORM_VENDOR: NVIDIA Corporation
                     [Platform 1] CL_PLATFORM_VERSION: OpenCL 1.2 CUDA 10.1.120
                     [Platform 1] CL_PLATFORM_PROFILE: FULL_PROFILE
                     [Platform 1] CL_PLATFORM_EXTENSIONS: cl_khr_global_int32_base_atomics cl_khr_global_int32_extended_atomics cl_khr_local_int32_base_atomics cl_khr_local_int32_extended_atomics cl_khr_fp64 cl_khr_byte_addressable_store cl_khr_icd cl_khr_gl_sharing cl_nv_compiler_options cl_nv_device_attribute_query cl_nv_pragma_unroll cl_nv_d3d10_sharing cl_khr_d3d10_sharing cl_nv_d3d11_sharing cl_nv_copy_opts cl_nv_create_buffer
    
                     [Platform 1] Number of devices available: 1
                     ---------------------------------------------
    
                             [Platform 1] Device ID: 0
                             ---------------------------
    
                                             [Platform 1] [Device 0] CL_DEVICE_NAME: GeForce GTX 1050
                                             [Platform 1] [Device 0] CL_DEVICE_VERSION: OpenCL 1.2 CUDA
                                             [Platform 1] [Device 0] CL_DRIVER_VERSION: 431.86
                                             [Platform 1] [Device 0] CL_DEVICE_OPENCL_C_VERSION: OpenCL C 1.2
                                             [Platform 1] [Device 0] CL_DEVICE_MAX_CLOCK_FREQUENCY: 1493 MHz
                                             [Platform 1] [Device 0] CL_DEVICE_MAX_COMPUTE_UNITS: 5
                                             [Platform 1] [Device 0] CL_DEVICE_GLOBAL_MEM_SIZE: 2048 MB
                                             [Platform 1] [Device 0] CL_DEVICE_MAX_MEM_ALLOC_SIZE: 512 MB
                                             [Platform 1] [Device 0] CL_DEVICE_LOCAL_MEM_SIZE: 48 KB
                                             [Platform 1] [Device 0] CL_DEVICE_MAX_WORK_GROUP_SIZE: 1024
                                             [Platform 1] [Device 0] CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS: 3
                                             [Platform 1] [Device 0] CL_DEVICE_MAX_WORK_ITEM_SIZES: 1024 1024 64
                                             [Platform 1] [Device 0] CL_DEVICE_IMAGE_SUPPORT: 1 (Available)
                                             [Platform 1] [Device 0] CL_DEVICE_IMAGE2D_MAX_WIDTH: 16384
                                             [Platform 1] [Device 0] CL_DEVICE_IMAGE2D_MAX_HEIGHT: 32768
                                             [Platform 1] [Device 0] CL_DEVICE_IMAGE3D_MAX_WIDTH: 16384
                                             [Platform 1] [Device 0] CL_DEVICE_IMAGE3D_MAX_HEIGHT: 16384
                                             [Platform 1] [Device 0] CL_DEVICE_IMAGE3D_MAX_DEPTH: 16384

1. Run test on Platform 0 Device ID 0 -  Intel(R) HD Graphics 630 (OpenCL 2.1 NEO):

    oclexplorer.exe -b test1.bin -k 6AA3789CFE067047480EED275D4A017B812D19AE6A4B82105E0B7DCEAF000000 -u

The follow output for running on Intel(R) HD Graphics 630:

    [test1.bin] loading...   loaded 2 records : 0.004139 sec
    Load program file: ./gpu.cl
    Compiling CL, can take minutes...Grid size: 3072x4096, total 12582912
    Modular inverse: 6144 threads, 2048 ops each
    
    ==[SELECTED DEVICE INFO]============================
    Device: Intel(R) HD Graphics 630
    Vendor: Intel(R) Corporation (8086)
    Driver: 26.20.100.7463
    Profile: FULL_PROFILE
    Version: OpenCL 2.1 NEO
    Max compute units: 24
    Max workgroup size: 256
    Global memory: 3386306560
    Max allocation: 1693153280
    
    
    Iteration 1 at [2020-01-23 18:59:26] from: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf000000
    
    
    ++++++++++++++++++++++++++++++++++++++++++++++++++
    TIME: 2020-01-23 18:59:31
    PRIV: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf64a1b5
    PUBL: 0446c360263b1794e429e7d672a878b5083c37d6ba177bfb68405eed3db01804a210bdfd925c8e2cd16c054c3919c6f0889376e96edc6b2baf5d06d8f139601268
    HASH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    ADDR: 1testBq2oGSeE3WVfi1MUbJZbUSWwEktC
    SALT: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf000000
    OFST: 6594996
    GPUH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    ++++++++++++++++++++++++++++++++++++++++++++++++++

[6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceb0800000], round 3: 2.49s (5.05 Mkey/s) [total 37748736 (3.63 Mkey/s)]

2. Run test on Platform 0 Device ID 1 - Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz (OpenCL 2.1)

    oclexplorer.exe -b test1.bin -k 6AA3789CFE067047480EED275D4A017B812D19AE6A4B82105E0B7DCEAF000000 -u -p 0 -d 1

The follow output for running on Intel(R) Core(TM) i7-7700HQ:

    [test1.bin] loading...   loaded 2 records : 0.000277 sec
    Load program file: ./gpu.cl
    Compiling CL, can take minutes...Grid size: 4096x4096, total 16777216
    Modular inverse: 65536 threads, 256 ops each
    
    ==[SELECTED DEVICE INFO]============================
    Device: Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz
    Vendor: Intel(R) Corporation (8086)
    Driver: 7.6.0.0814
    Profile: FULL_PROFILE
    Version: OpenCL 2.1 (Build 0)
    Max compute units: 8
    Max workgroup size: 8192
    Global memory: 8465768448
    Max allocation: 2116442112
    
    
    Iteration 1 at [2020-01-23 19:03:47] from: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf000000
    
    
    ++++++++++++++++++++++++++++++++++++++++++++++++++
    TIME: 2020-01-23 19:03:53
    PRIV: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf64a1b5
    PUBL: 0446c360263b1794e429e7d672a878b5083c37d6ba177bfb68405eed3db01804a210bdfd925c8e2cd16c054c3919c6f0889376e96edc6b2baf5d06d8f139601268
    HASH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    ADDR: 1testBq2oGSeE3WVfi1MUbJZbUSWwEktC
    SALT: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf000000
    OFST: 6594996
    GPUH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    ++++++++++++++++++++++++++++++++++++++++++++++++++

[6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceb2000000], round 4: 5.41s (3.10 Mkey/s) [total 67108864 (2.91 Mkey/s)]

As seen above performance on GPU cores much better than on the same CPU cores: On GPU core Intel(R) HD Graphics 630 (5.05 Mkey/s) of Intel(R) Core(TM) i7-7700HQ (3.10 Mkey/s)
And now look at third device on my test system.

3. Run test on Platform 1 Device ID 0 - NVIDIA GeForce GTX 1050 (OpenCL 1.2 CUDA)

    oclexplorer.exe -b test1.bin -k 6AA3789CFE067047480EED275D4A017B812D19AE6A4B82105E0B7DCEAF000000 -u -p 1 -d 0

The follow output for running on NVIDIA GeForce GTX 1050:

    [test1.bin] loading...   loaded 2 records : 0.000212 sec
    Load program file: ./gpu.cl
    Compiling CL, can take minutes...Grid size: 2560x2048, total 5242880
    Modular inverse: 5120 threads, 1024 ops each
    
    ==[SELECTED DEVICE INFO]============================
    Device: GeForce GTX 1050
    Vendor: NVIDIA Corporation (10de)
    Driver: 431.86
    Profile: FULL_PROFILE
    Version: OpenCL 1.2 CUDA
    Max compute units: 5
    Max workgroup size: 1024
    Global memory: 2147483648
    Max allocation: 536870912
    
    
    Iteration 1 at [2020-01-23 19:12:02] from: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf000000
    [6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf000000], round 1: 0.46s (11.51 Mkey/s) [total 5242880 (6.11 Mkey/s)]
    
    ++++++++++++++++++++++++++++++++++++++++++++++++++
    TIME: 2020-01-23 19:12:03
    PRIV: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf64a1b5
    PUBL: 0446c360263b1794e429e7d672a878b5083c37d6ba177bfb68405eed3db01804a210bdfd925c8e2cd16c054c3919c6f0889376e96edc6b2baf5d06d8f139601268
    HASH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    ADDR: 1testBq2oGSeE3WVfi1MUbJZbUSWwEktC
    SALT: 6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceaf500000
    OFST: 1352116
    GPUH: 09c4d5193ee73de349fb8237cc83d14ed47de115
    ++++++++++++++++++++++++++++++++++++++++++++++++++

[6aa3789cfe067047480eed275d4a017b812d19ae6a4b82105e0b7dceb6d00000], round 26: 0.39s (13.43 Mkey/s) [total 136314880 (12.84 Mkey/s)]

In this tests NVIDIA OpenCL looks much better than  Intel(R) OpenCL HD Graphics.

I will be glad any donate for beer:

	Bitcoin address: 13BpPaiENfzrUFrpPwGaVsVDwNr6Ccm7nZ
	Litecoin address: 1tc1qry72uty38jpj30lfuyxtdguf436xlsjuf57qxr




