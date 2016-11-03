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

* [1] [ble-shepherd usage](https://github.com/bluetoother/ble-shepherd#Usage)

```js
// see [1]
var BleShepherd = require('ble-shepherd');

var central = new BleShepherd('noble');		// 'noble' sub-module targeting on BLE4.0 USB adapter

central.on('ready', bleApp);	// central will fire a ready event when it is ready
central.start();				// start the central

function bleApp () { 
	// we will implement BLE application later
}
```

### 4. Allow BLE devices join the network

After the BLE server is up, you need to allow peripheral devices to join the network. When the device joins the network, you can do anything to them. In the example below, we're simply read the basic information of the incoming device.

Note: By default ble-shepherd will add all BLE peripheral devices to the network, but, you can use [central.blocker](https://github.com/bluetoother/ble-shepherd#blocker-class) to block unauthorized devices from join the network 

* [1] [central APIs](https://github.com/bluetoother/ble-shepherd#1-control-the-network)
* [2] [central 'ind' event](https://github.com/bluetoother/ble-shepherd#APIs)
* [3] [peripheral APIs](https://github.com/bluetoother/ble-shepherd#2-monitor-and-control-the-peripherals) and SIG-defined GATT [Services](https://www.bluetooth.com/specifications/gatt/services), [Characteristics](https://www.bluetooth.com/specifications/gatt/characteristics)

```js
var BleShepherd = require('ble-shepherd');

var central = new BleShepherd('noble');

central.on('ready', bleApp);	
central.start();

function bleApp () {
	// allow devices to join the network, see [1]
	console.log('Server is ready. Allow devices to join the network within 300 secs.');
	central.permitJoin(300);
	
	// see [2]
	central.on('ind', function (msg) {
		var dev = msg.periph;

		switch (msg.type) {
			case 'devIncoming': 
				// show device basic information, see [3]
				console.log('Device Name: ' 	  + dev.dump('0x1800', '0x2a00').value.name);
				console.log('Manufacturer Name: ' + dev.dump('0x180a', '0x2a29').value.manufacturerName);
				console.log('Model Number: ' 	  + dev.dump('0x180a', '0x2a24').value.modelNum);
				console.log('Firmware Revision: ' + dev.dump('0x180a', '0x2a26').value.firmwareRev);
				console.log('Hardware Revision: ' + dev.dump('0x180a', '0x2a27').value.hardwareRev);
				console.log('Software Revision: ' + dev.dump('0x180a', '0x2a28').value.softwareRev);

				break;

			default:
	            // Not deal with other msg.type in this example
	            break;
		}
	});
}
```


## Section B: Using TI's SensorTag to implement a small application

In the previous section, we have started the BLE server, and organize a Bluetooth machine network automatically. In this section, we want to use the TI's SensorTag to show you how to operate peripheral device to simply build up a Bluetooth application.

### 1. Install TI's SensorTag plug-in

plug-in is provided by third-party module manufacturer, it is used to help you recognize a third-party BLE device.

```js
npm install bshep-plugin-ti-sensortag1
```

### 2. Register plugin in server.js

When registering the plug-in, we will fill in the device name of 'ti-sensortag'. After the device joins the network, we can use this name to quickly determine whether the currently incoming device is TI's SensorTag.

```js
var BleShepherd = require('ble-shepherd');
var tiSensortagPlg = require('bshep-plugin-ti-sensortag1');		// require TI SensorTag plug-in

var central = new BleShepherd('noble');

// register SensorTag plug-in and fill in the device name
central.support('ti-sensortag', tiSensortagPlg); 			

central.on('ready', bleApp);	
central.start();

function bleApp () {
	console.log('Server is ready. Allow devices to join the network within 300 secs.');
	central.permitJoin(300);
	
	central.on('ind', function (msg) {
		var dev = msg.periph;

		switch (msg.type) {
			case 'devIncoming': 
				console.log('Device Name: ' 	  + dev.dump('0x1800', '0x2a00').value.name);
				console.log('Manufacturer Name: ' + dev.dump('0x180a', '0x2a29').value.manufacturerName);
				console.log('Model Number: ' 	  + dev.dump('0x180a', '0x2a24').value.modelNum);
				console.log('Firmware Revision: ' + dev.dump('0x180a', '0x2a26').value.firmwareRev);
				console.log('Hardware Revision: ' + dev.dump('0x180a', '0x2a27').value.hardwareRev);
				console.log('Software Revision: ' + dev.dump('0x180a', '0x2a28').value.softwareRev);

				// judge the device that joins the network is TI's SensorTag or not
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

### 3. Use TI's SensorTag to do something

After TI's SeneorTag joined the network, you can do some simple operation on it. This tutorial want to complete the following functions.

* Using simple key to enable/disable temperature and humidity sensors notification.
* When a notification of temp or humid is received, print it out.

In view of the above functions, we will need following features of SensorTag. Please see [SensorTag User Guide](http://processors.wiki.ti.com/index.php/SensorTag_User_Guide) for more information.

* [1] [IR temperature sensor](http://processors.wiki.ti.com/index.php/SensorTag_User_Guide#IR_Temperature_Sensor)
* [2] [Humidity Sensor](http://processors.wiki.ti.com/index.php/SensorTag_User_Guide#Humidity_Sensor_2)
* [3] [Simple Key](http://processors.wiki.ti.com/index.php/SensorTag_User_Guide#Simple_Key_Service)
	

```js
var BleShepherd = require('ble-shepherd');
var tiSensortagPlg = require('bshep-plugin-ti-sensortag1');	

var central = new BleShepherd('noble');

central.support('ti-sensortag', tiSensortagPlg); 

central.on('ready', bleApp);	
central.start();

// Declare service and characteristic UUID of SensorTag
var IR_TEMPERATURE_UUID			= '0xf000aa0004514000b000000000000000';	// see [1]
var IR_TEMPERATURE_DATA_UUID 	= '0xf000aa0104514000b000000000000000';	// see [1]
var IR_TEMPERATURE_CONFIG_UUID  = '0xf000aa0204514000b000000000000000'; // see [1]

var HUMIDITY_UUID				= '0xf000aa2004514000b000000000000000';	// see [2]
var HUMIDITY_DATA_UUID     		= '0xf000aa2104514000b000000000000000';	// see [2]
var HUMIDITY_CONFIG_UUID  		= '0xf000aa2204514000b000000000000000'; // see [2]

var SIMPLE_KEY_UUID 			= '0xffe0';    							// see [3]
var SIMPLE_KEY_DATA_UUID      	= '0xffe1';								// see [3]

var seneorTag;

function bleApp () {
	central.permitJoin(300);
	
	central.on('ind', function (msg) {
		var dev = msg.periph;

		switch (msg.type) {
			case 'devIncoming': 
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

					// Start sensors and perform measurements each second
					seneorTag.write(IR_TEMPERATURE_UUID, IR_TEMPERATURE_CONFIG_UUID, { config: 1 });
					seneorTag.write(HUMIDITY_UUID, HUMIDITY_CONFIG_UUID, { config: 1 });
				}
				break;

			case 'attNotify':
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

// simple key characteristic change handler
function simpleKeyChange (data) {
	var keyState = data.enable;

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

// convert temperature to Celsius, see [1]
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

// convert humidity to relative humidity, see [2]
function humidConverter (data) {
	return -6.0 + 125.0/65536 * data.humid;
}
```

### 4. Start the BLE machine network

```js
node server
```