Selenium - run tests on a virtual display
============================================

Set up
-------

Install xvfb

    sudo apt-get install xvfb

Install x11vnc

    sudo apt-get install x11vnc

Run tests on virtual display
----------------------------

Start xvbf (virtual display number 99)

    Xvfb -ac :99

Tell tests to run on virtual display

    export DISPLAY=:99

Or inside the test code (python)

    os.environ['DISPLAY'] = ':99'
    ...
    selenium = webdriver.Firefox(firefox_profile=ffp, firefox_binary=ffb)
    ...

Watch tests running on virtual display
--------------------------------------

Start x11vnc server on the same display:

    x11vnc -display :99

Use vnc client (for example, [gtkvncviewer](https://launchpad.net/gtkvncviewer))  to connect to the localhost and watch tests.


Links
-----------------
[Stackoverflow](http://serverfault.com/questions/273095/connect-to-xvfb-remote-to-fix-firefox-headless-crash)

[Xvfb + Firefox](http://www.semicomplete.com/blog/geekery/xvfb-firefox.html)

[Hudson Ci Server Running Selenium/Webdriver Cucumber In Headless Mode Xvfb](http://markgandolfo.com/?p=47)



