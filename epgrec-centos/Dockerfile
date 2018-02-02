FROM docker.io/centos:latest

RUN yum update -y
RUN yum -y install epel-release openssh-clients openssh-server;

RUN useradd -d /home/nginx -m -s /bin/bash nginx
RUN echo "export LANG=ja_JP.UTF-8" >> /home/nginx/.bash_profile
RUN yum -y install nginx

RUN yum -y install at php-cli php-fpm php-mbstring php-mysql php-process
RUN yum -y install gcc make gcc-c++
RUN yum -y install git
RUN yum -y install cmake
RUN yum -y install pcsc-lite-devel
RUN yum -y install automake

RUN yum -y reinstall glibc-common

ADD ./epgdump /tmp/epgdump

RUN usermod -s /bin/bash nginx

RUN localedef -v -c -i ja_JP -f UTF-8 ja_JP.UTF-8; echo "";
ENV LANG ja_JP.UTF-8

RUN sed -i 's/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm\/php-fpm.sock/' /etc/php-fpm.d/www.conf
RUN sed -i 's/user = apache/user = nginx/' /etc/php-fpm.d/www.conf
RUN sed -i 's/group = apache/group = nginx/' /etc/php-fpm.d/www.conf

RUN sed -i 's/;date.timezone =/date.timezone = Asia\/Tokyo/g' /etc/php.ini

RUN mkdir -p /var/www/html
RUN echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

RUN sed -i 's/PrivateTmp=true/PrivateTmp=false/' /lib/systemd/system/php-fpm.service
RUN systemctl enable nginx.service
RUN systemctl enable php-fpm.service

# epgdump
RUN cd /tmp/epgdump/ && make && make install

# ARIB library
RUN git clone https://github.com/stz2012/libarib25.git
RUN cd libarib25/ && cmake . && make && make install
RUN echo /usr/local/lib > /etc/ld.so.conf.d/usr-local-lib.conf
RUN ldconfig

# recpt1
RUN git clone https://github.com/stz2012/recpt1
RUN cd recpt1/recpt1/ && ./autogen.sh && ./configure --enable-b25 && make && make install

# prepare to build ffmpeg
#RUN apt-get -y install autoconf automake build-essential libass-dev libfreetype6-dev libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev wget && \
#apt-get -y install yasm libx264-dev libmp3lame-dev libopus-dev && \
#apt-get -y install libasound2 libass5 libvdpau1 libva-x11-1 libva-drm1 libxcb-shm0 libxcb-xfixes0 libxcb-shape0 libvorbisenc2 libtheora0 && \
RUN mkdir /tmp/ffmpeg_sources
RUN yum -y install wget
RUN yum -y install autoconf libtool
RUN yum -y install bzip2
RUN yum -y install yasm
RUN yum -y install nasm
RUN yum -y install libass-devel
RUN yum -y install libtheora-devel
RUN yum -y install libvorbis-devel

# build fdk-aac
RUN git clone --depth 1 git://github.com/mstorsjo/fdk-aac.git
RUN cd fdk-aac && autoreconf -fiv && ./configure && make && make install

# build lame
RUN git clone https://github.com/gypified/libmp3lame.git
RUN cd libmp3lame && ./configure && make && make install 

# build opus
RUN git clone git://github.com/xiph/opus.git
RUN cd opus && autoreconf -fiv && ./configure --disable-shared --with-pic && make -j && make install

# build x264
RUN git clone git://git.videolan.org/x264
RUN cd x264 && ./configure --enable-static --enable-pic --disable-asm && make -j && make install

#ffmpeg のビルド
RUN cd /tmp/ffmpeg_sources && wget http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2 && tar xjvf ffmpeg-snapshot.tar.bz2 
RUN cd /tmp/ffmpeg_sources/ffmpeg && \
PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ./configure \
  --prefix=/usr/local \
  --disable-shared \
  --pkg-config-flags=--static \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libx264 \
  --enable-nonfree
RUN cd /tmp/ffmpeg_sources/ffmpeg && make && make install 

RUN yum install -y pcsc-tools

# at
#RUN sed -i".org" 's/nginx//g' /etc/at.deny

# crontab
RUN yum -y install crontabs
ADD ./crontab /tmp/
RUN cat /tmp/crontab >> /etc/crontab

#RUN timedatectl set-timezone Asia/Tokyo
RUN /bin/cp -f /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

EXPOSE 80

CMD ["/usr/sbin/init"]

