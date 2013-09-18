Title: Eclipse.ini and Ubuntu 12.04
Date: 2012-09-22
Tags: ubuntu, eclipse

I was recently running into memory issues using the Scala IDE eclipse plugin
 and knew I needed to tweak my JVM heap size, but for some reason couldn’t
 find the location that Ubuntu hid eclipse.ini to make this change.

After a quick search, it turns out Ubuntu puts the eclipse.ini file in
 /etc/eclipse.ini.

To increase your JVM heap in Ubuntu, look for the following lines in eclipse.ini:

    :::bash
    -vmargs
    -Xms40m
    -Xmx384m

If you’re running things like Tomcat, Hibernate, Scala, PDT, or PyDev then
 the range of 40MB to 384MB of heap space is hardly enough. Change these
 values to larges ones such as:

    :::bash
    -vmargs
    -Xms512m
    -Xmx1024m

This will increase your heap size minimum from the default 40MB to 512MB and a
 maximum of 1GB.
