##Usage
An example implementation, including use of daylight and timer rules and a presence sensor, is included in `controller.py`.

`$ sudo python controller.py`

##Documentation

###class hue.PresenceSensor()
The `PresenceSensor` class provides a simple API to query whether the house is currently occupied, based on whether advertisement packets have recently been received from registered iBeacons associated with each member of the household.  The `query()` method returns `True` if the house is occupied (i.e. if any of the registered beacons are present), or `False` if none of the registered beacons have been detected for `SCAN_TIMEOUT` seconds.

The constructor method starts a subprocess to scan for Bluetooth LE devices using the `hcitool lescan` command.  Each time the `query()` method is called, an `hcidump` is started in another subprocess.  Its output is piped to a shell script in another subprocess, which parses the raw bluetooth packets to extract iBeacon advertisements and pipe them to the main process for handling.

Note that to detect beacons the python script must be run as root, as the `lescan` and `hcidump` commands require root permissions.

A configuration file `config.py` should be supplied including the following parameters:

|Parameter |Description |
|:---|:---|
|`HCI`|Bluetooth device to use, e.g. `hci0`|
|`SCAN_TIMEOUT`|Number of seconds to scan for registered beacons before timeout.|
|`DEVNULL`|Path to `\dev\null` (used to suppress stderr from subprocess)|

####hue.PresenceSensor.query()
Scans for advertisement packets from registered iBeacons.  Returns `True` as soon as one registered beacon is detected.  Otherwise waits `SCAN_TIMEOUT` seconds before returning `False` if no registered beacons are detected.

####hue.PresenceSensor.register_beacon(*MinorID*)
Add a beacon to the list of registered beacons in the household, by supplying the Minor ID of a new beacon as a string.  Currently the UUID and Major ID are not checked, so any valid iBeacon advertisement packet with the given MinorID would be parsed as the same beacon.

####hue.PresenceSensor.deregister_beacon(*MinorID*)
Remove the given beacon from the list of registered beacons in the household.

###class hue.DaylightSensor(*lat, lng*)
The DaylightSensor class provides a simple API to query whether a time supplied as an argument is within daylight hours. On initialisation, the constructor method queries the [sunrise-sunset.org](http://www.sunrise-sunset.org) API to obtain the sunset and sunrise times (UTC) for today at the location specifided by the latitute and longitude coordinates supplied as arguments.

The following  configuration parameters may be supplied in `config.py`:

Parameter | Description
:---|:---
`DAYLIGHT_UPDATE_FREQUENCY` (default=24) | Time between updating daylight times from [sunrise-sunset.org](http://www.sunrise-sunset.org).

####hue.DaylightSensor.query(*time*)
The argument `time` should be supplied as a `datetime` object in UTC.  If it has been more than `DAYLIGHT_UPDATE_FREQUENCY` hours since daylight times were last refreshed from [sunrise-sunset.org](http://www.sunrise-sunset.org), these are refreshed.  Then returns `True` if the date supplied as an argument is during daylight hours, `False` otherwise.

###class hue.HueLight(*name, ID*)
The HueLight class represents a single lamp connected to the bridge.  Instances of the HueLight class may be created and used seperately or (more conveniently) created for each light stored in the bridge by the constructor of a HueBridge object.  Various methods are implemented in order to control the lamp.

####hue.HueLight.on()
Switches the lamp on.  (Colour, brightness and other settings are unaffected.)

####hue.HueLight.off()
Switches the lamp off.  (Colour, brightness and other settings are unaffected.)

####hue.HueLight.dim()
Switches the lamp on and adjusts the brightness to a minimum setting.

####hue.HueLight.save_state()
Saves the current status of the lamp returned from the bridge into `self.state`.  May be used to e.g. reset lamp back to its previous on/off state after recalling a scene.  After calling this method `self.state['on']` is `True` if lamp was on, `False` if lamp was off when the method was called.

##Issues
- Verify registered beacons using UUID, Major & Minor ID
- Monitor battery level of ibeacon (if available) and warn if low
- Beacons may not be detected reliably if  disctance to receiver is large or SCAN_TIMEOUT is short, causing lights to cycle on and off while the house is occupied.  Experiment with extending the timeout period to 60s.
