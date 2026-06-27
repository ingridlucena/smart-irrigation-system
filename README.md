[irrigação_inteligente.cpp](https://github.com/user-attachments/files/29403676/irrigacao_inteligente.cpp)


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
        tft.print("MONITORAMENTO");
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
              tft.print("distancia:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Capacidade abaixo de 10%");
              tone(buzzer, 4000);
              distance=ultrasonic.read();
            } 
            noTone(buzzer);

            desenharCabecalho();

            if(distance<=13 && distance>10.5 ){
              PORTC=0b00000001;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distancia:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Reservatorio em 10% de sua capacidade.");
            }

            else if(distance<=10.5 && distance >9){
              PORTC=0b00000011;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distancia:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Reservatorio em 25% de sua capacidade.");
            }
            else if(distance<=9 && distance>5.5){
             PORTC=0b00000111;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distancia:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Reservatorio em 50% de sua capacidade.");
            }
            else if(distance<=5.5 && distance>2){
              PORTC=0b00001111;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distancia:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Reservatorio em 75% de sua capacidade.");
            }
            else if(distance<=3.8){
              PORTC=0b00011111;
              tft.setCursor(0,30);
              tft.setTextColor(ST7735_BLUE,ST7735_BLACK);
              tft.print("distancia:");
              tft.print(distance);
              tft.println("cm");
              tft.println("Reservatorio em 100% de sua capacidade.");
            }
        
            float hum = dht.readHumidity();
            float temp = dht.readTemperature();

            tft.setCursor(0,60);
            tft.setTextColor(ST7735_YELLOW,ST7735_BLACK);
            tft.print("UR do ar:");
            tft.print(hum);
            tft.println("%");
            tft.setCursor(0,70);
            tft.setTextColor(ST7735_RED,ST7735_BLACK);
            tft.print("Temperatura:");
            tft.print(temp);
            tft.println(" C");


            soil = map(analogRead(5), 1023, 0, 0, 100);
            soil = constrain(soil, 0, 100);
        

            tft.setCursor(0,80);
            tft.setTextColor(ST7735_GREEN,ST7735_BLACK);
            tft.print("Umidade do solo:");
            tft.print(soil);
            tft.println("%");

            if(soil<58){

                PORTD &= ~(1<<7);

                tft.setCursor(0,90);
                tft.setTextColor(ST7735_CYAN,ST7735_BLACK);
                tft.print("STATUS DA BOMBA: ");
                tft.println("ligada");
                delay(1000);

    
                PORTD |= (1<<7);

            }
            else{
            tft.setCursor(0,90);
            tft.setTextColor(ST7735_CYAN,ST7735_BLACK);
            tft.print("STATUS DA BOMBA: ");
            tft.println("desligada");

            }
      
        
        }

    }
