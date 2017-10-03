# VS_fstreun_AntiTheft

<b>Team</b>

streuseli (owner)<br>

copied from https://www.vs.inf.ethz.ch/edu/VS/exercises/A1/VS_HS2017_Assignment1.pdf

<b>Distributed Systems - Assignment 1.3  Sensing using Bluetooth Low Energy</b>

In this task, you will use Android’s built-in platform support for Bluetooth Low Energy (BLE) to con-
nect to the Sensirion HumiGadget to sense humidity and temperature. You should display the current
measurements to the user.
Helpful resources for this task are
• the Android guide on BLE: https://developer.android.com/guide/topics/
connectivity/bluetooth-le.html. Note, that it partly uses deprecated methods and
you should use the current methods which are available from version 5.0 (API 21) onwards (see
https://developer.android.com/about/versions/lollipop.html under
Advanced Connectivity)
1. Create a new project called VS_nethz_BleSensirion with an Activity called
MainActivity. The package name should be ch.ethz.inf.vs.a1.nethz.ble.
2. The Activity should first check if BLE is supported on the device. It should then check if bluetooth
is enabled, and display a dialog to the user requesting permission to enable it if it is disabled.
3. Implement a scan for available devices. You should limit the time of the scan and you should only
show Sensirion HumiGadget devices (e.g. by using a ScanFilter). Scanning should stop once
the user selects one of the listed devices, or the scan-time expires.
4. Note, that you also need the permission ACCESS_FINE_LOCATION. Since Android
6.0 you have to request this permission from the user the first time the app is
started. Find more information on https://developer.android.com/training/
permissions/requesting.html. Additionally, also since Android 6 the location service
has to be turned on to be able to perform a BLE-scan. You have to request the user to activate the
location services.
5. Basically, scanning for BLE devices works as follows:
(a) Request a BluetoothManager from the system and retrieve the BluetoothAdapter
from it.
(b) With the BluetoothAdapter you can check whether Bluetooth is turned on on the device
or not. If not, display the dialog.
(c) From the BluetoothAdapter, you can get the BluetoothLeScanner. With this
you can scan for Bluetooth devices. You have to provide a ScanCallback to receive the
results.
6. When the user selects a listed device, you should start a new activity which connects to
it. This is done by the method connectGatt(...), to which you have to passBluetoothGattCallback, which you have to implement. This callback monitors the con-
nection state. The method returns a BluetoothGatt object with which you can discover and
access the device’s services by discoverServices() and getService(...). When the
connection state changes to BluetoothProfile.STATE_CONNECTED, you should attempt
to start the service discovery on the device.
7. To access a service you can either iterate through the discovered services or use its unique
identifier ID (UUID) in getService(...). You should access two services, the humidity
and the temperature service. All UUIDs needed are listed below and are available in the class
SensirionSHT31UUIDS which we provide (to be able to omit the class name you can usestatic import for the class).
8. A service is composed of at least one characteristic (containing a single value), and a characteristic
holds descriptors (describing the characteristic’s value, optional).
9. Once
 you
 hold
 a
 service,
 you
 can
 access
 its
 characteristics
 using
service.getCharacteristic(...)
 with the corresponding UUID. Attempt to
read a value from the data characteristic.
10. You will realise, that there is a problem with reading data values. That is because the service
has to be activated on the device to use it. This is usually done by writing a one-byte in the
service’s configuration characteristic. The problem is that the characteristic has no permissions
set for writing. To change that we add our own characteristic with the same UUID and the correct
permissions using addCharacteristic(...). Basically, we are overwriting the previous
characteristic for that UUID.
11. Now you can read values from the data characteristic. Note, that these have to be converted to
decimal format with the following code.
Conversion for humidity values:

```java
private float convertRawValue(byte[] raw) {
ByteBuffer wrapper = ByteBuffer.wrap(raw).order(ByteOrder.LITTLE_ENDIAN);
return wrapper.getFloat();
}
```
12. Rather than reading or continuously polling the device, we want to be informed of new
values.
 This can be done via notifications.
 To enable notification you have to set
them for the app and the bluetooth device. First, enable them on the app by calling
bluetoothGatt.setCharacteristicNotification(...) with the corresponding
characteristic as argument. Second, for the bluetooth device, add a descriptor to the char-
acteristic with the following UUID: UUID_NOTIFICATION_DESCRIPTOR. Set the value
of the descriptor to BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE
and call writeDescriptor to the descriptor of the data characteristic. Hence forth,
whenever the values changes the method onCharacteristicChanged(...) in the
BluetoothGattCallback will be called, from where you can retrieve the value.
13. The process above is the same for both the humidity and the temperature service.
Hint: Note, that to transfer anything to the BLE device you have to set the value and then call
the corresponding write-method. However, you always have to make sure that a read or write
on the bluetooth device has finished before you can call the next one. Take this into account for
configuring the second service.
14. Finally, use a graph as in task 1 to display the values.
15. Be aware of handling the resources correctly, i.e. closing the connection when closing the app etc.
**Hint:* One source of errors we found is pairing the BLE device with your smartphone in the
Android settings. Do not do this.
• UUID_HUMIDITY_SERVICE: 00001234-b38d-4985-720e-0f993a68ee41
• UUID_HUMIDITY_CHARACTERISTIC: 00001235-b38d-4985-720e-0f993a68ee41
• UUID_TEMPERATURE_SERVICE: 00002234-b38d-4985-720e-0f993a68ee41
• UUID_TEMPERATURE_CHARACTERISTIC: 00002235-b38d-4985-720e-0f993a68ee41
• UUID_NOTIFICATION_DESCRIPTOR: 00002902-0000-1000-8000-00805f9b34fb
