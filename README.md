# Introduction

One of the important parts of setting up a linux server is setting up a timeserver to work with and understanding how to check your time. 

## Checking the current time

```
date
```

This is obvious command to use to check the system date. For example: 

```
$ date
Wed Dec 11 03:53:35 UTC 2019
```

This tells me that the server's default timezone is UTC. 

## Changing the time

```
tzselect
```

For your personal desktop, it's customary to use your local timezone, but for any server, you should 
use UTC. Ensuring you have a consistant timezone across all of your servers irrespective of timezones
or the craziness of Daylight Savings Time. Here we'll change the timezone to Pacific. 


```
$ tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the time zone using the Posix TZ format.
#? 2
Please select a country whose clocks agree with yours.
 1) Anguilla		  19) Dominican Republic    37) Peru
 2) Antigua & Barbuda	  20) Ecuador		    38) Puerto Rico
 3) Argentina		  21) El Salvador	    39) St Barthelemy
 4) Aruba		  22) French Guiana	    40) St Kitts & Nevis
 5) Bahamas		  23) Greenland		    41) St Lucia
 6) Barbados		  24) Grenada		    42) St Maarten (Dutch)
 7) Belize		  25) Guadeloupe	    43) St Martin (French)
 8) Bolivia		  26) Guatemala		    44) St Pierre & Miquelon
 9) Brazil		  27) Guyana		    45) St Vincent
10) Canada		  28) Haiti		    46) Suriname
11) Caribbean NL	  29) Honduras		    47) Trinidad & Tobago
12) Cayman Islands	  30) Jamaica		    48) Turks & Caicos Is
13) Chile		  31) Martinique	    49) United States
14) Colombia		  32) Mexico		    50) Uruguay
15) Costa Rica		  33) Montserrat	    51) Venezuela
16) Cuba		  34) Nicaragua		    52) Virgin Islands (UK)
17) Curaçao		  35) Panama		    53) Virgin Islands (US)
18) Dominica		  36) Paraguay
#? 49
Please select one of the following time zone regions.
 1) Eastern (most areas)	      16) Central - ND (Morton rural)
 2) Eastern - MI (most areas)	      17) Central - ND (Mercer)
 3) Eastern - KY (Louisville area)    18) Mountain (most areas)
 4) Eastern - KY (Wayne)	      19) Mountain - ID (south); OR (east)
 5) Eastern - IN (most areas)	      20) MST - Arizona (except Navajo)
 6) Eastern - IN (Da, Du, K, Mn)      21) Pacific
 7) Eastern - IN (Pulaski)	      22) Alaska (most areas)
 8) Eastern - IN (Crawford)	      23) Alaska - Juneau area
 9) Eastern - IN (Pike)		      24) Alaska - Sitka area
10) Eastern - IN (Switzerland)	      25) Alaska - Annette Island
11) Central (most areas)	      26) Alaska - Yakutat
12) Central - IN (Perry)	      27) Alaska (west)
13) Central - IN (Starke)	      28) Aleutian Islands
14) Central - MI (Wisconsin border)   29) Hawaii
15) Central - ND (Oliver)
#? 21

The following information has been given:

	United States
	Pacific

Therefore TZ='America/Los_Angeles' will be used.
Selected time is now:	Tue Dec 10 19:56:26 PST 2019.
Universal Time is now:	Wed Dec 11 03:56:26 UTC 2019.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
	TZ='America/Los_Angeles'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
America/Los_Angeles
vagrant@node-1:~$ date
Wed Dec 11 03:57:40 UTC 2019
```


## Setting up a time server (chrony)

Chrony can be installed into Ubuntu using apt-get: 

```
sudo apt-get install -y chrony
```

## Checking the status of time tracking using chronyc (chrony client)

```
chronyc tracking
```

This tells us some important data. 

System time - number of seconds off of NTP time
Last offset - the differene last done. 

```
$ chronyc tracking
Reference ID    : C63C16F0 (clock.xmission.com)
Stratum         : 2
Ref time (UTC)  : Tue Dec 10 18:50:27 2019
System time     : 35917.621093750 seconds slow of NTP time
Last offset     : +0.000175326 seconds
RMS offset      : 1358.954589844 seconds
Frequency       : 20.669 ppm fast
Residual freq   : +0.037 ppm
Skew            : 7.083 ppm
Root delay      : 0.035813998 seconds
Root dispersion : 0.001644461 seconds
Update interval : 59.6 seconds
Leap status     : Normal
```

Obviously this is wrong...  35917.621093750 seconds slow ... 9.97711697049 hours off! 

That's crazy. Also we can see the last offset is is changing the time by 0.0001 seconds...

That's crazy. It's updating every 59.6 seconds ... 1 minute... That will take forever. 

Another command to mix with this is the *watch* command. It will allow you to see the changes happen over time: 

```
sudo watch -n 0.1 chronyc tracking
```
Here you can see the changes happening: 

![2019-12-10 - Watch Chronyc Tracking](https://user-images.githubusercontent.com/2634673/70672442-1a19d880-1c34-11ea-9818-e9027f7f6308.gif)



## Syncing the time server if there is a large discrepancy

To do a time correction, we can use the *makestep* command. 

```
sudo chronyc makestep 0.5 1
```

The makestep has two parameters, the step threshold, and the update count. Here, we are 
saying, for 1 update, if the ntp server's time is 0.5 seconds off from the system time, 
"step" the clock. (meaning set the clock time to the server's time). 

Now, an update happens periodically, so it won't happen immediately unless you happen to 
be close to the next update. 

However, if you want to force an update to happen immediately, use the *burst* command. 

In this example, I use makestep and burst to set the system's clock. 

![2019-12-10 - make step](https://user-images.githubusercontent.com/2634673/70672892-7b8e7700-1c35-11ea-829c-a59219c10da6.gif)

## Forcing chrony to check the time servers. 

After using makestep, you can force the time servers to update using the burst command: 

```
sudo burst 1 8
```

Here we are saying, get 1 good measurement, and only take 8 tries. Usually it will happen 
on the first try, but hey, we'll give it 8. Because we have 8 servers that we're connected
to. 

## Forcing chrony to update the tracking info

The tracking info will be off when chrony first starts or when you change the date.  That is because it takes 3 measurements first. 
In order to force those three measurements, use the *burst* command. 

```
sudo burst 3/3
```

Here we have chrony knowing the correct time, but it is different than the system time. We can *force* the tracking to update by forcing 3 good checks. A step can be done using makestep. 

![2019-12-10 - Burst 3 3](https://user-images.githubusercontent.com/2634673/70672709-ebe8c880-1c34-11ea-8c5d-d8c143653cf6.gif)


