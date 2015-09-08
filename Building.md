Tio uses CMake (http://www.cmake.org/), a build system that generates makefiles and IDE projects.

Requisites:
  * A C++ compiler (tested on Visual C++ and GCC). On Windows you can use the free compiler that comes with the Platform SDK or use (the also free) Visual Studio Express.
  * CMake (http://www.cmake.org/)
  * Boost Libraries version 1.47 (http://www.boost.org).
  * Python sources if you want the Python plugin support. If you don't want, just change it on CMakeLists.txt file (change `SET(TIO_PYTHON_PLUGIN_SUPPORT 1)` to `SET(TIO_PYTHON_PLUGIN_SUPPORT 0)`). On Linux you need the Python headers to build, just having Python installed is not enough.

# Windows #
  * Download Boost. I suggest you to use the pre-compiled version available from [Boost Consulting](http://www.boostpro.com/download)
  * If you're going to build with Python plugin support, you need to download Python sources from http://www.python.org and built it first
  * Fix the paths on the file CMakeLists.txt to reflect your environment
  * Open the command prompt, cd to tio\tio directory and run "cmake .". It will generate a Visual Studio project or a makefile
  * If you prefer, you can use QtCreator instead, it knows how to open CMake files.
  * Build!

## Linux ##
  * You need gcc, CMake and Boost dev files to build. Check your distribution docs on which packages you need to install.
  * "$ cmake . && make" should do the job
  * If you have problems building and you use a distribution package for Boost, it's probably because of a wrong Boost version. Try downloading the version 1.44 from sources and building it.

### Ubuntu recipe ###
```
$sudo apt-get install mercurial make cmake libboost-all-dev build-essential
$hg clone https://tio.googlecode.com/hg/ tio
$cd tio/tio
$cmake . && make
```

## MacOSX ##
I guess CMake will generate a correct XCode project, but never tried. Although I've heard about people building it successfuly on Mac.