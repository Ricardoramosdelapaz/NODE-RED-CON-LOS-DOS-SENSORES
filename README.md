# NODE-RED-CON-LOS-DOS-SENSORES

# Practica NODE-RED simulado desde WOKWI
Este repositorio muestra como podemos llevar a cabo la simulacion en linea desde la interface de NODE-RED, dandonos visualmente los vamores que se programaron en el simulador de WOKWI de manera mas grafica que en dicho simulador.
## Introducción

### Descripción

El ```Node-RED``` es una herramienta de programación visual que se implementa en dispositivos controladores de hardware.la utilizamos en un entorno de adquisición de datos, en esta práctica ocuparemos haremos la programacion en WOKWI  en la cual incluiremos los sensores (```DHT22```) y (```HC-SR04```) para asi poderobservar  con graficas e indicadores por medio del Dashboard del Node-red.Cabe aclarar que esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/).


## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- [Node-red](http://localhost:1880/)
- Tarjeta ESP 32
- Sensor HC-SR04
- DHT22



## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas:
- Entrar a la plataforma [WOKWI](https://https://wokwi.com/).
- entras a el sitio [Node-RED](http://localhost:1880).

- Para poder entrar a [Node-RED] es importante seguir los siguientes pasos:
 [Instalar Node-Red](https://github.com/DiegoJm10/Node-red-instalacion/edit/main/README.md)


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación  en WOKWI y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
#include <LiquidCrystal_I2C.h> //Libreria de LCD
#define I2C1_ADDR    0x27
#define I2C2_ADDR    0x20
#define LCD1_COLUMNS 20
#define LCD1_LINES   4
// Update these with values suitable for your network.

 
const int trigPin = 4;
const int echoPin = 2;
const int trigPin2 = 17;
const int echoPin2 = 16;
const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "35.172.255.228";
String username_mqtt="RicardoR";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

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
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(trigPin, OUTPUT); //pin como salida
  pinMode(echoPin, INPUT);  //pin como entrada
  digitalWrite(trigPin, LOW);//Inicializamos el pin con 0
  pinMode(trigPin2, OUTPUT); //pin como salida
  pinMode(echoPin2, INPUT);  //pin como entrada
  digitalWrite(trigPin2, LOW);//Inicializamos el pin con 0
}

void loop()
{

  long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros
  long t2; //timepo que demora en llegar el eco
  long d2; //distancia en centimetros

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(trigPin, LOW);
  
  t = pulseIn(echoPin, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm

  digitalWrite(trigPin2, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(trigPin2, LOW);
  
  t2 = pulseIn(echoPin2, HIGH); //obtenemos el ancho del pulso
  d2 = t/59;             //escalamos el tiempo a una distancia en cm

Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
delay(1000);

Serial.print("Distancia2: ");
  Serial.print(d2);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
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
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["NOMBRE"] = "RICARDO RAMOS DE LA PAZ";
    doc["DISTANCIA"]= String(d);
    doc["DISTANCIA2"]= String(d2);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("PracticaNODERED", output.c_str());
  }
}


```
2. Instalar las librerías:
- **DHT sensor library for ESPx**
- **ArduinoJson**
- **PubSubClient**

![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/lib%20node.PNG?raw=true).

3. Hacer la conexion de **DHT2** con el sensor **HC-SR04** y el **DHT22**, como se muestra en la siguiente imagen.

![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/conex.PNG?raw=true).

### realizar la programacion en [Node-red](http://localhost:1880/)

## Una vez instalado y ya instalado el Dashboard:

- Colocar un inicio con un mqtt y lo configuramos con el la direccion ip actual y el nombre. (misma que se coloco en la programacion) tal como se muestra en la imagen.
  ![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/inicio.PNG?raw=true).
  
  Colocar un json y de igual manera tenemos que configurarlo como se muestra en la imagen.
  ![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/json.PNG?raw=true).
  
- Colocar un debug por default
- Colocar las funciones que nos daran los valores leidos de nuestro simulador y configurarlas como se muestra en la imagen.
- 
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/temp.PNG?raw=true).
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/hum.PNG?raw=true).
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/distancia.PNG?raw=true).

- Colocar las graficas y los indicadores.  
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/configuracion.PNG?raw=true).
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/DASHBOARD.PNG?raw=true).

-Como resultado nos queda de la siguiente manera:
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/DIAGRAMA.PNG?raw=true).

### Instrucciónes de operación

1. Iniciar simulador.
2. Visualizar los datos en el monitor serial.
4. Colocar la distancia dando click* al sensor ultrasonico **HC-SR04**.
5. Colocar temperatura y humedad en el sensor **DHT22**.
6. Abrir el Node-RED y correrlo en la parte de DEPLOYD.
7. Abrir los indicadores en la parte de flecha de la parte de la derecha al entrar a la parte de dashboard

## Resultados

Al iniciar la simulacion, verás los valores dentro del monitor serial y los datos obtenidos se veran graficamenteen node-red, y se actualizaran en tiempo real.

![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/SIMULACION.PNG?raw=true).
![](https://github.com/Ricardoramosdelapaz/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/GRAFICAS.PNG?raw=true).



# Créditos

Desarrollado por Ing. Ricardo Ramos De la paz

- [GitHub](https://github.com/Ricardoramosdelapaz)
