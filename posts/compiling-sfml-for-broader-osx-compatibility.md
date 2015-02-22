<!-- 
.. link: 
.. description: 
.. tags: SFML, games, C++, game programming, OSX
.. date: 2013/08/26 21:04:39
.. title: Compiling SFML for broader OS/X compatibility
.. slug: compiling-sfml-for-broader-osx-compatibility
-->

I recently competed in Ludum Dare 27, and one of the biggest struggles I had was making an application I built on the command line on my machine actually run for others. The first sign that I should have considered alternative frameworks was when I could only get SFML to link properly into my game when I built it from bleeding edge source. However, I'd been trucking along just fine using the compiled dylibs. I had been assuming the entire time that it was sufficient to include code and instructions for building the project yourself, but when the barrier to entry is too high, people will simply prefer all the Unity and HTML5 Canvas games. It's hard to blame them, really.

So, I set out to figure out how to package up SFML with my binary and ship it. After stumbling through a bunch of mediocre attempts to include the SFML dylibs in the zip file I submitted ot the competition, I finally realized the only thing left to do was construct a Mac app bundle.

Enter XCode.

<!-- TEASER_END -->

Getting my project set up was relatively easy, especially using the SFML templates that the SFML installer created for me. However, the first headache came when I tried to build the project against the default settings. After some Googling, I finally stumbled across a [SFML Tutorial](http://www.sfml-dev.org/tutorials/2.0/start-osx.php) about setting up a "Hello World" style Xcode project using SFML. I proceeded to import all the files I'd been working on into XCode, triggered a build and... linker errors. I tried going back to installing the packaged SFML, to no avail. Switched back to compiling frameworks, but had issues with them (though I now realize why that route failed, more below.) and finally went back to where I started, building dylibs from source.

After digging for a bit, I realized the errors I was getting were a lot better than I'd originally given them credit for. Namely, my issue is the default cmake settings for SFML only produce bleeding edge, current OS/X settings, and only for the active architecture. At this point, I should have realized toggling the architecture setting in XCode would have solved my framework issue, but I digress.

Toggling the "architecture" flag did it! To find it in XCode 4.3, click on the folder icon on the left side, then click on the very top element, which is your project. The center pane will update, with new tabs across the top. Click on "Build Settings" and play with "Valid Architectures" setting. Following this, I had success! Well, mostly. None of my game content was actually in the bundle. Now, I'm no XCode expert, so I'm sure this is the wrong way to do things, but adding said content was as simple as a Right-Click -> "Show Contents" on the bundle. From there, I traversed through Contents -> MacOS and dumped all the files there. Finally, it worked! I zipped up the game, uploaded it, and sent it some friends.

Immediate disaster! One of my friends is running OS/X 10.7, and was seeing errors about being unable to find SFML dylibs. Obviously, when you lose a weekend and large amounts of sleep to something you've built, you want others to at least see what you've done. Therefore, back to Google I went. I eventually stumbled on [this blog](http://www.mjbshaw.com/2013/02/building-sfml-2-with-c11-on-os-x.html) where he describes building SFML with different CMake settings. Coupled with a Stack Overflow post that pointed out the obvious; that your linked libraries need to be built with an SDK version <= your project SDK version. Derp.

The final pain was figuring out where the XCode SDKs had moved. Every link on Google said they should have been in /Developer, off the root. However, just like a million other annoying things, Lion moved it.

The new location is under /Application/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs

Anyway, here's the way to build SFML source:


    cmake -G 'Unix Makefiles' -DCMAKE_OSX_ARCHITECTURES='i386;x86_64'  -DSFML_INSTALL_XCODE4_TEMPLATES='ON' -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.7.sdk -DCMAKE_OSX_DEPLOYMENT_TARGET=10.7

Hope this helps someone!
