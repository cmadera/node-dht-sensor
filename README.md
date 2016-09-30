# node-dht-sensor

This node.js module supports querying air temperature and relative humidity from a compatible DHT sensor.

## Installation
``` bash
$ npm install node-dht-sensor
```

## Usage

This module depends on the [BCM2835](http://www.airspayce.com/mikem/bcm2835/) library that must be installed on your board before you can actually use this module.

To initialize the sensor, you have to specify the sensor type and the [GPIO pin](https://www.raspberrypi.org/documentation/usage/gpio/) where the sensor is connected to. It should work for DHT11, DHT22 and AM2302 sensors.

You should use sensorType value to match the sensor as follows:

| Sensor          | sensorType value |
|-----------------|:----------------:|
| DHT11           | 11               |
| DHT22 or AM2302 | 22               |

If the initialization succeeds when you can call the read function to obtain the latest readout from the sensor. Readout values contains a temperature and a humidity property.

### First Example

This sample queries the AM2302 sensor connected to the GPIO 4 every 2 seconds and displays the result on the console.

``` javascript
var sensorLib = require('node-dht-sensor');

var sensor = {
    initialize: function () {
        return sensorLib.initialize(22, 4);
    },
    read: function () {
        var readout = sensorLib.read();
        console.log('Temperature: ' + readout.temperature.toFixed(2) + 'C, ' +
            'humidity: ' + readout.humidity.toFixed(2) + '%');
        setTimeout(function () {
            sensor.read();
        }, 2000);
    }
};

if (sensor.initialize()) {
    sensor.read();
} else {
    console.warn('Failed to initialize sensor');
}
```

### Multiple Sensors Example

The following example shows a method for querying multiple sensors connected to the same Raspberry Pi. For this example, we have two sensors:

1. A DHT11 sensor connected to GPIO 17
2. High-resolution DHT22 sensor connected to GPIO 4

``` javascript
var sensorLib = require("node-dht-sensor");

var sensor = {
    sensors: [ {
        name: "Indoor",
        type: 11,
        pin: 17
    }, {
        name: "Outdoor",
        type: 22,
        pin: 4
    } ],
    read: function() {
        for (var a in this.sensors) {
            var b = sensorLib.readSpec(this.sensors[a].type, this.sensors[a].pin);
            console.log(this.sensors[a].name + ": " +
              b.temperature.toFixed(1) + "C, " +
              b.humidity.toFixed(1) + "%");
        }
        setTimeout(function() {
            sensor.read();
        }, 2000);
    }
};

sensor.read();
```

### Reference for building from source

Standard node-gyp commands are used to build the module. So, just make sure you have node and node-gyp as well as the Broadcom library to build the project.

1. Download BCM2835 library and follow installation  [instructions](http://www.airspayce.com/mikem/bcm2835/).

2. In case, you don't have node-gyp, install it first:
   ``` bash
   $ sudo npm install node-gyp -g
   ```

3. Generate the configuration files
   ``` bash
   $ node-gyp configure
   ```

4. Build the component
   ``` bash
   $ node-gyp build
   ```

### Verbose output

Verbose output from the module can be enabled by defining ```VERBOSE``` during the module compilation. For example, this can be enabled via the binging,gyp file:

``` javascript
{
  'targets': [
    {
      'target_name': 'node-dht-sensor',
      'sources': [ 'node-dht-sensor.cpp' ],
      'libraries': [ '-lbcm2835' ],
      'defines': [ 'VERBOSE']
    }
  ]
}
```

### Appendix A: Quick Node.js installation guide

There are many ways you can get Node.js installed on your Raspberry Pi. Here is just one of way you can do it.
``` bash
$ wget http://nodejs.org/dist/latest/node-v6.7.0-linux-armv7l.tar.xz
$ tar xvfJ node-v6.7.0-linux-armv7l.tar.xz
$ sudo mv node-v6.7.0-linux-armv7l /opt
$ sudo update-alternatives --install "/usr/bin/node" "node" "/opt/node-v6.7.0-linux-armv7l/bin/node" 1
$ sudo update-alternatives --set node /opt/node-v6.7.0-linux-armv7l/bin/node
```

### References

[1]: Node.js latest release - http://nodejs.org/dist/latest/

[2]: BCM2835 - http://www.airspayce.com/mikem/bcm2835/

[3]: Node.js native addon build tool - https://github.com/TooTallNate/node-gyp

[4]: GPIO: Raspbery Pi Models A and B - https://www.raspberrypi.org/documentation/usage/gpio/
