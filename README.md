# micropython-DS3231-AT24C32
MicroPython driver for DS3231 RTC and AT24C32 EEPROM module.

## DS3231N
Driver for the Dallas DS3231 highly accurate RTC module.
Driver implements both alarms and all available match triggers, interrupts and checks for both alarms.
Square wave output can be selected.
Checking for alarms, toggling square wave output do not allocate memory in the heap.

## Usage

### Create instance
Create an instance of the DS3231 class:

```python
from machine import Pin, I2C
from ds3231 import DS3231

i2c = I2C(sda=Pin(4), scl=Pin(5))

ds = DS3231(i2c)

### Time functions
set the time with:
```

# Set the date and time
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
`(2020, 10, 3, 6, 13, 55, 30, 0) # (year, month, mday, wday, hour, minute, second, 0)

Set the ESP8266 internal RTC using the DS3231
```python
import machine
rtc = machine.RTC()
rtc.datetime(ds.datetime)
```
### Alarm functions

The DS3231 has 2 internal alarms with can each be set independently to different match conditions.
The alarm has match options ranging from every second to every month.

Call an alarm without arguments and it will return the current alarm setting register.

*alarm 1*
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

*alarm 2*
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

*Checking and clearing alarms*
You can manually check to see if an alarm has triggered. This will return True when the alarm 
has been triggered and False otherwise. The act of checking clears the alarm register automatically.

`DS3231.check_alarm(alarm)`

alarm : alarm number, can be 1 or 2

### Square wave output
The DS3231 can output a square wave on the SQW/INT output pin. The following frequencies are availalbe:
* `FREQ_1`       1 Hz
* `FREQ_1024` 1024 Hz
* `FREQ_4096` 4096 Hz
* `FREQ_8192` 8192 Hz

When using the SQW/INT pin to output a square wave, the alarm interrupt not avaialable and if you would like
to check for alarms this will have to be done manually.

The DS3231 can also output the crystal frequency at 32768 Hz on a dedicated pin (32K). This output is enabled by default
on powerup and can me changed as follows:

```python
DS3231.output_32kHz() # Enable 32 kHz output
DS3231.output_32kHz(False) # Disable 32 kHz output
