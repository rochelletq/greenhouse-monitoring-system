
#include <Wire.h>
#include <SPI.h>
#include <Adafruit_BMP085.h> // for the BMP180 sensor
#include <Adafruit_GFX.h> // graphics library for OLED
#include <Adafruit_SSD1306.h> // OLED display library 
#include <Servo.h> // controlling servo motor 

//OLED display settings 
#define SCREEN_WIDTH 128 // OLED display width in pixels 
#define SCREEN_HEIGHT 64 // OLED display height in pixels
#define SERVO_PIN 11 // servo connected to pin 11
#define BUTTON_PIN 9 // push button connected to pin 9
#define BUZZER_PIN 4 // passive buzzer connected to pin 9
#define SEALEVELPRESSURE_HPA (101500) // sea level pressure 

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
Adafruit_BMP085 bmp; //BMP sesnor object 
Servo lidServo; // servo motor object 
bool isLidOpen = false; // Track lid status
bool lastButtonState = HIGH; // stores the last button state

// Button debounce variables


void setup() {
  Serial.begin(9600);
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("Failed to access OLED...");
    for (;;); // stops execution if the OLED fails 
  }

  delay(500); // delay for OLED to power up 
  display.clearDisplay(); // clear previous data from the display 

  bmp.begin(); //initlaise BMP180 sensor 

  lidServo.attach(SERVO_PIN); // attach servo motor to its pin 
  lidServo.write(0); // Ensure lid starts closed

  pinMode(BUTTON_PIN, INPUT_PULLUP); // set putton pin as an input with internal pull up
  pinMode(BUZZER_PIN, OUTPUT); // set buzzer pin as an output 
}

void loop() {
  bool currentButtonState = digitalRead(BUTTON_PIN);// read the current button state 

if (lastButtonState == HIGH && currentButtonState ==`= LOW) {// detect a button press 

    if (isLidOpen) {
      moveServo(0); // Close the lid if open
      isLidOpen = false;
      Serial.println("Lid closed");
      noTone(BUZZER_PIN);
      
    }
    else {
      moveServo(80); // if lid is closed, open the lid
      isLidOpen = true;
      Serial.println("Lid opened");
      tone(BUZZER_PIN,500);// turn on buzzer 
      delay(2500);// buzzer beeps for 2.5 seconds 
      noTone(BUZZER_PIN);// turn off buzzer 
      
    }
    delay(300);  // Prevent button bounce
  }
  lastButtonState == currentButtonState; // save thecurrent button state

  // Read temperature and pressure values from sensor 
  float temperature = bmp.readTemperature();
  float pressure = bmp.readPressure() / 100.0; // Convert to hPa

  // Smooth servo movement control based on temperature
  if (temperature >= 23.0 && !isLidOpen) {
    moveServo(80); // Open lid gradually
    isLidOpen = true;
    tone(BUZZER_PIN, 500);
    delay(2500);
    noTone(BUZZER_PIN);
  } else if (temperature < 23.0 && isLidOpen) {
    moveServo(0); // Close lid gradually
    isLidOpen = false;
    noTone(BUZZER_PIN);
  }

  // Update OLED Display with sensor data and lid status 
  updateDisplay(temperature, pressure, isLidOpen);

  // Print to Serial Monitor
  Serial.print("Temperature: ");
  Serial.print(temperature, 1);
  Serial.print(" C | Pressure: ");
  Serial.print(pressure, 1);
  Serial.print(" hPa | Lid: ");
  Serial.println(isLidOpen ? "OPEN" : "CLOSED");

  delay(1000); // delay before next loop 
}

// Function to gradually move the servo
void moveServo(int targetAngle) {
  int currentAngle = lidServo.read();
  while (currentAngle != targetAngle) {
    currentAngle += (currentAngle < targetAngle) ? 1 : -1;
    lidServo.write(currentAngle);
    delay(15); // Small delay for smooth movement
  }
}

// Function to update OLED display with sensor data
void updateDisplay(float temp, float press, bool lidOpen) {
  display.clearDisplay(); // clear the display memory 
  
  display.setTextSize(1);// set text size 
  display.setTextColor(WHITE);// set text colour 

  display.setCursor(0, 19);
  display.print("Temperature: ");
  display.print(temp, 1);
  display.print(" C"); // display temperature in degrees

  display.setCursor(0, 35);
  display.print("Pressure: ");
  display.print(press, 1);
  display.print(" hPa");// display pressure in hPa 

  display.setCursor(0, 50);  // Adjusted to avoid overlapping text
  display.print("Lid: ");
  display.print(lidOpen ? "OPEN" : "CLOSED");

  display.display();
}
