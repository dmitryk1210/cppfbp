CppFBP
===

### C++ Implementation of Flow-Based Programming (FBP)

General web site on Flow-Based Programming: https://jpaulm.github.io/fbp/ .

General
---

In computer programming, flow-based programming (FBP) is a programming paradigm that defines applications as networks of "black box" processes, which exchange data across predefined connections by message passing, where the connections are specified externally to the processes. These black box processes can be reconnected endlessly to form different applications without having to be changed internally. FBP is thus naturally component-oriented.

FBP is a particular form of dataflow programming based on bounded buffers, information packets with defined lifetimes, named ports, and separate definition of connections.

One interesting aspect of this implementation is that it supports the scripting language `Lua`, so large parts of your networks can be written in a scripting language if desired.

This implementation is based on an older C implementation called [THREADN](https://github.com/jpaulm/threadn/blob/master/README.md), which used `longjmp` and `setjmp` to control process scheduling.  The scheduling parts of CppFBP now use [Boost](https://www.boost.org/) instead of `longjmp` and `setjmp`. Much of the rest of the overall THREADN architecture has been incorporated into CppFBP - in particular, THREADN allowed networks to be defined dynamically or statically - this has been preserved in CppFBP.  See the description of "static" vs. "dynamic" in https://github.com/jpaulm/threadn/blob/master/README.md .

Web sites for FBP: 
* http://www.jpaulmorrison.com/fbp/
* https://github.com/flowbased/flowbased.org/wiki

Sample Component, Component API and Network Definitions
* http://www.jpaulmorrison.com/fbp/CppFBP.shtml
 
Services supported by Lua interface, and some sample CppFBP/Lua scripts
* http://www.jpaulmorrison.com/fbp/thlua.html


Prerequisites
---

Download and install Microsoft Visual Studio (Community)

If installing this for the first time, you may have to do the following:

    Go to Tools -> Customize.
    In the popup window, select the Commands tab.
    Select the Menu bar: button, and in the dropdown menu select View
    Click Reset All and confirm.
    
Download and install `Boost` - http://www.boost.org/


    Go to where the extracted file was c:\boost_x_yy_zz.
    Click on booststrap.bat (don't bother to type on the command window just wait and don't close the window that is the place I had my problem that took me two weeks to solve. After a while the booststrap will run and produce the same file, but now with two different names: b2, and bjam.
    Click on b2 and wait for it to run.
    This takes a looong time.
    Click on bjam and wait it to run. Then a folder will be produce called stage. [no longer needed, apparently]
    Right click on the project.
    Click on property.
    Click on linker.
    Click on general.
    Click on include additional library directory.
    Select the part of the library e.g. c:\boost_1_57_0\stage\lib.
    
Thanks, Wu Jie!    


Download and install `Lua` - http://www.lua.org/

Update following macros with correct version numbers: `BOOST_INCLUDE`, `BOOST_LIB`, `LUA_INCLUDE`, `LUA_LIB`, as follows:
- Go to `View/Other Windows/Property Manager/SolutionSettings`
- Expand, showing `Debug`
- Expand, showing `UserMacros`
- Select `UserMacros`
- Go to `UserMacros` under `Common Properties`
- Click on that, revealing the 4 macros; change version numbers if necessary
- Leave the `set this macro as an environment variable in the build environment` option set to selected
 
or, more simply, in Windows Explorer, just go to `SolutionSettings`, open it, open `UserMacros.props`, and make, apply and save your changes.

Then go to `File/Save UserMacros` (not sure if this is necessary).

Build FBP Project
---

Create empty `cppfbp` directory in your local GitHub directory

Do `git clone https://github.com/jpaulm/cppfbp`

Now go into Visual C++, and `Open/Project/Solution` `CppFBP.sln` (in the just cloned directory)

There will be a "solution" line, followed by a number of "projects" - two of which are `CppFBPCore` and `CppFBPComponents`.
 
Do these builds in this order:
- Right click on `CppFBPCore` and do a `Build`
- Right click on `CppFBPComponents` and do a `Build`
- Right click on `TestSubnets` and do a `Build`

Right click on the "solution" line, and do `Build Solution`

If you get errors, you may have to build individual projects (sometimes this will require more than one try); if you only get warnings, you can proceed.

If you get a message saying "unknown compiler version", try reinstalling Boost.

"Dynamic" Networks
---

The projects whose names are of the form `xxxDyn` are "dynamic" versions of the projects named `xxx` - they start by scanning off the `.fbp` network definitions (via the `CppFBP` module with the `DYNAM` parameter set to `true`), and then run the resulting network structures.

Testing Thxgen
---

This program takes .fbp file and generates a .cpp network definition. If you want to try it, run Thxgen on your cloned repository - in debug mode...  To change the file being scanned, change Command Arguments in Debugging Properties...
 
Testing "TimingTest1" (console application)
---

This test case has 5 Generate/Drop process pairs, over each of which 1,000,000 IPs travel concurrently.  This means 5,000,000 each of `create IP`, `send`, `receive`, and `drop`.  

To run it, right click on `TimingTest1` in Solution Explorer; Debug/Start new instance

You should see something like

    Elapsed time in seconds: 180.000
    Press any key to continue . . .

The elapsed time will of course depend on your machine processing speed, number of cores, etc.

Installing Lua (under VS 2015)
---
- Download latest version of Lua from http://www.lua.org/download.html; install in `C:Program Files [(x86)]`
- If Lua version is not 5.3.2, 
  - Copy all files from `C:Program Files [(x86)]\Lua\5.3\src`, except `lua.c` and `luac.c`, to `CppFBPLua\src` folder
  - Go into VS 2015
  - Do `Add/Existing Item` with the files just added to `CppFBPLua\src` of type `.c` to `CppFBPLua/Source Files`
  - Do `Add/Existing Item` with the files just added to `CppFBPLua\src` of type `.h` to `CppFBPLua/Header Files`
  - Rebuild `CppFBPLua` project
- Run tests

TryLua
---

This runs a simple network, where every process is a Lua script being executed by the general `ThLua` component.

This test case only has 10,000 IPs travelling through the network, where half are generated by each Generate process.  The network in free-form notation is as follows:

    Gen(ThLua), Gen2(ThLua), 
    Concat(ThLua), Repl(ThLua), Drop(ThLua),

    Gen OUT -> IN[0] Concat, Gen2 OUT -> IN[1] Concat,
    Concat OUT -> IN Repl OUT -> IN Drop,

    '5000' -> COUNT Gen,
    '5000' -> COUNT Gen2,
    'gen.lua' -> PROG Gen, 
    'gen.lua' -> PROG Gen2,
    'concat.lua' -> PROG Concat,
    'repl.lua' -> PROG Repl,
    'drop.lua' -> PROG Drop;


To run it, right click on `TryLua` in Solution Explorer; Debug/Start new instance

You should see something like

    5000
    5000
    10000
    Elapsed time in seconds: 19.000
    Press any key to continue . . .
    
(where the 10000 is generated by `drop.lua` for debugging purposes)    

