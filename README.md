
#include <Servo.h>
#include <OneWire.h>
#include <DallasTemperature.h>

// ==================================
// PIN CONFIGURATION
// ==================================

// LDR Sensors
const int LDR_TL = A0;
const int LDR_TR = A1;
const int LDR_BL = A2;
const int LDR_BR = A3;

// Soil Moisture Sensors
const int SOIL1 = A4;
const int SOIL2 = A5;

// Temperature Sensor
const int ONE_WIRE_BUS = 2;

// Servo Motors
const int HORIZONTAL_SERVO_PIN = 9;
const int VERTICAL_SERVO_PIN = 10;
const int IRRIGATION_SERVO_PIN = 11;

// ==================================
// SENSOR AND SERVO OBJECTS
// ==================================

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature temperatureSensors(&oneWire);

Servo horizontalServo;
Servo verticalServo;
Servo irrigationServo;

// ==================================
// SETTINGS
// ==================================

// Soil calibration values
// Adjust after testing your actual sensors.
const int DRY_VALUE = 800;
const int WET_VALUE = 350;

// Irrigation threshold
const int DRY_THRESHOLD = 40;
const int WET_THRESHOLD = 50;

// Solar tracking settings
const int TRACKING_TOLERANCE = 40;
const int SERVO_STEP = 1;

// Servo positions
int horizontalPosition = 90;
int verticalPosition = 90;

const int SERVO_MIN = 10;
const int SERVO_MAX = 170;

// Irrigation servo positions
const int VALVE_OPEN = 90;
const int VALVE_CLOSED = 0;

bool irrigationActive = false;

// ==================================
// SETUP
// ==================================

void setup() {

  Serial.begin(9600);

  // Initialize temperature sensors
  temperatureSensors.begin();

  // Attach servos
  horizontalServo.attach(HORIZONTAL_SERVO_PIN);
  verticalServo.attach(VERTICAL_SERVO_PIN);
  irrigationServo.attach(IRRIGATION_SERVO_PIN);

  // Initial servo positions
  horizontalServo.write(horizontalPosition);
  verticalServo.write(verticalPosition);
  irrigationServo.write(VALVE_CLOSED);

  Serial.println("================================");
  Serial.println("HYBRID RENEWABLE ENERGY SYSTEM");
  Serial.println("SOIL AND CROP MONITORING SYSTEM");
  Serial.println("================================");

  delay(1000);
}

// ==================================
// READ SOIL MOISTURE
// ==================================

int readSoilMoisture(int sensorPin) {

  int rawValue = analogRead(sensorPin);

  int moisture = map(
    rawValue,
    DRY_VALUE,
    WET_VALUE,
    0,
    100
  );

  moisture = constrain(moisture, 0, 100);

  return moisture;
}

// ==================================
// SOLAR TRACKING SYSTEM
// ==================================

void solarTracking() {

  int topLeft = analogRead(LDR_TL);
  int topRight = analogRead(LDR_TR);
  int bottomLeft = analogRead(LDR_BL);
  int bottomRight = analogRead(LDR_BR);

  int leftAverage = (topLeft + bottomLeft) / 2;
  int rightAverage = (topRight + bottomRight) / 2;

  int topAverage = (topLeft + topRight) / 2;
  int bottomAverage = (bottomLeft + bottomRight) / 2;

  // Horizontal tracking
  if (leftAverage > rightAverage + TRACKING_TOLERANCE) {

    horizontalPosition += SERVO_STEP;

  }
  else if (rightAverage > leftAverage + TRACKING_TOLERANCE) {

    horizontalPosition -= SERVO_STEP;

  }

  // Vertical tracking
  if (topAverage > bottomAverage + TRACKING_TOLERANCE) {

    verticalPosition += SERVO_STEP;

  }
  else if (bottomAverage > topAverage + TRACKING_TOLERANCE) {

    verticalPosition -= SERVO_STEP;

  }

  horizontalPosition = constrain(
    horizontalPosition, SERVO_MIN, SERVO_MAX
  );

  verticalPosition = constrain(
    verticalPosition, SERVO_MIN, SERVO_MAX
  );

  horizontalServo.write(horizontalPosition);
  verticalServo.write(verticalPosition);

  Serial.println("\n--- SOLAR TRACKING ---");

  Serial.print("Horizontal Position: ");
  Serial.println(horizontalPosition);

  Serial.print("Vertical Position: ");
  Serial.println(verticalPosition);
}

// ==================================
// TEMPERATURE MONITORING
// ==================================

void temperatureMonitoring() {

  temperatureSensors.requestTemperatures();

  int sensorCount = temperatureSensors.getDeviceCount();

  Serial.println("\n--- TEMPERATURE MONITORING ---");

  if (sensorCount == 0) {
    Serial.println("No temperature sensor detected.");
    return;
  }

  for (int i = 0; i < sensorCount; i++) {

    float temperature =
      temperatureSensors.getTempCByIndex(i);

    Serial.print("Temperature Sensor ");
    Serial.print(i + 1);
    Serial.print(": ");

    if (temperature == DEVICE_DISCONNECTED_C) {
      Serial.println("Sensor disconnected");
    }
    else {
      Serial.print(temperature);
      Serial.println(" C");
    }
  }
}

// ==================================
// AUTOMATIC IRRIGATION
// ==================================

void automaticIrrigation(int moisture1, int moisture2) {

  // Average moisture reading
  int averageMoisture = (moisture1 + moisture2) / 2;

  Serial.println("\n--- IRRIGATION SYSTEM ---");

  Serial.print("Average Soil Moisture: ");
  Serial.print(averageMoisture);
  Serial.println("%");

  // Activate irrigation when soil is dry
  if (averageMoisture <= DRY_THRESHOLD &&
      !irrigationActive) {

    irrigationServo.write(VALVE_OPEN);
    irrigationActive = true;

    Serial.println("SOIL IS DRY");
    Serial.println("IRRIGATION ACTIVATED");

  }

  // Stop irrigation when soil reaches desired moisture
  else if (averageMoisture >= WET_THRESHOLD &&
           irrigationActive) {

    irrigationServo.write(VALVE_CLOSED);
    irrigationActive = false;

    Serial.println("SOIL MOISTURE IS SUFFICIENT");
    Serial.println("IRRIGATION STOPPED");

  }

  else if (irrigationActive) {

    Serial.println("IRRIGATION IN PROGRESS");

  }

  else {

    Serial.println("SOIL MOISTURE IS NORMAL");
    Serial.println("IRRIGATION INACTIVE");

  }
}

// ==================================
// MAIN LOOP
// ==================================

void loop() {

  // Read soil moisture sensors
  int moisture1 = readSoilMoisture(SOIL1);
  int moisture2 = readSoilMoisture(SOIL2);

  Serial.println("\n================================");
  Serial.println("REAL-TIME MONITORING");
  Serial.println("================================");

  Serial.print("Soil Moisture 1: ");
  Serial.print(moisture1);
  Serial.println("%");

  Serial.print("Soil Moisture 2: ");
  Serial.print(moisture2);
  Serial.println("%");

  // Solar tracking
  solarTracking();

  // Temperature monitoring
  temperatureMonitoring();

  // Automatic irrigation
  automaticIrrigation(moisture1, moisture2);

  Serial.println("\n================================");

  delay(1000);
}
