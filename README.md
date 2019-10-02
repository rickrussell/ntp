# Building and Running with Docker Compose
---
## Pull and run -- it's this simple.
---
Using the docker-compose.yml file included in this git repo, you can build the container yourself (should you choose to).
*Note: this docker-compose files uses the `3.4` compose format, which requires Docker Engine release 17.09.0+

```
# clone, run
$ git clone https://github.com/rickrussell/ntp

$ cd ntp
$ docker-compose up -d 
```

## Test NTP
---
From any machine that has `ntpdate` you can query your new NTP container with the follow
command:

```
$> ntpdate -q <DOCKER_HOST_IP>
```

Here is a sample output from my environment:

```
$> ntpdate -q <IP of Docker Host>
server <IP of Docker Host>, stratum 1, offset 0.010089, delay 0.02585
17 Sep 15:20:52 ntpdate[14186]: adjust time server <IP of Docker Host> offset 0.010089 sec
```

If you see a message, like the following, it's likely the clock is not yet synchronized, or, perhaps you don't have the ntp ports forwarded
```
$> ntpdate -q <IP of Docker Host>
server <IP of Docker Host>, stratum 1, offset 0.005689, delay 0.02837
11 Dec 09:47:53 ntpdate[26030]: no server suitable for synchronization found
```

## Troubleshooting chronyd(NTP)
To see details on the chronyd(ntp) status, you can check with the below command on your
docker host:

### Connect to Server for Troubleshooting
```
$> cd ntp
projects/IaC/ntp $> docker-compose ps 
projects/IaC/ntp $> docker-compose ps
Name              Command               State                      Ports
--------------------------------------------------------------------------------------------
ntp    webproc --on-exit restart  ...   Up      0.0.0.0:123->123/udp, 0.0.0.0:8080->8080/tcp

```

#### Check Server (local daemon)
```
$> docker-compose exec ntp chronyc tracking
Reference ID    : 7F7F0101 ()
Stratum         : 1
Ref time (UTC)  : Mon Jul 22 18:14:32 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```
#### Check Sources
```
$> docker-compose exec ntp chronyc sourcestats
210 Number of sources = 6
Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
==============================================================================
time-b-g.nist.gov           0   0     0     +0.000   2000.000     +0ns  4000ms
time-b-wwv.nist.gov         0   0     0     +0.000   2000.000     +0ns  4000ms
triton.ellipse.net          0   0     0     +0.000   2000.000     +0ns  4000ms
server.nanoslim.org         0   0     0     +0.000   2000.000     +0ns  4000ms
cartwheel.spiderspace.co>   0   0     0     +0.000   2000.000     +0ns  4000ms
nu.binary.net               0   0     0     +0.000   2000.000     +0ns  4000ms
```
#### Debug Sources
```
$> docker-compose exec ntp chronyc sources
210 Number of sources = 6
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^? time-b-g.nist.gov             0  10     0     -     +0ns[   +0ns] +/-    0ns
^? time-b-wwv.nist.gov           0  10     0     -     +0ns[   +0ns] +/-    0ns
^? triton.ellipse.net            0  10     0     -     +0ns[   +0ns] +/-    0ns
^? server.nanoslim.org           0  10     0     -     +0ns[   +0ns] +/-    0ns
^? cartwheel.spiderspace.co>     0  10     0     -     +0ns[   +0ns] +/-    0ns
^? nu.binary.net                 0  10     0     -     +0ns[   +0ns] +/-    0ns
```
#### Manully force update of time
```
$> docker-compose exec ntp chronyc makestep
200 OK

```

Links: 
[chronyc (chrony cli)](https://chrony.tuxfamily.org/doc/3.5/chronyc.html "chronyc (chrony CLI)") 
[chronyd (chrony daemon)](https://chrony.tuxfamily.org/doc/3.5/chronyd.html "chronyd (chrony daemon)") 
[chrony.conf)](https://chrony.tuxfamily.org/doc/3.5/chrony.conf.html "chrony.conf)")
