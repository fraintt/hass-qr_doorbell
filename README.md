# HomeAssistant - qr_doorbell entry
A QR-Code Generator for HASS video doorbell gate opening

This is a short description about a solution I created for myself in order to let AirBnb Guests inside my property only with a QR-Code. 


Requirements: 
- Video Doorbell, with camera feed in HomeAssistant (camera Entity in HASS)
- Magnetic or Automatic Gate controlled by HASS (I have tasmota-relay wired into my automatic gate which "pushes" the button for the gate to open)

Helpers: 
 - input_text.qr_code_text

'''
shell_command:
  install_zbar: ssh -i /config/ssh/hass-asus-key.ppk -o 'StrictHostKeyChecking=no' pi@10.0.0.10 'sudo sh ./install_zbar.sh'
  generate_qr: ssh -i /config/ssh/hass-asus-key.ppk -o 'StrictHostKeyChecking=no' pi@10.0.0.10 'sudo sh ./qr_generate.sh'

image_processing:
  - platform: qrcode
    source:
     - entity_id: camera.doorbell
	 
binary_sensor:	 
  - platform: template
    sensors:
      qr_match:
        friendly_name: "QR Code Scan match"
        value_template: "{{(states.input_text.qr_code_text.state) == (states.image_processing.qr_doorbell.state) }}"
'''
