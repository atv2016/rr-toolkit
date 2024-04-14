# Realtime Results Toolkit (rr-toolkit)

# Collection of various networking tools, as well as running active iperf2 (mcast) and iperf3 (unicast) services.
# Throughput can be viewed on the Grafana webserver, or in Glances for system performance. 

# Starts up in TMUX mode but can be exited to shell by using Ctrl-C or TMUX Ctrl-B x prefix bindings.
# 
# To run the container and save the logfiles in your logfiles dir in your local homedir (specifying endpoint target, run intervals for iperf3 and iperf2, and if you want sound to announce the run):
# sudo docker run -it --rm --privileged --name=rr-toolkit-client -p 3000:3000 -p 8086:8086 -p 61208:61208 -v $PWD/logfiles:/logfiles -h CLIENT -e TARGET='172.17.0.1' -e IPERF3_INT='15' -e IPERF2_INT='30' -e SOUND='1' rr-toolkit-client
# Time unit interval is in minutes!

# To build the image: sudo docker build . -t rr-toolkit-client

# To get a shell:
# docker exec -it rr-toolkit-client /bin/bash
#
# Raw output from netperf, iperf, iperf3 is logged on the /logfiles volume in the container and your local /logfiles directory setup as a BIND mount (this directory has to be created)

# Processed output can be viewed on your http://localhost:3000 using Grafana 
# InfluxDB is reachable on port 8806
#
# System statistics can be looked at on http://localhost:61208 using Glances

# New jobs can be added to the telegraf.conf file which is copied during the docker build. As long as the service is running on the toolkit-server, you can add it (requires a rebuild).
# The ip address it is targeting you specify in the docker run commandline with the -e option. If you don't want to hit any servers, just leave empty or specify 127.0.0.1. Just make
# sure you type the arguments as per the instructions (ENV comes before the image name).

FROM ubuntu:latest 

ENV TZ=Europe/London
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

MAINTAINER YOUR NAME <your.name@you.com>

ENV TERM=xterm-256color \
    DEBUG_COLORS=true \
    COLORTERM=truecolor \
    TTY=true \
    INFLUX_TOKEN=rJK0zVb2OxDkzhKwMRtIIFsZplmX5iMrnQEr1BLFICobHaCkogyeOId6zhEYo4NJ077E1qiZdmpPfJ517nRi8w== \
    IPERF2_INT_DEFAULT=30 \
    IPERF3_INT_DEFAULT=60 \
# Update if changing order of TMUX string
    WALL_TTY=3

COPY telegraf.conf /etc/telegraf/telegraf.conf 
COPY local.yml /etc/grafana/provisioning/dashboards/local.yml
COPY iperf3.json /var/lib/grafana/dashboards/iperf3.json
COPY datasources.yml /etc/grafana/provisioning/datasources/datasources.yml

# Install binaries and remove cache
RUN echo "wireshark-common wireshark-common/install-setuid boolean true" | debconf-set-selections \
    && echo PS1="'"'\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\ \033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '"'" >> .bashrc \
    && apt-get update \
# Get prereqs Grafana
    && apt-get install -y apt-transport-https \
    && apt-get install -y software-properties-common wget \
    && wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key \
    && echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | tee -a /etc/apt/sources.list.d/grafana.list \
# Get pre-reqs Influxdb2
    && wget -q https://repos.influxdata.com/influxdata-archive_compat.key \
    && echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null \
    && echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | tee /etc/apt/sources.list.d/influxdata.list \
# Get pre-reqs Telegraf 
    && wget -q https://repos.influxdata.com/influxdata-archive_compat.key \
    && echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null \
    && echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | tee /etc/apt/sources.list.d/influxdata.list \
# Get trex
    && wget --no-check-certificate --no-cache https://trex-tgn.cisco.com/trex/release/latest \
# Update and install a whole bunch of cool stuff
    && apt-get update \
    && apt-get install -y pciutils pv kmod beep nano vim zsh cpanminus ssh telegraf influxdb2 grafana-enterprise tmux bashtop atop btop nmon dstat saidar ftp tftp lolcat figlet toilet iputils-ping telnet ntopng tshark net-tools iperf iperf3 nmap tcpdump iproute2 arping htop glances \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir logfiles \
# Set init.d symlinks as we don't use systemd
    && update-rc.d grafana-server defaults \
    #&& update-rc.d influxdb2 defaults \
# Work around for telegraf variables
    && echo 'echo INFLUX_TOKEN=$INFLUX_TOKEN >> /etc/default/telegraf' >> /etc/profile.d/startup.sh \
    && echo 'echo TARGET=$TARGET >> /etc/default/telegraf'  >> /etc/profile.d/startup.sh \
# Populate startup.sh shell script
    && echo '/usr/bin/toilet WELCOME TO RR-TOOLKIT -f metal -t -f bigmono12 |/usr/games/lolcat' >> /etc/profile.d/startup.sh \
    && echo '/usr/bin/figlet Realtime Results Toolkit -w 200 -f small | /usr/games/lolcat' >> /etc/profile.d/startup.sh \
    && echo '/usr/bin/figlet Client Version 1.12 -w 200 -f small | /usr/games/lolcat' >> /etc/profile.d/startup.sh \
    && echo '/usr/bin/toilet RR-TOOLKIT: $TARGET -f metal -t -f pagga' >> /etc/profile.d/startup.sh \
# Load beep mod for announcing runs
    && echo 'if [[ $SOUND = 1 ]]  ; then echo "Sound enabled! If you do not hear sound use the --privileged flag or expose the input device"; beep -f 1198.64491 -l 200.00000 -n -f 1345.43319 -l 200.00000 -n -f 1007.93648 -l 100.00000 -n -f 1007.93648 -l 100.00000 -D 100.00000 -n -f 800.00000 -l 100.00000 -n -f 951.36544 -l 100.00000 -n -f 897.96948 -l 100.00000 -n -f 800.00000 -l 100.00000 -D 100.00000 -n -f 800.00000 -l 200.00000; else echo "Sound Disabled!";fi' >> /etc/profile.d/startup.sh \
# Prepare logfiles and setup glances system monitor
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
    && echo 'echo 'Waiting for iperf3 connection...' >> /logfiles/iperf3.log' >> /etc/profile.d/startup.sh \
    && echo 'echo 'Waiting for iperf2 connection...' >> /logfiles/iperf.log' >> /etc/profile.d/startup.sh \
# Startup trex
# Startup glances
    && echo '/usr/bin/glances -w -p 61208 &' >> /etc/profile.d/startup.sh \
# Create influxdb init script
    && echo 'cp /usr/lib/influxdb/scripts/init.sh /etc/init.d/influxd' >> /etc/profile.d/startup.sh \
    && echo 'chmod +x /etc/init.d/influxd' >> /etc/profile.d/startup.sh \
# Move InfluxDB to bind mount 
    && echo "sed -i 's|/var/lib|/logfiles|g' /etc/influxdb/config.toml" >> /etc/profile.d/startup.sh \
# Startup InfluxDB
    && echo 'service influxd start' >> /etc/profile.d/startup.sh \
# Kick of InfluxDB bucket and retention, can also be done on localhost:8806
    && echo 'influx setup \
  --org COMPANY \
  --bucket logs \
  --username admin \
  --password ExAmPl3PA55W0rD \
  --token $INFLUX_TOKEN \
  --force' >> /etc/profile.d/startup.sh \
# Setup pre-configured Grafana Dashboard
    && echo 'echo [dashboards] >> /etc/grafana/grafana.ini' >> /etc/profile.d/startup.sh \
    && echo 'echo min_refresh_interval = 5s >> /etc/grafana.ini' >> /etc/profile.d/startup.sh \
    && echo 'echo default_home_dashboard_path = /var/lib/grafana/dashboards/iperf3.json >> /etc/grafana/grafana.ini' >> /etc/profile.d/startup.sh \
# Add Grafana database to bindmount so the sqllite database is persistent 
    && echo "sed -i 's|DATA_DIR=/var/lib/grafana|DATA_DIR=/logfiles|g' /etc/default/grafana-server" >> /etc/profile.d/startup.sh \
# Startup Grafana
    && echo 'service grafana-server start' >> /etc/profile.d/startup.sh \
# Create Telegraf init script
    && echo 'cp /usr/lib/telegraf/scripts/init.sh /etc/init.d/telegraf' >> /etc/profile.d/startup.sh \
    && echo 'chmod +x /etc/init.d/telegraf' >> /etc/profile.d/startup.sh \
# Set Telegraf timers explicitly if not set by user
    && echo 'if [[ $IPERF2_INT ]]; then echo "IPERF2 interval set"; else echo "IPERF2 interval not set, using default timer of $IPERF2_INT_DEFAULT";IPERF2_INT=$IPERF2_INT_DEFAULT;fi' >> /etc/profile.d/startup.sh \
    && echo 'if [[ $IPERF3_INT ]]; then echo "IPERF3 interval set"; else echo "IPERF3 interval not set, using default timer of $IPERF3_INT_DEFAULT";IPERF3_INT=$IPERF3_INT_DEFAULT;fi' >> /etc/profile.d/startup.sh \
    && echo 'export IPERF2_T_INT="$IPERF2_INT"m; echo IPERF2_T_INT="$IPERF2_INT"m >> /etc/default/telegraf' >> /etc/profile.d/startup.sh \
    && echo 'export IPERF3_T_INT="$IPERF3_INT"m; echo IPERF3_T_INT="$IPERF3_INT"m >> /etc/default/telegraf' >> /etc/profile.d/startup.sh \
# Startup Telegraf Agent and API key
    && echo 'service telegraf start' >> /etc/profile.d/startup.sh \
# Install job announcer crontab
    && echo 'if [[ $SOUND = 1 ]] ; then' >> /etc/profile.d/startup.sh  \ 
    && echo 'echo 'Installing crontab announcer job'' >> /etc/profile.d/startup.sh \
    && echo 'crontab << EOF' >> /etc/profile.d/startup.sh \
    && echo '*/$IPERF3_INT * * * * for n in 1 2 3 ; do for f in 1 2 1 2 1 2 1 2 1 2 ; do /usr/bin/beep -f 2000 -l 20; done; done ; echo "\033[31;5;7mIperf3 client run starting!\033[0m" > /dev/pts/$WALL_TTY ; ps -ef | grep iperf3 | grep -v grep > /dev/pts/$WALL_TTY' >> /etc/profile.d/startup.sh \
    && echo '*/$IPERF2_INT * * * * for n in 1 2 3 ; do for f in 1 2 1 2 1 2 1 2 1 2 ; do /usr/bin/beep -f 2000 -l 20; done; done ; echo "\033[31;5;7mIperf2 client run starting!\033[0m" > /dev/pts/$WALL_TTY ; ps -ef | grep iperf | grep -v grep | grep -v iperf3 > /dev/pts/$WALL_TTY' >> /etc/profile.d/startup.sh \
    && echo 'EOF' >> /etc/profile.d/startup.sh \
    && echo 'else' >> /etc/profile.d/startup.sh \
    && echo 'crontab << EOF' >> /etc/profile.d/startup.sh \
    && echo '*/$IPERF3_INT * * * * echo "\033[31;5;7mIperf3 client run starting!\033[0m" > /dev/pts/$WALL_TTY && ps -ef | grep iperf3 | grep -v grep > /dev/pts/$WALL_TTY' >> /etc/profile.d/startup.sh \
    && echo '*/$IPERF2_INT * * * * echo "\033[31;5;7mIperf2 client run starting!\033[0m" > /dev/pts/$WALL_TTY && ps -ef | grep iperf | grep -v grep | grep -v iperf3 > /dev/pts/$WALL_TTY' >> /etc/profile.d/startup.sh \
    && echo 'EOF' >> /etc/profile.d/startup.sh \
    && echo 'fi' >> /etc/profile.d/startup.sh \
# Start crond
    && echo 'service cron start' >> /etc/profile.d/startup.sh \
# Fixing glances bug
    && wget https://github.com/nicolargo/glances/archive/refs/tags/v3.2.7.tar.gz; tar -xzf v3.2.7.tar.gz;cp -r glances-3.2.7/glances/outputs/static/public /usr/lib/python3/dist-packages/glances/outputs/static \
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
# Startup statistics view and startup system
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
    && echo 'echo 'System bootstrap finished.'' >> /etc/profile.d/startup.sh \
    && echo 'echo ''' >> /etc/profile.d/startup.sh \
    && echo 'sleep 3' >> /etc/profile.d/startup.sh \
    && echo 'tmux new-session -d glances \;  split-window btop --utf-force \; split-window bash \; select-layout tiled \; attach-session' >> /etc/profile.d/startup.sh \
# Give iperf3 output on stdout, we can add more panes in TMUX for iperf2 if we need it and have 2 bash terminals with output from both processes in addition to json to influx
    && echo 'tail -f /logfiles/iperf3.log > /dev/pts/$WALL_TTY' >> /etc/profile.d/startup.sh 

# Expose the default iperf3, iperf, webserver, glances, grafana and influxdb server port
EXPOSE 5201/tcp 5201/udp 5001/tcp 5001/udp 80/tcp 443/tcp 61208/tcp 3000/tcp 8086/tcp

ENTRYPOINT ["/bin/bash","-l","-c" ]

CMD ["/usr/bin/bash"]
