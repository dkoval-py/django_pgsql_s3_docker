## What is this?
Bash script for measuring latency between Kubernetes (in our case GKE) and CloudSQL on the Transport OSI level by opening the TCP connection using socket and measure th    e execution command time.

## Components:
1. ``src/check.sh`` - Bash script for measuring and logginig measures to stdout and log file.
2. ``src/latency-check`` - Logrotate configuration file (here's defined the parameters for rotating logs).
3. ``src/latency`` - Cronjob file, in which is defined the frequency of running log rotation.

## Run from source:
If you'd like to run script manually, the first thing should be done is defining environment variables: ``HOST`` and ``PORT``.<br />
``HOST`` - is IP address, latency to which from your VM you'd like to measure.<br />
``PORT`` - is a port on remote VM to which script will open TCP connection using socket. Port should be openned.<br />
Manually define environment variables:
```
  export HOST=127.0.0.1
  export PORT=80
```

As soon as ENV variables are defined, you can run the script:
```
  /bin/bash src/check.sh
```
Your console will be blocked by executing script, and you'll be able to observe the measuring results on your terminal session.<br />
Logs file you can find by the location: ``/ping-logs/latency.log``

## Run inside docker container:
The first thing should be done is changing ENV variables values in the Dockerfile by your values in file ``src/Dockerfile``. <br />
Change the following strings:
```
  ENV HOST=127.0.0.1
  ENV PORT=22
```
Build docker image:
```
  docker build --network host -t latency-monitoring src/.
```
Run docker image:
 ```
  docker run -d --restart unless-stopped latency-monitoring
```
Check script running:
```
  docker logs ${container-id}
     or
  docker exec -it ${container-id} /bin/bash
  tail -f /ping-logs/latency.log
```

## Run inside K8S:
Install application using helm chart. <br />
By default helm chart without any changes will be built for DEV environment.
```
  helm install dev .
```
If it's necessary to install helm chart for another environment (STAGE or PROD), you should redefine some variables. <br />
Run for stage:
```
  helm install --set Environment.dev.enabled=false --set Environment.stage.enabled=true \
  --set container.envStage.HOST=${{YOUR-HOST-IP}} --set container.envStage.PORT=${{YOUR-PORT}} stage .
```
Run for prod:
```
  helm install --set Environment.dev.enabled=false --set Environment.prod.enabled=true \
  --set container.envProd.HOST=${{YOUR-HOST-IP}} --set container.envProd.PORT=${{YOUR-PORT}} prod .
```
