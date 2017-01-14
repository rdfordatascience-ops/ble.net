# ble.net ![Build status](https://img.shields.io/vso/build/nexussays/ebc6aafa-2931-41dc-b030-7f1eff5a28e5/7.svg?style=flat-square) [![NuGet](https://img.shields.io/nuget/v/ble.net.svg?style=flat-square)](https://www.nuget.org/packages/ble.net) [![MPLv2 License](https://img.shields.io/badge/license-MPLv2-blue.svg?style=flat-square)](https://www.mozilla.org/MPL/2.0/)

`ble.net` is a Bluetooth Low Energy (aka BLE, aka Bluetooth LE, aka Bluetooth Smart) PCL to enable simple development of BLE clients on Android, iOS, and Windows 10 (advertising only at the moment, the underlying UWP connection API is pretty bad).

Currently only client operations are supported. Server operations will be added in the future if there is significant enough interest.

## Setup

### 1. Install NuGet packages

Install `ble.net` in your PCL
```powershell
Install-Package ble.net
```

In each platform project, install the relevant package:
```powershell
Install-Package ble.net-android
```
```powershell
Install-Package ble.net-ios
```
```powershell
Install-Package ble.net-uwp
```

> Note that there are two packages for Android, `ble.net-android` and `ble.net-android21`; the API 21+ package is not materially different for client operations but will be needed if/when server operations are implemented. You should probably just use `ble.net-android`.

### 2. Obtain a reference to `BluetoothLowEnergyAdapter`

Each platform project has a class `BluetoothLowEnergyAdapter` with a static method `BluetoothLowEnergyAdapter.ObtainDefaultAdapter()`. Obtain this reference and then provide it to your application code using whatever dependency injector or manual reference passing you are using in your project.

#### Android-specific setup

If you want the adapter enable/disable functions to work, in your main `Activity`:
```csharp
protected override void OnCreate( Bundle bundle )
{
   // ...
   
   BluetoothLowEnergyAdapter.InitActivity( this );

   // ...
}
```

If you want `IBluetoothLowEnergyAdapter.OnStateChanged` to work, in your calling `Activity`:
```csharp
protected sealed override void OnActivityResult( Int32 requestCode, Result resultCode, Intent data )
{
   BluetoothLowEnergyAdapter.OnActivityResult( requestCode, resultCode, data );
}
```

## API

> See sample Xamarin Forms app included in the repo for a complete example.

All the exmaples presume you have some `adapter` passed in as per the setup notes above:
```csharp
IBluetoothLowEnergyAdapter adapter = /* platform provided adapter value */;
```

### Scan for devices/advertisements/beacons

```csharp
await adapter.ScanForDevices(
      ( IBlePeripheral peripheral ) =>
      {
         // connect to peripheral
         //m_adapter.ConnectToDevice( peripheral, TimeSpan.FromSeconds( 5 ) );
         // or read the advertising data
         //peripheral.Advertisement.DeviceName
         //peripheral.Advertisement.Services
         //peripheral.Advertisement.ManufacturerSpecificData
         //peripheral.Advertisement.ServiceData
      },
      // scan for 30 seconds and then stop
      TimeSpan.FromSeconds( 30 ) );
```

### Connect to a BLE device

```csharp
var device = await adapter.ConnectToDevice( peripheral, TimeSpan.FromSeconds( 5 ) );

// read a specific value
try
{
   var value = await m_device.ReadCharacteristicValue( KNOWN_SERVICE_GUID, KNOWN_CHARACTERISTIC_GUID );
}
catch(GattException ex)
{
   Debug.WriteLine( ex.ToString() );
}

// enumerate all services on the device
foreach(var guid in await m_device.ListAllServices())
{
   Debug.WriteLine( guid );
}
```