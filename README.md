
# elasticall

Shell tool to monitor Elasticsearch running advanced queries.

**Motivation**. I want to be able to run take a random sample from Elasticsearch, I want to do it from the shell and I want the thing to be as practical as possible. I would like the computer to manage a question like this: "Jarvis, show me a random sample log lines made in the last 2 days". 

The script `elasticall` was made to throw very complex queries to Elasticsearch and beutify the output results. As such, it is a tool that frees the user from the tedious interactions with JSON. Taking random sample is just anohter, not trivial, Elasticsearch query.

## Installation & run

***CAVEAT.*** The procedure is tested working for **LCLS-logs** Elasticsearch index, for the program to be installed in *psmetric01*. Other users in other networks need to properly adapt the Elasticsearch address and index in the *elasticall* file.

- Get the source from GitHub
- Run the installation script: `$> sudo install`
- Test run, shows the builtin manual: 
```
$> elasticall -h
```
```
=> MANUAL TEXT ...
```

- Easy task, show the number all log lines stored into LCLS-logs index.

```
$> elasticall --s0='*' /usr/local/etc/elasticall/paramSearchAndCount
```

```
=> querys: '*', num.doc: 557,668,598
```

## Heads up ! 

**CAVEAT for LCLS users.** Before you do any test you must be aware of this important fact. `elasticall` runs in *psmetric01* but the Elasticsearch server runs in *psmetric04*. If you run a search that takes time and kill `elasticall` by e.g. `Ctrl+C,` or `kill`, the search will keep on running on *psmetric04*! 

If you intend to run multiple queries via `elasticall` and kill them librerally then keep an eye on *psmetric04* load with `top`. A CPU load of about *500%* in `top` is expected when you run long and complex queries.


## Random Sample to measure performace

The first reason I for me to want a random sample over Elasticsearch  was to measure its Elasticsearch performance. Indeed, a random sample, if well performed, implies that all the sample space must be visited at each run. It makes no sense to cache results. 


***CASE.1*** Random sample without parameters, by default runs on all of Elasticsearch data and returns 20 lines. 

```
$> time elasticall queries/paramRandomSample2
time elasticall /usr/local/etc/elasticall/paramRandomSample2 
ATTENTION: this query can take some time ... 
total hits: 557,365,125
Mar  3 04:42:38 psnfs02 kernel: [4453638.338888] sd 1:0:45:0: [sdaj] Bad block number requested
Feb 27 23:48:26 psnfs02 kernel: [4176799.486066] sd 1:0:45:0: [sdaj] Bad block number requested
==== CUTTED LINES ====
Mar  9 02:49:47 psana1216 monit[1956]: Processing queued event /var/monit/1549399756_1c92a80

real    1m18.535s
user    0m0.191s
sys     0m0.026s
```

**OBSERVE**
- There are about 550 milion lines stored into Elasticsearch. 
- The search took 1 minute and 20 seconds, it is quite a lot. This time is expected to increase on increasing the amount of data stored into Elasticsearch.


***CASE.2*** Random sample of 5 lines over all lines produced in the last betwwn `now` and 3 days ago `now-3d`.

```
 time elasticall --s0=5 --s2='now-3d' --s3='now'  \
                  /usr/local/etc/elasticall/paramRandomSample2
```

```
ATTENTION: this query can take some time ... 
total hits: 10,214,480
May 14 01:07:16 psbackup05 kernel: sd 1:0:44:0: [sdar] Sense Key : Not Ready [current] [descriptor] 
May 13 17:24:41 ioc-hpl-03.pcdsn kernel: EtherCAT 0: Domain 0: 4 working counter changes - now 9/9.
May 16 00:50:40 psanagpu104 kernel: [2959316.460953] NFS: state manager: check lease failed on NFSv4 server psnfs02 with error 13
May 15 03:50:06 psrelay.pcdsn dhcpd: DHCPDISCOVER from 0c:c4:7a:1c:3c:2b via eth1.661: network 172.21.48/22: no free leases
May 14 04:50:40 psbackup05 kernel: mpt3sas_cm0: #011request_len(4096), underflow(4096), resid(4096)

real    0m1.557s
user    0m0.188s
sys     0m0.029s
```

***OBSERVE***. 
- "total hits: 10,214,480". This query ran across 10 milion documents to get its 5 elements selection.
- The query took about 0.2 seconds to be resolved.
- This amount of time is expected not to grow much in future.

***CASE.3*** Random sample of 5 lines over all lines produced in the last month and matching the Lucene query `nmingott AND NOT psmetric`.

```
$> time elasticall --s0=5 --s1='nmingott' \
                   --s2='now-3d' --s3='now' \
                   /usr/local/etc/elasticall/paramRandomSample2 
```
```
ATTENTION: this query can take some time ... 
total hits: 408
May 14 03:56:29 psmetric01 crontab[15949]: (root) LIST (nmingott)
May 13 17:22:48 psmetric01 sudo: pam_unix(sudo:session): session opened for user root by nmingott(uid=0)
May 15 22:56:29 psmetric01 crontab[8537]: (root) LIST (nmingott)
May 14 09:26:31 psmetric01 crontab[10929]: (root) LIST (nmingott)
May 16 10:55:39 psmetric04 sshd[28092]: Accepted gssapi-with-mic for nmingott from 134.79.101.10 port 42160 ssh2

real    0m0.262s
user    0m0.190s
sys     0m0.022s
```


## Random Sample to detect anomalies 


```
$> time elasticall --s0=20 --s2='now-2d' --s3='now'\
                    /usr/local/etc/elasticall/paramRandomSample2
```
```
ATTENTION: this query can take some time ... 
total hits: 6,876,159
May 16 11:10:01 psana1408 crond[30014]: (root) FAILED to authorize user with PAM (Failure setting user credentials)
May 16 06:25:11 psanagpu104 rpc.gssd[1273]: ERROR: No credentials found for connection to server psnfs01.pcdsn
May 14 20:43:38 psanagpu104 rpc.gssd[1273]: ERROR: gssd_refresh_krb5_machine_credential: Input/output error while initializing krb5 context
May 16 02:41:44 psrelay.pcdsn dhcpd: DHCPDISCOVER from 0c:c4:7a:1d:0d:e7 via eth1.661: network 172.21.48/22: no free leases
May 16 17:33:23 psanagpu104 rpc.gssd[1273]: ERROR: gssd_refresh_krb5_machine_credential: Input/output error while initializing krb5 context
May 15 05:47:00 ioc-hpl-03.pcdsn kernel: EtherCAT 0: Domain 0: 4 working counter changes - now 9/9.
May 15 03:41:35 psanagpu104 rpc.gssd[1273]: ERROR: gssd_refresh_krb5_machine_credential: Input/output error while initializing krb5 context
May 16 10:28:18 ioc-cxi-usrmot1.pcdsn kernel: input: Winbond Electronics Corp Hermon USB hidmouse Device as /class/input/input3
May 15 18:11:21 ioc-hpl-03.pcdsn kernel: EtherCAT WARNING: Datagram ffff810364a3b198 (domain0-0-main) was SKIPPED 2 times.
May 14 20:51:03 psbackup05 kernel: sd 1:0:129:0: [sddt] CDB: Read(16) 88 00 00 00 00 00 00 00 00 00 00 00 00 08 00 00
May 16 02:49:46 psbackup05 kernel: mpt3sas_cm0: #011handle(0x003a), ioc_status(scsi data underrun)(0x0045), smid(361)
May 14 23:52:04 psanaphi102 systemd-logind[1486]: Removed session 233810.
May 14 18:21:45 ioc-hpl-03.pcdsn kernel: EtherCAT WARNING 0: 2 datagrams UNMATCHED!
May 16 06:55:31 psanaphi101 systemd[1]: Created slice user-15622.slice.
May 15 17:05:10 psmetric04 telegraf: 2019-05-16T00:05:10Z E! [inputs.snmp]: Error in plugin: agent switch-tst-b950-208: performing get on field hostname: Request timeout (after 3 retries)
May 15 08:17:04 psanagpu104 rpc.gssd[1273]: ERROR: gssd_refresh_krb5_machine_credential: Input/output error while initializing krb5 context
May 15 02:32:46 psanagpu104 rpc.gssd[1273]: ERROR: No credentials found for connection to server psnfs02.pcdsn
May 16 05:36:11 psanagpu104 rpc.gssd[1273]: ERROR: gssd_refresh_krb5_machine_credential: Input/output error while initializing krb5 context
May 15 12:37:04 daq-mec-cam01 NetworkManager[1157]: <info>  [1557949024.8139] device (enp5s0f1): Activation: starting connection 'Wired connection 1' (623b2eaa-e903-30ca-af9b-26e4a0cb6b6e)
May 15 21:03:48 psanagpu104 rpc.gssd[1273]: ERROR: gssd_refresh_krb5_machine_credential: Input/output error while initializing krb5 context

real    0m0.837s
user    0m0.194s
sys     0m0.024s
```

Compare with a sample of 20 messages taken in the space of log messages produced between 1 and 2 months ago. 

```
 $> time elasticall --s0=20 --s2='now-2M' --s3='now-1M' \
                     /usr/local/etc/elasticall/paramRandomSample2
```

```
ATTENTION: this query can take some time ... 
total hits: 72,552,603
Mar 18 17:56:06 psana1119 monit[1859]: Processing queued event /var/monit/1550881320_1eac350
Apr 11 04:15:21 psanagpu104 sshd[7769]: Received disconnect from 172.21.49.203: 11: disconnected by user
Mar 17 22:46:46 psana1403 monit[1785]: Processing queued event /var/monit/1549389644_1511570
Apr  6 19:21:38 psrelay.pcdsn dhcpd: DHCPDISCOVER from 0c:c4:7a:1c:3c:2a via eth1.661: network 172.21.48/22: no free leases
Apr 11 22:36:27 psrelay.pcdsn dhcpd: DHCPACK on 172.21.68.220 to 00:25:90:ac:66:9a via 172.21.68.2
Mar 29 23:22:30 psanaphi102 systemd[1]: Starting user-15622.slice.
Mar 17 01:06:21 psana1104 monit[1873]: Processing queued event /var/monit/1551421314_2642150
Mar 18 13:17:15 psana1217 monit[1698]: Processing queued event /var/monit/1549986733_10afb60
Mar 19 19:14:47 psana1209 monit[1864]: Processing queued event /var/monit/1548493934_fe8830
Mar 30 00:31:43 psanaphi102 systemd[1]: Stopping user-15622.slice.
Mar 17 01:46:50 psana1306 monit[1863]: Processing queued event /var/monit/1549396185_718460
Apr  5 09:04:48 psanagpu101 sshd[14699]: pam_unix(sshd:session): session opened for user psjhub by (uid=0)
Apr 11 21:53:36 ioc-hpl-03.pcdsn kernel: EtherCAT WARNING 0: 1 datagram UNMATCHED!
Mar 20 19:08:31 psana1310 monit[1852]: Processing queued event /var/monit/1550864313_9b97b0
Mar 18 22:57:42 psana1218 monit[1784]: Processing queued event /var/monit/1551420561_1f71400
Mar 19 19:56:36 psana1215 monit[1904]: Processing queued event /var/monit/1550863982_f465b0
Mar 30 07:51:33 psrelay.pcdsn dhcpd: DHCPDISCOVER from 0c:c4:7a:01:da:a0 via 134.79.165.2: network 134.79.165/24: no free leases
Mar 19 15:35:22 psana1402 monit[1842]: Processing queued event /var/monit/1550080196_16748d0
Mar 18 18:10:40 psana1103 monit[1874]: Processing queued event /var/monit/1549394307_8662c0
Apr 16 14:04:25 psrelay.pcdsn dhcpd: DHCPDISCOVER from 00:25:90:02:ef:d4 via 172.21.62.2

real    0m9.999s
user    0m0.188s
sys     0m0.028s
```


The analysis is very crude but we can already establish with a good degree of confidence ***we have a problem in *psanagpu104****. Indeed, its frequent presence in the sample carries error messages and it is not reflected in the sample produced from old logs.


## Counting documents, aka log lines




