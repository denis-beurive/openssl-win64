# OpenSSL compilation on Windows 11

## Versions and architecture

OS Version: `Windows 11 Famille`

OpenSSL version: `openssl-3.3.0`

MSVC version: `Compilateur d'optimisation Microsoft (R) C/C++ version 19.40.33808 pour x64`

Architecture: `Intel(R) Core(TM) i7` / `64 bits, processeur x64`

## Preparation

* Download: [https://www.openssl.org/source/](https://www.openssl.org/source/)
* Install [Perl](https://strawberryperl.com/) (default installation directory: `C:\Strawberry\perl\bin`)
* Install [NASM](https://www.nasm.us/) (default installation directory: `"%userprofile%"\AppData\Local\bin\NASM`)
* Make sure both Perl and NASM are on your `%PATH%`.
* Start PowerShell with administrator privileges.
* Extract the content of the OpenSSL source archive.
* `cd openssl-3.3.0`
* Run `C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat amd64`.
* Run `perl Configure VC-WIN64A` from the OpenSSL top directory.

## Configuring the environment

### Add rc.exe to the %PATH%

Add the path to the MSVC tools to your `%PATH%` (search for `rc.exe`, for example).

For example: `C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x64`

    cmd /c for %A in ("C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x64") do @echo %~sA

The result is: `C:\PROGRA~2\WI3CF2~1\10\bin\100261~1.0\x64`

Under PowerShell, you can add this new path for the duration of the session:

    $env:Path += ';C:\PROGRA~2\WI3CF2~1\10\bin\100261~1.0\x64'

### C++ compiler (CL.EXE): configure the search paths for header files

Look for the directories used by the compiler to search for header files. These directories are:

* `C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.40.33807\include`
* `C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\ucrt`
* `C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\um`
* `C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\shared`

Get the shortened paths:

    cmd /c for %A in ("C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.40.33807\include") do @echo %~sA
    cmd /c for %A in ("C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\ucrt") do @echo %~sA
    cmd /c for %A in ("C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\um") do @echo %~sA
    cmd /c for %A in ("C:\Program Files (x86)\Windows Kits\10\Include\10.0.26100.0\shared") do @echo %~sA

Shortened paths are:

* `C:\Progra~1\Micros~4\2022\Community\VC\Tools\MSVC\14.40.33807\include`
* `C:\PROGRA~2\WI3CF2~1\10\Include\100261~1.0\ucrt`
* `C:\PROGRA~2\WI3CF2~1\10\Include\100261~1.0\um`
* `C:\PROGRA~2\WI3CF2~1\10\Include\100261~1.0\shared`

Edit the file "`makefile`", and add the following variables:

    ##### User defined commands and flags ################################

    INCLUDE1=C:\Progra~1\Micros~4\2022\Community\VC\Tools\MSVC\14.40.33807\include
    INCLUDE2=C:\PROGRA~2\WI3CF2~1\10\Include\100261~1.0\ucrt
    INCLUDE3=C:\PROGRA~2\WI3CF2~1\10\Include\100261~1.0\um
    INCLUDE4=C:\PROGRA~2\WI3CF2~1\10\Include\100261~1.0\shared

Modify the `CFLAGS` variable:

    CFLAGS=/W3 /wd4090 /nologo /O2 --help -I"$(INCLUDE1)" -I$(INCLUDE2) -I$(INCLUDE3) -I$(INCLUDE4)

### Resource compiler (RC.EXE): configure the search paths for header files

Edit the file "`makefile`", and modify the `RCFLAGS` variable:

    RC="rc"
    RCFLAGS=/i $(INCLUDE1) /i $(INCLUDE2) /i $(INCLUDE3) /i $(INCLUDE4)

### Linker (LINK.EXE): configure the search paths for libraries

Look for the directories used by the linker to find the required libraries. These directories are:

* `C:\Program Files (x86)\Windows Kits\10\Lib\10.0.26100.0\um\x64`
* `C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.40.33807\lib\x64`
* `C:\Program Files (x86)\Windows Kits\10\Lib\10.0.26100.0\ucrt\x64`

Get the shortened paths:

    cmd /c for %A in ("C:\Program Files (x86)\Windows Kits\10\Lib\10.0.26100.0\um\x64") do @echo %~sA
    cmd /c for %A in ("C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.40.33807\lib\x64") do @echo %~sA
    cmd /c for %A in ("C:\Program Files (x86)\Windows Kits\10\Lib\10.0.26100.0\ucrt\x64") do @echo %~sA

Shortened paths are:

* `C:\PROGRA~2\WI3CF2~1\10\Lib\100261~1.0\um\x64`
* `C:\PROGRA~1\MICROS~4\2022\COMMUN~1\VC\Tools\MSVC\1440~1.338\lib\x64`
* `C:\PROGRA~2\WI3CF2~1\10\Lib\100261~1.0\ucrt\x64`

Edit the file "`makefile`", add the following variables, and modify the `RCFLAGS` variable:

    LIB1=C:\PROGRA~2\WI3CF2~1\10\Lib\100261~1.0\um\x64
    LIB2=C:\PROGRA~1\MICROS~4\2022\COMMUN~1\VC\Tools\MSVC\1440~1.338\lib\x64
    LIB3=C:\PROGRA~2\WI3CF2~1\10\Lib\100261~1.0\ucrt\x64

    LD="link"
    LDFLAGS=/nologo /debug /LIBPATH:"$(LIB1)" /LIBPATH:"$(LIB2)" /LIBPATH:"$(LIB3)"

## Start the compilation

* Run `nmake`

## Test and install

    nmake test
    nmake install

Once installed, the OpenSSL files can be found here: `C:/Program Files/OpenSSL`