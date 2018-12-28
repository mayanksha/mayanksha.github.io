---
layout: posts 
title: "Real UIDs vs Effective UIDs and Setuid Programs"
---


## Intro
In all \*nix systems, a UID or a "User ID Number" is an integer which maps your particular username (`whoami`) to this unique integer.
These mappings between the actual username and the UIDs is maintained in the `/etc/passwd` file. 
Every running process has at least two UIDs present in the process control block (`task_struct` in linux source code) which can be seen looked seen by this command

	cat /proc/${PID}/status | grep 'Uid'

On my system, this results into `1000	1000	1000	1000` which are respectively the uid, euid, suid and fsuid respectively for this process.

## What Real and Effective UIDs are? 
Simply stated, real UIDs identify the user who launched the process, while the effective UIDs determine what resources the process is eligible to access or has access to. 

A simple analogy can be drawn \- Within a company, the ID card of every person tells us about who the person is whereas the Secret Keys that they have authorizes them to various resouces in company. It's the ID cards vs secret keys thing. 

Doesn't clarifies...hang on a sec!

To make things more clear, we'll have to dive deeper into what set-uid programs are. 
## Set-UID programs

Whenever you change your password using the `passwd` command, the changes are made to `/etc/passwd` and `/etc/shadow` files.
The passwords aren't  stored directly in `/etc/passwd`; instead  they are stored in `/etc/shadow` in hashed form. 
Furthermore, any user can read the `/etc/passwd` file whereas only the root user is allowed to read the `/etc/shadow` file. 

The `passwd` command when run as a non-root user offers us to change the password of the current user only, but changing the password requires changes to be made to `/etc/shadow` and `/etc/passwd` files. 
How come the `passwd` command edit these files even when it doesn't have sufficient privileges? 

The answer lies in the fact that `passwd` is a Set-UID program meaning that it changes to some other user (here root) for a moment to make changes to these files. 

## Set-UID bit in Inodes
Consider a little program in C:

	#include<stdio.h>
	#include<unistd.h>
	void printIDs(){
		printf("ruid = %d, euid = %d\n", getuid(), geteuid());
	}

	int main(){
		printIDs();
		int ret_val = setuid(0);
		if (ret_val == -1){
			perror("Errored!");
			_exit(-1);
		}
		printIDs();
		return 0;
	}

>   Output (a.out): \
					 UID           GID  
	Real      1000  Real      1001  
	Effective 1000  Effective 1001  
	Errored!: Operation not permitted

Why doesn't setuid work? And how was it working in the case of `passwd`? The answer is the "Set-UID" bit present in Inode of the executable of this program. 

Set-UID bit runs the program with the effective UID set as the owner of this file instead of the one who is running this file, so that if `root` owns this executable and `msharma` runs this executable, then reuid will be that of `msharma` (1000) while euid will be that of `root` (0).

Suppose `a.out` was the object file for this program. `stat a.out` command tells us that `msharma` is the owner of this file since `msharma` has compiled this file. If we change the ownership of this file to root, and set the setuid bit and run our executable

	sudo chown root:root ./a.out
	sudo chmod +s ./a.out


>   Output (a.out) : \
	ruid = 1000, euid = 0 \
	ruid = 0, euid = 0

Here, the program started off with root privileges since euid was 0 initially, and when we did setuid(0), we changed the ruid too. Since, here we were `root` this changes doesn't makes much sense but if setuid bit was set by some other  non-root user, then the advantages would be apparent. 
Check this [SO Question](https://stackoverflow.com/questions/8499296/realuid-saved-uid-effective-uid-whats-going-on) which explains temporarily or permanently dropping root privileges (consider the case when you wish to `fork` a new process, and then `exec` in child with lowered privileges). 

>	Note: Make sure your partition is not mounted with `nosuid` flag. This disables the whole set-uid bit business  at the file-system level.

## Extra Readings

There's yet another kind of UIDs namely set-saved UIDs which can help to regain access once we had dropped privileges. You can check them out below.

[Tips for Writing Secure Setuid Programs](https://www.gnu.org/software/libc/manual/html_node/Tips-for-Setuid.html#Tips-for-Setuid) \\
[The Excellent - GNU libc manual](https://www.gnu.org/software/libc/manual/html_node/Setuid-Program-Example.html) \\
[SO Question - Using the setuid bit properly](https://unix.stackexchange.com/questions/166817/using-the-setuid-bit-properly) 

I shall create another post explaining in what ways we can potentially break set-uid programs and how much of a security risk they possess if not coded properly.




