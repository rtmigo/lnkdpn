# [depz](https://github.com/rtmigo/depz)

[![Generic badge](https://img.shields.io/badge/ready_for_use-no-red.svg)](#)
[![Actions Status](https://github.com/rtmigo/depz/workflows/CI/badge.svg?branch=master)](https://github.com/rtmigo/depz/actions)
[![Generic badge](https://img.shields.io/badge/CI_OS-MacOS,_Ubuntu-blue.svg)](#)
[![Generic badge](https://img.shields.io/badge/CI_Python-3.7--3.9-blue.svg)](#)


Command line tool for symlinking directories with reusable source code into the project.

Language-agnostic. With Python and Flutter specific extensions.

# Why

**Reusing code** should be simple. If I have the needed code in a **directory on a local drive**, 
I just want to **include it** into the project. Without packaging it as a library 
for distribution or messing with IDE settings.

My first thought is to **create a symlink**:

```bash
$ ln -s /abc/libs/mylib /abc/myproject/mylib
```

Now `myproject` sees `mylib` as a local directory `myproject/mylib`. I can edit both `myproject` 
and `mylib` while working on `myproject`.

But here problems arise:
- **Portability**. How to make the symlinks easy to recreate on another system?
- **Recursive dependencies**. How to include not only `mylib`, but all the dependencies of `mylib`?

The answer is `depz`. It reduces these tasks to a one-line command.

# Install

```bash
$ pip3 install depz
```

Make sure that it is installed:

```bash
$ depz --help
```

# Use

- Specify dependencies in `depz.txt`
- Run the command `depz`

## Specify dependencies

File `xxx/depz.txt` lists dependencies for `xxx`:
- `/abc/myproject/depz.txt` for `myproject`
- `/abc/libs/mylib/depz.txt` for `mylib`

The `depz.txt` format:
```sh

# lines prefixed by hashes are ignored

# lines that specify local directory 
# names are LOCAL dependencies:
  
/absolute/path/to/mylib1
../libs/mylib2
~/path/mylib3

# lines that cannot be resolved to an existing 
# directory are considered EXTERNAL dependencies
 
requests
numpy
```
### Local dependencies are recursive

When a project depends on local `mylib`, it means, it also depends on all 
the dependencies of `mylib`. So after scannig `myproject/depz.txt` we will also 
scan `mylib/depz.txt` the same way.

### Paths are relative to parent of depz.txt

But resulting symlinks always go directly into the project dir.

The following examples show how the direcroies will be linked when running `depz` for `/abc/project`:

#### --mode=default

| File  | Line | Resolves to | Creates symlink |
|--------------------|------------|---------------|--------|
|/abc/project/depz.txt|/abc/libs/xxx|/abc/libs/xxx|/abc/project/xxx|
|/abc/project/depz.txt|../libs/xxx|/abc/libs/xxx|/abc/project/xxx|
|/abc/libs/xxx/depz.txt|../zzz|/abc/libs/zzz|/abc/project/zzz|

#### --mode=layout

Useful for frameworks with strict directory structure. 

| File  | Line | Resolves to | Creates symlink |
|--------------------|------------|---------------|--------|
| /abc/project/depz.txt | /abc/libs/xxx|/abc/libs/xxx/src<br/>/abc/libs/xxx/test|/abc/project/src/xxx<br/>/abc/project/test/xxx |
| /abc/project/depz.txt | ../libs/xxx|/abc/libs/xxx/src<br/>/abc/libs/xxx/test|/abc/project/src/xxx<br/>/abc/project/test/xxx |
| /abc/libs/xxx/depz.txt | ../zzz|/abc/libs/zzz/src<br/>/abc/libs/zzz/test|/abc/project/src/zzz<br/>/abc/project/test/zzz |

 
# Run

```bash
$ cd /abc/myproject
$ depz
```
or
```bash
$ depz --project /abc/myproject
```

Recursively scans `/abc/myproject/depz.txt` and prints all the found dependencies. Does not change anything in the filesystem. 

---------

```bash
$ cd /abc/myproject
$ depz --relink
```

Removes all the symlinks found in `/abc/myproject`. Adds new symlinks to the local dependencies. Prints external dependencies.
 
---------

```bash
$ depz --help
```

Shows additional command-line options.

# External dependencies

By default, a list of all external dependencies is simply printed to the terminal like that:

```txt
$ depz
--

Scanning /abc/myproject/depz.txt
...
External dependencies: pandas numpy requests
```

The `-e` argument causes the command to print only the list of dependencies.

#### In one line splitted by spaces:

```txt
$ depz -e line
--

pandas numpy requests
```

Sample usage:
```txt
$ pip3 install $(depz -e line)
```

#### One dependency per line:
```txt
$ depz -e multi
--

pandas
numpy
requests
```

Sample usage:
```txt
$ depz -e multi > requrements.txt
```