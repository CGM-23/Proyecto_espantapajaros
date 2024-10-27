# Proyecto_espantapajaros
#include<WiFi.h>
#include<WiFiClientSecure.h>
#include<ThingSpeak.h>
#include<ESP32Servo.h>
#include<UniversalTelegramBot.h>

const char* red="Galaxyred";
const char* contra="abcd1234";
const int mychannelnumber=2322086;
const char* myApiKey="XWJ48CTR13K0HIBD";
const char* server= "api.thingspeak.com";
const char* botToken= "6720834358:AAHeveZg-jJXikSSLByrS8br4pQKsBbGzSc";
#define id "6842190577"
int led = 15;
WiFiClientSecure secure_client;
WiFiClient client;
UniversalTelegramBot bot(botToken,secure_client);
int porcentaje=25;
Servo myservo;
int pinservo1 = 19;
int pinservo2 = 33;
int pin_musica = 5;
int pin_sensor = 2;
int t_p = 4;
int e_p = 18;
float ss = 0.034;
long dur;

float dist_cm;
int Do=261;
int Do_H=523;
int Do_SH=554;
int Re=294;
int Re_H=587;
int Re_SH=622;
int Mi=329;
int Mi_H=659;
int Fa=349;
int Fa_H=698;
int Fa_SH=740;
int Sol=391;
int Sol_S=415;
int Sol_H=783;
int Sol_SH=830;
int La=440;
int La_S=455;
int La_H=880;
int Si=466;
bool servoMoving = false;
int delayRequest=500;

void handleTelegramCommand(String command) {
if (command == "/ledoff") {

digitalWrite(led, LOW);
analogWrite(led, 0);
bot.sendMessage(id, "LED apagado", "");
} else if (command == "/ledon") {
digitalWrite(led, HIGH);
analogWrite(led, 255);

bot.sendMessage(id, "LED encendido", "");

} else if (command.startsWith("/pwm")) {
// Analiza el comando para ajustar el valor PWM personalizado
int pwmValue = command.substring(4).toInt();
porcentaje=pwmValue;
if (pwmValue >= 0 && pwmValue <= 255) {
analogWrite(led, pwmValue);
analogWrite(led, 0);
bot.sendMessage(id, "LED PWM ajustado a " + String(pwmValue), "");
} else {
bot.sendMessage(id, "Valor PWM no válido. Debe estar entre 0 y 255",
"");
}
}

else {
// If client send /start
if (command == "/start") {
String welcome = "Hola,\n";
welcome += "Los comandos son los siguientes: \n";
welcome += "/pwm(valor de pwm de la led)\n";
welcome += "/ledoff\n";
welcome += "/ledon\n";
bot.sendMessage(id, welcome);
}
}
}
void setup() {
Serial.begin(115200);
pinMode(led, OUTPUT);
pinMode(pin_musica, OUTPUT);
digitalWrite(pin_musica, LOW);
pinMode(pin_sensor, INPUT_PULLUP);
pinMode(t_p, OUTPUT);
pinMode(e_p, INPUT);
noTone(pin_musica);
myservo.attach(pinservo1);
myservo.attach(pinservo2);
myservo.write(0);
WiFi.begin(red,contra);
Serial.println("Conectando a la red ...");
secure_client.setCACert(TELEGRAM_CERTIFICATE_ROOT);
while(WiFi.status() !=WL_CONNECTED){

delay(1000);
Serial.println("No esta conectado el WIFI");
}
Serial.println("Wifi conectado");
Serial.println("IP local: "+String(WiFi.localIP()));
bot.sendMessage(id,"bot conectado", "");
ThingSpeak.begin(client);
}

void loop() {
ThingSpeak.setField(1, 1);
int melodiaAleatoria[4];
int duracionAleatoria[4];
int sensorValue = digitalRead(pin_sensor);
int melodia_star1[] = {La, Do, Si, La_H};
int duracion_star1[] = {500, 500, 500, 1200};
int melodia_star2[] = {Do, Re, Mi, Fa_SH};
int duracion_star2[] = {500, 500, 500, 1200};
int melodia_star3[] = {Sol, Fa, Mi, La};
int duracion_star3[] = {500, 500, 500, 1300};
int notas_star = 4;

digitalWrite(t_p, LOW);
delayMicroseconds(2);
digitalWrite(t_p, HIGH);
delayMicroseconds(10);
digitalWrite(t_p, LOW);
dur = pulseIn(e_p, HIGH);

dist_cm = dur * ss / 2;
ThingSpeak.setField(2, dist_cm);
int numUpdates = bot.getUpdates(bot.last_message_received + 1);
for (int i = 0; i < numUpdates; i++) {
String command = bot.messages[i].text;
handleTelegramCommand(command);
}

int x = ThingSpeak.writeFields(mychannelnumber, myApiKey);
if (sensorValue == HIGH && dist_cm < 60) {
Serial.println("Movimiento detectado");
digitalWrite(led, HIGH);

Serial.println("Movimiento enviado a ThingSpeak");
analogWrite(led, porcentaje);
Serial.println("Se decteto un movimiento a una distacia de "+
String(dist_cm) + " cm");
int opcionMelodia = random(1, 4);
if (opcionMelodia == 1) {
for (int i = 0; i < notas_star; i++) {
melodiaAleatoria[i] = melodia_star1[i];
duracionAleatoria[i] = duracion_star1[i];
}

} else if (opcionMelodia == 2) {
for (int i = 0; i < notas_star; i++) {
melodiaAleatoria[i] = melodia_star2[i];
duracionAleatoria[i] = duracion_star2[i];
}

} else {
for (int i = 0; i < notas_star; i++) {
melodiaAleatoria[i] = melodia_star3[i];
duracionAleatoria[i] = duracion_star3[i];
}
}
for (int posicion = 0; posicion < notas_star; posicion++) {
toca(melodiaAleatoria[posicion], duracionAleatoria[posicion]);
delay(30);
}
bot.sendMessage(id,"Se detectado un movimiento", "");
bot.sendMessage(id,"A una distancia de: " + String(dist_cm) + " cm", "");
Serial.println();
analogWrite(led, 0);
}
delay(30);
}

void toca(int nota, int duracion) {
tone(pin_musica, nota, duracion);
delay(duracion + 40);
noTone(pin_musica);
}
