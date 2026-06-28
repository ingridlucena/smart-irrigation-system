For the Portuguese version, click [here](README.md).  
For the English version, you are already here.

---

# Intelligent Monitoring and Automated Irrigation System 

This project consists of developing a prototype for an automated system to control, read sensors, and monitor a water reservoir and soil irrigation. The main objective is the practical application of automation, water efficiency, and low-level hardware manipulation concepts.

## System Operation
The system operates in a closed loop, continuously monitoring two main variables to make real-time control decisions:
* **Reservoir Monitoring:** The ultrasonic sensor measures the distance to the water level in centimeters. The algorithm processes this data to calculate and display the volumetric capacity of the reservoir (such as 75% or 100% of its maximum capacity) on the TFT screen.
* **Irrigation Control:** The capacitive soil moisture sensor constantly monitors how dry the soil is. When the moisture drops below the programmed critical threshold, the microcontroller triggers the relay module to turn on the water pump. As soon as the soil reaches ideal moisture, the system cuts the relay signal, stopping the irrigation to prevent water waste.
* **Environmental Data:** The DHT sensor performs secondary readings of ambient temperature and relative humidity for monitoring purposes on the display.

## Software Features
Unlike the conventional high-level scripts from the Arduino IDE, the software was developed by exploring the microcontroller's native architecture in pure C/C++:
* **Register Manipulation:** Manual and direct configuration of input and output ports through data direction registers (`DDRC` and `DDRD`) and direct state writing (`PORTD` and `PORTC`).
* **Classic Architecture:** Implementation using the `int main(void)` main function and a `while(1)` infinite control loop, optimizing execution flow and chip response time.

## Hardware Used
* **Microcontroller:** ATmega328P (Arduino Platform)
* **Visual Interface:** 1.8" TFT Screen (ST7735 Driver)
* **Sensors:** Capacitive Soil Moisture Sensor, Ultrasonic Sensor, and Ambient Temperature and Humidity Sensor (DHT)
* **Actuator:** Relay Module to trigger the water pump system

## Electronic Schematic
The complete diagram containing all electrical connections, ATmega328P pinouts, and interconnections for all sensors and modules is available for reference:

* 📄 [Click here to open the Complete Schematic PDF](esquematico_irrigacao_inteligente.pdf)

## Academic Context
Project developed as a practical activity for the Control and Automation Engineering course at UFRPE - UABJ, serving as a technical record and practical application report of programming and electronic circuit concepts.

    #include "DHT.h"
    #include <Adafruit_GFX.h> 
    #include <Adafruit_ST7735.h>  
    #include <SPI.h>
    #include <Ultrasonic.h>
    
    #define TFT_CS  10
    #define TFT_DC  9
    #define TFT_RST 8 
    
    Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_RST);
    
    #define DHTPIN 2        
    #define DHTTYPE DHT22  
    DHT dht(DHTPIN, DHTTYPE);
    #define SOIL A5
    Ultrasonic ultrasonic(3,4);
    
    #define buzzer 5
    #define rele 7
    
    float distance;
    float soil;
    
    
    void desenharCabecalho() {
        tft.fillRect(0, 0, 160, 20, ST7735_RED);
        tft.setCursor(25, 6);
        tft.setTextColor(ST7735_WHITE);
        tft.setTextSize(1);
        tft.print("MONITORING");
    }
    
    int main(void) {
    
        init(); 
        DDRC = 0b00011111;
        DDRD = 0b10101000;
        PORTD |= (1<<7);
        dht.begin();
        SPI.begin(); 
        tft.initR(INITR_BLACKTAB);
        tft.setRotation(1);
        tft.fillScreen(ST7735_BLACK);
    
    
        while (1) {
    
            distance=ultrasonic.read();
    
             while(distance>12.9){
              PORTC = 0b00000001;
              delay(500);
              PORTC =0b00000000;
              tft.fillScreen(ST7735_BLACK);
              desenharCabecalho();
              tft.setCursor(0,50);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distance:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Capacity below 10%");
              tone(buzzer, 4000);
              distance=ultrasonic.read();
            } 
            noTone(buzzer);
    
            desenharCabecalho();
    
            if(distance<=13 && distance>10.5 ){
              PORTC=0b00000001;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distance:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Tank at 10% capacity.");
            }
    
            else if(distance<=10.5 && distance >9){
              PORTC=0b00000011;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distance:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Tank at 25% capacity.");
            }
            else if(distance<=9 && distance>5.5){
             PORTC=0b00000111;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distance:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Tank at 50% capacity.");
            }
            else if(distance<=5.5 && distance>2){
              PORTC=0b00001111;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distance:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Tank at 75% capacity.");
            }
            else if(distance<=3.8){
              PORTC=0b00011111;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distance:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Tank at 100% capacity.");
            }
        
            float hum = dht.readHumidity();
            float temp = dht.readTemperature();
    
            tft.setCursor(0,60);
            tft.setTextColor(ST7735_YELLOW,ST7735_BLACK);
            tft.print("Air humidity:");
            tft.print(hum);
            tft.println("%");
            tft.setCursor(0,70);
            tft.setTextColor(ST7735_RED,ST7735_BLACK);
            tft.print("Temperature:");
            tft.print(temp);
            tft.println(" C");
    
    
            soil = map(analogRead(5), 1023, 0, 0, 100);
            soil = constrain(soil, 0, 100);
        
    
            tft.setCursor(0,80);
            tft.setTextColor(ST7735_GREEN,ST7735_BLACK);
            tft.print("Soil moisture:");
            tft.print(soil);
            tft.println("%");
    
            if(soil<58){
    
                PORTD &= ~(1<<7);
    
                tft.setCursor(0,90);
                tft.setTextColor(ST7735_CYAN,ST7735_BLACK);
                tft.print("PUMP STATUS: ");
                tft.println("on");
                delay(1000);
    
    
                PORTD |= (1<<7);
    
            }
            else{
            tft.setCursor(0,90);
            tft.setTextColor(ST7735_CYAN,ST7735_BLACK);
            tft.print("PUMP STATUS: ");
            tft.println("off");
    
            }
      
        
        }
    
    }
