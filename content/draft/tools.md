# general notes

- Various tools may be a part of other packages, for example:
  - Majority of Windows GCC distributions apart from compilers also include GNU Binutils.
  - Some Linux tools are available on Windows as parts of additional shells (Git Bash, Cygwin, MSYS).
- Tools that are ported from Linux to Windows may have slightly different behaviour
- Documentation on command line tools available on GNU/Linux: https://linux.die.net/man/

## nm

Lists symbols in files.

The tool can be used to debug "undefined reference" errors. You can run the tool against any binary file:
- `.o`, `.obj` (object code)
- `.so`, `.dll` (dynamic link library)
- `.a`, `.lib` (static link library)
- `.out`, `.exe` (executable)
- and more

Note that various binary files may have different types of symbols. `nm` will mark the symbol type with a letter.

Example entries:

```
$ nm -SsCg ./Release/filter_spirit.exe
00000000004c21a0 R .refptr._ZSt4cerr
00000000004c21b0 R .refptr._ZSt4cout
00000000004c21c0 R .refptr._ZSt7nothrow
00000000004b0190 T ___CTOR_LIST__
00000000004b01e8 T ___DTOR_LIST__
00000000004402d0 T ___lc_codepage_func
00000000004c3dd0 R __fu10__ZTVN10__cxxabiv117__class_type_infoE
00000000004e8de0 I __imp_SSL_accept
00000000004e8de8 I __imp_SSL_connect
00000000004ee008 D _tls_end
00000000004e64fc B _tls_index
00000000004ee000 D _tls_start
00000000004c1840 R _tls_used
                 U std::__throw_bad_variant_access(char const*)
00000000004a60d0 T non-virtual thunk to boost::wrapexcept<boost::bad_function_call>::~wrapexcept()
00000000004c2eb0 R construction vtable for std::ostream-in-boost::beast::detail::static_ostream
00000000004a5b90 T void std::call_once<void (std::thread::*)(), std::thread*>(std::once_flag&, void (std::thread::*&&)(), std::thread*&&)
0000000000436570 T std::terminate()
00000000004a5d20 T std::operator==(std::error_code const&, std::error_condition const&)
0000000000436568 T std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)
0000000000000000 R typeinfo name for boost::exception_detail::clone_base
0000000000000000 R typeinfo for std::call_once<void (std::thread::*)(), std::thread*>(std::once_flag&, void (std::thread::*&&)(), std::thread*&&)::{lambda()#1}
0000000000000000 R typeinfo name for __cxxabiv1::__forced_unwind
                 U vtable for __cxxabiv1::__class_type_info
                 U memcpy
                 U memmove
                 U memset
                 U strerror
                 U strlen
                 U strncpy
```

For debug builds, you will usually get more symbols as debug code is not cleaned up of redundant functions for debugability reasons.

Symbol types documentation: https://linux.die.net/man/1/nm

## objdump

Displays information about object files.

Example:

```
$ objdump.exe -f ./Release/filter_spirit.exe

./Release/filter_spirit.exe:     file format pei-x86-64
architecture: i386:x86-64, flags 0x0000013a:
EXEC_P, HAS_DEBUG, HAS_SYMS, HAS_LOCALS, D_PAGED
start address 0x00000000004014ae
```

This tool can be used to extract various information contained in binary files, eg symbol names, imports/exports, architecture information, hardcoded data (numbers, strings etc).

https://linux.die.net/man/1/objdump

## ldd

Lists shared library dependencies.

```
$ ldd ./Release/filter_spirit.exe
        ntdll.dll => /c/WINDOWS/SYSTEM32/ntdll.dll (0x7ffc6cd50000)
        KERNEL32.DLL => /c/WINDOWS/System32/KERNEL32.DLL (0x7ffc6a680000)
        KERNELBASE.dll => /c/WINDOWS/System32/KERNELBASE.dll (0x7ffc696b0000)
        msvcrt.dll => /c/WINDOWS/System32/msvcrt.dll (0x7ffc6c050000)
        USER32.dll => /c/WINDOWS/System32/USER32.dll (0x7ffc6c490000)
        win32u.dll => /c/WINDOWS/System32/win32u.dll (0x7ffc69540000)
        GDI32.dll => /c/WINDOWS/System32/GDI32.dll (0x7ffc6a320000)
        gdi32full.dll => /c/WINDOWS/System32/gdi32full.dll (0x7ffc69320000)
        msvcp_win.dll => /c/WINDOWS/System32/msvcp_win.dll (0x7ffc695c0000)
        ucrtbase.dll => /c/WINDOWS/System32/ucrtbase.dll (0x7ffc6a070000)
        WS2_32.dll => /c/WINDOWS/System32/WS2_32.dll (0x7ffc6c680000)
        RPCRT4.dll => /c/WINDOWS/System32/RPCRT4.dll (0x7ffc6bf20000)
        libgcc_s_seh-1.dll => /mingw64/bin/libgcc_s_seh-1.dll (0x61440000)
        libboost_program_options-mt.dll => /c/mingw64/mingw64/bin/libboost_program_options-mt.dll (0x67f80000)
        libcrypto-1_1-x64.dll => /c/mingw64/mingw64/bin/libcrypto-1_1-x64.dll (0x3010000)
        WSOCK32.dll => /c/WINDOWS/SYSTEM32/WSOCK32.dll (0x7ffc53710000)
        libwinpthread-1.dll => /mingw64/bin/libwinpthread-1.dll (0x64940000)
        mcfgthread-12.dll => /c/mingw64/mingw64/bin/mcfgthread-12.dll (0x6c080000)
        libssl-1_1-x64.dll => /c/mingw64/mingw64/bin/libssl-1_1-x64.dll (0x6d480000)
        libstdc++-6.dll => /mingw64/bin/libstdc++-6.dll (0x6fc40000)
        ??? => ??? (0x27a0000)
        libcrypto-1_1-x64.dll => /c/mingw64/mingw64/bin/libcrypto-1_1-x64.dll (0x3010000)

```

Here we see that an executable is using:
- a bunch of Windows system-provided libraries:
  - `ntdll.dll` - NT stuff, camed first with Windows NT),
  - `KERNEL32.DLL`, `KERNELBASE.dll` - probably kernel API, I guess syscalls
  - `msvcrt.dll` - **M**icro**s**oft **V**isual **C** **r**un**t**ime
  - `USER32.dll` - I guess some stuff for 32-bit userspace programs
  - `WSOCK32.dll`, `WS2_32.dll` - windows sockets (networking)
  - more things needed for any Windows program
- `libboost_program_options-mt.dll` - Boost Program Options with **m**ulti**t**hreading support
- `libssl-1_1-x64.dll` - OpenSSL 1.1
- `libcrypto-1_1-x64.dll` - OpenSSL 1.1 (cryptographic utilities)
- `libwinpthread-1.dll` - Windows pthread implementation
- `mcfgthread-12.dll` - MCF thread implementation
- `libgcc_s_seh-1.dll` - GCC's **s**tructured **e**xception **h**andling
- `libstdc++-6.dll` - C++ standard library
- `???` - something unknown for the tool

If we would like to deploy, tools such as `ldd` can help by listing libraries that are needed. When the program is developed, all the libraries likely reside in some dedicated place for easier building (here in the compiler `bin` directory) but a typical user is unlikely to have a compiler installed (let alone the required version) so it's better to just copy the non-system libraries to executable's directory and ship them together.

## size

Displays the sizes of sections inside binary files.

```
$ size ./Release/filter_spirit.exe
   text    data     bss     dec     hex filename
 903248   36268    9088  948604   e797c ./Release/filter_spirit.exe
```

## strip

Removes symbols and sections from files. Can be used to "optimize" executables by discarding any unneeded data.

```
$ ll ./Release/
total 3478
-rwxr-xr-x 1 admin 1049089 3550413 Mar  5 10:35 filter_spirit.exe*
-rw-r--r-- 1 admin 1049089    1651 Mar  5 10:35 makefile
-rw-r--r-- 1 admin 1049089     318 Mar  5 10:35 objects.mk
-rw-r--r-- 1 admin 1049089     658 Mar  5 10:35 sources.mk
drwxr-xr-x 1 admin 1049089       0 Mar  4 10:16 src/
$ strip ./Release/filter_spirit.exe
$ ll ./Release/
total 934
-rwxr-xr-x 1 admin 1049089 943104 Mar  5 13:53 filter_spirit.exe*
-rw-r--r-- 1 admin 1049089   1651 Mar  5 10:35 makefile
-rw-r--r-- 1 admin 1049089    318 Mar  5 10:35 objects.mk
-rw-r--r-- 1 admin 1049089    658 Mar  5 10:35 sources.mk
drwxr-xr-x 1 admin 1049089      0 Mar  4 10:16 src/
```

Here, `strip` with default options reduced a release build binary from ~3.5 MB to ~940 KB.
