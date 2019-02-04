---
layout: post
title: "Compile goaccess from source on CentOS 7"
date: 2016-07-07 22:00:32 -0400
comments: true
tags: [linux, web-apps]
---
[Goaccess](https://goaccess.io) is a neat little utility which scans through your web server (I use nginx) logs and generates a nice HTML report of 
your site's access statistics. Here is a [sample from their site](http://rt.goaccess.io). I run CentOS 7 on an EC2 Instance where I run a pet project. I wanted a light web analyzer that just works. I have
[epel](https://fedoraproject.org/wiki/EPEL) enabled on my instance so I just installed goaccess by `sudo yum install goaccess`. It worked and I was able
to see some stats. But then I figured out that the version from repositories is quite an old one at 0.9.8. The latest one is 1.0.2 from the website.
So, I just uninstalled it `sudo yum remove goaccess`.

I downloaded the source tarball and after a few hiccups I was able to get it running. There are two quirks you need to watch out for. First one is
that, enabling geoip in goaccess requires installation of maxmind's geoip database. The installed database from yum is very small, so you need to
update it and change symlink /usr/share/GeoIP/GeoIP.dat to point to /usr/share/GeoIP/GeoLiteCountry.dat. Second quirk is that by default the configure
script doesn't figure out the geoip devel libs path so you need to manually point to it while configuring.

```sh
sudo yum install GeoIP
geoipupdate
sudo rm /usr/share/GeoIP.dat
sudo ln -s /usr/share/{GeoLiteCountry,GeoIP}.dat 

wget http://tar.goaccess.io/goaccess-1.0.2.tar.gz
tar -xzvf goaccess-1.0.2.tar.gz
cd goaccess-1.0.2
LD_FLAGS='-L/usr/lib64/' ./configure --enable-utf8 --enable-geoip
make
sudo make install
```

I also created a daily crontab entry to generate reports daily and put it in my `public` folder of rails directory, all the files in which I serve
statically from nginx. So I can just type http://mydomain.com/access.html and see the reports. Here is my crontab entry.

```sh crontab
@daily /bin/zcat -f /var/log/nginx/access.log* | /usr/local/bin/goaccess -a -o /home/ec2-user/my_rails_dir/public/access.html
```
