# How to add an proxy to web-888
There arem ultiple ways to do it. You can even add nginx to have https on side. But I used a little bit simpler solution:

```
apk add socat

# add startup script for
(sleep 30 && socat TCP-LISTEN:80,fork TCP:127.0.0.1:8073) &
```

This will start proxy staff on port 80 and forward all incomming packets to port 8073. There is also posibility to reconfigure web-888 directly to that port,
but many application use as default 8073, so I want to keep it.

So if you have defined DNS record for your local IP address, then you can localy access  http://yourradio.sdr.com. However http://yourradio.sdr.com/admin will not work, since
your local IP is different from 127.0.0.1 where is a proxy. However http://localIP:8073/admin will work without restriction.
