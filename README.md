**mike0xt's fork of stelford's home-assistant_inkbird:**
See updated description below

  ****I am not capable to provide any support!!**
# Description
This repo adds support for the Inkbird IBS-TH1 under Home Assistant.

![Example Output inside Home Assistant](room-temps.png)

# Installation
Install this by dropping it into your config folder (path may vary
from install to install). In my install, this would be at
/config/custom_components/inkbird. 

## Home Assistant Container Install
Home Assistant Container should have NET_ADMIN capability
https://stackoverflow.com/questions/38758627/how-can-we-add-capabilities-to-a-running-docker-container
```
# in docker-compose.yml
cap_add:
  - NET_ADMIN
```
Access HA container bash to install bluepy
```
docker exec -it homeassistant bash
# in bash:
pip install --find-links $WHEELS_LINKS bluepy==1.3.0
# From <https://github.com/home-assistant/core/issues/24441> 
# lib cap and setcap may not be needed??
apk add labcap
setcap 'cap_net_raw,cap_net_admin+eip' /usr/local/lib/python3.8/site-packages/bluepy/bluepy-helper
```
## Configuration
Change your /config/configuration.yaml to have something like:

```
sensor:
  - platform: inkbird
    devices:
      - mac: '90:E2:02:9B:45:3E'
        name: 'Cians Room'
        monitored_conditions:
          - temperature
          - humidity
          - battery
      - mac: '90:e2:02:9b:4b:64'
        name: 'Kats Room'
        monitored_conditions:
          - temperature
          - humidity
          - battery
```

Obviously, the MAC and name you will change to your devices. The MAC you 
can find by using the scan.py inside helper_scripts. You can also
test in a 'once off' fashion by using the test_btle.py script with your
MAC updated inside it. NOTE: help scripts use old method to get values. Final sensor.py uses new method to get fff2 values.

Every time a scan_interval is hit (set at 60s)
then the Inkbird.Updater will scan the btle for any broadcasts for 10s.
The Inkbird sends out a broadcast every 10s as well. This means that
from time to time, we won't get lucky and listen at the right time.

That said, there is no more btle connections happening (as it's using
broadcasted data from the devices only now). It's also vastly more
power efficient.


# Changelog

## [1.0.1] - 2021-04-20
Tested on a Raspberry Pi 3B running RaspberryPiOS with HA Container
### Added
- Better instructions for installation into Home Assistant Container. May fix the error: "bluepy.btle.BTLEManagementError: Failed to execute management command ‘le on’ (code: 20, error: Permission Denied)"

### Changed
- Determination of sensor data only occurs on devices specified in configuration.yaml instead of all BT devices detected
- Cleaned up and simplified debug logging
- Removed waiting for next cycle for bluepy to return BTLE results. If no results are found, will restart the process and try again once for results. May result in HA warnings logged: "Update of inkbird.updater is taking over 10 seconds".
- Updated the manifest.json to fix "No 'version' key in the manifest file" warning.

### Fixed
- MAC addresses are now NOT case sensitive. May fix this error: https://community.home-assistant.io/t/inkbird-engbird-thermometer-and-hygrometer-sensor-need-help-trying-to-add-them-to-ha/94628/15
- Fixed memory leak likely caused by bluepy-helper crashing by killing process. See https://github.com/IanHarvey/bluepy/issues/267#issuecomment-657183840
