# NODE-RED-CON-DHT22-Y-ULTRASONICO
Se mandan los datos de los sensores a la node red para presentarlos en graficas 

## Introduccion 

### Descripcion

La node red nos mostrará los valores recolectados de los sensores con la dashboard 

### Material necesario

- WOKWI
- Tarjeta ESP32
- Sensor ultrasonico
- Sensor DHT22
- Red Wifi en este caso sera la de WOKWI Guest
- None Red (SMD Ejecutar como administrador de tareas y poner node-red y dar enter) y abrir en internet con localhost:1880
- Servidor MQTT presionar connect y copiar lo que aparece adelante de HOST (SMD)

  https://www.mqtt-dashboard.com/ 

## Instrucciones

### Requisitos previos
Para poder usar este repositorio deberás entrar a la plataforma WOKWI

https://wokwi.com/

Instrucciones de prepracion de entorno
Abrir la terminal de programación y colocar las siguientes líneas

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 18;
const char* ssid = "Wokwi-GUEST"; //red wifi
const char* password = ""; //contraseña de la red wifi
const char* mqtt_server = "52.29.87.71"; //servidos mqtt adresses
String username_mqtt="DiploIC";
String password_mqtt="12345";

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
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
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
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {

 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
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
    doc["Nombre"] = "Irving Cardoso";
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"] = String(d);

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("DiplomadoIC", output.c_str());
  }
}
```
2.Instalar las librerías siguientes 

![image](https://github.com/user-attachments/assets/9478397b-f105-434e-8a7a-357c16f2163b)

3.Hacer la conexión como se muestra en la siguiente imagen

![image](https://github.com/user-attachments/assets/3e14fa82-28b9-46e6-9c7d-cce0c7d6aaa3)

## Instrucciones de operacion
1.Iniciar el simulador
2. Abrir la dashboard donde se mostraran las graficas de los datos recolectados
3. Variar los parametros dando doble click en los sensores para que estos puedan modificar las graficas 

## Resultados 

Cuando el programe funcione correctamente te arrojara los siguientes resultados los cuales pueden variar dependidendo del rango de los sensores y los rangos ajustados en la node red 

![image](https://github.com/user-attachments/assets/c2470cf8-57ba-4f88-b9eb-1a9512ddfadb)







