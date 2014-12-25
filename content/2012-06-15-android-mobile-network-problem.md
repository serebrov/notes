Andriod - moblie network problem after BusyBox update
============================================
I have LG p500 (Optimus One), rooted with original ROM.

After update to [BusyBox](https://play.google.com/store/apps/details?id=stericson.busybox) 1.20.1 my moblie GPRS network became broken. I got network indicator (E with two arrows), but can not access network.

I suspected that this was someting with dns settings, but did not find any solution. I uninstalled busybox, but this didn't help. I did a factory reset - also didn't help. Actually I thought factory reset wil return my device to the original state, but then noticed that I still have root and busybox.

So I started examining system logs and found this error:

    Failed to replace default route com.android.server.NativeDaemonConnectorException
    06-12 17:04:40.999 E/NetworkManagmentService(1560)com.android.server.NetworkManagementService 994
    Failed to replace default route com.android.server.NativeDaemonConnectorException:
    Cmd {route replace def v4 rmnet0 31.144.159.49} failed with code 400 : {failed to replace default route for [rmnet0 -4] (I/O error)}: for iface [rmnet0]
    06-12 17:04:40.999 E/NetworkStateTracker(1560)android.net.NetworkStateTracker 249
    Unable to add default route.

I googled this error and found XDA thread which mention this error, page where it mentioned first - [XDA forum thread, see post 4174](http://forum.xda-developers.com/showthread.php?t=1314898&page=418).

Note that I use original LG ROM (not the rom discussed in that thread). Few pages later there is a more detailed examination - [see post 4272 and 4275](http://forum.xda-developers.com/showthread.php?t=1314898&page=428).

During discussion they suggested to try this command (I modified it a bit because original forum version gave error about wrong arguments. I issued this command in the [Script Manager](https://play.google.com/store/apps/details?id=os.tools.scriptmanager) via su):

    su -c "ip -4 route replace default via 109.46.130.66 dev rmnet0"

Where ip (109.46.130.66) is ip address from the com.android.server.NativeDaemonConnectorException error above - I think this is dynamic dns server I got from my mobile network provider (different on each connection).
And this command makes network work! But only for curren session. When I reconnect GPRS the network doesn't work again (because new dns server ip is issued every time).

The final solution is to take an "ip" binary from the original rom and replace a busybox's version:

* Get the [stock rom](http://forum.xda-developers.com/showthread.php?t=1260920)
* Extract system files from kdz format, [how to](http://forum.xda-developers.com/showthread.php?t=901417)
* After extracting "system.mbn": copy system/bin/ip to the device (somewhere, for example, sd card root).
* Rename /system/bin/ip to /system/bin/ip_busybox (just a backup). To access system folder on the device I used [ES File Explorer](https://play.google.com/store/apps/details?id=com.estrongs.android.pop) with root mode enabled.
* Copy original "ip" (from sdcard root) to /system/bin/

Finally I want to thank [BusyBox installer](https://play.google.com/store/apps/details?id=stericson.busybox) developer, Stephen. I contacted him by email when I realized that problem was with busybox installer and he answered and put effort to help me find the solution. Also he posted about this issue in the mentioned above XDA thread (I could not do this because have no permission to post into developer topics on XDA).
