# modorganizer-umbrella
An umbrella- (super-) project for modorganizer.

## Purpose
This repository contains a meta build-system that is able to download and build MO subprojects and dependencies as necessary.
It can be used to build the whole project to produce a build that should be equivalent to the release or to build subprojects (i.e. plugins) with the minimum of dependencies.

This umbrella project can optionally produce ide projects for developers.

## Notes
* While mostly functional this project is work in progress and should only be used by those with the will to spend some time.
* Currently all dependencies are built from source, including monsters like Qt and python. This is necessary to produce releasable bundles (pre-built python would introduce additional run-time dependencies, pre-built Qt doesn't provide debug symbols (pdbs)) but it is overkill if all you want to do is contribute to a plugin.

## Concept
At its base this is actually a fairly simple program. Arbitrary command calls are wrapped inside Task objects (grouped as projects) and put into a dependency graph.
The tasks are then executed in proper order, with the umbrella providing the environment so you don't need to have all required tool in your global PATH variable.

There are specialised task implementations to conveniently specify sources to be retrieved from a repository or to get the correct make tool invoked.

Now one thing that may be a bit confusing is that all tasks have to be fully initialized before processing starts but since tasks will usually build upon each other, not all information may be available at that time.
In these cases functions/lambdas can be passed as parameters in task initialization which will then be invoked when that task is processed which will be after all dependencies are complete.

Some more details:
- Successfully completed tasks are memorized (in the "progress" directory) and will not be run again
- Names for tasks are generated so they may not be very user-friendly
- Technically, independent tasks could be executed in parallel but that is not (yet) implemented

## Open Problems

While conceptually this isn't particularly complicated, the actual build process for some tools are massively complex. Some issues I have not been able to work around yet:
- I can't seem to "make install" the qt webkit subproject properly. After building I have to manually move around files to impossible locations (#include "../../../../Source/and/so/on"-the f???)
- The windows variant of grep depends on dlls and the way the qt build system calls it they apparently can't be found even if they reside in the same directory as the exe.

## Dependencies
* python 2.7
  * decorator
* cmake
* visual C++ 2013 or newer
* 7zip - specifically, the command line version (7za)

## Usage
First, check config and see if all paths are set correctly.

```
usage: unimake.py [-h] [-f FILE] [-d DESTINATION] [target [target ...]]

positional arguments:
  target                make target

optional arguments:
  -h, --help            show this help message and exit
  -f FILE, --file FILE  sets the build script
  -d DESTINATION, --destination DESTINATION
                        output directory (base for download and build)
```

I'd suggest to use a destination folder that isn't too deep, some dependencies don't handle long paths well.
If the make target is left empty, everything is built.
