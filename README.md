## Background
Like some others, I have expired Google Nest Protects, which I wanted to replace with a cheaper / scalable alternative, but above all, I also wanted to realise a trustworthy alarm system, which can integrate with HomeAssistant.

In persuit of this, I initially purchased the FireAngel Pro Connected Gateway, but found it to be a dreadfully unreliable piece of kit.
I removed all the alarms at once, and it didn't notice! Pressing 'test' still reported all alarms 'online'. 
In fact, after leaving the gateway completely unplugged for almost a year, the FireAngel app still says one alarm is online, and shows some obnoxious message "Monitoring the risk to your home in real time using complex algorithms… Everything is OK”.
So when you consider the safety and reliability of a DIY aproach, just remember, FireAngel have set the bar very low with their commercial 'smart' product offering.

My next step, after being bitterly disappointed with the gateway (and not minding if I destroyed it), was to attached it to a debugger, to see exactly what it was doing (with the mindset of intercepting its comms to create a local HA integration).
I could see all the JSON state changes being sent to AWS. But I found the messages were slow and that that it stopped updating locally if the internet connection went down. So in the end, I decided to build my own bridge from a donor radio module.

You can check here if you're interested in the debug log I trapped from the gateway.
todo: link

The WiSafe2 radios uses an 868MHz radio. I considered useing a generic 868MHz transiver to intercept the comms. But the data over the air is encrypted and and the specification is not documented for the public anyway.
So rather than trying to simulate a FireAngel radio, I looked at a radio module and noted that it uses SPI to communicate with the alarm board. The path of least resistance was therefore to take a radio module as a donor and use it to build my own bridge.

With the radio on the bench, I used PulseView to trap the communication and reverse-engineered everything that I could figure out. 
Check here if you're interested in the communication specification. Maybe you have some inside knowlege and can add to it?

## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

This project explains how I used a WiSafe2 radio module, along with an Arduino Nano, to bridge my network of FireAngel WiSafe2 alarms into HomeAssistant.
Using this method, all communication is local (unlike the FireAngel Connected Gateway product, which pushes states to AWS).


This WiSafe2HA bridge should work with any WiSafe2 alarm. But it has been specifically tested to work with the used the following alarms:
* FP2620W2
* FP1720W2-R
* WST-630
* W2-SVP-630
* W2-CO-10X


With this project, via Home Assistant, I am able to receive notifications when:
* An alarm is tested (the type of test - heat, smoke, CO and the result - OK/Not OK)
* An emergency event occurs (and the type of event - Fire/CO)
* An alarm is removed from / attached to its base
* An alarm's battery is running low

I can also send data back to the network, including:
* Fire Test
* CO Test
* Silence an alarm

And I can perform some basic configuration, like check and enable network pairing mode.
The bridge also produces a heart-beat, so we can be confident in it's connection to HA.


To build the WiSafe2HA bridge, you'll need:
* Home Assistant (optional Google Home / Amazon Alexa for TTS & Mobile Phone companion app for notifications)
* A WiSafe2 radio module
* An Arduino Nano
* PCB
* 4ch Level Converter (x2)
* 2.54mm jumper
* Enclosure (optional)




## Obtaining the parts
The WiSafe2 radio modules can be obtained from:
* Any old alarm. Perhaps you can find one which has expired, or has gone faulty.
* The W2-SVP-630 strobe units can regually be found for sale on ebay and are otherwise worthless 2nd-hand... I picked mine up for £5 :)
* The radio can be bought as a stand-alone module. But expect to over-pay if buying it new

The Arduino Nano:
Just a regular Arduino Nano (or clone). Should be 5v @ 16MHz.

The PCB:
Here's a link to the PCB I made for this:
* JSON file (for loading into easy-EDA): 
* gerber files (for ordering direct from JLCPCB, etc): 
I made this PCB using easy easy-EDA and I ordered mine from JLCPCB using easy-EDA's built in PCB ordering function. 

Level Converter:
You need 2x 4ch level converters. Just search ebay for "4ch Logic Level Shifter Converter Module 5V to 3.3V"

2.54mm jumper:
Rummage through your drawerers. There's always one in there somewhere!

2.54mm male header:
Search ebay for 2.54mm male header. You only need 2 pins, for attaching the jumper to.
If this is a problem, you can always just bridge the contacts together with solder.

Enclosure:
Here's a link to the STL file for the enclosure I made.



## More information
I built this bridge using an Arduino Nano, as opposed to an ESP8266 because:
* I liked the reliability of a direct USB connection (no dependency on wifi network)
* I needed to use SPI slave mode (the radio is designed to be the master)... The ESP8266 only supports SPI as a master



## Building the project
Program the Arduino Nano, using the sketch code provided. You shouldn't need to change anything.
ToDo: add link:

There are variables for the EMBEDDED DEVICE ID and MODEL, which you can change if you like. Basically, the radio thinks it's connected to an alarm. The parameters we set here define the type of alarm that the Nano will simulate.
This is used when we initiate a CO/Fire test from HA. Or silence the alarms.
The information here just needs to be unique (i.e. not the same as another device on your network).
Chances are you can leave it all alone.

The only library dependency is the SPI library, which should be installed into the Arduino IDE by default.
If you encounter any trouble uploading the sketch, try setting the processor to ATmega328P (old bootloader). I found is nearly always required if using an Arduino Nano 'compatible'.

Then you can solder all the components to the PCB.
This is self-explainitory. The PCB silk-screen shows the correct way to insert the level converters and the Arduino Nano.
The WiSafe2 radio can only be soldered in one way.

Radio modules from alarms have a backup battery. Whereas the modules from the Strobe (plug-in) units don't. If your radio has a battery, I recommend removing it. This ensures the Arduino will initialise the radio from a cold-start if the power is ever cycled.
Removing the battery is also required if you want to use the 3D-printed enclosure I provide.


## Configuring the hardware
There is a jumper which toggles the driver mode. For normal operation with HomeAssistant, the jumper should be on.

If you want to receive raw wi-safe2 hex data from the alarm network, for the purpose of further development (to add support for other alarms or functions, etc) then remove the jumper and connect the gateway to a serial debugger. You'll see the raw WiSafe2 hex data.
See my communication analysis notes for more info:


# Setting up in HomeAssistant
You'll be plugging the Arduino Nano into your HA device via USB. If it's the only USB to Serial device attached, it will show up as /dev/ttyUSB0
If you have more USB serial devices attached, it could be USB1, etc. You can check in HomeAssistant's Hardware Page.
I'll use /dev/ttyUSB0 for the example configuration.

These changes are done inside the HA configuration.yaml file.
Note: Whilst my HA configuration files (yaml) work fine for me, I am far from being a HA expert. So I'm sure the HA gurus amongst you will be able to improve/optimise this configuration significantly.
Please feel free to contribute any improvements and optimisations to these HA configuration files if this is your thing.

First, we'll adjust the recorder. The heartbeat of the bridge updates every 20 seconds or so. It does not seem logical to have HomeAssistant record the history of the heartbeat. We can disable entity recording by adding this:

recorder:
  exclude:
    entity_globs: 
      - sensor.fireangeldata   


The input from the bridge is built from a sensor template. This takes the serial data and makes individual sensors out of it, which we can use inside HA.

Here we create the sensor template, with 'heartbeat' and 'messages'.
Below the 'messages' section, I've included 3 example alarms; Smoke, Heat and CO.
Using these 3 examples, hopefully you can see how you could add as many alarms as you need to.

We just give each alarm a name, so we can identify it in HA.
And we set the ID of each alarm, so we can identify it from the WiSafe2 network (the id is the hex number). I'll explain how to get the IDs of your alarms shortley.

 ```yaml
#-------------------------
#FireAngel template sensor
#-------------------------
sensor:
  - platform: serial
    name: fireangeldata
    serial_port: /dev/ttyUSB0
    baudrate: 115200       
  - platform: template
    sensors:
    
      #HeartBeat 
      #Result is Online or Offline based on heartbeat changes!
      fireangel_radio_heartbeat:
        friendly_name: 'HeartBeat'
        icon_template: mdi:heart-pulse
        value_template: >
          {% if states.sensor.fireangeldata.state !='unknown' and (now() - states.sensor.fireangeldata.last_changed).total_seconds() < 125 %}
          Online
          {% else %}
          Offline!
          {% endif %}

      #Messages from the radio to the user
      fireangel_radio_messages:
        friendly_name: 'Messages'
        icon_template: mdi:message-processing
        value_template: >          
          {% if not '{' in states.sensor.fireangeldata.state %}
          {{states.sensor.fireangeldata.state}}
          {% endif %}
          
      #-----------------------------------    
      #2D8D01 | FP2620W2 | Smoke | Bedroom
      #-----------------------------------  
      #EVENT
      fireangel_event_2d8d01: 
        unique_id: 'fireangel_event_2d8d01'
        friendly_name: 'Bedroom Smoke Alarm'
        icon_template: mdi:fire
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='2d8d01'.upper() %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'MISSING' in states.sensor.fireangel_event_2d8d01.state %}
          {{ states.sensor.fireangel_event_2d8d01.state }}
          {% elif states.sensor.fireangeldata.attributes.event=='SILENCE' %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'EMERGENCY' in states.sensor.fireangel_event_2d8d01.state %}
          {{ states.sensor.fireangel_event_2d8d01.state }}
          {% endif %}
      #BATTERY
      fireangel_battery_2d8d01:
        unique_id: 'fireangel_battery_2d8d01'
        friendly_name: 'Bedroom Smoke Alarm'
        icon_template: >
          {% if states.sensor.fireangel_battery_2d8d01.state=='OK' %}
          {{'mdi:battery'}}
          {% elif states.sensor.fireangel_battery_2d8d01.state=='unknown' %}
          {{'mdi:battery-unknown'}}
          {% else %}
          {{'mdi:battery-low'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='2d8d01'.upper() and states.sensor.fireangeldata.attributes.battery is defined %}
          {{ states.sensor.fireangeldata.attributes.battery }}
          {% else %}
          {{ states.sensor.fireangel_battery_2d8d01.state }}
          {% endif %}
      #BASE
      fireangel_onbase_2d8d01:
        unique_id: 'fireangel_onbase_2d8d01'
        friendly_name: 'Bedroom Smoke Alarm'
        icon_template: >
          {% if states.sensor.fireangel_onbase_2d8d01.state=='ON' %}
          {{'mdi:home'}}
          {% elif states.sensor.fireangel_onbase_2d8d01.state=='unknown' %}
          {{'mdi:help-circle'}}
          {% else %}
          {{'mdi:alert-circle'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='2d8d01'.upper() %}
          {{ states.sensor.fireangeldata.attributes.base }}
          {% else %}
          {{ states.sensor.fireangel_onbase_2d8d01.state }}
          {% endif %}
          
      #-----------------------------------  
      #A76F18 | FP1720W2-R | Heat | Kitchen
      #-----------------------------------  
      #EVENT
      fireangel_event_a76f18: 
        unique_id: 'fireangel_event_a76f18'
        friendly_name: 'Kitchen Heat Alarm'
        icon_template: mdi:fire
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='a76f18'.upper() %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'MISSING' in states.sensor.fireangel_event_a76f18.state %}
          {{ states.sensor.fireangel_event_a76f18.state }}
          {% elif states.sensor.fireangeldata.attributes.event=='SILENCE' %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'EMERGENCY' in states.sensor.fireangel_event_a76f18.state %}
          {{ states.sensor.fireangel_event_a76f18.state }}
          {% endif %}
      #BATTERY
      fireangel_battery_a76f18:
        unique_id: 'fireangel_battery_a76f18'
        friendly_name: 'Kitchen Heat Alarm'
        icon_template: >
          {% if states.sensor.fireangel_battery_a76f18.state=='OK' %}
          {{'mdi:battery'}}
          {% elif states.sensor.fireangel_battery_a76f18.state=='unknown' %}
          {{'mdi:battery-unknown'}}
          {% else %}
          {{'mdi:battery-low'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='a76f18'.upper() and states.sensor.fireangeldata.attributes.battery is defined %}
          {{ states.sensor.fireangeldata.attributes.battery }}
          {% else %}
          {{ states.sensor.fireangel_battery_a76f18.state }}
          {% endif %}
      #BASE
      fireangel_onbase_a76f18:
        unique_id: 'fireangel_onbase_a76f18'
        friendly_name: 'Kitchen Heat Alarm'
        icon_template: >
          {% if states.sensor.fireangel_onbase_a76f18.state=='ON' %}
          {{'mdi:home'}}
          {% elif states.sensor.fireangel_onbase_a76f18.state=='unknown' %}
          {{'mdi:help-circle'}}
          {% else %}
          {{'mdi:alert-circle'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='a76f18'.upper() %}
          {{ states.sensor.fireangeldata.attributes.base }}
          {% else %}
          {{ states.sensor.fireangel_onbase_a76f18.state }}
          {% endif %}
		  
      #-----------------------------------  
      #AD8003 | W2-CO-10X | CO | Kitchen
      #-----------------------------------  
      #EVENT
      fireangel_event_ad8003: 
        unique_id: 'fireangel_event_ad8003'
        friendly_name: 'Kitchen CO Alarm'
        icon_template: mdi:molecule-co
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='ad8003'.upper() %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'MISSING' in states.sensor.fireangel_event_ad8003.state %}
          {{ states.sensor.fireangel_event_ad8003.state }}
          {% elif states.sensor.fireangeldata.attributes.event=='SILENCE' %}
          {{ states.sensor.fireangeldata.attributes.event }}
          {% elif 'EMERGENCY' in states.sensor.fireangel_event_ad8003.state %}
          {{ states.sensor.fireangel_event_ad8003.state }}
          {% endif %}
      #BATTERY
      fireangel_battery_ad8003:
        unique_id: 'fireangel_battery_ad8003'
        friendly_name: 'Kitchen CO Alarm'
        icon_template: >
          {% if states.sensor.fireangel_battery_ad8003.state=='OK' %}
          {{'mdi:battery'}}
          {% elif states.sensor.fireangel_battery_ad8003.state=='unknown' %}
          {{'mdi:battery-unknown'}}
          {% else %}
          {{'mdi:battery-low'}}
          {% endif %}       
        value_template: >
          {% if states.sensor.fireangeldata.attributes.device=='ad8003'.upper() and states.sensor.fireangeldata.attributes.battery is defined %}
          {{ states.sensor.fireangeldata.attributes.battery }}
          {% else %}
          {{ states.sensor.fireangel_battery_ad8003.state }}
          {% endif %}   
 ```


To talk back to the bridge, you can add this configuration:

 ```yaml
#---------------------------------------------------------
#FireAngel shell commands (used to send data to the alarms)
#---------------------------------------------------------
shell_command:
   fireangel_test_co: /bin/bash -c "echo -e -n "1~" > /dev/ttyUSB0"
   fireangel_test_smoke: /bin/bash -c "echo -e -n "2~" > /dev/ttyUSB0"
   fireangel_test_all: /bin/bash -c "echo -e -n "3~" > /dev/ttyUSB0"
   fireangel_emergency_co: /bin/bash -c "echo -e -n "4~" > /dev/ttyUSB0"
   fireangel_emergency_smoke: /bin/bash -c "echo -e -n "5~" > /dev/ttyUSB0" 
   fireangel_silence_co: /bin/bash -c "echo -e -n "6~" > /dev/ttyUSB0"    
   fireangel_silence_smoke: /bin/bash -c "echo -e -n "7~" > /dev/ttyUSB0"    
   fireangel_get_pairing: /bin/bash -c "echo -e -n "8~" > /dev/ttyUSB0"
   fireangel_start_pairing: /bin/bash -c "echo -e -n "9~" > /dev/ttyUSB0"
 ```   
   
Now you can start adding some entities to your dashboard.

Here's my lovelace configuration. In case you want to borrow anything from it:
 ```yaml
  - title: FireAngel
    path: fireangel
    panel: false
    theme: ''
    badges: []
    cards:
      - type: entities
        entities:
          - entity: sensor.fireangel_event_13f43e
            secondary_info: last-updated
          - entity: sensor.fireangel_event_06f93e
            secondary_info: last-updated
          - entity: sensor.fireangel_event_2d8d01
            secondary_info: last-updated
          - entity: sensor.fireangel_event_a76f18
            secondary_info: last-changed
          - entity: sensor.fireangel_event_ad8003
            secondary_info: last-changed
          - entity: sensor.fireangel_event_11ce01
            secondary_info: last-updated
        title: Alarm Events
        show_header_toggle: true
        state_color: false
      - type: entities
        entities:
          - entity: sensor.fireangel_battery_13f43e
          - entity: sensor.fireangel_battery_06f93e
          - entity: sensor.fireangel_battery_2d8d01
          - entity: sensor.fireangel_battery_a76f18
          - entity: sensor.fireangel_battery_ad8003
          - entity: sensor.fireangel_battery_11ce01
        title: Battery Status
      - type: entities
        entities:
          - entity: sensor.fireangel_onbase_13f43e
          - entity: sensor.fireangel_onbase_06f93e
          - entity: sensor.fireangel_onbase_2d8d01
          - entity: sensor.fireangel_onbase_a76f18
        show_header_toggle: true
        state_color: false
        title: Base Status
      - type: entities
        entities:
          - entity: sensor.fireangel_radio_heartbeat
          - entity: sensor.fireangel_radio_messages
          - entity: sensor.fireangeldata
        title: Gateway Information
        state_color: false
      - name: Check Radio Pairing
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_get_pairing
        type: button
        icon: mdi:signal-variant
        show_state: false
        icon_height: 50px
      - name: Initiate Radio Pairing
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_start_pairing
        type: button
        icon: mdi:signal-distance-variant
        show_state: true
        icon_height: 50px
      - name: Test Fire
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_test_smoke
        type: button
        icon: mdi:fire
        show_state: false
        icon_height: 50px
      - name: Test Carbon-monoxide
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_test_co
        type: button
        icon: mdi:molecule-co
        show_state: false
        icon_height: 50px
      - name: Silence Alarms
        show_icon: true
        show_name: true
        tap_action:
          action: call-service
          service: shell_command.fireangel_silence_smoke
        type: button
        icon: mdi:volume-off
        show_state: false
        icon_height: 50px
  - title: Bedroom
    path: bedroom
    badges: []
    cards:
      - type: button
        tap_action:
          action: toggle
        entity: switch.wardrobe_light_power
        icon: hass:lightbulb
      - type: button
        tap_action:
          action: toggle
        entity: switch.wardrobe_light_brighter
        icon: hass:brightness-5
      - type: button
        tap_action:
          action: toggle
        entity: switch.wardrobe_light_dimmer
        icon: hass:brightness-4
      - type: button
        tap_action:
          action: toggle
        entity: switch.main_light_power
        icon: hass:lightbulb
      - type: media-control
        entity: media_player.bedroom_speaker
 ```

At this point, you probably want to make sure that you at least an entities card on your dashboard called 'Gateway Information' and include: 
* sensor.fireangel_radio_heartbeat 
* sensor.fireangel_radio_messages
* sensor.fireangeldata

If the bridge is communicating with HA, you should see that the heartbeat reports 'online'.
If it isn't working, check the USB connection and configuration.

Next, add yourself some buttons for each of the shell commands you're interested in.
The buttons just call services. e.g. service shell_command.fireangel_start_pairing
Again, see my lovelace configuration for inspiration.

Now you'll want to pair your bridge with your WiSafe network.
Depending where you got your radio from, this may or may not be paired already. 

If you used a new (or reset) radio, you can place the bridge into pairing mode, by pressing the button you created on the dashboard for pairing.
Once in pairing mode, press the physical button one one of your alarms (as normal). The messages sensor will show the result.

If you find the network is already paired (because your radio module is 2nd hand), then you should reset it with a double push of the physical button on the back of the radio.
See FireAngel's instructions for unpairing an alarm from the network.
Use the 'Check Radio Pairing' dashboard button and observe the message to confirm that the radio is unpaired, then you can proceed to pair with your network as previously described.

At this point, when you test any alarm in your network, sensor.fireangeldata will show you the ID of the alarm. Take this ID to create the sensors for each alarm, based on my sample configuration.
Note: Use lower-case when using the ID in the HA configuration file.

Add sensors for each of your alarms, and you're pretty much done.

## Value Add
For the full experiance, I recommend making sure you have TTS (Google/Alexa) and Mobile Phone notifications setup and working. I won't go into detail here, as this is just HA stuff, and not specific to this project. 
But I'll quickly share the congiguration I use for goole wavenet:

 ```yaml
tts:
  - platform: google_cloud
    base_url: http://192.168.0.200:8123
    key_file: googlecloud.json
    voice: en-GB-Wavenet-B
    encoding: linear16
    speed: 0.9
    pitch: -1.0
 ```
 
And for mobile Phone notifications:

 ```yaml
notify:
  - platform: group
    name: all_mobile_app_devices
    services:
      - service: mobile_app_iphone_1
      - service: mobile_app_iphone_2
 ```

With that, I use these automations (automations.yaml) to trigger alerts and notifications:
I receive notifications if the gateway goes offline, as well as any alarm events.
I also have the events configured as 'critical', so that my phone will make a sound, even if it is on silent.

 ```yaml
- id: '1620428541168'
  alias: FireAngel Test Fail Notifications
  description: FireAngel Test Fail Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
  - platform: state
    entity_id: sensor.fireangel_event_601a03
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
  condition:
  - condition: template
    value_template: '{{ ''TEST'' in trigger.to_state.state }}'
  - condition: template
    value_template: '{{ states.sensor.fireangeldata.attributes.result==''FAIL''}}'
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-test-notification
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        There is a fault with the alarm! Check or replace it as necessary. See the
        user manual for full details.
      title: FireAngel Alarm Test
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        There is a fault with the alarm. Check or replace it as necessary.
  mode: single
- id: '1620428541169'
  alias: FireAngel Test Pass Notifications
  description: FireAngel Test Pass Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
  - platform: state
    entity_id: sensor.fireangel_event_601a03
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
  - platform: state
    entity_id: sensor.fireangel_event_a5b813
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
  condition:
  - condition: template
    value_template: '{{ ''TEST'' in trigger.to_state.state }}'
  - condition: template
    value_template: '{{ states.sensor.fireangeldata.attributes.result==''PASS''}}'
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-test-notification
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        The alarm is functioning normally.
      title: FireAngel Alarm Test
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: A {{ trigger.to_state.state }} event was triggered by the {{ trigger.to_state.attributes.friendly_name}}.
        This is only a test. The alarm is functioning normally.
  mode: single
- id: '1620428541170'
  alias: FireAngel Base Notifications
  description: FireAngel Base Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_onbase_13f43e
    to: 'OFF'
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_onbase_2d8d01
    to: 'OFF'
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_onbase_06f93e
    to: 'OFF'
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_onbase_a76f18
    to: 'OFF'
    for: '1'
  condition: []
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-off-base-notification
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} was removed
        from its base
      title: FireAngel Alarm Off Base
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} was removed
        from its base
  mode: single
- id: '1620428541171'
  alias: FireAngel Emergency Notifications
  description: FireAngel Emergency Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
  - platform: state
    entity_id: sensor.fireangel_event_601a03
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
  - platform: state
    entity_id: sensor.fireangel_event_a5b813
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
  condition:
  - condition: template
    value_template: '{{ ''EMERGENCY'' in trigger.to_state.state }}'
  action:
  - data_template:
      data:
        push:
          sound:
            name: default
            critical: 1
            volume: 1.0
      message: WARNING! A {{ trigger.to_state.state }} has been reported by the {{
        trigger.to_state.attributes.friendly_name}}. Take action now!
      title: FireAngel Alarm Emergency
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: WARNING! WARNING! A {{ trigger.to_state.state }} has been reported
        by the {{ trigger.to_state.attributes.friendly_name}}. Take action now! This
        is not a test. Repeat. A {{ trigger.to_state.state }} has been reported by
        the {{ trigger.to_state.attributes.friendly_name}}. Take action now! This
        is not a test.
  mode: single
- id: '1620428541172'
  alias: FireAngel Offline Notifications
  description: FireAngel Offline Notifications
  trigger:
  - platform: state
    entity_id: sensor.fireangel_radio_heartbeat
    to: Offline!
    for: '35'
  condition: []
  action:
  - data_template:
      data:
        push:
          sound:
            name: default
            critical: 1
            volume: 1.0
      message: WARNING! The FireAngel Gateway is Offline
      title: FireAngel Gateway Offline
    service: notify.all_mobile_app_devices
  mode: single
- id: '1620428541173'
  alias: FireAngel Missing Notifications
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.fireangel_event_13f43e
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_2d8d01
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_06f93e
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_601a03
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_11ce01
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_a76f18
    to: MISSING
    for: '1'
  - platform: state
    entity_id: sensor.fireangel_event_ad8003
    to: MISSING
    for: '1'
  condition: []
  action:
  - data_template:
      data:
        apns_headers:
          apns-collapse-id: fireangel-missing-notification
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} has beeen
        reported as missing by other alarms in the network
      title: FireAngel Alarm Off Base
    service: notify.all_mobile_app_devices
  - service: tts.google_cloud_say
    data:
      entity_id: media_player.broadcast_group
      message: Warning! The {{ trigger.to_state.attributes.friendly_name }} has beeen
        reported as missing by other alarms in the network 
 ```




Of course, no one template will serve all projects since your needs may be different. So I'll be adding more in the near future. You may also suggest changes by forking this repo and creating a pull request or opening an issue. Thanks to all the people have contributed to expanding this template!

Use the `BLANK_README.md` to get started.

<p align="right">(<a href="#top">back to top</a>)</p>



### Built With

This section should list any major frameworks/libraries used to bootstrap your project. Leave any add-ons/plugins for the acknowledgements section. Here are a few examples.

* [Next.js](https://nextjs.org/)
* [React.js](https://reactjs.org/)
* [Vue.js](https://vuejs.org/)
* [Angular](https://angular.io/)
* [Svelte](https://svelte.dev/)
* [Laravel](https://laravel.com)
* [Bootstrap](https://getbootstrap.com)
* [JQuery](https://jquery.com)

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

This is an example of how you may give instructions on setting up your project locally.
To get a local copy up and running follow these simple example steps.

### Prerequisites

This is an example of how to list things you need to use the software and how to install them.
* npm
  ```sh
  npm install npm@latest -g
  ```

### Installation

_Below is an example of how you can instruct your audience on installing and setting up your app. This template doesn't rely on any external dependencies or services._

1. Get a free API Key at [https://example.com](https://example.com)
2. Clone the repo
   ```sh
   git clone https://github.com/your_username_/Project-Name.git
   ```
3. Install NPM packages
   ```sh
   npm install
   ```
4. Enter your API in `config.js`
   ```js
   const API_KEY = 'ENTER YOUR API';
   ```

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage

Use this space to show useful examples of how a project can be used. Additional screenshots, code examples and demos work well in this space. You may also link to more resources.

_For more examples, please refer to the [Documentation](https://example.com)_

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ROADMAP -->
## Roadmap

- [x] Add Changelog
- [x] Add back to top links
- [ ] Add Additional Templates w/ Examples
- [ ] Add "components" document to easily copy & paste sections of the readme
- [ ] Multi-language Support
    - [ ] Chinese
    - [ ] Spanish

See the [open issues](https://github.com/othneildrew/Best-README-Template/issues) for a full list of proposed features (and known issues).

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE.txt` for more information.

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Your Name - [@your_twitter](https://twitter.com/your_username) - email@example.com

Project Link: [https://github.com/your_username/repo_name](https://github.com/your_username/repo_name)

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

Use this space to list resources you find helpful and would like to give credit to. I've included a few of my favorites to kick things off!

* [Choose an Open Source License](https://choosealicense.com)
* [GitHub Emoji Cheat Sheet](https://www.webpagefx.com/tools/emoji-cheat-sheet)
* [Malven's Flexbox Cheatsheet](https://flexbox.malven.co/)
* [Malven's Grid Cheatsheet](https://grid.malven.co/)
* [Img Shields](https://shields.io)
* [GitHub Pages](https://pages.github.com)
* [Font Awesome](https://fontawesome.com)
* [React Icons](https://react-icons.github.io/react-icons/search)

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/othneildrew/Best-README-Template.svg?style=for-the-badge
[contributors-url]: https://github.com/othneildrew/Best-README-Template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/othneildrew/Best-README-Template.svg?style=for-the-badge
[forks-url]: https://github.com/othneildrew/Best-README-Template/network/members
[stars-shield]: https://img.shields.io/github/stars/othneildrew/Best-README-Template.svg?style=for-the-badge
[stars-url]: https://github.com/othneildrew/Best-README-Template/stargazers
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/othneildrew/Best-README-Template/issues
[license-shield]: https://img.shields.io/github/license/othneildrew/Best-README-Template.svg?style=for-the-badge
[license-url]: https://github.com/othneildrew/Best-README-Template/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/othneildrew
[product-screenshot]: images/screenshot.png
