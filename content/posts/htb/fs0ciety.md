---
title: "Hack the Box - fs0ciety"
date: 2019-09-04T21:15:03+01:00
draft: true
---

The other day I picked up Hack the Box again. 
While I don't do boxes, I do challenges given that they are more self contained and don't require much of a setup.

I took up `fs0ciety`, a plain reference to Mr. Robot, however I don't think it has much to do with it.

Our first challenge is to crack a ZIP file.
I don't know about you but I've *never* cracked a ZIP before, but I've done brute forcing.

Still, I didn't really knew where to start.
I found [`fcrackzip`](http://manpages.ubuntu.com/manpages/trusty/man1/fcrackzip.1.html) 
and a John Hammond [video](https://www.youtube.com/watch?v=KY8uM4j8EOY) on ZIP cracking.

In the video John uses the same tool to crack the ZIP and a list, 
the ["rockyou"](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) list!

While the list is nothing more than a brute-force dictionary, it means we don't have to make up the passwords. 
We just try the ones available, if that fails, we fall back to making them up.

And so we run:

```bash
$ fcrackzip -v -D -u -p ~/Downloads/rockyou.txt fsociety.zip
```

Further reading `man fcrackzip` we see the descriptions for the used flags.

```
-v, --verbose
              Each -v makes the program more verbose.

-D, --dictionary
              Select dictionary mode. In this mode, fcrackzip will read passwords from a file, which must contain one password per line and should be alphabetically sorted (e.g. using sort(1)).

-u, --use-unzip
              Try to decompress the first file by calling unzip with the guessed password. This weeds out false positives when not enough files have been given.

-p, --init-password string
              Set initial (starting) password for brute-force searching to string, or use the file with the name string to supply passwords for dictionary searching.
```

I think the descriptions are clear enough, if you do not understand something, Google is your friend (well not always, maybe try DuckDuckGo!).

The command from before outputs:

```
found file 'sshcreds_datacenter.txt', (size cp/uc    198/   729, flags 9, chk 8d9c)


PASSWORD FOUND!!!!: pw == justdoit
```

It worked!
Now that we have a password we process to the next step.

Reading the file with `cat`:

```
cat sshcreds_datacenter.txt
```

We get:

```                                                                                          
*****************************************************************************************
Encrypted SSH credentials to access Blume ctOS : 

MDExMDEwMDEgMDExMDAxMTAgMDEwMTExMTEgMDExMTEwMDEgMDAxMTAwMDAgMDExMTAxMDEgMDEwMTExMTEgMDExMDAwMTEgMDEwMDAwMDAgMDExMDExMTAgMDEwMTExMTEgMDAxMDAxMDAgMDExMDExMDEgMDAxMTAwMTEgMDExMDExMDAgMDExMDExMDAgMDEwMTExMTEgMDExMTAxMTEgMDExMDEwMDAgMDEwMDAwMDAgMDExMTAxMDAgMDEwMTExMTEgMDExMTAxMDAgMDExMDEwMDAgMDAxMTAwMTEgMDEwMTExMTEgMDExMTAwMTAgMDAxMTAwMDAgMDExMDAwMTEgMDExMDEwMTEgMDEwMTExMTEgMDExMDEwMDEgMDExMTAwMTEgMDEwMTExMTEgMDExMDAwMTEgMDAxMTAwMDAgMDAxMTAwMDAgMDExMDEwMTEgMDExMDEwMDEgMDExMDExMTAgMDExMDAxMTE=

*****************************************************************************************
```

Which at first sight is a bunch of gibberish, 
however that `=` gives us the hint that it might be encoded as [Base64](https://en.wikipedia.org/wiki/Base64).

For this I used [CyberChef](https://gchq.github.io/CyberChef), 
which is a really handy tool for this kind of problems.

We paste our text there and add the "From Base64" operation to our recipe and bake it.

We get the following output:

(I broke it down to 4 columns for easier visualization)

```
01101001 01100110 01011111 01111001 
00110000 01110101 01011111 01100011 
01000000 01101110 01011111 00100100 
01101101 00110011 01101100 01101100 
01011111 01110111 01101000 01000000 
01110100 01011111 01110100 01101000 
00110011 01011111 01110010 00110000 
01100011 01101011 01011111 01101001 
01110011 01011111 01100011 00110000 
00110000 01101011 01101001 01101110 
01100111
```

Adding the "From Binary" operation yields:

```
if_y0u_c@n_$m3ll_wh@t_th3_r0ck_is_c00king
```

Which is our flag!
