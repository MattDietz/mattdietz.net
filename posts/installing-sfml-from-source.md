<!-- 
.. title: Installing SFML from Source on OSX
.. slug: installing-sfml-from-source
.. date: 2015-08-23 05:57:05 UTC
.. tags: gamedev, SFML, programming, tutorials
.. category: 
.. link: 
.. description: 
.. type: text
-->

It's fairly easy to install SFML from source on OSX. Following the instructions in [Compiling SFML with CMake](http://www.sfml-dev.org/tutorials/2.3/compile-with-cmake.php) you'll need CMake. The problem is CMake isn't included with OSX default. Fortunately, [Homebrew](http://brew.sh/) remedies that for us. Simply follow the instructions on the Homebrew site to get started.

Once that finishes installing, follow the current brew instructions for updating your formula list. Then, simply install CMake:

        brew install cmake

Notice I omitted sudo. That's because homebrew installs everything under /usr/local/, and you're going to have a bad time if random bits and pieces under that hierarchy are owned by root.

Next, snag SFML from source, cd into the cloned directory, and then kick off CMake

        git clone https://github.com/SFML/SFML.git
        cd SFML
        cmake CMakeLists.txt

Once CMake finishes running (it'll take a couple seconds) you'll have our old friend Makefile generated in the local directory. You probably know what to do at this point

        make
        sudo make install

At this point, you should have some fancy new headers and libraries under /usr/local. Congrats!

P.S. I'm a masochist and like to build SFML games from the command line, so I found out the hard way that /usr/local/include and /usr/local/lib aren't in your default search path. To tell clang you want it to look somewhere else, give it the -I (include path) and -L (library path) flags.

        CFLAGS=-L/usr/local/lib -I/usr/local/include ...
