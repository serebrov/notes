selenium webdriver - set php session cookie
============================================
To set the php session cookie we can use the addcookie (or python version add_cookie) method of the webdriver. But it accepts only name and value and does not allow to set additional cookie parameters like domain, path, etc.

Fortunately it is easy to do with javascript. Here is example of a JS code to set the cookie:

    document.cookie = "PHPSESSID=9ojofgkb21nujvhulvgq4drh06; domain=.myhost.com; path=/";

And here is python code version (assume you have set 'cookie' and 'domain' variables:

    webdriver.execute_script("document.cookie = 'PHPSESSID="+cookie+"; domain="+domain+"; path=/'")
