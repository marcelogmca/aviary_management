
# Capture and Visualization of Remote Aviary Data
A project with the objective of modernizing a group of medium scale aviaries.

Part of the code used in this project is provided in this repository.

## Reasons to start the project

My parents own a few small sized quail aviaries, and as any job that involves animals, they need to be checked and maintained multiple times a day, everyday. Making it more difficult, these are in different locations causing the need for constant moving.
A lot of time is spent in repetitive tasks that can be cheaply and efficiently automated.

### What can be easily automated?

 - Checking **Temperature** and **Humidity** in near real time (To make sure those are in a **safe** interval);
 - Keeping record of all temperature and humidity changes;
 - Recording water consumption;
 - Check for potentially dangerous substances in the air, such as smoke and gas;
 - Alerts if problems occur.

## Location and Connection
Buildings can be up to one kilometer distance from the main building, and so different connection technologies were analysed to find the most adequate,

|Technology|Advantages|Disadvantages|
|--|--|--|
|Cabled Internet|Reliable|Monthly subscription; Paying the "cable pulling"|
|Long-Range WiFi|Low-cost hardware|Vulnerable to Obstacles; Still relatively Low Range|
|LoRaWAN|Potentially High Range|Needs expensive hardware to achieve such High Range (Antennas)|
|Cellular-Internet|Low-cost hardware; Can use a twin sim card reducing costs significantly; Signal virtually anywhere|Monthly subscription can become expensive|

Due to all the buildings having good cellular signal, mobile routers costing around 40€, and the possibility of having twin sim cards the Cellular-Internet was the ideal choice for a cheap and efficient way to connect all systems to a central server via internet.

# Main server

After the network is completed, a server will be required to communicate with sensors, devices and serve as a database (and many other things that may come in handy).
It's possible to rent online servers for such a task, but for the small scale of this project a local server is enough.

## Choice of hardware
Since there is no need for a high performance server, due to the simplicity of the project, the ideal choice of hardware is a Raspberry PI.


![Raspberry PI 3](https://images-na.ssl-images-amazon.com/images/I/91zSu44%2B34L._SX355_.jpg)


A Raspberry can be incredibly cheap and has enough performance to do well in all sorts of operations.
After having all services of this project up and running, as well as a few other unrelated services and being accessed by a few users, it still remained at less than 60% RAM usage and less than 3% average CPU load.
The simplest and most cost effective storage solution is having an external hard drive connected.
Keeping in mind Raspberry itself will not have enough energy output from it's USB ports to power an external HDD, so a powered drive or USB hub is required.
#### Protection against power loss
Raspberry and it's operating system, just like most systems, can have its file-system become corrupted and sometimes have hardware damage, so it's important to always have an UPS.
In this project an UPS is powering the raspberry and the router connected to the internet, so when power fails, internet can still be accessible (as long as the ISP did not suffer from the power loss as well, which is unlikely).
Some UPSs provide a USB/serial connection that can give status of its battery, so it's possible to create a save, quit and shutdown procedure just before losing power.

## Software
Raspbian (Debian) will be the operation system most suited for the raspberry, it has a lot of documentation and support online and is simple to setup.

Software installed:
 - Docker - It will be used to install and manage multiple services with ease.
 - Node Js - It will do all the processing needed and can act as a web page server, its a very well-documented technology, made to support many simultaneous calls efficiently.

### Docker services
- Influxdb
	- Open source time series database, made for event based data and real-time visualization.
- Grafana
	- Open source data visualization dashboard, with alert configuration support.

### NodeJS endpoints
- Data Reception
	- The data received from the sensor nodes is converted into Influxdb compatible data and sent to its service.
- Alerts Configuration
	- Since Grafana does not provide an easy way to change alerts specifications for the less tech savvy, a simple web-service using Grafana's API enables quick configuration of all alerts.
- Alert forwarding to Twillio
	- Not available externally; Grafana sends it's alerts and this service promptly formats it and sends it to Twillio.

## Server processing
Using Node Js, it's possible to create a service that will receive any relevant data via HTTP/TCP and process it, in this project it will receive temperature, humidity, water readings and others, convert it into a ready to be stored format as forward it to the database.

### Database (InfluxDB)
Influxdb is a an open source time series platform. Contrary to more common database engines, the data does not need to fit in a predefined table or have relation to other tables, with influxdb the **emphasis** is data's **time** of arrival, columns do not need to be predefined, so it's possible to have new sensors reporting new types of data without having to modify anything.
All data from this project is timed, being the information being reported back from all the sensors, so a time series platform makes the most sense, and brings certain advantages for this use case.
From [influxdata.com](https://www.influxdata.com/blog/why-build-a-time-series-data-platform/), four reasons to use influxdb:
- Time series data needs to focus on fast ingestion.
- High-precision data is kept for some short period of time with longer retention periods for summary data at medium or lower precision. 
- An agent or the database itself must continuously compute summaries from the high-precision data for longer term storage.
- The query pattern of time series can be quite different from other database workloads. 

### Webpage (Grafana)
Grafana is an open source analytics and monitoring visualization platform, it presents very useful features out of the box such as integration with influxdb and ability to configure alerts.
Instead of creating a custom website to handle data visualization, and having the risk of having a constantly changing environment that needs to be re-coded, with grafana changes can be easily done with it's editor mode. New components can be added, configured, moved and resized easily.
![Real time data visualization](https://i.imgur.com/hziz0rv.png)
![Graphed data history](https://i.imgur.com/mJGTGUB.png)
![Alert configuration](https://i.imgur.com/Kk4B3uX.png)

### Alerts (Twillio and Grafana)
When one value is within a dangerous interval, the server should contact the responsible people via mobile phone (call or sms).
For this, Twillio.com was the choice I made, they provide a very well documented API with a reasonable price.
*When an SMS is not alerting enough...*
[Tasker](https://tasker.joaoapps.com/) is an android application that among its vast amount of features contains the possibility to execute an action based on a received SMS, so it's possible to configure a custom tune or vibration when a warning is received.

### Backups
It's important to not rely in just one piece of hardware, problems do happen and it's better to be prepared.
For this, a Linux **crontask** was made to send a compressed image of the database to **cloud** service as well as separate **local storage**, **every few days**. The database having such a short amount of records make this a very basic and easy to implement solution.
Installing the operating system and required software solely in raspberry's SD card can also be an easy way to make full system backups, since it's possible to save the entire image.

# IoT
Every building will need to have multiple sensors which need to be connected to a device capable of reading them.

In this project the following devices are used:
 - An [arduino](https://www.arduino.cc/) to read, process and send sensor data.
 - ESP8266 ESP-01, a wireless module that allows arduino to connect to WiFi.
 - DHT11, a temperature and humidity sensor.
 - MLX90614, an infrared temperature sensor.
 - YF-S201, a water flow meter.
 - MQ-135, an air quality sensor.
 - MAX4466, a noise level meter.
 - LDR, a light level meter.

#### Cabling
In order to connect the various sensors to the arduino device, sometimes at distances over 20m, standard cat5 ethernet cables were used.

### ESP8266 Module
This WiFi module works with arduino by communicating via serial (RX and TX connections).
Arduino will write AT commands to ESP8266, and will receive a reply response from the module.
[\[List of available AT commands\]](https://www.espressif.com/sites/default/files/documentation/4a-esp8266_at_instruction_set_en.pdf) (Pay attention to firmware versions)

**Warning**: Some use cases may require a change in ESP's baud rate, but ***DO NOT USE AT+IPR COMMAND***, it will most likely **break** your device, use AT+UART_DEF or AT+UART_CUR instead. 

#### Commands to connect to a wifi network and send data

    AT+CWMODE=1 //configure as station mode (will not serve as an access point)
    AT+CWJAP="ssid","password" //connect to wifi
    AT+CIFSR //prints ip address
    AT+CIPSTART="TCP","ip",port //connect to a tcp server running
This commands are necessary to establish a connection to the server.

[AT+CIPSEND](https://github.com/espressif/ESP8266_AT/wiki/CIPSEND) command to send data to server

### Recovering from errors
ESP8266 module will reply with an appropriate error response when a problem occurs, and so if there is a persisting problem with the connection the device should attempt to reconnect and completely restart as last resort.
##### Errors observed
The mobile routers using cellular-internet lose connection and renew their IPs every few days, and when this occurs the ESP8266s stop being able to reach the internet without being restarted.
Sometimes due to external circumstances data fails to be sent, this should not prompt a restart unless it persists for a long period of time.
Due to the cheap nature of ESP-01 modules *very rarely* they would stop answering to commands unless hardware resettled.
The sensors may fail, but the data of the still working sensors should be sent and the list of non working sensors should be potentially sent to the server.
##### Restarting ESP-01 in case of failure
There are two ways of restarting the ESP-01 module, performing a software reset or an hardware reset.
A **software reset** consists in sending an *AT command* to the module to order a reset, however if the device is irresponsible after a potential failure this won't work and an hardware reset is required.
The way of implementing an **hardware reset** on this module is by cutting its current for a few seconds, for this a relay can be used between arduino's **+5v** and the **ESP-01**. This type of reset can also be useful to make sure the volatile memory is cleared.
##### Using arduino's watchdog, the best practice
Watchdog is a timer that runs in parallel to the arduino's core processes that implement a countdown. A range between 15ms to 8s can be configured, and when this countdown runs out the arduino will be **completely** restarted (not keeping any memory values).

***But why is this countdown useful?*** This countdown can be restarted at any moment, and so it's possible to implement a system that keeps resetting the countdown (preventing the restart) while the process is running smoothly, but in case of failure letting the countdown complete.

How to use the watchdog to manage failures:

    #include <avr/wdt.h>
    
    void watchDogSetup(){
      cli();  //Disable all interrupts
      wdt_reset(); //Reset the wdt timer
      //Enter Watchdog Configuration Mode:
      WDTCSR |= (1<<WDCE)|(1<<WDE);
      //Set watchdog settings:
      WDTCSR = (1<<WDIE)|(1<<WDE)|(1<<WDP3)|(0<<WDP2)|(0<<WDP1)|(1<<WDP0);
      sei(); //Reenable all interrupts
    }
The countdown time can be modified **up to 8s** by changing between "1<<" or "0<<" preceding WDP3, WDP2, WDP1 and WDP0 according to the following table:
|WDP3|WDP2|WDP1|WDP0|Time|
|--|--|--|--|--|
|0|0|0|0|16ms|
|0|0|0|1|32ms|
|0|0|1|0|64ms|
|0|0|1|1|0.125s|
|0|1|0|0|0.25s|
|0|1|0|1|0.5s|
|0|1|1|0|1s|
|0|1|1|1|2s|
|1|0|0|0|4s|
|1|0|0|1|8s|

Reseting the countdown:

    wdt_reset() 

Forcing a reset (this should only be done if it's an unrecoverable failure):
 

    delay(delay_ms);

*dalay_ms* must be any value higher than the countdown. The arduino's main process will not be able to restart the timer in any way, and so the restart will be triggered during the delay.

### Sensors

#### DHT11 (Temperature and Humidity)
This module will measure both temperature and humidity, the data sent to arduino is digital and so it is recommend to use a dht11 library for it to work with arduino as simple as possible.
The values captured by this module are not the most accurate or precise, but are reasonable for this project, and their incredibly low price point make this a very good sensor, especially for the conditions it needs to endure.
The following are some the characteristics of this module:
|Property|Value|
|--|--|
|Temperature Range|0-50ºC|
|Humidity Range|20-90%|
|Temperature Precision|+-2ºC|
|Humidity Precision|+-5%|
|Polling Rate|2s|

#### MLX90614 (Temperature)
The temperature of a room is rarely uniform, especially when heating or cooling systems are at play, and so having a measure of a single point in the room may not be ideal.
This module can average the temperature of a large surface area using infrared, with a minor caveat. Dusty environments can cover the sensor, stopping it from measuring as effectively however it can still not only provide the temperature of the dust, but also of the component itself and so at least turning into a single point sensor instead of being ineffective.

#### YF-S201 (Water flow)
This module will be connected to a water pipe, assuming it fits, it can support up to 30L/m which is plenty for this project.
#####  Tuning
From my experience, different flow meters will show different results out of the box for the same amount of water. A way to tune each flow meter to be as precise as possible is to funnel a fixed amount of water (say, a few liters) and tune it via code until it matches. In this case it will be used to measure total water that ran through the pipe, and so it will need to be calculated using the amount of turns made by the "propeller" inside.

#### MQ-135 (Air quality)
This module can sense air quality by detecting NH3, NOx, alcohol, benzene, smoke, CO2, etc.
Animals produce CO2, and some heating solutions use Gas, this sensor can capture a general idea of the room's air quality and provide a way to know if this dangerous materials exceed the normal amounts.

#### MAX4466 (Noise)
A noise level sensor in an aviary can monitor animal activity based on the noise their produce as well as external noise nearby, this is important for their stress and behavior, deaths due to strange noises such as grass cutters have happened in the past and so this can potentially give us insight into these types of events.

#### LDR (Light)
This is a very rudimentary sensor, it can be implemented to analyse light coming from sunlight and lamps during night time.
