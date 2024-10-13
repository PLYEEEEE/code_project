#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

#define LIGHT_SENSOR_PIN A0
#define IN_1 D5
#define IN_2 D6

int lightThreshold = 500;

ESP8266WebServer server(80);

void curtainStop() {
  Serial.printIn("Stop");
  digitalWrite(IN_1, LOW);
  digitalWrite(IN_2, LOW);
}

void curtainOpen() {
  Serial.printIn("Open");
  digitalWrite(IN_1, HIGH);
  digitalWrite(IN_2, LOW);
}

int readSensor() {
  return analogRead(LIGHT_SENSOR_PIN);
}

void curtainClose() {
  Serial.printIn("Close");
  digitalWrite(IN_1, LOW);
  digitalWrite(IN_2, HIGH);
}

void handleRoot() {
  // อ่านค่าเซ็นเซอร์แสง
  int lightValue = readSensor();

  String page = "<html><read><title>My Curtain Control</title></head><body 
style= 'text-align: center;'>";
  page += "<h1>My curtain Control</h1>";
  page += "<p>Light Sensor Value: " + String(lightValue) + "</p>"

  // สร้างปุ่มเพื่อควบคุมการเปิด-ปิดม่าน
   page += "<form action='/control' method='get'>";
   page += "<button type='submit' name='action' value='open'>Open
Curtain</button>";
   page += "<button type='submit' name='action' value='close'>Close
Curtain</button>";
   page += "<button type='submit' name='action' value='stop'>Stop
Curtain</button>";
   page += "</form>";

   page += "</body></html>";

   server.send(200, "text/html", page);
}

void handleControl() {
   if (server.hasArg("action")) {
     String action = server.arg("action");
     if (action == "open") {
       curtainOpen();
     } else if (action == "close") {
       curtainClose();
     } else if (action == "stop") {
       curtainStop();
     }
  }
  handleRoot(); // แสดงหน้าเว็บควบคุมม่านอีกครั้งหลังจากดำเนินการเสร็จสิ้น
}

void setup() {
  Serial.begin(9600);
  // put your setup code here, to run once:
  pinMode(LIGHT_SENSOR_PIN, INPUT);
  pinMode(IN_1, OUTPUT);
  pinMode(IN_2, OUTPUT);
  delay(1000);


  curtainStop();

  Serial.println("The curtain is ready");

  // เริ่มต้นตั้งค่า WiFi Access Point
  WiFi.softAP("My Curtain"); // ตั้งชื่อ WiFi ใหม่เป็น "My Curtain"

  // รับทราบ IP Address ของ Access Point และแสดงผลใน Serial Monitor
  Serial.print("AP IP Address");
  Serial.println(WiFi.softAPIP());

  // เริ่มต้นตั้งค่าเว็บเซิร์ฟเวอร์
  server.on("/", handleRoot);
  server.on("/control", handleControl); // สร้างเส้นทางสำหรับควบคุมม่าน
  server.begin();
}

void loop() {
  server.handleClient();
  delay(500);
}






















  
