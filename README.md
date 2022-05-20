# Geiger_lanph: a simple Geiger counter design

Author:  Areg Danagoulian

This repository provides documentation for very simple design for a Geiger Counter.  It is based on a synthesis of various open source Geiger counter
designs, and is optimized for simplicity rather than performance. 
The design consists of on an analog design for a high voltage (HV) module, a very minimalist analog signal processing configuration, followed up by Arduino DAQ instrumentation.

The design can be split into three main parts.

**HV module**

The HV module takes the 4.5 V from a battery pack (or 5V from USB interface) and boost converts it to ~300V.  
This is achieved by using a 555 timer as a switching device in feedback mode.  The "core" of the HV is an inductor coupled
to a capacitor through a rectifier diode.  To achieve the HV the inductor-ground connection is "switched" on and off via a HV transistor,
which takes the switching input from the 555 into its base.  This is the basic principle of how spark plugs on an internal compustion 
engine work. 

**Radiation Sensor***

The HV is then used to bias the Geiger-Muller tube, which in our case is the classic Soviet SBM-20 ($20 on ebay) via a 5.1 MOhm input resistor.
The negative terminal of the tube is sent to ground via a 5.1 kOhm resistor.  The readout of the signal is directly from the negative terminal.  

**Data Aquisition (DAQ)***

The DAQ is based on a small Arduino microcontroller.  It takes the output directly from the Geiger tube into one of its digital pins.  
The digital input requires a TTL signal, and while the output of the Geiger tube is not exactly TTL it's close enough to achieve reliable
triggering.  The Arduino sketch can be found on one of our other repositories [here](https://github.com/ustajan/GeigerDAQ/tree/main/GeigerCounter).

For any DAQ system the key question is ask is -- what is the dead time?  We estimate that the dead time for the DAQ configuration employed in this
design is <1ms. 

The DAQ performs the following actions:
* estimates the count rate via a variable integration window and displays it on a 16x2 standard LCD.  The integration time is determined as following:
  + When the observed rate is <20 CPM the integration time is 30 sec
  + else, if its <100 CPM the integration time is 10 sec
  + else: 1 second
* In addition to the count rate the DAQ also displays the dose rate in uR/hr.  A few points:  
  + The conversion coefficient for SBM-20 is 0.57 (uR/hr)/CPM.
  + Note that this conversion is based on 137Cs standard, i.e. 662 keV.  This conversion may be completely inaccurate for lower energy sources.  To "flatten" the energy dependence the tube can be wrapped in ~mm of lead as a way of achieving what's known as energy compensation.

**Readout to computer via the serial connection**

In addition to analyzing the count and dose rates and displaying those on the LCD, the Arduino DAQ software can also "talk" to a computer via the serial bus.
The software for this can be found in the above mentioned repository [here](https://github.com/ustajan/GeigerDAQ) -- look for ```logger.py``` python code.
The code reads and prints to file the timestamp of every event in microseconds, allowing for subsequent detailed statistical analysis.
