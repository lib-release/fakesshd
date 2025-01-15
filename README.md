run a fake ssh server to collect login attempts information.
you will see many brute force password attacks requests.

### quick startup
```bash
# startup fakesshd, server will listen at: 0.0.0.0:22
# so, you should edit '/etc/ssh/ssh_config',
# and set 'ListenAddress' to other address, or set 'Port' to another.
# !!!
# make sure you can login your sever use other methods,
# because address 0.0.0.0:22 will be used to fake ssh server by default
# !!!
./fakesshd

#login attempts records will save to directory './Logs/SSH2'.
```

### startup options
```bash
# set listen address and port
# you can listen your server public ip address, eg '8.8.8.8:22',
# and leave '127.0.0.0:22' to real ssh server
--listen "0.0.0.0:22"

# support use http to view login attempts records
# http listen port is same as '--listen'
--http

# change http listen address and port
--http "0.0.0.0:8080"

# set ssh server software
# default value is 'OpenSSH_8.4p1 Debian-5+deb11u3'
--software "OpenSSH_8.4p1 Debian-5+deb11u3"

# response success to client when they attempt login
# after they authenticate success,
# they may send channel open request to out fake server
--password-login-response-success
--publickey-login-response-success

# if response success to client,
# server will auto disconnect connection
# time uint is 'second'
--disconnect-timeout-after-login-success 5

# allow client sent channel open request
# and our fake server will do nothing
# just tell the client channel open success
--allow-channel-open

# server will auto close channel
# time uint is 'second'
--close-timeout-after-channel-open 1

```
### example
the command will start up fakesshd at 0.0.0.0:22, http server runs at 0.0.0.0:22 too.

you can see login username and password,channel open request, channel requestwhen client attempt to login our fake ssh server
```bash
./fakesshd --listen 0.0.0.0:22 \
    --http \
    --software "OpenSSH_8.4p1 Debian-5+deb11u3" \
    --password-login-response-success \
    --publickey-login-response-success
```

browser may does not allow you visit port 22 buy http protocol.

you can change '--http' to other address, eg. '--http 0.0.0.0:8080'

then you can use port 8080 to visit logs.

alse, you can add ningx location rule to handle http on port 22. 
```nginx
location ~ ^/fakesshd_logs
{
    proxy_http_version 1.1;
    proxy_pass http://xxxxxx:22; # your fake fake server address
    proxy_set_header Connection 'close';
    proxy_pass_header Server;
}
```

then you can visits '/fakesshd_logs' to view login attempts log
