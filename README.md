# Gas meter sensor for IGA AC-5M for Home assistant
To measure the gas use on an IGA AC-5M gas-meter (danish gas meter), it is possible to attach a small magnetometer outside the case and monitor the internal measuring disk which has a magnetic signature.

![Gas meter](/gas%20meter.png)

## Tools
- ESP8266
- HMC5883L Magnetometer (https://esphome.io/components/sensor/hmc5883l.html?highlight=hmc5883l)

## Installation
- Gaffatape the magnetometer onto the gas-meter according to the drawing
- Attach the magnetometer to e.g. D1 and D2 on the ESP8266
- In Home Assistant, figure out which of the axis-readings (x, y or z) gives out a variable signal when gas is being used (Turn on the stove to force gas use if possible). Comment out the axis-readings that are not neeeded. In the code below, the z-axis is the one giving the needed data.
- Finally, a template sensor is needed in Home Assistant to detect each revolution of the measuring disc in the gas-meter. The readout from the sensor describes a sinus-curve for each revolution of the measuring disc and in the HA code below, I have identified the middle of the sinus-curve to be 10 µT.

![Magentometer reading](/magnetometer%20reading.png)

## ESPHome Code

```
esphome:
  name: esp8266-gas-meter

esp8266:
  board: nodemcuv2

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:
  password: "YOURSECRET"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp8266-gas-meter Fallback Hotspot"
    password: "ANOTHER SECRET"

captive_portal:

i2c:
  sda: D1
  scl: D2

sensor:
  - platform: qmc5883l
    address: 0x0D
    range: 800uT
#    field_strength_x:
#      name: "HMC5883L Field Strength X"
#    field_strength_y:
#      name: "HMC5883L Field Strength Y"
    field_strength_z:
      name: "HMC5883L Field Strength Z"
    update_interval: 1s
```

## Home assistant code to detect a revolution of the measuring disc
```
 - platform: threshold
    name: Gascycle
    entity_id: sensor.hmc5883l_field_strength_z
    upper: -10 
    hysteresis: 2
```
- "upper" detects when the measuring disc in the gas meter (measured as a sinus-curve) is halfway through its revolution
- "hysteresis" ensures that if the meter stops very close to the "upper" limit and small vibrations on the sensor makes it jump up and down from the limit, that it does not measure them as full revolutions of the measuring disc
