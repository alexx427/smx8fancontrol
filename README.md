Supermicro X8 - Fan control (smx8fancontrol)

A while ago, i bought two used Supermicro server to build our lab with Proxmox. These servers are Supermicro SuperServer 6016T-MTLF equipped with X8DTL-if motherboard. These are stable servers but a bit old. Still, for our current needs, these servers meet our expectation in terms of performance. With 24 GB Ram each it’s enough for our Proxmox installation.

That said, those two are very noisy and it’s not even the power supply making the noise, it’s the 5 fans running at max speed every time the CPU is solicited a little. Under very small load, the fans are kicking off at full speed and within 30 sec it gets back to normal.

Looking at the BIOS settings, the number of options to control the fans are minimal:

    Full Speed/FS

    Performance/PF

    Balanced/BL

    Energy Saving/ES.

The best one for our need is the “ Energy Saving/ES” but it has this strange behavior of suddenly running all the fans at 100%.

After a little digging, i found out the fans are controlled by a Winbond W83627DHG-P chipset. Also confirmed by Supermicro user manual. The chipset data sheet reveals it’s capable of quite a lot. There are many options to control the fan speed based on the system temperature, which should be enough to tweak the fan speed to our expectation, but the Supermicro BIOS is not displaying these options to the user.

The Linux Kernel is providing a driver to manage this chipset and it may allow us to reconfigure the chipset with sane values. After experimenting with the driver, it’s not a viable solution because the driver polling the chipset at regular intervals. After a period of time, it conflicts with the polling done by the BCM (IPMI) and causes a deadlock making the chipset unresponsive. So it’s a bad solution, because we lose control of the fans and lose the sensor data retrieved by BCM (IPMI).

Finally, with few experimentation, I find out it’s possible to reconfigure this chipset manually using i2c in a very similar way as the driver’s doing.

Installation

To install it on your system, you need to install few dependencies.

sudo apt-get install python-smbus

You also need to enable the i2c-dev kernel module by editing the file /etc/modules:

i2c-dev

Finally, download the script:

sudo wget -O /usr/local/bin/smx8fancontrol https://gist.github.com/ikus060/26a33ce1e82092b4d2dbdf18c3610fde/raw/2b12bff6d880c9dec69c0b74e0fb9025b2be559c/smx8fancontrol 
sudo chmod +x /usr/local/bin/smx8fancontrol

Usage

Once installed, you should be able to execute the script from command line. By default, the script will output the configuration of the chipset.

$ sudo smx8fancontrol

Temp1
Mode: Smart Fan Control
Fan mapping Relationships (T1FMR): Fan1 Fan2 Fan3 Fan4 Fan5 Fan6 Fan7 Fan8
Smart Fan Control Table (SFIV)
   42C   47C   47C   47C   47C   47C   47C
   40%  100%  100%  100%  100%  100%  100%
Thermal Cruise (TTTI): 40C
Critical Temperature (T0CTFS): 47C
Hysteresis (HT0): 5C

Temp2
Mode: Smart Fan Control
Fan mapping Relationships (T2FMR): Fan1 Fan2 Fan3 Fan4 Fan5 Fan6 Fan7 Fan8
Smart Fan Control Table (SFIV)
   42C   47C   47C   47C   47C   47C   47C
   40%  100%  100%  100%  100%  100%  100%
Thermal Cruise (TTTI): 40C
Critical Temperature (T1CTFS): 47C
Hysteresis (HT1): 5C

Temp3
Mode: Smart Fan Control
Fan mapping Relationships (T3FMR): 
Smart Fan Control Table (SFIV)
   42C   47C   47C   47C   47C   47C   47C
   40%  100%  100%  100%  100%  100%  100%
Thermal Cruise (TTTI): 40C
Critical Temperature (T2CTFS): 47C
Hysteresis (HT2): 5C

Temp4
Mode: Smart Fan Control
Fan mapping Relationships (T4FMR): 
Smart Fan Control Table (SFIV)
   42C   47C   47C   47C   47C   47C   47C
   40%  100%  100%  100%  100%  100%  100%
Thermal Cruise (TTTI): 40C
Critical Temperature (T3CTFS): 47C
Hysteresis (HT3): 5C

Temp5
Mode: Smart Fan Control
Fan mapping Relationships (T5FMR): Fan1 Fan2 Fan3 Fan4 Fan5 Fan6 Fan7 Fan8
Smart Fan Control Table (SFIV)
   42C   47C   47C   47C   47C   47C   47C
   40%  100%  100%  100%  100%  100%  100%
Thermal Cruise (TTTI): 40C
Critical Temperature (T4CTFS): 47C
Hysteresis (HT4): 5C

Temp6
Mode: Smart Fan Control
Fan mapping Relationships (T6FMR): 
Smart Fan Control Table (SFIV)
   42C   47C   47C   47C   47C   47C   47C
   40%  100%  100%  100%  100%  100%  100%
Thermal Cruise (TTTI): 40C
Critical Temperature (T5CTFS): 47C
Hysteresis (HT5): 5C

Fan1
Output Nonstop Value (F1ONV): 40%

Fan2
Output Nonstop Value (F2ONV): 40%

Fan3
Output Nonstop Value (F3ONV): 40%

Fan4
Output Nonstop Value (F4ONV): 40%

Fan5
Output Nonstop Value (F5ONV): 40%

Fan6
Output Nonstop Value (F6ONV): 40%

Fan7
Output Nonstop Value (F7ONV): 40%

Fan8
Output Nonstop Value (F8ONV): 40%

At first look, the output if a bit cryptic, so will analyze it.

There are 6 temperature sensors available on this chipset. I’m not sure all of them are wired to something of importance. By default, only 3 of them are used to control the fan. For each temperature sensors, we have a block of information similar to this one.
Temp1 	Description
Mode: Smart Fan Control 	Define the mode of operation for this temperature sensor. Either Smart Fan Control or Thermal Cruise Control.
Fan mapping Relationships (T1FMR): Fan1 Fan2 Fan3 Fan4 Fan5 Fan6 Fan7 Fan8 	Used by Smart Fan Control or Thermal Cruise Control. Define the relationship between the temperature sensors and the fans. This is typically used to define zone. e.g.: The CPU should be cold by the fan attached to the CPU. In our use case, it’s rather useless, since the CPU is passively cooled. So we configured all the fans to be controlled by Temp1 and Temp2.
Smart Fan Control Table (SFIV) 	Used by Smart Fan Control. This table provides a connection between the temperature and the fan duties. When the temperature is <=42⁰C, the fan will run at 25%. When temperature is >=60⁰C, the fan will run at 100%. In between values are computed linearly. e.g.: If temperature is 52⁰C, fan duties will be ~69%.
42C 47C 47C 47C 47C 47C 47C 	 
40% 100% 100% 100% 100% 100% 100% 	 
Thermal Cruise (TTTI): 40C 	Used by Thermal Cruise Control.Define the targeted temperature for this sensor.
Critical Temperature (T0CTFS): 47C 	Used by Smart Fan Control or Thermal Cruise Control.Define the upper limit. When this temperature is reached, fan duties controlled by these sensors will be set up to 100%.
Hysteresis (HT0): 5C 	Used by Smart Fan Control or Thermal Cruise Control.To avoid changing the fan speed when the temperature sensors are unstable, it uses the hysteresis to smooth the value.

e.g.: With a temperature sensor of 45⁰C and a Hysteresis set to 2⁰C, the temperature may fluctuate between 43⁰C to 47⁰C without affecting the fan speed. If the temperature reach 48⁰C, then the fan speed will be changed according to the current mode of control.

When using this script, you may realize the values define by default in the BIOS are not appropriate. The lower limit is defined to 42⁰C and the upper limit is 47⁰ with Hysteresis of 5⁰C. Basically, telling the chipset to run the fan at 40% all the time. Except if upper then 47⁰C run at 100%. There is not increase because of the hysteresis of 5⁰C which equals the difference between 47⁰C and 42⁰C. Hopefully, using this script you may set the values to sane one.

$ sudo smx8fancontrol --help
usage: smx8fancontrol.py [-h] [-m {smart,cruise}] [-p PWM] [-t TEMP]
                         [-H HYSTERSIS]

Change SuperMicro X8 Fan Control.

optional arguments:
  -h, --help            show this help message and exit
  -m {smart,cruise}, --mode {smart,cruise}
                        Set the fan mode: smart or cruise. smart: to use Smart
                        Fan mode. cruise: to use Thermal Cruise mode.
  -p PWM, --pwm PWM     Set Fan Duty (in percentage).
  -t TEMP, --temp TEMP  Set Temperature (in Celsius) associated to pwm.
  -H HYSTERSIS, --hystersis HYSTERSIS
                        Set Hystersis value in degree (0-15)

On my system, to get a good trade between noise and temperature, I’m running this command to change the Smart Fan Control and the Hysteresis.

sudo smx8fancontrol -m smart -p 25,50,100 -t 42,47,60 -H 2

I hope this script will be useful to others despite the material being a bit old.
