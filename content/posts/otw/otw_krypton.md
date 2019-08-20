---
title: "Over The Wire - Krypton Walkthrough"
date: 2019-07-22T18:52:37+01:00
draft: true
---

# Level 0

The first level is easy, the password is a Base64 encoded string and in order to continue we need to decode it.

We can use an assortment of tools, from the Linux terminal to [CyberChef](https://gchq.github.io/CyberChef/).

We will use the terminal.

```bash
echo "S1JZUFRPTklTR1JFQVQ=" | base64 -d
```

Running the command above yields:

```
KRYPTONISGREAT
```

Easy enough.

# Level 1

This time the cipher we have is [ROT13](https://en.wikipedia.org/wiki/ROT13) which is a fancy name for a [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher) in which the rotation is known beforehand.

You can read more about substitution ciphers [here](https://en.wikipedia.org/wiki/Substitution_cipher).

While you can do the cipher fairly quick by hand, or using code, this time we will use [rot13.com](https://rot13.com/)

Pasting in the message bellow

```
YRIRY GJB CNFFJBEQ EBGGRA
```

Yields:

```
LEVEL TWO PASSWORD ROTTEN
```

# Level 2

In this level we do not know the rotation value.

While breaking a Caesar cipher is fairly cipher using frequency analysis or even brute-force,
if you do not know what kind of text you are expecting it may be harder.

For example, in our case we know that the text will probably be plain english,
however it could be gibberish what we were actually looking for and we would not notice.

Looking at the `/krypton/krypton2` folder we have:

```
krypton2@krypton:/tmp/tmp.XBEPsD3nDP$ ls -la /krypton/krypton2
total 32
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

```
Well done.  You've moved past an easy substitution cipher.

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

Reading the `README` gives us two direct hints:

> The message plaintexts are in English (*** very important)

> They were produced from the same key (*** even better!)

And further reading the `HINT` files yields:

> Some letters are more prevalent in English than others.

> "Frequency Analysis" is your friend.

Given that we have a substitution cipher in our hands, 
we know for a fact that it likely is weak when attacked with a frequency table.

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