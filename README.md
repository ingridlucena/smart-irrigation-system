[irrigação_inteligente.cpp](https://github.com/user-attachments/files/29403676/irrigacao_inteligente.cpp)

Para a versão em Português, você já está aqui.  
For the English version, click [here](README.en.md).

---

# Sistema Inteligente de Monitoramento e Irrigação Automatizada

Este projeto consiste no desenvolvimento do protótipo de um sistema automatizado para controle, leitura de sensores e monitoramento de um reservatório de água e irrigação de solo. O objetivo principal é a aplicação prática de conceitos de automação, eficiência hídrica e manipulação de hardware.

## Funcionamento do Sistema
O sistema opera em malha fechada monitorando continuamente duas variáveis principais para tomar decisões de controle em tempo real:
* **Monitoramento do Reservatório:** O sensor ultrassônico mede a distância até a lâmina d'água em centímetros. O algoritmo processa esses dados para calcular e exibir na tela TFT a capacidade volumétrica útil do reservatório (como 75% ou 100% de sua capacidade máxima).
* **Controle de Irrigação:** O sensor de umidade do solo monitora constantemente o nível de secura da terra. Quando a umidade cai abaixo do limite crítico programado, o microcontrolador aciona o módulo relé para ligar a bomba de água. Assim que o solo atinge a umidade ideal, o sistema corta o sinal do relé, interrompendo a irrigação para evitar o desperdício de água.
* **Dados Ambientais:** O sensor DHT faz a leitura secundária da temperatura e da umidade ativa do ar ambiente para fins de monitoramento no display.

## Características do Software
Diferente da estrutura convencional de scripts da IDE do Arduino, o software foi desenvolvido explorando a arquitetura nativa do microcontrolador em C/C++:
* **Manipulação de Registradores:** Configuração manual e direta das portas de entrada e saída através dos registradores de direção de dados (`DDRC` e `DDRD`) e escrita de estados (`PORTD` e `PORTC`).
* **Estrutura Clássica:** Implementação utilizando a função principal `int main(void)` e um loop infinito de controle `while(1)`, otimizando o fluxo de execução e o tempo de resposta do chip.

## Hardware Utilizado
* **Microcontrolador:** ATmega328P (Plataforma Arduino)
* **Interface Visual:** Tela TFT 1.8" (Driver ST7735)
* **Sensores:** Sensor de Umidade do Solo Capacitivo, Sensor Ultrassônico e Sensor de Temperatura e Umidade Ambiente (DHT)
* **Atuador:** Módulo Relé para acionamento do sistema de bombagem

## Esquema Eletrónico
O diagrama completo com todas as conexões elétricas, pinagens do ATmega328P e interligações de todos os sensores e módulos está disponível para consulta:

* 📄 [Clique aqui para abrir o PDF do Esquematico Completo](esquematico_irrigacao_inteligente.pdf)

## Contexto Acadêmico
Projeto desenvolvido como atividade prática para o curso de Engenharia de Controle e Automação na UFRPE - UABJ, servindo como registro técnico e relatório de aplicação prática de conceitos de programação e circuitos eletrônicos.

---


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
