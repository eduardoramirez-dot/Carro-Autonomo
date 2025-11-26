//Carro-Autonomo
//Codigo C en arduino uno para el comportamiento de un carro autonomo
#include <Servo.h>

// ----------------------------------------------
// PINES
// ----------------------------------------------
const int SERVO_PIN = 11;

// Motor L298N
const int IN1 = 8;
const int IN2 = 13;
const int ENA = 10;

// Ultrasonidos izquierda y derecha (LOS TUYOS)
const int TRIG_I = 7;
const int ECHO_I = 6;

const int TRIG_D = 3;
const int ECHO_D = 2;

// Ultrasonido frontal (NUEVO)
const int TRIG_F = 5;
const int ECHO_F = 4; 

// ----------------------------------------------
// VALORES DEL SERVO (IGUAL QUE ANTES)
// ----------------------------------------------
int SERVO_IZQ = 1600;
int SERVO_CENTRO = 2000;
int SERVO_DER = 2500;

Servo direccion;

// ----------------------------------------------
void setup() {
  Serial.begin(115200);

  // Servo
  direccion.attach(SERVO_PIN, 500, 2500);
  direccion.writeMicroseconds(SERVO_CENTRO);

  // Motor
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(ENA, OUTPUT);

  // Ultrasonidos izquierdo y derecho
  pinMode(TRIG_I, OUTPUT);
  pinMode(ECHO_I, INPUT);
  pinMode(TRIG_D, OUTPUT);
  pinMode(ECHO_D, INPUT);

  // Ultrasonido frontal
  pinMode(TRIG_F, OUTPUT);
  pinMode(ECHO_F, INPUT);

  Serial.println("Auto ESP32 con 3 ultrasonidos listo!");
}

// ----------------------------------------------
// MOTOR
// ----------------------------------------------
void motorAdelante(int vel) {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, vel);
}

void motorAtras(int vel) {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  analogWrite(ENA, vel);
}

void motorStop() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 0);
}

// ----------------------------------------------
// ULTRASONIDO
// ----------------------------------------------
float leerDistancia(int trig, int echo) {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  long duracion = pulseIn(echo, HIGH, 25000);
  float dist = duracion * 0.034 / 2;

  if (dist == 0 || dist > 300) return 300;
  return dist;
}

// ----------------------------------------------
// LOOP
// ----------------------------------------------
void loop() {
  float distI = leerDistancia(TRIG_I, ECHO_I);
  float distD = leerDistancia(TRIG_D, ECHO_D);
  float distF = leerDistancia(TRIG_F, ECHO_F);

  Serial.print("Izq: "); Serial.print(distI);
  Serial.print(" | Der: "); Serial.print(distD);
  Serial.print(" | Front: "); Serial.print(distF);
  Serial.println(" cm");

  int MIN_DIST = 200;
  int MIN_DISTF = 10;
  int VEL = 255;

  // --- SI EL FRENTE REBASA EL MÍNIMO ---
  if (distF < MIN_DISTF) {
    Serial.println("OBSTÁCULO EN FRENTE");

    motorStop();
    direccion.writeMicroseconds(SERVO_CENTRO);
    delay(200);

    motorAtras(VEL);
    delay(600);
    motorStop();
    delay(200);

    // Decide la mejor dirección usando izquierda vs derecha
    if (distI > distD) {
      Serial.println("Evitando por IZQUIERDA");
      direccion.writeMicroseconds(SERVO_IZQ);
    } else {
      Serial.println("Evitando por DERECHA");
      direccion.writeMicroseconds(SERVO_DER);
    }

    delay(500);
    motorAdelante(VEL);
    delay(1300);
    motorStop();
    direccion.writeMicroseconds(SERVO_CENTRO);
    delay(200);

    return; // evita seguir leyendo abajo
  }

  // --- OBSTÁCULO LATERAL ---
  if (distI > MIN_DIST || distD > MIN_DIST) {

    motorStop();
    direccion.writeMicroseconds(SERVO_CENTRO);
    delay(200);

    motorAtras(VEL);
    delay(400);
    motorStop();
    delay(200);

    if (distI < distD) {
      Serial.println("Girando DERECHA");
      direccion.writeMicroseconds(SERVO_DER);
    } else {
      Serial.println("Girando IZQUIERDA");
      direccion.writeMicroseconds(SERVO_IZQ);
    }

    delay(500);

    motorAdelante(VEL);
    delay(1200);
    motorStop();
    direccion.writeMicroseconds(SERVO_CENTRO);
    delay(200);

  } else {
    // --- CAMINO LIBRE ---
    direccion.writeMicroseconds(SERVO_CENTRO);
    motorAdelante(VEL);
  }

  delay(100);
}
