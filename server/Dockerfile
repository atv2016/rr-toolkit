# RR-Toolkit Realtime Results Toolkit - Server

# Collection of various networking tools, as well as running active iperf2 (mcast) and iperf3 (unicast) services.
# This can be used with a regular iperf client, or using the client Dockerfile that also houses Telegraf, Grafana and stores results in a Influx Database. 

# Starts up in statistics mode (btop) but can be exited to shell by using Ctrl-C or TMUX Ctrl-B x prefix bindings.
# 
# To run the container and save the logfiles in your logfiles dir in your local homedir:
# sudo docker run -it --rm --name=rr-toolkit-server -p 5201:5201/tcp -p 5201:5201/udp -p 5001:5001/tcp -p 5001:5001/udp -p 80:80 -p 443:443 -p 61209:61209 -v $PWD/logfiles:/logfiles -h SERVER rr-toolkit-server:latest

# To build the image: sudo docker build . -t rr-toolkit-server:latest

# To get a shell:
# docker exec -it rr-toolkit-server /bin/bash
#
# Raw output from netperf, iperf, iperf3 is logged on the /logfiles volume in the container and your local /logfiles directory setup as a BIND mount (this directory has to be created)

# Processed output can be viewed if you use this in conjunction with toolkit:client which starts up scheduled iperf client sessions and logs them to a database. See that README for more instructions.
#
# System statistics can be looked at on http://localhost:61208 using Glances

FROM ubuntu:latest 

ENV TZ=Europe/London
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

MAINTAINER YOUR NAME <your.name@you.com>

ENV TERM=xterm-256color \
    DEBUG_COLORS=true \
    COLORTERM=truecolor \
    TTY=true 

# Install binaries and remove cache
RUN echo "wireshark-common wireshark-common/install-setuid boolean true" | debconf-set-selections \
    && echo PS1="'"'\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\ \033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '"'" >> .bashrc \
    && apt-get update \
# Get trex
    && apt-get install -y apt-transport-https \
    && apt-get install -y software-properties-common wget \
    && wget --no-check-certificate --no-cache https://trex-tgn.cisco.com/trex/release/latest \
# Update and install a whole bunch of cool stuff
    && apt-get update \
    && apt-get install -y pciutils pv nano vim zsh cpanminus ssh tmux bashtop atop btop nmon dstat saidar ftp tftp lolcat figlet toilet iputils-ping telnet ntopng tshark net-tools iperf iperf3 nmap tcpdump iproute2 arping htop glances \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir logfiles \
# Populate startup.sh shell script
    && echo '/usr/bin/toilet WELCOME TO RR-TOOLKIT -f metal -t -f bigmono12 |/usr/games/lolcat' > /etc/profile.d/startup.sh \
    && echo '/usr/bin/figlet Realtime Results Toolkit -w 200 -f small | /usr/games/lolcat' >> /etc/profile.d/startup.sh \
    && echo '/usr/bin/figlet Server Version 1.12 -w 200 -f small | /usr/games/lolcat' >> /etc/profile.d/startup.sh \
    && echo '/usr/bin/toilet RR-TOOLKIT -f metal -t -f pagga' >> /etc/profile.d/startup.sh \
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
# Startup iperf3
    && echo '/usr/bin/iperf3 -sD -p 5201 --logfile /logfiles/iperf3.log' >> /etc/profile.d/startup.sh \
    && echo 'echo 'Waiting for iperf3 connection...' >> /logfiles/iperf3.log' >> /etc/profile.d/startup.sh \
# Startup iperf2
    && echo '/usr/bin/iperf -sD -u -B 224.1.1.1 -i 1 >> /logfiles/iperf.log' >> /etc/profile.d/startup.sh \
    && echo 'echo 'Waiting for iperf2 connection...' >> /logfiles/iperf.log' >> /etc/profile.d/startup.sh \
# Startup trex
# Startup glances
    && echo '/usr/bin/glances -w -p 61209 &' >> /etc/profile.d/startup.sh \
# Fixing glances bug
    && wget https://github.com/nicolargo/glances/archive/refs/tags/v3.2.7.tar.gz; tar -xzf v3.2.7.tar.gz;cp -r glances-3.2.7/glances/outputs/static/public /usr/lib/python3/dist-packages/glances/outputs/static \
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
# Setup DPD and Trex
    && echo 'mkdir -p /opt/trex;cd /opt/trex' >> /etc/profile.d/startup.sh \
    && echo 'mv /latest /opt/trex' >> /etc/profile.d/startup.sh  \
    && echo 'echo 'Installing DPD/trex...'' >> /etc/profile.d/startup.sh \
    && echo 'pv latest | tar -xz' >> /etc/profile.d/startup.sh \
    && echo 'cd v3.03;./dpdk_setup_ports.py -s' >> /etc/profile.d/startup.sh \
    && echo './dpdk_setup_ports.py -t' >> /etc/profile.d/startup.sh \
    && echo 'echo 'Run ./dpkdk_setup_ports -i on the commandline to specify EVEN interfaces'' >> /etc/profile.d/startup.sh \
# Setup TMUX
    && echo 'set-option -g status-utf8 on' >> .tmux.conf \ 
    && echo set -g status-right '#(rainbarf)' >> .tmux.conf \
    && echo 'unbind-key x' >> .tmux.conf \
    && echo 'bind x kill-window' >> .tmux.conf \
# Startup statistics view and start system
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
    && echo 'echo 'System bootstrap finished.'' >> /etc/profile.d/startup.sh \
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
    && echo 'sleep 3' >> /etc/profile.d/startup.sh \
    && echo 'btop --utf-force' >> /etc/profile.d/startup.sh
    #&& echo 'tmux new-session -d glances \;  split-window tcpdump \; split-window tail -f /logfiles/iperf3.log \; split-window tail -f /logfiles/iperf.log \; select-layout even-vertical \; attach-session' >> /etc/profile.d/startup.sh

# Expose the default iperf3, iperf, webserver, glances, grafana and influxdb server port
EXPOSE 5201/tcp 5201/udp 5001/tcp 5001/udp 80/tcp 443/tcp 61209/tcp

ENTRYPOINT ["/bin/bash","-l","-c" ]

CMD ["/usr/bin/bash"]
