# Mozilla Firefox in a container, with pulseaudio passthrough support
#
# docker build -t firefox:latest .
#
# docker run --name firefox --hostname firefox.domain.net -d \
#           -v /var/run/dbus:/var/run/dbus \
#           -v /run/user/<uid>/pulse:/run/user/<uid>/pulse \
#           -v /tmp/.X11-unix:/tmp/.X11-unix \
#           -v /home/<username>/.pulse-cookie:/home/firefox/.pulse-cookie \
#           -e DISPLAY=:1
#           -v /home/<username>/Downloads:/home/firefox/Downloads:rw \
#           -v /home/<username>/.mozilla:/home/firefox/.mozilla:rw \
#           -v /dev/shm:/dev/shm \
#           firefox:latest
#
# Change UID/GID in the container to match the permissions of the exported
# downloads and home volumes, or update the local mount path to match uid a
# and gid to 1001
#

FROM archlinux:latest
MAINTAINER fmount<fmount@inventati.org>
LABEL description="Firefox in a Container"

# define user to be used as owner of the contained process
ENV user firefox
ENV group firefox
ENV uid 1001
ENV gid 1001

# passthrough services variables
ENV PULSE_PATH /run/user/$uid/pulse
ENV PULSE_SERVER unix:$PULSE_PATH/native
ENV HOME /home/$user
ENV PULSE_COOKIE $HOME/.pulse-cookie
ENV DBUS_PATH /var/run/dbus
ENV DISPLAY :0

# create user and home directory
RUN groupadd -g $gid $group; \
    useradd -u $uid -g $group -G audio $user; \
    mkdir -p /home/$user; \
    chown -R $user:$group /home/$user

# install firefox and additional packages
RUN pacman --noprogressbar --noconfirm -Syu; \
     pacman --noprogressbar --noconfirm -S util-linux sed firefox

RUN /bin/sh -c apt-get update && apt-get install -y util-linux sed xvfb x11vnc supervisor xterm xrdp novnc websockify firefox xpra flashplugin-installer && apt-get clean


RUN ln -s /usr/share/novnc/vnc_lite.html /usr/share/novnc/index.html

ADD supervisord.conf /etc/supervisord.conf
ADD xrdp.ini /etc/xrdp/xrdp.ini

RUN chmod +x /entry.sh

ENV DISPLAY :0
ENV RESOLUTION=1024x768

# generate random uuid for this container
RUN uuidgen | sed 's/\-//g' > /etc/machine-id

# deploy firefox autoconfiguration profile
COPY config/mozilla.cfg /usr/lib/firefox/
COPY config/autoconfig.js /usr/lib/firefox/defaults/pref/

# set volume
RUN mkdir -p /home/$user/Downloads && chown -Rv $user:$group /home/$user
RUN mkdir -p /home/$user/.mozilla && chown -Rv $user:$group /home/$user
VOLUME /home/$user/Downloads
VOLUME /home/$user/.mozilla

# set user
USER $user

EXPOSE 5901 6901

# entrypoint
ENTRYPOINT ["firefox"]
