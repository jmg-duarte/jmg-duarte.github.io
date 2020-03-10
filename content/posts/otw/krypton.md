---
title: "Over The Wire - Krypton Walkthrough"
date: 2019-07-22T18:52:37+01:00
draft: true
---

# Level 0

Level 0 blocks our entry on the server right on the site.

Giving us a [Base64](https://en.wikipedia.org/wiki/Base64) encoded string to decode before.

According to Wikipedia, Base64 is:

---

*Base64 is a group of binary-to-text encoding schemes that represent binary data in an ASCII string format by translating it into a radix-64 representation.*

---

There are a ton of tools online to decode Base64, if you're using Linux, 
you have one on your computer! 
The `base64` command! Let's run it with `--help`.

```
Usage: base64 [OPTION]... [FILE]
Base64 encode or decode FILE, or standard input, to standard output.

With no FILE, or when FILE is -, read standard input.

Mandatory arguments to long options are mandatory for short options too.
  -d, --decode          decode data
  -i, --ignore-garbage  when decoding, ignore non-alphabet characters
  -w, --wrap=COLS       wrap encoded lines after COLS character (default 76).
                          Use 0 to disable line wrapping

      --help     display this help and exit
      --version  output version information and exit

The data are encoded as described for the base64 alphabet in RFC 4648.
When decoding, the input may contain newlines in addition to the bytes of
the formal base64 alphabet.  Use --ignore-garbage to attempt to recover
from any other non-alphabet bytes in the encoded stream.

GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
Full documentation at: <https://www.gnu.org/software/coreutils/base64>
or available locally via: info '(coreutils) base64 invocation'
```

We can see it only takes files as input, 
but in the UNIX world [everything is a file](https://en.wikipedia.org/wiki/Everything_is_a_file)!
This means we can read standard output and input as files!

We will redirect the `stdout` of `echo` to the `stdin` of `base64` using a [pipe](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-4.html)!

```bash
echo "S1JZUFRPTklTR1JFQVQ=" | base64 -d
```

Running the command above yields:

```
KRYPTONISGREAT
```

Great success!

# Level 1

We're told that the file:

---

*It is 'encrypted' using a simple rotation called ROT13.*

---

Which means we have a [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher) on our hands!
A Caesar cipher is a substitution cipher, the main idea behind substitution cipher is the replacement
of the original characters by others, hence substitution.
You can read more about substitution ciphers [here](https://en.wikipedia.org/wiki/Substitution_cipher).

We also know the rotation, 13 characters as told by the name ([ROT13](https://en.wikipedia.org/wiki/ROT13))!

We could use an online tool but I wrote a tool especially for this purpose, [`rot`](https://github.com/jmg-duarte/rot), so I'll be using my own tool.

We just run it like so:

```
rot -d=13 "YRIRY GJB CNFFJBEQ EBGGRA"
```

Resulting in the following output:

```
LEVEL TWO PASSWORD ROTTEN
```

We got our password!

# Level 2

Once more we have a Caesar cipher as told by the `README`:

---

The password for level 3 is in the file krypton3.  It is in 5 letter
group ciphertext.  It is encrypted with a Caesar Cipher.

---

However, in this level we do not know the rotation value.

While breaking a Caesar cipher is fairly cipher using [frequency analysis](https://en.wikipedia.org/wiki/Frequency_analysis) or even [brute-force](https://en.wikipedia.org/wiki/Brute-force_attack),
if you do not know what kind of text you are expecting it may be harder.

For example, in our case we know that the text will probably be plain english,
however it could be gibberish what we were actually looking for and we would not notice.

Looking at the `/krypton/krypton2` folder we have:

```
drwxr-xr-x 2 root     root     4096 Jul 22 04:22 .
drwxr-xr-x 8 root     root     4096 Jul 22 04:22 ..
-rw-r----- 1 krypton2 krypton2 1815 Jul 22 04:22 README
-rwsr-x--- 1 krypton3 krypton2 8970 Jul 22 04:22 encrypt
-rw-r----- 1 krypton3 krypton3   27 Jul 22 04:22 keyfile.dat
-rw-r----- 1 krypton2 krypton2   13 Jul 22 04:22 krypton3
```

We can see the `encrypt` binary and the file it uses as key, `keyfile.dat`.

Using the `encrypt` binary we can encrypt plaintext we wrote in order to establish a mapping and discover the rotation value.

We can do it by running the following commands:

```bash
cd $(mktemp -d)                     # this will create a new temp directory and change our directory to there
ln -s /krypton/krypton2/keyfile.dat # creates a symbolic link in order for the binary to read the key
echo "ABCDEFGHIJKLMNOPQRSTUVWXYZ" > plaintext                # create our plaintext only containing A
/krypton/krypton2/encrypt plaintext # encrypt the plaintext
cat ciphertext                      # read the result
```

The last command yields `MNOPQRSTUVWXYZABCDEFGHIJKL` so we can map the cipher as bellow.

```
ABCDEFGHIJKLMNOPQRSTUVWXYZ
MNOPQRSTUVWXYZABCDEFGHIJKL
```

Resulting in 

```
OMQEMDUEQMEK
CAESARISEASY
```

# Level 3

Diving straight for the `README` we see:

```
Well done. You've moved past an easy substitution cipher.

Hopefully you just encrypted the alphabet a plaintext 
to fully expose the key in one swoop.

The main weakness of a simple substitution cipher is 
repeated use of a simple key.  In the previous exercise
you were able to introduce arbitrary plaintext to expose
the key.  In this example, the cipher mechanism is not 
available to you, the attacker.

However, you have been lucky.  You have intercepted more
than one message.  The password to the next level is found
in the file 'krypton4'.  You have also found 3 other files.
(found1, found2, found3)

You know the following important details:

- The message plaintexts are in English (*** very important)
- They were produced from the same key (*** even better!)


Enjoy.
```

Giving us the hints:

> The message plaintexts are in English 

> They were produced from the same key 

And further reading the `HINT` files yields:

> Some letters are more prevalent in English than others.

> "Frequency Analysis" is your friend.

Given that we have a substitution cipher in our hands, 
we know for a fact that it likely is weak when attacked with a [frequency table](https://en.wikipedia.org/wiki/Letter_frequency).

The `HINT` files give that away, plus we know the text is in English we know what 
table to use.

You can write a script to perform the analysis, however we can also use [this website](https://www.guballa.de/substitution-solver).

Using it we get the following:

```
found1 - qazwsxedcrfvtgbyhnujmikolp
found2 - qazwsxedcrfvtgbyhnujmikolp
found3 - qazwsxedcrfvtgbyhnujmikolp
```

So we can break our ciphertext with a simple script:

```python
cipher = 'KSVVW BGSJD SVSIS VXBMN YQUUK BNWCU ANMJS'
key = 'QAZWSXEDCRFVTGBYHNUJMIKOLP'
alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXY'
d = dict(zip(key, alphabet))
d[' '] = ' ' # just so the space doesn't throw an error

print("".join(list(map(lambda c: d[c],cipher))))
```

Resulting in:

```
WELLD ONETH ELEVE LFOUR PASSW ORDIS BRUTE
```

# Level 4

Reading the `README` before diving in is really important, especially because it tells us the cipher!

```
This level is a Vigenère Cipher.  You have intercepted two longer, english language messages.
```

There is no need to be fancy while cracking this one,
just go to [DCode's Vigenère Cipher](https://www.dcode.fr/vigenere-cipher) and use the hints to break it.

We have two long texts which we should use to get the key, and we also know the key length!

For the long texts we select the length option and use the resulting key to decode the cipher text.

Which gets us:

```
CLEAR TEXT
```

# Level 5