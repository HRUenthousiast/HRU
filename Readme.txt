Home Assistant for controlling a Brink Flair 400 HRU and optimizing the bypass

By HRUenthousiast, August 27th 2024


INTRODUCTION

The purpose of these files is controlling a Brink Flair 400 
Heat  Recovery Unit (HRU) with Home Assistant by hand and automated. 
With the automation the operation of the bypass is optimized.

This is realized with the following steps: 
A) Connecting the Brink Flair HRU with Home Assistant. This is done 
using ESPHome for transferring the modbus signal of the HRU over the 
WIFI network.
The file Flair400ModbusEspHome.yaml contains the necessary coding 
for programming a Nodemcu ESP8266 (Lolin Nodemcu) with UART TTL to 
RS485 Converter. 
This code can easily be adapted for other ESP chips like EPS32 and 
other HRU's. 
I tried to make adaptions easier for non specialists with links to 
background information.

B) DashboardForFlairModbus.yaml containing the raw file format for a 
dashboard in Home Assistant for manual control and 
ConfigurationAddition.yaml with an addition for the configuration 
file to create a helper switch for the HRU in Home Assistant.
   
C) ExampleAutomationsFlair400.yaml with automations for automatic 
control of the bypass with Home Assistant. 

The files are tested on Home Assistant 2024.8.3 installed in 
supervisor mode as described on 
https://github.com/home-assistant/supervised-installer on  Debian 
GNU/Linux 12 (bookworm) on a INTELNUC (from the Windows XP era).
I have no reason to think it won't work on installations that differ 
from this one. I doubt however the feasibility of a Raspberry Pi 
with SSD memory. The error prevention of this is really simple. 
Adding a HRU as described will add entities and history, increasing 
the volatility and size of the database increasing the possibility 
of stability issues with the SSD, see for example the experience 
written in 
https://community.home-assistant.io/t/pi-ram-requirements/447375/10. 
I expect an oodroid computer with MMC or an Intel NUC will do a 
better job.

General background information: 
- Inspiration: 
https://community.home-assistant.io/t/brink-flair-325-heat-recovery-
unit-esphome-modbus-integration-5/423182 
- Preparing the hardware and writing the instructions in 
Flair400ModbusEspHome.yaml: 
https://esphome.io/components/modbus_controller.html 
- Connecting with the HRU: Installation Manual Brink Flair 400
- Numbers and type of the modbus registers:  Brink's Installation 
Regulation for Modbus UWA2-B/UWA2-E (not dated).




A) ESP8266 AND THE FILE FLAIR400MODBUSESPHOME.YAML 

As a first step you have to setup the hardware as described on 
https://esphome.io/components/modbus_controller.html
and on https://community.home-assistant.io/t/brink-flair-325-heat-
recovery-unit-esphome-modbus-integration-5/423182
For the 5V I used the USB port of the HRU. 

The UART protocol describes that Rx must be connected with Tx and 
vice versa but I had to connect Rx of the ESP8266 board with the Rx 
of the Uart convertor and the Tx with the Tx. 

Furthermore, you have to install the ESPHome dashboard (see: 
https://esphome.io/guides/getting_started_hassio.html )

The file Flair400ModbusEsphome.yaml contains the software needed for
the ESP. You can install this with Home Assistant as described at 
https://esphome. Other possibilities are there described as well. 
Steps: 
1) In the ESPHome dashboard: click on new device.
2) Fill in the required information regarding your ESP version, 
board and WIFI connection.
3) You get the message 'Installation created', click Install and 
Cancel. 
4) You'll see a tile representing your installation. Click Edit. 
5) The coding you see, is a basic installation that has to be 
combined with the installation in the file 
Flair400ModbusEsphome.yaml with the following steps: 
   
- componnents esphome: and esp32: don't alter
- component logger: replace by the component logging in 
Flair400ModbusEsphome.yaml including the comments. You can uncomment 
these when you have to check the logs.
- components api: and ota: don't change.				  
- component WIFI: replace by the component logging in 
Flair400ModbusEsphome.yaml and replace the ip-values by the values 
that are applicable for your network.
- component captive portal: replace by the text in 
Flair400ModbusEsphome.yaml starting with captive portal with a few 
alterations: 
   - component uart: replace the values by the pin numbers for ESP 
and the values for your HRU
   - component modbus_controller: check the address of the modbus of 
your HRU
  - other components: the register addresses of Brink Modbus are 
used, for other devices you can use this as an example. 
								 
The first upload is done with an USB cable, see 
https://esphome.io/guides/getting_started_hassio.
After that you can upload changes via WIFI. After uploading the 
fieldnames used in this file are used by the ESP chip and recognized 
in Home Assistant.

Attention for the following:
1) Placing the housing with the ESP board between the metal air 
piping of the Brink weakens the WIFI signal and the WIFI of ESP 
boards without external antenna is weak already. Attention to the 
place of the board or purchase of a board with external antenna will 
prevent connection problems.
2) The ESP8266 chip has modest possibilities of memory and velocity, 
the ESP32 has more computing power. 
With limitations, the ESP8266 was sufficient in my case. I describe 
my experience the possibilities and limitations in comments in the 
file Flair400ModbusEspHome.yaml. 



B) DASHBOARDFORFLAIRMMODBUS.TXT and CONFIGURATIONADDITION.YAML 

The first file contains coding for the raw configuration editor of 
Home Assistant with the same fieldnames as used in the file 
Flair400ModbusEspHome.yaml. See the map Images for a screenprint.
You can install this file in in the following way: 
- Add a new dashboard for example with the name HRU under het Home 
Assistant menu Settings / Dashboards.
- A new dashboard appears in the menu. Select that dashboard, select 
the pencil right-above, select the three dots menu, select the menu 
item raw configuration editor. Copy the content of the file 
DashbordForFlairModbus.txt in the file. Save, click Done and the 
register fields of the Flair400 appear.

The file ConfigurationAddition.yaml contains a helper switch for 
setting the HRU. Add this file to your configuration.yaml. As an 
alternative you can manually define a helper switch 
under Settings / Devices&services with the values as defined in the 
file.


  
C) EXAMPLEAUTOMATIONSFLAIR400.YAML

These automations are maybe a inspiration for you. For understanding 
these automations the following is of interest:

I have installed the HRU in an existing house and was lucky I could 
use a bit thicker ducts than mostly used resulting in low pressure 
and low energy consumption of the HRU. 
In this installation I omitted volume controls resulting in less 
ductwork and less pressure loss.
In this setup I couldn't control the air volume of each room and had 
to adjust the air volume at a higher level. The extra energy 
necessary was really low because of the low pressure of the 
installation. 
Furthermore, humidity control was not necessary because I couldn't 
connect the bathroom with this HRU. 
I also omitted CO2 control but controlled the unit the preset 
Air control and the possibility to change the airflow manually if 
necessary.  

Your situation can be different and you can need other automations 
but maybe you find inspiration in my automations:

1) Automation for steering the bypass. The Brink Flair has the 
"cooling" function and the "keep it warm" function, 
not the "keep it cool" and the "warming" function. The automation 
has all four functions.

2) Automation for control of bypass boost. Air contains not much 
energy and only with big difference in temperature outside and 
inside it can be useful to let the HRU work harder than required
normally. 

3) Automation for setting the airflow of the HRU. This automation 
listens to the helper switch defined in the file 
ConfigurationAddition.yaml.
With this switch you can set the HRU on 'Automatic' (controlled 
by Home Assistant) or manually choosing the desired flow rate.

4) Automation resets the helper switch. If set for a longer period 
than 10 hours on a manual setting, this automation sets it back to 
Automatic.

5)  Automation for controlling the airflow, depending on time, 
presence of people at home and the like.
