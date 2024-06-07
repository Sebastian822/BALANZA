// librerias del motor
#include <WiFi.h>
#include <WebServer.h>
// Librerias de la celda de carga
#include "HX711.h"
#include <EEPROM.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>

// Configura el SSID y la contraseña de tu red WiFi
const char* ssid = "Aula tecnica";
const char* password = "Madygraf32";

// Configura los pines de salida
const int pinOutput_ENA = 18; // enciende el motor con LOW
const int pinOutput_DIR = 19; // sentido de giro
const int pinOutput_PUL = 13; // pulso,  debe tener 1000 microsegundos entre semiciclo positivo y negativo

WebServer server(80);

LiquidCrystal_I2C lcd(0x27, 16, 2);

HX711 balanza;

const int zero = 2;
int DT = 4;
int CLK = 5;

int peso_calibracion = 185; // Es el peso referencial a poner, en mi caso mi celular pesa 185g (SAMSUMG A20)
long escala;

int state_zero = 0;
int last_state_zero = 0;




void handleRoot() {
  server.send(200, "text/plain", "ESP32 Web Server");
}

void handleOutputENA_f() {
  digitalWrite(pinOutput_ENA, LOW); // ENA se enciende con LOW
  digitalWrite(pinOutput_DIR, LOW); //Forward
  server.send(200, "text/plain", "Motor en Marcha");
  for (int i = 0; i < 200; i++) {
    digitalWrite(pinOutput_PUL, HIGH);
    delayMicroseconds(1000);
    digitalWrite(pinOutput_PUL, LOW);
    delayMicroseconds(1000);
  }

}

void handleOutputENA_r() {
  digitalWrite(pinOutput_ENA, LOW); // ENA se apaga con HIGH
  digitalWrite(pinOutput_DIR, HIGH); //Reverse
  server.send(200, "text/plain", "Motor en Reversa");
    for (int i = 0; i < 200; i++) {
    digitalWrite(pinOutput_PUL, HIGH);
    delayMicroseconds(1000);
    digitalWrite(pinOutput_PUL, LOW);
    delayMicroseconds(1000);
  }
}



//Función calibración
void calibration() { // despues de hacer la calibracion puedes borrar toda la funcion "void calibration()"
  boolean conf = true;
  long adc_lecture;

  // restamos el peso de la base de la balaza
  lcd.setCursor(0, 0);
  lcd.print("Calibrando base");
  lcd.setCursor(4, 1);
  lcd.print("Balanza");
  delay(3000);
  balanza.read();
  balanza.set_scale(); //La escala por defecto es 1
  balanza.tare(20);  //El peso actual es considerado zero.
  lcd.clear();

  //Iniciando calibración
  while (conf == true) {

    lcd.setCursor(1, 0);
    lcd.print("Peso referencial:");
    lcd.setCursor(1, 1);
    lcd.print(peso_calibracion);
    lcd.print(" g        ");
    delay(3000);
    lcd.clear();
    lcd.setCursor(1, 0);
    lcd.print("Ponga el Peso");
    lcd.setCursor(1, 1);
    lcd.print("Referencial");
    delay(3000);

    //Lee el valor del HX711
    adc_lecture = balanza.get_value(100);

    //Calcula la escala con el valor leido dividiendo el peso conocido
    escala = adc_lecture / peso_calibracion;

    //Guarda la escala en la EEPROM
    EEPROM.put( 0, escala );
    delay(100);
    lcd.setCursor(1, 0);
    lcd.print("Retire el Peso");
    lcd.setCursor(1, 1);
    lcd.print("referencial");
    delay(3000);
    lcd.clear();
    lcd.setCursor(1, 0);
    lcd.print("READY!!....");
    delay(3000);
    lcd.clear();
    conf = false; //para salir de while
    lcd.clear();

  }
}



void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.println("WiFi connected");

  // Imprime la dirección IP obtenida
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  // Configura los pines de salida
  pinMode(pinOutput_ENA, OUTPUT);
  pinMode(pinOutput_DIR, OUTPUT);
  pinMode(pinOutput_PUL, OUTPUT);

  // Configura los pines en estado inicial
  digitalWrite(pinOutput_ENA, HIGH); // El motor está apagado inicialmente
  digitalWrite(pinOutput_DIR, LOW); // El sentido de giro es hacia adelante inicialmente
  digitalWrite(pinOutput_PUL, LOW); // El pulso está en estado bajo inicialmente

  server.on("/", handleRoot);
  server.on("/ena_f", handleOutputENA_f);
  server.on("/ena_r", handleOutputENA_r);

  server.begin();
  Serial.println("HTTP server started"); //iniciando

  balanza.begin(DT, CLK);//asigana los pines para el recibir el trama del pulsos que viene del modulo
  pinMode(zero, INPUT);//declaramos el pin2 como entrada del pulsador
  pinMode(13,OUTPUT);
  lcd.init(); // Inicializamos el lcd
  lcd.backlight(); // encendemos la luz de fondo del lcd

  EEPROM.get( 0, escala );//Lee el valor de la escala en la EEPROM

  if (digitalRead(zero) == 1) { //esta accion solo sirve la primera vez para calibrar la balanza, es decir se presionar ni bien se enciende el sistema
    calibration();
  }
  balanza.set_scale(escala); // Establecemos la escala
  balanza.tare(20);  //El peso actual de la base es considerado zero.

}




void loop() {
  server.handleClient();

  int state_zero = digitalRead(zero);
  float peso;
  peso = balanza.get_units(10);  //Mide el peso de la balanza

  //Muestra el peso
  lcd.setCursor(1, 0);
  lcd.print("Peso: ");
  lcd.print(peso, 0);
  lcd.println(" g        ");
  delay(5);

  //Botón de zero, esto sirve para restar el peso de un recipiente 
  if ( state_zero != last_state_zero) {

    if (state_zero == LOW) {
      balanza.tare(10);  //El peso actual es considerado zero.
    }
  }
  last_state_zero  = state_zero;

  if (peso>=500)digitalWrite(13,1);
  else if(peso<=500)digitalWrite(13,0);



}

