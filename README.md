# lampara-iot
Código de la lámpara IoT en Arduino Cloud para ESP8266

#include <Wire.h>

#include <Adafruit_GFX.h>
#include <Adafruit_GrayOLED.h>
#include <Adafruit_SPITFT.h>
#include <Adafruit_SPITFT_Macros.h>
#include <gfxfont.h>

#include <Adafruit_SSD1306.h>
#include <splash.h>

#include "thingProperties.h"

Adafruit_SSD1306 display(128, 64, &Wire, -1); // instanciamos la pantalla oled

int led_interno = 2;
int LED_PIN = 14;
int diferencia = 0;
int threshold = 10;

void setup() {
  // conectar a arduino cloud
  Serial.begin(9600);
  delay(1500); 

  initProperties();
  
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);
  
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();

  pinMode(led_interno, OUTPUT);
  pinMode(LED_PIN, OUTPUT);
  pinMode(A0,INPUT); // A0, fotorresistencia
  pinMode(12, OUTPUT); // zumbador

  // intensidad por defecto
  intensidad = 300;

  // iniciamos el I2C
  Wire.begin(D2,D1);

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.clearDisplay();
  display.setTextSize(3);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 10);
}

void loop() {
  ArduinoCloud.update();
  
  if(estado) {
    if(automatico) {
      intensidad = 700-(analogRead(A0)*3); // hemos experimentado con los valores y esto se ajusta a una buena cantidad de luz y variación
    }
    analogWrite(LED_PIN, intensidad);
  } else {
    analogWrite(LED_PIN, 0);
  }
  
  display.clearDisplay();
  display.setTextSize(4);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 10);
  time_t now = ArduinoCloud.getLocalTime();
  // sólo si la hora es válida
  if(now > 1000) {
    // representar la hora en la pantalla
    struct tm *timeInfo = localtime(&now);
    display.print(timeInfo->tm_hour);
    display.print(":");
    display.print(timeInfo->tm_min);

    // código de la alarma
    time_t alarmaTime = alarma;
    struct tm *alarmaInfo = localtime(&alarmaTime);
    if(hayAlarma) {
      if(timeInfo->tm_hour == alarmaInfo->tm_hour &&
      timeInfo->tm_min == alarmaInfo->tm_min) {
        for(int i=0;i<5;i++) {
          digitalWrite(12, HIGH);
          delay(20);
          digitalWrite(12, LOW);
          delay(40);
          digitalWrite(12, HIGH);
          delay(20);
          digitalWrite(12, LOW);
          delay(500);
        }
      }
    }
  } else {
    display.setTextSize(2);
    display.println("Conectando...");
  }
  display.display();
  
  delay(1000); // actualizar la hora cada segundo también significa actualizar la luz con el modo automático cada segundo (los cambios no son tan suaves)
  // en el caso de bajar el delay, la pantalla oled tiene problemas por refrescarse tan rápido
}

void onEstadoChange()  {
  digitalWrite(led_interno,!estado);
  digitalWrite(12, HIGH);
  delay(20);
  digitalWrite(12, LOW);
  delay(40);
  digitalWrite(12, HIGH);
  delay(20);
  digitalWrite(12, LOW);
}

void onIntensidadChange()  {
  // Add your code here to act upon Intensidad change
}

void onAutomaticoChange()  {
  // Add your code here to act upon Automatico change
}

void onAlarmaChange()  {
  // Add your code here to act upon Alarma change
}

void onHayAlarmaChange()  {
  // Add your code here to act upon HayAlarma change
}
