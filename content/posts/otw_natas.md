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