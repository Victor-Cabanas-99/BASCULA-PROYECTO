# BASCULA
## Practica con un sensor de peso **HX711(5Kg)**, en una placa de desarrollo ESP32 y comunicado en Node-RED. 
Este repositorio muestra como podemos programar en una **ESP32** con el sensores **HX711(5Kg)** y mostrar los datos optenidos en **Node-RED** y en una **LCD de 12c**.

### Introducción
**Descripción:** 
La **ESP32** la utilizamos en un entorno de adquision de datos, en esta practica ocuparemos un **sensor de peso HX711(5Kg)** para medir el peso de alguna pieza, equipaje, cantidad de material, etc. Los datos optenidos seran visualizados en forma de grafica y como indicador en Node-RED, como a su vez en wokwi en una LCD de 12c y como indicadores visuales tres leds.

### Material Necesario
Para realizar esta practica necesitas lo siguiente:
- WOKWI
- Tarjeta ESP32
- Sensor de peso HX711 (5Kg)
- Node-RED
- LCD 12C
- RELAYS

### Requisitos previos:
Para poder usar este repositorio necesitas entrar a la plataforma WOKWI como tener instalado Node-RED.

### Instrucciones de preparación de entorno:
1.-Abrir la terminal de programación en **WOKWI** y colocar la siguente programación:
```
#include "HX711.h"
#include <LiquidCrystal_I2C.h>
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
const int led1 = 15;
const int led2 = 5;
const int led3 = 16;
#define BUILTIN_LED 2
const int DOUT=4; //pin 4 DT
const int CLK=2; //pin 2 SCK
#define I2C_ADDR    0x27 //pines 22=SCL 21=SDA 
#define LCD_COLUMNS 20
#define LCD_LINES   4

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "3.76.79.164"; //servidor local de la CMD public MQTT
String username_mqtt="DIXZO";
String password_mqtt="238";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

HX711 bascula;//declaramos la funcion bascula
LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(9600);
  bascula.begin(DOUT, CLK);
  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  lcd.init();
  lcd.backlight();
  Serial.println("No ponga ningun  objeto sobre la bascula");
  Serial.println("...");
  Serial.println("Listo para pesar");  

  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  
}

void loop() {
  
 if (bascula.wait_ready_timeout(1000)) {
    long leyendo = bascula.read();
    //Serial.print("leyendo sensor: ");//se lee para despues ajustar ESCALA=valor de lectura/valor real
    //Serial.println(leyendo); // ESCALA=2100/5Kg 
    bascula.set_scale(420); // Establecemos la escala (0.42 en gramos) (420 en kilogramos)
  } 
 //d=bascula.get_units(10);
 if (bascula.get_units(10) >=1 && bascula.get_units(10) <=2.8){ //1 a 2.79
  //secuencia de leds
  digitalWrite(led1, HIGH); 
  digitalWrite(led2, LOW);
  digitalWrite(led3, LOW);
 lcd.clear(); 
  lcd.setCursor(2, 0);
  lcd.print("peso:" + String(bascula.get_units(10),2) + "kg");
  lcd.setCursor(2, 1);
  lcd.print("pieza chica");
  delay(1000);
 }    
else if (bascula.get_units(10) >=2.8 && bascula.get_units(10) <=3.5){ //2.8 a 3.49
  //secuencia de leds
  digitalWrite(led1, HIGH);
  digitalWrite(led2, HIGH);
  digitalWrite(led3, LOW);
 lcd.clear(); 
  lcd.setCursor(2, 0);
  lcd.print("peso:" + String(bascula.get_units(10),2) + "kg");
  lcd.setCursor(2, 1);
  lcd.print("pieza mediana");
  delay(1000);
 } 
 else if (bascula.get_units(10) >=3.5 && bascula.get_units(10) <=4.6){ //3.5 a 4.6
  //secuencia de leds
  digitalWrite(led1, HIGH);
  digitalWrite(led2, HIGH);
  digitalWrite(led3, HIGH);
 lcd.clear(); 
  lcd.setCursor(2, 0);
  lcd.print("peso:" + String(bascula.get_units(10),2) + "kg");
  lcd.setCursor(2, 1);
  lcd.print("pieza grande");
  delay(1000);
 } 
 else if (bascula.get_units(10) >=4.6 && bascula.get_units(10) <=5){ //4.61 a 4.99
  digitalWrite(led1, LOW);
  digitalWrite(led2, LOW);
  digitalWrite(led3, LOW);
 lcd.clear(); 
  lcd.setCursor(2, 0);
  lcd.print("peso:" + String(bascula.get_units(10),2) + "kg");
  lcd.setCursor(0, 1);
  lcd.print("error en la elaboracion");
  delay(1000);
 }
 else {
  digitalWrite(led1, LOW);
  digitalWrite(led2, LOW);
  digitalWrite(led3, LOW);
  lcd.clear(); 
  lcd.setCursor(6, 0);
  lcd.print("ERROR");
  delay(1000);
 }

  Serial.print("Peso: ");
  Serial.print(bascula.get_units(10),2);
  Serial.println(" kg");
  delay(1000);

if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    doc["PESO"] = String(bascula.get_units(10),2);
    doc["NOMBRE"] = "Diego Bahena";

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("GG", output.c_str()); //nos unimos a un canal TOPICO
  }
}
```

2.- Instalar las librerias necesarias para el correcto funcionamiento del programa, como se muestra en la siguente imagen.

![image](https://github.com/user-attachments/assets/e2a90073-c2ca-4925-b937-093ab4eba498)

3.- Hacer la conexion de **HX711** y de los relay con la **ESP32** como tambien, conectar el LCD como se muestra en la siguente imagen.

![image](https://github.com/user-attachments/assets/b2828fdd-08e7-454d-aa86-0df9b3e65eaf)

4.- abrimos la **CMD** como administrador, para ejecutar el Programa **Node-red** debemos escribir en el **CMD**: ```node-red``` 

5.- Para abrir la aplicación nos vamos algun explorador y colocamos el siguente link: ```localhost:1880```

6.- ejecutamos el **CMD** como administrador escribimos ```nslookup broker.hivemq.com``` y copiamos el Address que nos genera.

7.- una vez en **node-red** colocamos un bloque ```mqqtt in``` y configuramos el bloque con el puerto mqtt con el ip ```3.76.79.164``` como se muestra en la imagen.

![image](https://github.com/user-attachments/assets/174f4487-100b-4477-91cb-1a6580d3ebc3)

8.- Colocar el bloque ```json``` y configurarlo como se muestra en la imagen.

![image](https://github.com/user-attachments/assets/1d8e839f-d648-4096-976e-ce02f5d30b69)

9.- Colocamos un bloque ```function``` y lo configuramos con el siguentes codigo.

![image](https://github.com/user-attachments/assets/e96c06aa-8c2e-4886-b1e6-c137ae56f26e)

10.- Colocamos un bloque de ```chart```  ```gauge``` y dos de  ```switch``` ```set.msg.payload``` ```show dialog``` estos ultimos para mostrar un mensaje.

![image](https://github.com/user-attachments/assets/f6057c72-1df4-4d6f-8018-3be2663c4649)

### Instrucciónes de operación
- Iniciar simulador en WOKWI.
- Seleccionamos DEPLOY en node-red para iniciar la comunicacion. 
- Seleccionamos DASHBOARD en node-red. 
- Abrimos el localhost en el dashboard. 
- Visualizamos la grafica como el parametro de peso.
  
### Resultados
Cuando haya funcionado, verás los valores en el localhost como en la LCD asi como se muestra en la siguente imagen.
- Un peso de menos de 1kg se comstrara lo siguiente tanto en el LCD como en Node-RED.

![image](https://github.com/user-attachments/assets/9f871358-31b0-4dda-b00c-33104b40511c)

![image](https://github.com/user-attachments/assets/99228a72-bfcf-49f9-b956-5640e7ccf6d9)

- un peso entre 1kg y 2.8Kg se enciende el LED 1 y se comstrara lo siguiente tanto en el LCD como en Node-RED.

![image](https://github.com/user-attachments/assets/d65095c7-1564-4045-87e2-6698ed73bed2)

![image](https://github.com/user-attachments/assets/6f5a1caf-c5ed-450c-abac-5847f8aad1d5)

- un peso entre 2.8kg y 3.5Kg se encienden el LED 1 como el LED 2 y se comstrara lo siguiente tanto en el LCD como en Node-RED.

![image](https://github.com/user-attachments/assets/c4e5b775-0bf4-4e94-9c03-40df9441ed7e)

![image](https://github.com/user-attachments/assets/70827b5f-8afc-41cf-9b0b-34abcd968e1d)

- un peso entre 3.5kg y 4.6Kg se encienden los tres LEDS y se comstrara lo siguiente tanto en el LCD como en Node-RED.

![image](https://github.com/user-attachments/assets/dba5cc74-5258-423f-9626-8e88664da72f)

![image](https://github.com/user-attachments/assets/230b6ac8-f649-448f-b7f7-7cd7c6c2fba9)

- un peso entre 4.6kg y 5Kg se apagan los tres LEDS y se comstrara lo siguiente tanto en el LCD como en Node-RED.

![image](https://github.com/user-attachments/assets/240bfa2b-8328-471a-bb0b-0e92c6ca18ea)

![image](https://github.com/user-attachments/assets/94ddf0e8-baf5-45f4-9464-d017c1877ab0)

### Desarrollado por 
- Guillermo de Jesús Hernandez Yáñez
- Diego David Bahena Galan
- Victor Emanuel Cabañas Montes
- Raul Castañeda Sotelo

- [GitHub](https://github.com/inward182)
- [GitHub](https://github.com/DiegoDBG)
- [GitHub](https://github.com/Victor-Cabanas-99)
- [GitHub](https://github.com/RaulCasS)

### Evidencia del funcionamiento en youtube
- [Youtube](https://youtu.be/E66pZOY2fUM)
