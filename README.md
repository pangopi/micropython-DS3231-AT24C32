# ds3231_with_eeprom.py
MicroPython driver for DS3231 RTC and AT24C32 EEPROM circuitboard.

Authored by Conrad Storz as a vibe coding project. July 22, 2025

This is an effort to expand on the work of Willem Peterse and his project to access the DS3231.

I am adding EEPROM access and access to the temperature sensor on the circuitboard.

# Example usage of the DS3231_With_EEPROM class
# Uncomment the following lines to test the functionality with a real I2C bus
# from machine import I2C, Pin
# i2c = I2C(0, scl=Pin(22), sda=Pin(21))  # Adjust pins as needed
# ds = DS3231_With_EEPROM(i2c)  # Create an instance of the RTC with EEPROM support  
    
# Example usage:
         
# ds.write_eeprom(0, b'Hello')            # Write 'Hello' to EEPROM at address 0 Valid addresses are 0-1023
# data = ds.read_eeprom(0, 5)             # Read 5 bytes from EEPROM starting at address 0
# print(data)                             # Should print b'Hello', 'b' is a bytes object in Python 
# temp = ds.read_temperature()            # Read temperature
# print("Temperature:", temp, "C")        # Print temperature in Celsius

# Note: Ensure that the I2C bus is properly initialized and the DS3231 circuitboard is connected correctly.
# The EEPROM operations are blocking, so they may take some time depending on the I2C speed and the EEPROM's write cycle time.
# The temperature reading is in Celsius and can be used for various applications like logging or triggering events
# based on temperature thresholds. Another choice would be as a time and temperature display device.  

The following is copied from Willem Peterse original project for your reference.

## Usage

### Create instance
Create an instance of the DS3231 class:

```python
from machine import Pin, I2C
from ds3231 import DS3231

i2c = I2C(sda=Pin(4), scl=Pin(5))

ds = DS3231(i2c)
```
### Time functions

Example of setting the date and time:
```python
year = 2020 # Can be yyyy or yy format
month = 10
mday = 3
hour = 13 # 24 hour format only
minute = 55
second = 30 # Optional
weekday = 6 # Optional

datetime = (year, month, mday, hour, minute, second, weekday)
ds.datetime(datetime)
```

returns in format of the ESP8266 RTC:
`(2020, 10, 3, 6, 13, 55, 30, 0) # (year, month, mday, wday, hour, minute, second, 0)`

Set the ESP8266 internal RTC using the DS3231
```python
import machine
rtc = machine.RTC()
rtc.datetime(ds.datetime)
```

#### Get the date and time
Call `ds.datetime()` to get the current date and time. This will print a warning on the REPL when the Oscillator Stop Flag (OSF) is set. When not using the REPL the OSF

### Alarm functions

The DS3231 has 2 internal alarms with can each be set independently to different match conditions.
The alarm has match options ranging from every second to every month.

Call an alarm without arguments and it will return the current alarm setting register.

#### alarm 1
Set alarm 1:
`DS3231.alarm1([time[, match[, int_en[, weekday]]]])`

time    : tuple, (second,[ minute[, hour[, day]]])
weekday : bool, select mday (False) or wday (True)
match   : int, match const
int_en  : bool, enable interrupt on alarm match on SQW/INT pin (disables SQW output)


Alarm 1 has the following match options:
```python
ds.AL1_EVERY_S      # Alarm every second
ds.AL1_MATCH_S      # Alarm when seconds match (every minute)
ds.AL1_MATCH_MS     # Alarm when minutes, seconds match (every hour)
ds.AL1_MATCH_HMS    # Alarm when hours, minutes, seconds match (every day)
ds.AL1_MATCH_DHMS   # Alarm when day|wday, hour, min, sec match (specific wday / mday) (once per month/week)
```

Example:
```python
# Set alarm 1 for 16:10:15 every day
ds.alarm1((15, 10, 16), match=ds.AL1_MATCH_HMS)
```

#### alarm 2
Set alarm 2:
`DS3231.alarm2([time[, match[, int_en[, weekday]]]])`

time    : tuple, (minute[, hour[, day]])
weekday : bool, select mday (False) or wday (True)
match   : int, match const
int_en  : bool, enable interrupt on alarm match on SQW/INT pin (disables SQW output)


Alarm 2 has the following match options:
```python
ds.AL2_EVERY_M # Alarm every minute on 00 seconds
ds.AL2_MATCH_M # Alarm when minutes match (every hour)
ds.AL2_MATCH_HM # Alarm when hours and minutes match (every day)
ds.AL2_MATCH_DHM # Alarm when day|wday match (once per month/week)
```

Example:
```python
# Set alarm 1 for 19:30 every Tuesday
ds.alarm2((30, 19, 2), match=ds.AL2_MATCH_DHM, weekday=True)
```

#### Checking and clearing alarms
You can manually check to see if an alarm has triggered. This will return True when the alarm has been triggered and False otherwise. The act of checking clears the alarm register automatically.

`DS3231.check_alarm(alarm)`

alarm : alarm number, can be 1 or 2

### Square wave output
The DS3231 can output a square wave on the SQW/INT output pin. The following frequencies are availalbe:
* `FREQ_1`       1 Hz
* `FREQ_1024` 1024 Hz
* `FREQ_4096` 4096 Hz
* `FREQ_8192` 8192 Hz

Example:
```python
ds.square_wave(freq=ds.FREQ_1024) # Enable 1024 Hz output
ds.square_wave(freq=False) # Disable SQW output
```

Note:
* When using the SQW/INT pin to output a square wave, the alarm interrupt is not avaialable. Checking for alarms will have to be done manually.
* When disabling the SQW output signal, the alarm interrupts (even when set before) remain disabled. They can to be set manually afterwards, if desired with `ds.alarm_int()`.

The DS3231 can also output the crystal frequency at 32768 Hz on a dedicated pin (32K). This output is enabled by default on powerup and can me changed as follows:

```python
ds.output_32kHz() # Enable 32 kHz output
ds.output_32kHz(False) # Disable 32 kHz output
```
