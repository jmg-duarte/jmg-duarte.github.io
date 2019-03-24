---
title: "Over The Wire - Natas Walkthrough"
date: 2019-03-24T15:59:46Z
draft: true
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