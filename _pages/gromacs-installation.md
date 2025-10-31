---
layout: single
title: "GROMACS"
permalink: /wiki/tutorials/gromacs-installation/
author_profile: true
parent: Tutorials
grand_parent: Wiki
---
# Installation Guide for GROMACS on IKKEM HPC (with Real Debugging Examples)

This guide walks you through installing GROMACS on the IKKEM HPC system, showing actual error messages and how to fix them. Screenshots and terminal outputs demonstrate what you may see during the process.

---

## Step 1: Download and Extract GROMACS

Download the source code and extract it:

```bash
wget https://ftp.gromacs.org/gromacs/gromacs-2025.2.tar.gz
tar xfz gromacs-2025.2.tar.gz
cd gromacs-2025.2
```

---

## Step 2: Create a Build Directory

```bash
mkdir build
cd build
```

---

## Step 3: Initial CMake Configuration

Try to configure with:

```bash
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
```

### ❗ Error Example 1: CMake Version Too Low

```
CMake Error at CMakeLists.txt:34 (cmake_minimum_required):
  CMake 3.28 or higher is required.  You are running version 3.20.2

-- Configuring incomplete, errors occurred!
```

**How to fix:**  
Check available CMake versions and load a newer one.

```bash
module avail cmake
module load cmake/3.31
```

---

## Step 4: CMake Error — GCC Version Too Low

Try configuring again:

```bash
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
```

### ❗ Error Example 2: GCC Version Too Low

```
CMake Error at cmake/gmxTestCompilerProblems.cmake:76 (message):
  GCC version 11 or later required.  Earlier versions may not have full C++17
  support.
Call Stack (most recent call first):
  CMakeLists.txt:104 (gmx_test_compiler_problems)
```

**How to fix:**  
Load a compatible GCC module, and set environment variables if needed:

```bash
module avail gcc
module load gcc/11
export CC=$(which gcc)
export CXX=$(which g++)
```

**If you made changes, start with a clean build directory:**

```bash
cd ..
rm -rf build
mkdir build
cd build
```

Then configure again:

```bash
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON
```

---

## Step 5: Successful Configuration

If you see:

```
-- Configuring done (27.6s)
-- Generating done (5.7s)
-- Build files have been written to: /public/home/bisheng/gromacs_tutorial/gmx_install/test/gromacs-2025.2/build
```

You are ready to compile.

---

## Step 6: Compile and Test

```bash
make -j32
make check
```

If all is well, you should see something like:

```
100% tests passed, 0 tests failed out of 94

Label Time Summary:
GTest             = 145.59 sec*proc (90 tests)
IntegrationTest   =  64.58 sec*proc (29 tests)
...
[100%] Built target check
```

---

## Step 7: Attempt Installation (and Permission Issue)

Try installing with:

```bash
sudo make install
```

### ❗ Error Example 3: Permission Denied / Admin Rights Needed

You will see:

```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:
  #1) Respect the privacy of others.
  #2) Think before you type.
  #3) With great power comes great responsibility.
```

**How to fix:**  
Install to your user directory instead:

```bash
cmake .. -DGMX_BUILD_OWN_FFTW=ON          -DREGRESSIONTEST_DOWNLOAD=ON          -DCMAKE_INSTALL_PREFIX=$HOME/gromacs-install
make -j32
make install
```

---

## Step 8: Source GROMACS and Verify

Activate your GROMACS environment:

```bash
source ~/gromacs-install/bin/GMXRC
```

Check if it is sourced correctly:

```bash
which gmx
gmx -h
```

If the help text appears, installation was successful!

---

## Summary of Common Errors and Solutions

| Error/Message Example                                                         | Solution                                                     |
|-------------------------------------------------------------------------------|--------------------------------------------------------------|
| `CMake 3.28 or higher is required. You are running version 3.20.2`            | `module load cmake/3.31`                                     |
| `GCC version 11 or later required. Earlier versions may not have full C++17`  | `module load gcc/11` + set `CC`/`CXX` variables              |
| `We trust you have received the usual lecture from the local System ...`      | Use `-DCMAKE_INSTALL_PREFIX=$HOME/gromacs-install`           |

---

**If you run into further problems, consult the [GROMACS documentation](https://manual.gromacs.org/) or your system administrator.**
