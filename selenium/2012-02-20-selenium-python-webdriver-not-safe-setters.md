selenium - python Firefox webdriver - unsafe setters in firefox_profile.py
============================================

Problem
-------

I tried to disable native events for Firefox webdriver in a following way:

    ffp = webdriver.firefox.firefox_profile.FirefoxProfile(path)
    if (Config.ff_native_events_enabled == False):
        ffp.native_events_enabled = False
    ffb = webdriver.firefox.firefox_binary.FirefoxBinary(firefox_path=Config.browser_binary)
    selenium = webdriver.Firefox(firefox_profile=ffp, firefox_binary=ffb)

After that Firefox starts, but python code can not connect to the webdriver extension.

Test fails with error like this:

    Traceback (most recent call last):
    ...
    File "/usr/local/lib/python2.6/dist-packages/selenium/webdriver/firefox/webdriver.py", line 45, in __init__
        self.binary, timeout),
    File "/usr/local/lib/python2.6/dist-packages/selenium/webdriver/firefox/extension_connection.py", line 46, in __init__
        self.binary.launch_browser(self.profile)
    File "/usr/local/lib/python2.6/dist-packages/selenium/webdriver/firefox/firefox_binary.py", line 44, in launch_browser
        self._wait_until_connectable()
    File "/usr/local/lib/python2.6/dist-packages/selenium/webdriver/firefox/firefox_binary.py", line 86, in _wait_until_connectable
        self.profile.path, self._get_firefox_output()))
    WebDriverException: Message: "Can't load the profile. Profile Dir: /tmp/tmp2aEvmI/webdriver-py-profilecopy Firefox output: "

Actually this error is not related to the profile loading and it fails in `_wait_until_connectable` method.

If I comment out the `ffp.native_events_enabled = False` string then everything works.

Solution
---------

After some examination I found that webdriver (python code) passes parameters to the fireforx extension through the user.js file in the Firefox profile folder.
And it looked like this:

    user_pref("security.warn_entering_weak", false);
    user_pref("security.fileuri.strict_origin_policy", false);
    user_pref("webdriver_enable_native_events", False);
    ...

Note that here we have lowercase 'false' for all options except the `webdriver_enable_native_events`.
The python code which is responsible for this:

    # my code
    ffp.native_events_enabled = False

    # webdriver code
    @native_events_enabled.setter
    def native_events_enabled(self, value):
        self.default_preferences['webdriver_enable_native_events'] = str(value)

Here python boolean value is converted into the "False" string.
The solution is to use in my code lowercase string instead of boolean:

    # my code
    ffp.native_events_enabled = "false"

I also added a [bug report](http://code.google.com/p/selenium/issues/detail?id=3400) and hope this will be fixed in selenium code.
The same problem exist for other boolean-like setter `accept_untrusted_certs`.

Links
-----------------
[bug report](http://code.google.com/p/selenium/issues/detail?id=3400)




