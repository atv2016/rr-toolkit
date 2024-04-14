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
