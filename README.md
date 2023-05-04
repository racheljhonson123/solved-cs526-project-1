Download Link: https://assignmentchef.com/product/solved-cs526-project-1
<br>
Smashing the Stack for Fun and Credit<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a>

<h1>1       Overview</h1>

In this assignment you will learn about security vulnerabilities in software. In particular, you will find and exploit memory corruption vulnerabilities in order to better understand how simple programming errors can lead to whole system compromise. You will also examine some common mitigation techniques and analyze exploit code (i.e., shellcode). This assignment is due by no later than 12:30pm on Friday, September 20, 2019. See Section 5 for submission details.

The code and other answers you submit must be entirely your own work. You may consult with other students about the conceptualization of the project and the meaning of the questions, but you may not look at any part of someone else’s solution or collaborate with anyone else. You may consult published references, provided that you appropriately cite them (e.g., with program comments), as you would in an academic paper.

<h2>Objectives</h2>

<ul>

 <li>Be able to identify and avoid buffer overflow vulnerabilities in native code</li>

 <li>Understand the severity of buffer overflows and the necessity of standard defenses</li>

 <li>Gain familiarity with machine architecture and assembly language</li>

</ul>

<h2>Read this First</h2>

This project asks you to develop attacks and test them in a virtual machine you control. Attempting the same kinds of attacks against others’ systems without authorization is prohibited by law and university policies and may result in <em>fines, expulsion, and jail time</em>. You must not attack anyone else’s system without authorization! Per the course ethics policy, you are required to respect the privacy and property rights of others at all times, <em>or else you will fail the course.</em>

<h1>2       Getting started</h1>

Buffer-overflow exploitation depends on specific details of the target system, so we are providing a Kali Linux VM in which you should develop and test your attacks. This will also promote a consistent experience for everyone in the class and facilitate the grading process. We’ve slightly tweaked the configuration to disable security features that are commonly used in the wild but would complicate your work. We’ll use this precise configuration to grade your submissions, so do not use your own VM instead. You must download the targets into a folder owned by the VM, running targets in a shared folder will cause incorrect behavior.

To get started, first download and install a copy of <em>Oracle VirtualBox </em>for Windows, GNU/Linux, or Mac OS

X.<a href="#_ftn2" name="_ftnref2"><sup>[2]</sup></a> Next, download the GNU/Linux virtual machine image that we will use for this project and the Project 1 materials<a href="#_ftn3" name="_ftnref3"><sup>[3]</sup></a> from <a href="https://www.cs.purdue.edu/homes/clg/CS526/">https://www.cs.purdue.edu/homes/clg/CS526/</a><a href="https://www.cs.purdue.edu/homes/clg/CS526/">.</a> Project 1 materials are also available via Blackboard.

The login information for the two available users is:

Username: cs526

Password: cs526

Username: root Password: root

<ol>

 <li>Download VirtualBox from <a href="https://www.virtualbox.org/">https://www.virtualbox.org/</a> and install it on your computer. VirtualBox runs on Windows, Linux, and Mac OS.</li>

 <li>Get the VM file at <a href="https://www.cs.purdue.edu/homes/clg/CS526/projects/526-kali.ova">https://www.cs.purdue.edu/homes/clg/CS526/projects/526-k</a> <a href="https://www.cs.purdue.edu/homes/clg/CS526/projects/526-kali.ova">ova</a><a href="https://www.cs.purdue.edu/homes/clg/CS526/projects/526-kali.ova">.</a> This file is 2 GB, so we recommend downloading it from campus.</li>

 <li>Launch VirtualBox and select File B Import Appliance to add the VM.</li>

 <li>Start the VM. The username and password required to login are cs526 and cs526.</li>

 <li>Download <a href="https://www.cs.purdue.edu/homes/clg/CS526/projects/CS526.Fall2019.Project1.tgz">https://www.cs.purdue.edu/homes/clg/CS526/projects/CS526.Fall</a></li>

</ol>

<a href="https://www.cs.purdue.edu/homes/clg/CS526/projects/CS526.Fall2019.Project1.tgz">Project1.tgz</a> from inside the VM. This file contains the target programs you will exploit.

<ol start="6">

 <li>tar -xzvf CS526.Fall2019.Project1.tgz</li>

 <li>cd targets/</li>

 <li>Each of your targets will be slightly different. Personalize the targets by running:</li>

</ol>

./setcookie <em>username</em>

Make sure your username is your Purdue Career Username (and correct)!

<ol start="9">

 <li>sudo make (The password you’re prompted for is cs526. <em>You must run this command as sudo</em>)</li>

</ol>

<h1>3       Resources, Guidelines, and Helpful Information</h1>

No Attack Tools!            You may not use special-purpose tools meant for testing security or exploiting vulnerabilities. You must complete the project using only general purpose tools, such as gdb.

Control Hijacking Before you begin this project, review the lecture slides from software security lectures. Read “Smashing the Stack for Fun and Profit,” available at <a href="http://phrack.org/issues/49/14.html">http://phrack.org/issues/49/14. </a><a href="http://phrack.org/issues/49/14.html">html</a><a href="http://phrack.org/issues/49/14.html">.</a>

GDB You will make use of the GDB debugger for dynamic analysis within the VM. Useful commands that you may not know are “disassemble”, “info reg”, “x”, and “stepi”. See the GDB help for details, and don’t be afraid to experiment! This quick reference may also be useful: <a href="https://web.eecs.umich.edu/~sugih/pointers/Gdb-reference-card.pdf">https://web.</a>

<a href="https://web.eecs.umich.edu/~sugih/pointers/Gdb-reference-card.pdf">eecs.umich.edu/</a><a href="https://web.eecs.umich.edu/~sugih/pointers/Gdb-reference-card.pdf"><sub>˜</sub></a><a href="https://web.eecs.umich.edu/~sugih/pointers/Gdb-reference-card.pdf">sugih/pointers/Gdb-reference-card.pdf</a><a href="https://web.eecs.umich.edu/~sugih/pointers/Gdb-reference-card.pdf">.</a>

Figure 1: If you use ssh to connect to your virtual machine and want to enable core dump files, you must execute ulimit -c unlimited and for setuid executables execute echo 1 &gt; /proc/sys/fs/suiddumpable as root.

x86 Assembly There are many good references for Intel assembly language, but note that this project targets the 32-bit x86 ISA. The stack is organized differently in x86 and x86 64. If you are reading any online documentation, ensure that it is based on the x86 architecture, not x86 64.

<h2>3.1      Enabling core dump files</h2>

Core dump files can be useful for analyzing the cause of an execution crash. Once enabled, the operating system will create a file named core in your current working directory when a process crashes (e.g., due to a segmentation fault). Used in conjunction with gdb, the core file can give you a complete view of the process state when it crashed. To analyze a crashed process, simply launch gdb with the names of the executable file and its core file as arguments.

You can enable core dump files for processes that crash by using the command ulimit -c unlimited from the command prompt. However, to enable core dump for setuid executables, you should use an additional command (echo 1 &gt; /proc/sys/fs/suiddumpable). If you need assistance enabling core files, see Figure 1 or ask for help via the course discussion board.

<h2>3.2      Miscellaneous</h2>

You may find it helpful to write an additional program or script that will assist in figuring out the correct input to exploit any of the vulnerabilities. Note that this can be separate from the script/program you will write to exploit the vulnerabilities. If you’re unsure about what is acceptable for this assignment, please don’t hesitate to email us. Be sure to turn in the code for any additional program or script that you write.

<h1>4       Targets</h1>

The target programs for this project are simple, short C programs with (mostly) clear security vulnerabilities. We have provided source code and a Makefile that compiles all the targets. Your exploits must work against the targets as compiled and executed within the provided VM.

These targets are owned by the root user and have the suid bit set. Your goal is to cause them to launch a shell, which will therefore have root privileges. This and the following targets all take input as commandline arguments rather than from stdin. Unless otherwise noted, you should use the shellcode we have provided in shellcode.py<a href="#_ftn4" name="_ftnref4"><sup>[4]</sup></a> for you to use. Successfully placing this shellcode in memory and setting the instruction pointer to the beginning of the shellcode (e.g., by returning or jumping to it) will open a shell. A successful run is shown below.

For each program that we have provided, we ask that you explicitly:

<ol>

 <li>Briefly describe the behavior of the program.</li>

 <li>Identify and describe the vulnerability as well as its implications.</li>

 <li>Discuss how your program or script exploits the vulnerability and describe the structure of your attack.</li>

 <li>Provide your attack as a self-contained program written in Python.</li>

 <li>Suggest a fix for the vulnerability. How might you systematically eliminate vulnerabilities of thistype?</li>

</ol>

For vulnerable4.c, provide answers to the following questions in addition to those listed above.

<ol>

 <li>What is the value of the stack canary? How did you determine this value?</li>

 <li>Does the value change between executions? Does the value change after rebooting your virtual machine?</li>

 <li>How does the stack canary contribute to the security of vulnerable4?</li>

</ol>

<h2>vulnerable1: Redirecting control to shellcode</h2>

What to submit Create a Python program named sol1.py that prints a line to be used as the commandline argument to the target. Test your program with the command line:

./vulnerable1 $(python sol1.py)

If you are successful, you will see a root shell prompt (#). Running whoami will output “root”.

If your program segfaults, you can examine the state at the time of the crash using GDB with the core dump: gdb ./vulnerable1 core. The file core won’t be created if a file with the same name already exists. Also, since the target runs as root, you will need to run it using sudo ./vulnerable1 in order for the core dump to be created.

<h2>vulnerable2: Overwriting the return address indirectly</h2>

What to submit Create a Python program named sol2.py that prints a line to be used as the commandline argument to the target. Test your program with the command line:

./vulnerable2 $(python sol2.py)

<h2>vulnerable3: Beyond strings</h2>

This target takes as its command-line argument the name of a data file it will read. The file format is a 32-bit count followed by that many 32-bit integers. Create a data file that causes the provided shellcode to execute and opens a root shell.

What to submit Create a Python program named sol3.py that outputs the contents of a data file to be read by the target. Test your program with the command line:

python sol3.py &gt; tmp; ./vulnerable3 tmp

<h2>vulnerable4: Bypassing DEP</h2>

This program resembles target2, but it has been compiled with data execution prevention (DEP) enabled.

What to submit    This program is a little bit different. Create two text files for us:

<ul>

 <li>txt The input file you used in targeting vulnerable4</li>

 <li>txt A text file containing the user commands used for targeting vulnerable4 Test your files with the command line:</li>

</ul>

./vulnerable4 sol4input.txt

Warning: do not try to create a solution that depends on you manually setting environment variables. You cannot assume that the grader will run your solution with the same environment variables that you have set. Please make sure that your program exits gracefully when you close the spawned shell as well (it should not segfault).

Note that in this vulnerable program <em>you will not achieve a root shell with just a basic attack</em>. This is okay!

This is due to protections in newer versions of bash/dash that are specifically intended to drop setuid privileges when a shell is spawned through /bin/sh.

If your program spawns a shell in the default environment and a root shell in zsh, then you have successfully completed the exploit for this class.

If you would like to know more about this, please read on…

From the documentation for the system call <a href="#_ftn5" name="_ftnref5"><sup>[5]</sup></a>:

”system() will not, in fact, work properly from programs with set-user-ID or set-group-ID privileges on systems on which /bin/sh is bash version 2, since bash 2 drops privileges on startup.”

bash is explicitly programmed to check for setuid<a href="#_ftn6" name="_ftnref6"><sup>[6]</sup></a>:

if (runningsetuid &amp;&amp; privilegedmode == 0)

disableprivmode ();

Basically, if dash/bash detects that it is executed in a setuid process, it immediately changes the effective user ID to the process’s real user ID, essentially dropping the privilege. This was not always true in previous versions.

To see that this is actually just an artifact of the type of shell we are running, the VM also has zsh installed, which does not include these protections. We can switch the shell to zsh and run the exact same exploit with a very different result. See the example or the commands below for how to do this.

To switch to zsh:

sudo rm /bin/sh sudo ln -s /bin/zsh /bin/sh

Open a zsh shell by running: zsh

Once you have opened the new shell, try running your exploit again. This should give you a root shell.

If we want to reset back to the default shell:

sudo rm /bin/sh sudo ln -s /bin/dash /bin/sh

There are potential workarounds for this dropping of privileges in the default shell<a href="#_ftn7" name="_ftnref7"><sup>[7]</sup></a>… However, if your program spawns a shell in the default environment and a root shell in zsh, then you have successfully completed the exploit for this project.

Extra credit: For extra credit, create a Python program that will spawn a root shell using the default environment and shell. If you complete the extra credit, please let us know in the answers.pdf file and submit this solution instead.

<h2>vulnerable5: Variable stack position</h2>

When we constructed the previous targets, we ensured that the stack would be in the same position every time the vulnerable function was called, but this is often not the case in real targets. In fact, a defense called ASLR (address-space layout randomization) makes buffer overflows harder to exploit by changing the starting location of the stack and other memory areas on each execution. This target resembles vulnerable1, but the stack position is randomly offset by 0–255 bytes each time it runs. You need to construct an input that always opens a root shell despite this randomization.

What to submit Create a Python program named sol5.py that prints a line to be used as the commandline argument to the target. Test your program with the command line:

./vulnerable5 $(python sol5.py)

A word of caution If you see any output from the program before a root shell is opened, you have not done vulnerable5 of the project correctly and your solution will not be accepted by the grader.

<h2>vulnerable6: Return-oriented programming [Extra credit]                                      (<em>Difficulty: Hard</em>)</h2>

This target is identical to vulnerable1, but it is compiled with DEP enabled. Implement a ROP-based attack to bypass DEP and open a root shell. It may be helpful to use a tool such as ROPgadget (<a href="https://github.com/JonathanSalwan/ROPgadget">https: </a><a href="https://github.com/JonathanSalwan/ROPgadget">//github.com/JonathanSalwan/ROPgadget</a><a href="https://github.com/JonathanSalwan/ROPgadget">)</a>.

<ol>

 <li>Though there are a number of ways you could implement a return-oriented program, your ROP shoulduse the execve system-call to run the ”/bin/sh” binary. This is equivalent to: execve(”/bin/sh”, 0, 0);</li>

 <li>For an extra push in the right direction, int 0x80 is the assembly instruction for interrupting executionwith a syscall, and if the EAX register contains the number 11, it will be an execve. Now all you need to figure out is what values you need for EBX, ECX, and EDX, and set them using ROP gadgets!</li>

</ol>

What to submit Create a Python program named sol6.py that prints a line to be used as the commandline argument to the target. Test your program with the command line:

./vulnerable6 $(python sol6.py)

You may find the objdump utility helpful.

For this target, it’s acceptable if the program segfaults after the root shell is closed.

<h2>vulnerable7: Heap-based exploitation [Extra credit]                                                (<em>Difficulty: Hard</em>)</h2>

This program implements a doubly linked list on the heap. It takes three command-line arguments. Figure out a way to exploit it to open a root shell. You may need to modify the provided shellcode slightly.

What to submit Create a Python program named sol7.py that prints lines to be used for each of the command-line arguments to the target. Your program should take a single numeric argument that determines which of the three arguments it outputs. Test your program with the command line:

./vulnerable7 $(python sol7.py 1) $(python sol7.py 2) $(python sol7.py

3)


