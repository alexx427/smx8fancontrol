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
