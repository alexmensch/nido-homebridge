*This respository is the NodeJS plugin for [Homebridge](https://github.com/nfarina/homebridge) that allows the Nido smart thermostat to be controlled via [Apple HomeKit](https://www.apple.com/ios/home/). If you're looking for instructions on how to run Nido on a Raspberry Pi, see <https://github.com/alexmensch/nido> for instructions and the full project background.*

## Quickstart
```bash
npm install homebridge-nido
```

## Running the homebridge-nido plugin locally for development
### Requirements and initial configuration
1. Follow the instructions for [installing Homebridge](https://github.com/nfarina/homebridge#installation).
2. Install the plugin locally by running `npm install -g .` in the base directory.
3. Update the Homebridge configuration file in `~/.homebridge/config.json` with the settings needed by the homebridge-nido plugin. This [configuration file](https://github.com/alexmensch/nido/blob/master/homebridge/config.json) will work with minor modifications. (See description of configuration settings, below.)

### Starting Homebridge and testing against the Nido API
1. Start Homebridge by running `homebridge`. Confirm that the homebridge-nido plugin is loaded at startup. See the [Homebridge documentation](https://github.com/nfarina/homebridge#installation-details) for hints to debug any issues.
2. Start the Nido backend and API. Instructions can be found at [https://github.com/alexmensch/nido-python](https://github.com/alexmensch/nido-python#running-the-application-locally-for-development).
3. Pair your iOS device by scanning the QR code in the Homebridge startup log, or manually pairing with the default code `94812494`.

## Homebridge-nido plugin configuration settings
The following settings are all required. The homebridge-nido plugin expects to load them from Homebridge's `config.json` on initial startup. If your actions on your iOS device are not being received by the Nido API, or settings are not being loaded in the thermostat accessory representation in HomeKit, check the Homebridge logs for any errors.

 Setting     |Description
:------------|------------
`accessory`  |This must match the Accessory class name.
`name`       |The name of the Accessory.
`baseAPIUrl` |The base request URL to which all other URLs will be appended.
`getCHCSUrl` |Path to the getter for the CurrentHeatingCoolingState Characteristic.
`getTHCSUrl` |Path to the getter for the TargetHeatingCoolingState Characteristic.
`setTHCSUrl` |Path to the setter for the TargetHeatingCoolingState Characteristic. A trailing slash is in place as the target state is appended to this path.
`getCTUrl`   |Path to the getter for the CurrentTemperature Characteristic.
`getTTUrl`   |Path to the getter for the TargetTemperature Characteristic.
`setTTUrl`   |Path to the setter for the TargetTemperature Characteristic. A trailing slash is in place as the target temperature is appended to this path.
`getTDUUrl`  |Path to the getter for the TemperatureDisplayUnits Characteristic. **Note:** This characteristic does not change the units shown by the Accessory in HomeKit. Temperature units shown in HomeKit are controlled by the user's OS-wide setting; the Accessory should always return values in Celsius. This Characteristic (and the corresponding setter) are only intended to control the temperature display units on the physical thermostat. Since Nido has no physical display, this setting has no effect, but is required by HomeKit for the standard Thermostat Accessory defined in HomeKit.
`setTDUUrl`  |Path to the setter for the TemperatureDisplayUnits Characteristic.
`getCRHUrl`  |Path to the getter for the CurrentRelativeHumidity Characteristic.
`secret`     |The pre-defined secret that is checked to authorize calls to the Nido API.
`modemapping`|A dictionary that maps modes as defined by HomeKit to the corresponding mode name that the Nido API accepts. The HomeKit definition of modes can be found [here](https://github.com/KhaosT/HAP-NodeJS/blob/master/lib/gen/HomeKitTypes.js#L2341).
`validmodes` |The corresponding set of modes that Nido will accept. This limits the available modes that are shown to the user in HomeKit.

## Operation and design notes
In general, the plugin operates as a very simple translation layer between the HomeKit and Nido APIs with little complexity. There are just a few modifications from the default Accessory behavior as defined in HomeKit:
- The valid temperature range of the CurrentTemperature Characteristic has been expanded to -10C to 50C.
- The TargetTemperature range has been reduced to 12C to 30C, which more than covers a reasonable and comfortable range for a heater.
- The plugin polls and updates the CurrentHeatingCoolingState, CurrentTemperature and CurrentRelativeHumidity Characteristics every 5 minutes, which will trigger callbacks to HomeKit.
