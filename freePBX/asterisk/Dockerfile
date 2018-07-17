FROM centos:6
LABEL maintainer="thaild <thaiattt@gmail.com>"
RUN  yum -y install \
    yum groupinstall "Development Tools" -y \
    yum-plugin-priorities \
    pcre-devel \
    autoconf \
    openssl-devel \
    epel-release \
    make wget gcc vim git\
    centos-release-scl centos-release-scl-rh \
    apr-devel \
    apr-util-devel \
    pkgconf re2c \
    expat-devel \
    pcre pcre-devel 

# install freePBX
RUN yum -y update &&\
    yum -y groupinstall core base "Development Tools" &&\
    yum install -y gcc-c++ lynx bison mysql-devel mysql-server php php-mysql php-pear php-mbstring php-xml tftp-server httpd ncurses-devel \
    libtermcap-devel sendmail sendmail-cf caching-nameserver sox newt-devel libxml2-devel libtiff-devel audiofile-devel gtk2-devel \
    subversion kernel-devel subversion kernel-devel php-process crontabs cronie cronie-anacron wget php-xml uuid-devel libtool \
    sqlite-devel unixODBC mysql-connector-odbc libuuid-devel binutils-devel php-ldap

RUN chkconfig --level 345 mysqld on && service mysqld start
RUN chkconfig --level 345 httpd on && service httpd start

RUN pear channel-update pear.php.net && pear install db-1.7.14

RUN sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux 

# Install iksemel
RUN cd /usr/src &&\ 
    wget https://src.fedoraproject.org/lookaside/pkgs/iksemel/iksemel-1.4.tar.gz/532e77181694f87ad5eb59435d11c1ca/iksemel-1.4.tar.gz &&\
    tar xf iksemel-*.tar.gz &&\
    cd iksemel-* &&\
    ./configure  &&\
    make &&\
    make install 

# Install Asterisk-Extra-Sounds
RUN mkdir -p /var/lib/asterisk/sounds &&\
    cd /var/lib/asterisk/sounds &&\ 
    wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-core-sounds-en-wav-current.tar.gz &&\
    wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-extra-sounds-en-wav-current.tar.gz &&\
    tar xvf asterisk-core-sounds-en-wav-current.tar.gz  &&\
    rm -f asterisk-core-sounds-en-wav-current.tar.gz &&\
    tar xfz asterisk-extra-sounds-en-wav-current.tar.gz &&\
    rm -f asterisk-extra-sounds-en-wav-current.tar.gz &&\
    # Wideband Audio download
    wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-core-sounds-en-g722-current.tar.gz   &&\
    wget http://downloads.asterisk.org/pub/telephony/sounds/asterisk-extra-sounds-en-g722-current.tar.gz &&\
    tar xfz asterisk-core-sounds-en-g722-current.tar.gz &&\
    rm -f asterisk-core-sounds-en-g722-current.tar.gz &&\
    tar xfz asterisk-extra-sounds-en-g722-current.tar.gz &&\
    rm -f asterisk-extra-sounds-en-g722-current.tar.gz


# Add the Asterisk User
RUN adduser asterisk -m -c "Asterisk User"

#Install and Configure Asterisk
RUN cd /usr/src &&\ 
    wget http://downloads.asterisk.org/pub/telephony/dahdi-linux-complete/dahdi-linux-complete-current.tar.gz &&\ 
    wget http://downloads.asterisk.org/pub/telephony/libpri/libpri-current.tar.gz &&\ 
    wget http://soft-switch.org/downloads/spandsp/spandsp-0.0.6.tar.gz &&\ 
    wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz &&\ 
    wget -O jansson.tar.gz https://github.com/akheron/jansson/archive/v2.7.tar.gz &&\ 
    wget http://www.pjsip.org/release/2.4/pjproject-2.4.tar.bz2

USER root
# RUN yum install -y linux-headers-4.15.0-24-generic
# Compile and install DAHDI and LibPRI
# RUN cd /usr/src &&\ 
#     tar xvfz dahdi-linux-complete-current.tar.gz &&\ 
#     tar xvfz libpri-current.tar.gz &&\ 
#     rm -f dahdi-linux-complete-current.tar.gz libpri-current.tar.gz &&\ 
#     cd dahdi-linux-complete-* &&\ 
#     make all &&\ 
#     make install &&\ 
#     make config &&\ 
#     cd /usr/src/libpri-* &&\ 
#     make &&\ 
#     make install 

# Compile and install pjproject
RUN cd /usr/src &&\ 
    tar -xjvf pjproject-2.4.tar.bz2 &&\ 
    rm -f pjproject-2.4.tar.bz2 &&\  
    cd pjproject-2.4 \ 
    && CFLAGS='-DPJ_HAS_IPV6=1' ./configure --enable-shared --disable-sound --disable-resample --disable-video --disable-opencore-amr \
    && make dep \
    && make \ 
    && make install \
    && rm -r /usr/src/pjproject-2.4

#  Compile and Install jansson
RUN cd /usr/src  &&\ 
    tar vxfz jansson.tar.gz  &&\ 
    rm -f jansson.tar.gz  &&\ 
    cd jansson-*  &&\  
    autoreconf -i  &&\  
    ./configure --libdir=/usr/lib64  &&\ 
    make  &&\ 
    make install

# Compile and install SpanDSP
RUN cd /usr/src &&\
    tar -xzf spandsp-0.0.6.tar.gz &&\
    cd spandsp-0.0.6 &&\ 
    ./configure --libdir=/usr/lib64 &&\
    make &&\
    make install

# Compile and install Asterisk
RUN cd /usr/src &&\
    tar xvfz asterisk-13-current.tar.gz &&\
    rm -f asterisk-13-current.tar.gz &&\
    cd asterisk-* &&\
    contrib/scripts/install_prereq install &&\
    ./configure --libdir=/usr/lib64 &&\ 
    contrib/scripts/get_mp3_source.sh &&\
    make menuselect.makeopts &&\

    # RUN ./menuselect/menuselect --list-options    
    ./menuselect/menuselect --enable=chan_sip --enable=format_mp3 --disable=BUILD_NATIVE    &&\
    cat menuselect.makeopts &&\
    make &&\
    make install \
    && make config \
    && ldconfig 


RUN chown asterisk. /var/run/asterisk &&\
    chown -R asterisk. /etc/asterisk &&\
    chown -R asterisk. /var/{lib,log,spool}/asterisk &&\
    chown -R asterisk. /usr/lib64/asterisk &&\
    chown -R asterisk. /var/www/ 

# A few small modifications to Apache.
RUN sed -i 's/\(^upload_max_filesize = \).*/\120M/' /etc/php.ini &&\
    # sed -i 's/\(^memory_limit = \).*/\256M/' /etc/php.ini &&\
    sed -i 's/^\(User\|Group\).*/\1 asterisk/' /etc/httpd/conf/httpd.conf &&\
    sed -i 's/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf &&\
    service httpd restart

#Install and Configure FreePBX 
RUN cd /usr/src  &&\
    wget http://mirror.freepbx.org/modules/packages/freepbx/freepbx-13.0-latest.tgz  &&\
    tar xfz freepbx-13.0-latest.tgz  &&\
    rm -f freepbx-13.0-latest.tgz  &&\ 
    cd freepbx  &&\
    ./start_asterisk start  &&   service mysqld start  &&  service httpd start  &&\
    ./install -n

#SUPERVISOR 
RUN yum install python-setuptools -y && easy_install supervisor
# RUN /usr/bin/echo_supervisord_conf > /etc/supervisord.configure

#SETTING CONTAINER
ADD supervisor /etc/supervisor
RUN mkdir -p /var/log/supervisor/ && touch /var/log/supervisor/supervisord.log
RUN chown -R asterisk. /var/www/ var/log/asterisk/
ENTRYPOINT ["supervisord", "--nodaemon", "-c", "/etc/supervisor/supervisord.conf"]

# USER asterisk
# CMD [ "/usr/sbin/asterisk", "-f" ]

# START FREEPBX
# ADD freepbx.service /etc/systemd/system/freepbx.service
# RUN chkconfig --level 345 freepbx.service on 
# CMD [ "amportal", "restart" ]