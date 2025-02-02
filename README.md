<h1 align="center">Setup Multiple Provers on Hyperbolic</h1>
<br>

1. Download dan Setup Prover for address1
```
sudo apt install -y curl
curl -L github.com/cysic-labs/phase2_libs/releases/download/v1.0.0/setup_prover.sh > ~/setup_prover.sh
bash ~/setup_prover.sh address1
```
2. Download params20, params24, and params25
```
mkdir -p ~/.scroll_prover/params
curl -L --retry 999 -C - circuit-release.s3.us-west-2.amazonaws.com/setup/params20 -o ~/.scroll_prover/params/params20
curl -L --retry 999 -C - circuit-release.s3.us-west-2.amazonaws.com/setup/params24 -o ~/.scroll_prover/params/params24
curl -L --retry 999 -C - circuit-release.s3.us-west-2.amazonaws.com/setup/params25 -o ~/.scroll_prover/params/params25
```
3. Run prover address1
```
cd cysic-prover
bash start.sh
```
4. If an 'address already' error appears, replace it with the key that has been backed up

5. Copy Folder for address2
```
cd /home/ubuntu
cp -r cysic-prover cysic-prover-2
```
6. Copy key file to address2 folder (/home/ubuntu/cysic-prover-2/~/.cysic/assets)
   
7. Change EVM address on config.yaml file for address2
```
sudo apt install -y nano
nano /home/ubuntu/cysic-prover-2/config.yaml
```
8. Saave file ( CTRL+X -> Y -> ENTER )
		
9. Install supervisor
```
sudo apt install -y supervisor
```
10. Create file supervisord.conf
```
nano supervisord.conf
```
11. Copy the following code
```
[unix_http_server]
file=/tmp/supervisor.sock   ; the path to the socket file

[supervisord]
logfile=/tmp/supervisord.log ; main log file; default $CWD/supervisord.log
logfile_maxbytes=50MB        ; max main logfile bytes b4 rotation; default 50MB
logfile_backups=10           ; # of main logfile backups; 0 means none, default 10
loglevel=info                ; log level; default info; others: debug,warn,trace
pidfile=/tmp/supervisord.pid ; supervisord pidfile; default supervisord.pid
nodaemon=false               ; start in foreground if true; default false
silent=false                 ; no logs to stdout if true; default false
minfds=1024                  ; min. avail startup file descriptors; default 1024
minprocs=200                 ; min. avail process descriptors;default 200
strip_ansi=true


[rpcinterface:supervisor]
supervisor.rpcinterface_factory=supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket

[program:cysic-prover]
command=/home/ubuntu/cysic-prover/prover
numprocs=1
directory=/home/ubuntu/cysic-prover
priority=999
autostart=true
redirect_stderr=true
stdout_logfile=/home/ubuntu/cysic-prover/cysic-prover.log
stdout_logfile_maxbytes=1GB
stdout_logfile_backups=1
environment=LD_LIBRARY_PATH="/home/ubuntu/cysic-prover",CHAIN_ID="534352"

[program:cysic-prover-2]
command=bash -c "sleep 3600 && /home/ubuntu/cysic-prover-2/prover"
numprocs=1
directory=/home/ubuntu/cysic-prover-2
priority=999
autostart=true
redirect_stderr=true
stdout_logfile=/home/ubuntu/cysic-prover-2/cysic-prover.log
stdout_logfile_maxbytes=1GB
stdout_logfile_backups=1
environment=LD_LIBRARY_PATH="/home/ubuntu/cysic-prover-2",CHAIN_ID="534352"
```
12. Save file ( CTRL+X -> Y -> ENTER )

13. Run supervisor
```
supervisord -c supervisord.conf
```
14. Log check
  - Prover1
```
supervisorctl tail -f cysic-prover
```
  - Prover2 (after 1 hour)
```
supervisorctl tail -f cysic-prover-2
```
