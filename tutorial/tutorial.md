#Toturial

This tutorial will show you how to build a BLE IoT network with `ble-shepherd`. `ble-shepherd` will become a BLE network server once it is started, and you can use it to control or manage the devices in the BLE network.

Before you start, you need to prepare some hardware devices

* PC, Raspberry, beaglebone or other Node.js platform
* [BLE 4.0 USB Adapter](https://github.com/sandeepmistry/node-bluetooth-hci-socket#compatible-bluetooth-40-usb-adapters)
* Any BLE peripheral device
* [TI SensorTag](http://www.ti.com/ww/en/wireless_connectivity/sensortag2015/?INTC=SensorTag&HQS=sensortag)



## Section A: Using ble-shepherd to organize a Bluetooth machine network

In this section, we'll show you how to use `ble-shepherd` to create a BLE machine server, and if you have any BLE device in the vicinity of your environment, you can see that they are joining the network.

### 1. Create a folder /bleserver and a server.js in it

```js
mkdir bleserver && cd bleserver
```

```js
touch server.js
```

### 2. Install the ble-shepherd module in /bleserver folder

```js
npm install ble-shepherd
```

### 3. Start to run ble-shepherd

```js
var BleShepherd = require('ble-shepherd');
var central = new BleShepherd('noble');

central.on('ready', bleApp);	// central will fire a ready event when it is ready
central.start();				// start the central

function bleApp () { }
```

### 4. Allow BLE devices join the network

After the BLE server is up, you need to allow peripheral devices to join the network. When the device joins the network, you can do anything to them. In the example below, we're simply read the basic information of the device.

// [todo] If you do not want all your devices to join the network, you can use central.blocker to filter out unwanted devices

* [1] [central 'ind' event and peripheral APIs](https://github.com/bluetoother/ble-shepherd#APIs)
* [2] [SIG-defined GATT Services](https://www.bluetooth.com/specifications/gatt/services) and [SIG-defined GATT Characteristics](https://www.bluetooth.com/specifications/gatt/characteristics)

```js
var BleShepherd = require('ble-shepherd');

var central = new BleShepherd('noble');

central.on('ready', bleApp);	
central.start();				// start the central

function bleApp () {
	central.permitJoin(300);
	
	// see [1]
	central.on('ind', function (msg) {
		var dev = msg.periph;

		switch (msg.type) {
			case 'devIncoming': 
				// show device basic information
				// see [2]
				console.log('Device Name: ' + dev.dump('0x1800', '0x2a00').value.name);
				console.log('Manufacturer Name: ' + dev.dump('0x180a', '0x2a29').value.manufacturerName);
				console.log('Model Number: ' + dev.dump('0x180a', '0x2a24').value.modelNum);
				console.log('Firmware Revision : ' + dev.dump('0x180a', '0x2a26').value.firmwareRev);
				console.log('Hardware Revision : ' + dev.dump('0x180a', '0x2a27').value.hardwareRev);
				console.log('Software Revision : ' + dev.dump('0x180a', '0x2a28').value.softwareRev);

				break;

			default:
	            // Not deal with other msg.type in this example
	            break;
		}
	});
}
```


## Section B: Using TI's SensorTag to implement a small application

In the previous section, we have started the BLE server, and organize a Bluetooth machine network automatically. In this section, we want to use the SensorTag to show you how to operate the network device to simply build up a Bluetooth application

### 1. Install TI's SensorTag plugin

plugin is provided by third-party module, it is used to help you recognize a third-party BLE device

```js
npm install bshep-plugin-ti-sensortag1
```

### 2. Using plugin in server.js



```js
var BleShepherd = require('ble-shepherd');
var tiSensortagPlg = require('bshep-plugin-ti-sensortag1');		// require ti SensorTag plugin

var central = new BleShepherd('noble');

central.support('ti-sensortag', tiSensortagPlg); 				// register plugin 

central.on('ready', bleApp);	
central.start();				// start the central

function bleApp () {
	central.permitJoin(300);
	
	central.on('ind', function (msg) {
		var dev = msg.periph;

		switch (msg.type) {
			case 'devIncoming': 
				// show device basic information
				console.log('Device Name: ' + dev.dump('0x1800', '0x2a00').value.name);
				console.log('Manufacturer Name: ' + dev.dump('0x180a', '0x2a29').value.manufacturerName);
				console.log('Model Number: ' + dev.dump('0x180a', '0x2a24').value.modelNum);
				console.log('Firmware Revision : ' + dev.dump('0x180a', '0x2a26').value.firmwareRev);
				console.log('Hardware Revision : ' + dev.dump('0x180a', '0x2a27').value.hardwareRev);
				console.log('Software Revision : ' + dev.dump('0x180a', '0x2a28').value.softwareRev);

				if (dev.name === 'ti-sensortag') {
					// Do what you'd like to do with your 'ti-sensortag' here
				}

				break;

			default:
	            // Not deal with other msg.type in this example
	            break;
		}
	});
}
```

### 3. 


### 4. 

	* IR temperature sensor
	* Humidity Sensor
	* Simple Key
	
	* using simple key to enable/disable sensors notification
	* If sensor value changes, print it out




```js
var BleShepherd = require('ble-shepherd');
var tiSensortagPlg = require('bshep-plugin-ti-sensortag1');		// require ti sensortag plugin

var central = new BleShepherd('noble');

central.support('ti-sensortag', tiSensortagPlg); 				// register plugin 

central.on('ready', bleApp);	
central.start();	// start the central

// Declare service and characteristic UUID of sensor tag
var IR_TEMPERATURE_UUID                     = 'f000aa0004514000b000000000000000';
var HUMIDITY_UUID                           = 'f000aa2004514000b000000000000000';
var SIMPLE_KEY_UUID                         = 'ffe0';    

var IR_TEMPERATURE_DATA_UUID                = 'f000aa0104514000b000000000000000';
var HUMIDITY_DATA_UUID                      = 'f000aa2104514000b000000000000000';
var SIMPLE_KEY_DATA_UUID                    = 'ffe1';

var seneorTag;

function bleApp () {
	central.permitJoin(300);
	
	central.on('ind', function (msg) {
		var dev = msg.periph;

		switch (msg.type) {
			case 'devIncoming': 
				// show device basic information
				console.log('Device Name: ' + dev.dump('0x1800', '0x2a00').value.name);
				console.log('Manufacturer Name: ' + dev.dump('0x180a', '0x2a29').value.manufacturerName);
				console.log('Model Number: ' + dev.dump('0x180a', '0x2a24').value.modelNum);
				console.log('Firmware Revision : ' + dev.dump('0x180a', '0x2a26').value.firmwareRev);
				console.log('Hardware Revision : ' + dev.dump('0x180a', '0x2a27').value.hardwareRev);
				console.log('Software Revision : ' + dev.dump('0x180a', '0x2a28').value.softwareRev);

				if (dev.name === 'ti-sensortag') {
					// Do what you'd like to do with your 'ti-sensortag' here
					seneorTag = dev;

					// Register a handler to handle notification of simple key
					seneorTag.onNotified(SIMPLE_KEY_UUID, SIMPLE_KEY_DATA_UUID, simpleKeyChange);

					// Enable Simple keys notification
					seneorTag.configNotify(SIMPLE_KEY_UUID, SIMPLE_KEY_DATA_UUID, true);
				}

				break;

			case 'attChange':
				var data = msg.data;
				 
				if (data.sid.uuid === IR_TEMPERATURE_UUID && data.cid.uuid === IR_TEMPERATURE_DATA_UUID) {
					console.log('>> IR Temperature value changed: ' + irTempConverter(data.value));
				} else if (data.sid.uuid === HUMIDITY_UUID && data.cid.uuid === HUMIDITY_DATA_UUID) {
					console.log('>> Humidity value changed: ' + humidConverter(data.value));
				}
				break;
			default:
	            // Not deal with other msg.type in this example
	            break;

			
		}
	});
}

function simpleKeyChange (data) {
	var keyState = data.state;

	if (keyState === 1) { 			//  The left key is pressed
		// Enable IR Temperature and Humidity sensors notification
		seneorTag.configNotify(IR_TEMPERATURE_UUID, IR_TEMPERATURE_DATA_UUID, true);
		seneorTag.configNotify(HUMIDITY_UUID, HUMIDITY_DATA_UUID, true);
		
	} else if (keyState === 2) {	//  The right key is pressed
		// Disable IR Temperature and Humidity sensors notification
		seneorTag.configNotify(IR_TEMPERATURE_UUID, IR_TEMPERATURE_DATA_UUID, false);
		seneorTag.configNotify(HUMIDITY_UUID, HUMIDITY_DATA_UUID, false);
	}
}

function irTempConverter (data) {
    var rawT1, rawT2, m_tmpAmb, Vobj2, Tdie2,  
        Tref = 298.15, 
        S, Vos, fObj, tempVal;

    rawT1 = data.rawT1;
    rawT2 = data.rawT2;

    if (rawT2 > 32768)
        rawT2 = rawT2 - 65536;

    // convert temperature to Celsius 
    m_tmpAmb = rawT1 / 128.0;
    Vobj2 = rawT2 * 0.00000015625;
    Tdie2 = m_tmpAmb + 273.15;

    S = (6.4E-14) * (1 + (1.75E-3) * (Tdie2 - Tref) + (-1.678E-5) * Math.pow((Tdie2 - Tref), 2));
    Vos = -2.94E-5 + (-5.7E-7) * (Tdie2 - Tref) + (4.63E-9) * Math.pow((Tdie2 - Tref), 2);
    fObj = (Vobj2 - Vos) + 13.4 * Math.pow((Vobj2 - Vos), 2);
    tempVal = Math.pow(Math.pow(Tdie2, 4) + (fObj/S), 0.25);
    tempVal = Number((tempVal - 273.15).toFixed(2));

    return tempVal;
}

function humidConverter (data) {
	return -6.0 + 125.0/65536 * data.humid;
}
```