selenium - problem with loading x\_ignore\_nofocus.so
===================================================

Problem
-------

Selenium fails to start Firefox with following error:

    'The browser appears to have exited before we could connect.
    The output was: Failed to dlopen /usr/lib/libX11.so.6\ndlerror
    says: /usr/lib/libX11.so.6: wrong ELF class: ELFCLASS32\n'

In my case it was reproduced on the 64 bit machine with [Amazon Linux AMI](http://aws.amazon.com/amazon-linux-ami/).
The problem itself is known and there is [an issue in selenium tracker](http://code.google.com/p/selenium/issues/detail?id=2852).

It is because `x_ignore_nofocus` library tries to load 32bit version of the `libX11` instead of 64bit.
In my system there are following versions of `libX11`:

    # find / | grep libX11.so.6
    /usr/lib64/libX11.so.6       <-- symbolic link to `libX11.so.6.3.0'
    /usr/lib64/libX11.so.6.3.0 <-- ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, stripped
    /usr/lib/libX11.so.6          <-- symbolic link to `libX11.so.6.3.0'
    /usr/lib/libX11.so.6.3.0    <-- ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, stripped

At the right side (after "<--") is an output of `file /usr/libX/libX11.so.6xxx`.

Solution
--------

Change symbolic link `/usr/lib/libX11.so.6` to point to the 64-bit version `/usr/lib64/libX11.so.6.3.0`:

    # mv /usr/lib/libX11.so.6 /usr/lib/libX11.so.6.bak
    # ln -s /usr/lib64/libX11.so.6.3.0 /usr/lib/libX11.so.6

After that selenium started to work.

Links
-----------------
[bug reprot](http://code.google.com/p/selenium/issues/detail?id=2852).
