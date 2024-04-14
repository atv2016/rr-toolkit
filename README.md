### RR-toolkit Realtime Results Toolkit

Rr-toolkit is a docker container that uses Telegraf to execute certain configured commands, which are then logged into Influx, and can then further be configured or displayed in Grafana.
It exposes various ports to your docker host, one of which is the Grafana port where you can see the results of what you have scheduled in Telegraf in realtime in Grafana. This is very
useful for seeing results right away (think bandwidth testing, snmpgets, logs, security alerts, check if the coffee machine is still working, or whatever else you can think of) and put actions
in Grafana based on thresholds or dashboards you set up.

Everytime a command is executed in Telegraf, an audible beep will alert you (if you set that parameter) that a new test run is about to start. The database as well as the Grafana dashboard
is pre-provisioned, but you can ofcourse setup better ones and setup threshold markers, email alerts, etc. You can also work with the Influx Dashboard should you want to.

As the files and databases are stored on a bind mount, when the docker container is killed and started back up again, it will use those same files to start where it left off. This also allows
anyone else to use these files, so the bind mount could be a network drive and anyone could import these results. This way you can store the results of past Telegraf runs and compare them from
a year ago. Ofcourse, they could just browse to the docker host ip address and look at the Grafana dashboard you are running for a realtime feed while you have it running.

You can run this on anything that supports docker, and by nature of a docker container it will be self-contained and immutable, requiring no support or changes from anyone until you want to
make changes to it, in which case you will have to rebuild it. 

The docker server is only needed for testing, if you have a target service running you have no need for that and you only need to run the client container.

### How to use

1. Edit the telegraf.conf in the client directory so it uses the commands you want it to. The current .conf file is setup for using iperf but you can run any command you want to. It helps if it
   outputs to CSV or JSON format so Influx can handle it more easily. Remember to rebuild your docker image you want to include those changes.

```
[[inputs.exec]]
# We cannot use the --logfile flag if we want to have json output put in the InfluxDB
commands=["/usr/bin/iperf3 -c $TARGET -J"]
#interval = "5m"
interval = "$IPERF3_T_INT"
timeout = "240s"
data_format = "json"
json_query = "end"
name_override = "iperf3 server"
```

2. Run the following:
```
sudo docker run -it --rm --privileged --name=rr-toolkit-client -p 3000:3000 -p 8086:8086 -p 61208:61208 -v $PWD/logfiles:/logfiles -h CLIENT -e TARGET='172.17.0.1' -e IPERF3_INT='15' -e IPERF2_INT='30' -e SOUND='1' rr-toolkit-client
```
Where 3000 is Grafana, 8086 is Influx and 61208 is a Glances service. As the current telegraf file is geared towards running iperf, we also specify some variables that the docker build image can use. And for kicks we enable some sound if available. You can use Grafana, but you are free to use any of the backends Telegraf supports and send it straight to your favorite logging platform.

### Troubleshooting

I am not available for troubleshooting. Use at your own risk. You most likely will have to edit it to your situation, but all the core components should be there and you should be able to get it
to work how you want it to fairly quickly. I have used this for various projects in the past to ingest data quickly from a variety of sources, continous monitoring for anomalies in the data etc.
