#include <Arduino.h>

// Ορισμός pins μοτέρ
#define M1A 8
#define M1B 9
#define M2A 10
#define M2B 11

// Ορισμός pins αισθητήρων (8 IR)
int sensorPins[8] = {26, 27, 28, 29, 0, 1, 2, 3}; // αλλάξε αν χρειάζεται
int sensorValues[8];

void setup() {
  // Ρυθμίσεις μοτέρ
  pinMode(M1A, OUTPUT);
  pinMode(M1B, OUTPUT);
  pinMode(M2A, OUTPUT);
  pinMode(M2B, OUTPUT);

  // Ρυθμίσεις αισθητήρων
  for(int i=0; i<8; i++){
    pinMode(sensorPins[i], INPUT);
  }
}

void setMotors(float leftSpeed, float rightSpeed){
  // leftSpeed και rightSpeed από -1.0 έως 1.0
  // μετατρέπουμε σε PWM 0-255
  int leftPWM = constrain(abs(leftSpeed*255), 0, 255);
  int rightPWM = constrain(abs(rightSpeed*255), 0, 255);

  if(leftSpeed >= 0){
    analogWrite(M1A, leftPWM);
    analogWrite(M1B, 0);
  } else {
    analogWrite(M1A, 0);
    analogWrite(M1B, leftPWM);
  }

  if(rightSpeed >= 0){
    analogWrite(M2A, rightPWM);
    analogWrite(M2B, 0);
  } else {
    analogWrite(M2A, 0);
    analogWrite(M2B, rightPWM);
  }
}

void loop() {
  int position = 0;
  int totalActive = 0;

  // Διαβάζουμε αισθητήρες
  for(int i=0; i<8; i++){
    sensorValues[i] = digitalRead(sensorPins[i]);
    if(sensorValues[i]){
      position += i;
      totalActive++;
    }
  }

  if(totalActive == 0){
    // Καμία γραμμή
    setMotors(0.2, -0.2);
  } else {
    float avgPos = (float)position / totalActive;
    
    if(avgPos < 3){
      // Γραμμή αριστερά
      setMotors(0.4, 0.6);
    } else if(avgPos > 4){
      // Γραμμή δεξιά
      setMotors(0.6, 0.4);
    } else {
      // Ευθεία
      setMotors(0.5, 0.5);
    }
  }

  delay(10); // μικρή καθυστέρηση
}
