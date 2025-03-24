# -#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_Sensor.h>
#include "HX711.h"

// OLED 디스플레이 설정
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// MPU6050 객체
Adafruit_MPU6050 mpu;

// HX711 객체 및 핀
#define HX_DT 4
#define HX_SCK 5
HX711 scale;

void setup() {
  Serial.begin(115200);

  // I2C 통신 시작 (SDA = GPIO8, SCL = GPIO9)
  Wire.begin(8, 9);

  // OLED 초기화
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED init 실패");
    while (true);
  }

  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.setCursor(0, 0);
  display.println("Initializing...");
  display.display();

  // MPU6050 초기화
  if (!mpu.begin()) {
    Serial.println("MPU6050 init 실패");
    while (true);
  }

  // HX711 초기화
  scale.begin(HX_DT, HX_SCK);
  scale.set_scale();  // 나중에 보정값 지정 가능
  scale.tare();       // 0점 설정

  delay(1000);
}

void loop() {
  // MPU6050 센서 읽기
  sensors_event_t a, g, temp;
  mpu.getEvent(&a, &g, &temp);

  // HX711 압력 값 읽기
  float pressure = scale.get_units(5);  // 5회 평균값

  // OLED 출력
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Smart Shoe Monitor");

  // MPU6050 출력
  display.setCursor(0, 12);
  display.print("Accel X:");
  display.println(a.acceleration.x, 1);
  display.print("       Y:");
  display.println(a.acceleration.y, 1);
  display.print("       Z:");
  display.println(a.acceleration.z, 1);

  // HX711 압력 출력
  display.setCursor(0, 48);
  display.print("Pressure: ");
  display.println(pressure, 1);

  display.display();

  delay(500);  // 0.5초마다 업데이트
}
