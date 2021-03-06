---
title: "The Blech compiler - blechc"
linkTitle: "Blech compiler"
weight: 10
description: >
  The Blech compiler `blechc` compiles Blech programs to C programs.
  Learn how to build and install the Blech compiler from source.
---


## Build blechc from source

Clone the blech compiler project using

```
git clone https://github.com/blech-lang/blech
```

To build the project, you need **.NET Core** installed. Go to the [Microsoft .NET website](https://dotnet.microsoft.com/download) and follow their instructions to install the latest stable **.NET Core** available for your operating system.

Navigate to the folder where you checked out the Blech project. It should contain the file `Blech.sln`. Now you have choices:
  * For a simple **debug build** run
    ```
    dotnet build
    ```
    This creates `./src/blechc/bin/Debug/netcoreapp3.1/blechc.dll`.
    Use the dotnet command to start the compiler like so
    ```
    dotnet ./src/blechc/bin/Debug/netcoreapp3.1/blechc.dll
    ```
  * For a release build additionally use the `-c Release` option.

  * Finally, for a **self-contained release build**, which is operating system dependent, you need to run `dotnet publish` with a specific runtime identifier like so
    ```
    dotnet publish -c Release -r win-x64
    ```
    For Linux use `linux-x64`, for MacOS use `osx-x64`.

    This creates a folder `./src/blechc/bin/Release/netcoreapp3.1/win-x64/publish` which contains all files needed for execution. The folder as a whole can be moved arbitrarily.
    Inside the folder invoke the binary
    ```
    blechc
    ```
    to run the Blech compiler.

## Use `blechc`

Assuming you have a binary of the Blech Compiler `blechc` for your operating system, or you have built the Blech project yourself from sources, this sections tells you how to use it.

If the `blechc` binary is in your `$PATH`, you can invoke the compiler by simply writing
```
blechc
```
on the command line interface.
If you do not have a standalone (publish) build and want to use your local Debug (or Release) build, use the `dotnet` command to start the compiler from your blech working directory. 

```
dotnet <path-to>/blech/src/blechc/bin/Debug/netcoreapp3.1/blechc.dll
```


**From here on out we assume ```blechc``` to be a synonym for either call above**

The following shows some typical invocations of `blechc`.

Compile a Blech program:
```
blechc someBlechFile.blc
```

Translates ```someBlechFile.blc``` to C code silently. You will only see output on the command line if there are problems with the translation.
This will generate a file `blech.c` in the same folder which contains a runnable program. It runs in a main loop and calls the entry point activity of `someBlechFile.blc` 60 times. Furthermore in the subfolder `./blech` two files are generated `someBlechFile.c` and `someBlechFile.h`. They contain the C code and interface declarations respectively.

Compile with diagnostic messages:
```
blechc -v d someBlechFile.blc
```

Does the same as above but prints out textual representations of intermediate compilation results including: typed syntax tree, control flow graphs and block graphs for activities, C code.

Just check the programs correctness: 
```
blechc --dry-run someBlechFile.blc
```

Runs through all compilation phases but do not write any C files. This is useful to check for problems without actually touching anything on the file system.


Try all the options
```
blechc --help
```

## Compile generated C files

By default the compiler produces a main program in `blech.c` which can be used as a first test. To compile this code you need to include Blech specific C header files. These are located in `<path-to>/blech/src/blechc/include`. 
On Windows C compilation may look like this.
```
mingw32-cc.exe -I. -I<path-to>/blech/src/blechc/include blech.c
```
Note that the current folder `.` is explicitly added as a path to be included.
The resulting executable will run the program for 60 reactions and print the variable evaluations after every reaction in JSON format.

To include the generated C code in your own project inspect `blech.c` for details. In particular, make sure you call the `init` function on initial startup and then `tick` with every reaction.
For programmers familiar with Arduino these correspond to the `setup` and `loop` functions.
