# HomeAssistant - qr_doorbell entry
A QR-Code Generator for HASS video doorbell gate opening

This is a short description about a solution I created for myself in order to let AirBnb Guests inside my property only with a QR-Code. 


Requirements: 
- Video Doorbell, with camera feed in HomeAssistant (camera Entity in HASS)
- Magnetic or Automatic Gate controlled by HASS (I have tasmota-relay wired into my automatic gate which "pushes" the button for the gate to open)
-  QR Installed on the System where the qr_generate.sh will be run. (https://pypi.org/project/qrcode/)

Helpers: 
 - input_text.qr_code_text

  ---------

 # Entries needed in configuration.yaml

```
shell_command:
  install_zbar: ssh -i /config/ssh/hass-asus-key.ppk -o 'StrictHostKeyChecking=no' pi@10.0.0.10 'sudo sh ./install_zbar.sh'
  generate_qr: ssh -i /config/ssh/hass-asus-key.ppk -o 'StrictHostKeyChecking=no' pi@10.0.0.10 'sudo sh ./qr_generate.sh'

# The shell commands are needed for:

# install_zbar:  Installing the zbar Package as this is needed for the QR-Code ImageProcessing Platform, check below for Docker Instructions.
#  https://www.home-assistant.io/integrations/qrcode/

# The above shell_command (install_zbar) is maybe only needed for non-docker users, 
# all othery may run the contents of the script only once, 
# and the zbar package will be permanently installed. 
# This is not the case for docker installations, see more further below. 

# generate_qr: A script which generates a random string of numbers, 
#  saves this string into a Input-Text in HASS, and 
#  generates qr-code image which is saved into the HASS Media Folder. 

image_processing:
  - platform: qrcode
    source:
     - entity_id: camera.doorbell
       name: qr_doorbell

# The ImageProcessing Integration which processes images from an camera, 
# the state of this entity is the String which is being "processed" from the camera.
	 
binary_sensor:	 
  - platform: template
    sensors:
      qr_match:
        friendly_name: "QR Code Scan match"
        value_template: "{{(states.input_text.qr_code_text.state) == (states.image_processing.qr_doorbell.state) }}"

# A binary sensor which changes to "true" if the input_text and image_processing equal. 

```

 ------- 
 # Automations

The following automation is triggered by the above mentioned binary sensor, sends me a notification when a QR-Code was read by the image processing platform and opens the gate. 

 ```
alias: Notification - QR Code Scanned
description: "Various actions when a qr-code is scanned"
trigger:
  - platform: state
    entity_id: image_processing.qr_doorbell
    from: unknown
    id: qr_read
  - platform: state
    entity_id:
      - binary_sensor.qr_match
    to: "on"
    id: qr_match
condition: []
action:
  - choose:
      - conditions:
          - condition: trigger
            id: qr_match
        sequence:
          - service: script.gate_open
            data: {}
            enabled: false
          - service: notify.mobile_app_iphone
            data:
              message: "Match: {{ states.image_processing.qr_doorbell.state }}"
              title: QR Code Scanned!
      - conditions:
          - condition: trigger
            id: qr_read
        sequence:
          - service: notify.mobile_app_iphone
            data:
              message: "Read: {{ states.image_processing.qr_doorbell.state }}"
              title: QR Code Scanned!
mode: single
 ```

 ONLY FOR Docker users !!

 As the standard docker-image of homeassistant does not come with zbar pre-installed, we need to manually re-install it after each update of the container. 

 For this I have created the following automation: 

 ```
alias: Install ZBAR
description: "Installs ZBAR when home-assistant start and entity is not defined"
trigger:
  - platform: homeassistant
    event: start
    id: hass_start
condition:
  - condition: template
    value_template: "{{ states.image_processing.qr_doorbell.state is not defined }}"
action:
  - delay:
      hours: 0
      minutes: 5
      seconds: 0
      milliseconds: 0
  - service: shell_command.install_zbar
    data: {}
  - service: notify.mobile_app_iphone
    data: null
    message: HASS Started.
mode: single


 ```

 ------- 
 # Scripts

 This script generates the QR-Code and sends me a notification with the QR-Code Picture in order for me to Screenshot it and send it to the Guest

```
alias: Generate QR Code
entity_id: generate_qr
sequence:
  - service: shell_command.generate_qr
    data: {}
  - delay:
      hours: 0
      minutes: 0
      seconds: 4
      milliseconds: 0
  - service: notify.mobile_app_iphone
    data:
      message: QR Code was generated !
      data:
        attachment:
          url: /media/local/token.png
          hide-thumbnail: false
mode: single
```
