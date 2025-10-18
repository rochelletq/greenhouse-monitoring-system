# greenhouse temperature monitoring system
A temperature and pressure monitoring system with real-time data display built with Arudino
## features 
- real-time data collection using the BMP180 temperature and pressure sensor
- OLED display - for visual display of temperature, pressure and lid status
- servo controlled lid - opens and closes automatically depending on set temperarure thresholds
- manual override - push button used for lid control
- audio feeback - a buzzer when the lip opens

## tech stack
  - microcontroller: Arduino Uno
  - programming language: C++ (Arduino)
  - components: BMP180 Sensor (temperature and presssure), OLED display (128x64, I2C), servo motor, push button, passive buzzer
    
## how it works
- reads temperature and presssure from the BMP180 sensor
- displays reading on the OLED display
- if the temperature exceeds 23°C the lid opens automatically and triggers the buzzer
- when the temperature drops below 23°C the lid closes
- psuh button allows for manual togging of the lid
- servo moves gradually for smooth mechanisms 

## futher improvments 
- add data for humidity
- add bluettoth for an app for user interface

## developed by Rochelle Thompson-Quartey for a first year engineering project (dec 2024 - april 2025)
