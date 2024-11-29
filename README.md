#include <SoftwareSerial.h>
SoftwareSerial simSerial(10, 11); // TX 10 ; RX 11

#include <LCDI2C_Generic.h>
#include <LCDI2C_Katakana.h>
#include <LCDI2C_Katakana_Symbols.h>
#include <LCDI2C_Katakana_Vietnamese.h>
#include <LCDI2C_Latin.h>
#include <LCDI2C_Latin_Symbols.h>
#include <LCDI2C_Latin_Vietnamese.h>
#include <LCDI2C_Multilingual.h>
#include <LCDI2C_Russian.h>
#include <LCDI2C_RussianLatin.h>
#include <LCDI2C_RussianLatin_Symbols.h>
#include <LCDI2C_RussianLatin_Vietnamese.h>
#include <LCDI2C_Russian_Symbols.h>
#include <LCDI2C_Russian_Vietnamese.h>
#include <LCDI2C_Symbols.h>
#include <LCDI2C_Vietnamese.h>
#include <DHT.h>
#include <DHT_U.h>
#include <Wire.h> 
#include <LiquidCrystal_I2C.h> // Vào Library Manager tải thư viện Thư viện Màn hình I2C
LiquidCrystal_I2C lcd(0x27,16,2);//Khai báo địa chỉ I2C (0x27 or 0x3F) và Màn hình 16x02

#define DHTPin  6        // Chân xuất tín hiệu của cảm biến nối chân số 6 Arduino
#define DHTType  DHT11   // Khai báo kiểu cảm biến là DHT11
#define buzzer 3
// Khai báo chân kết nối
const int flamePin = 2; // Chân D0 của cảm biến lửa kết nối với chân số 2 của Arduino
#define mq2Pin A1  // Thiet lap chan analog cho mq2
#define value 160
const int ledPin = 13;  // Chân LED tích hợp trên Arduino

#define PHONE_NUMBER      "0985881608"
#define SET_BAUDRATE      "AT+IPREX=115200"  // Thay đổi baudrate
#define MCU_SIM_BAUDRATE  115200 

DHT dht(DHTPin, DHTType); // Khai báo thư viện chân cảm biến và kiểu cảm biến.

void sim_at_wait() {
    delay(100);
    while (simSerial.available()) {
        Serial.write(simSerial.read());
    }
}

bool sim_at_cmd(String cmd) {
    simSerial.println(cmd);
    sim_at_wait();
    return true;
}

bool sim_at_send(char c) {
    simSerial.write(c);
    return true;
}

void send_sms(String message) {
    sim_at_cmd("AT+CMGF=1"); // Chuyển module sang chế độ nhắn tin
    String temp = "AT+CMGS=\"";
    temp += (String)PHONE_NUMBER;
    temp += "\"";
    sim_at_cmd(temp);
    simSerial.print(message); // Nội dung tin nhắn
    sim_at_send(0x1A);  // End character for SMS
    sim_at_wait();
}

void make_call() {
    String temp = "ATD";
    temp += PHONE_NUMBER;
    temp += ";";
    sim_at_cmd(temp);
    delay(5000); // Gọi trong 20 giây
    sim_at_cmd("ATH"); // Ngắt cuộc gọi
}

void setup() {
    pinMode(flamePin, INPUT); // Thiết lập chân flamePin là đầu vào
    pinMode(mq2Pin, INPUT); // thiet lap chan mq2 la dau vao
    pinMode(buzzer, OUTPUT); // Thiet lap loa bao dong
    pinMode(ledPin, OUTPUT);  // Thiết lập chân ledPin là đầu ra
    Serial.begin(MCU_SIM_BAUDRATE);
    delay(200);
    simSerial.begin(MCU_SIM_BAUDRATE);
    delay(200);

    Serial.println("\n\n\n\n-----------------------\nSystem started!!!!");

    sim_at_cmd("AT"); // Kiểm tra AT Command
    delay(200);
    sim_at_cmd("AT+IPREX?"); // Kiểm tra tốc độ baud rate
    delay(200);
    sim_at_cmd(SET_BAUDRATE); // Cài tốc độ baudrate
    delay(200);
    sim_at_cmd("ATI"); // Thông tin module SIM
    delay(200);
    sim_at_cmd("AT+CPIN?"); // Kiểm tra trạng thái của thẻ SIM
    delay(200);
    sim_at_cmd("AT+CSQ"); // Kiểm tra tín hiệu mạng
    delay(200);
    sim_at_cmd("AT+CIMI");
    delay(200);

    // Khởi động hệ thống gửi tin nhắn kiểm tra
    send_sms("He thong hoan tat");
    delay(5000); // Delay 5s

    // Thực hiện cuộc gọi kiểm tra
    make_call();

    Serial.begin(9600);
  
    lcd.begin(); // Khởi động LCD                    
    lcd.backlight(); // Mở đèn màn hình
  
    dht.begin(); // khởi động cảm biến DHT
}

void loop() 
{
  float DoAm = dht.readHumidity(); // Đọc độ ẩm môi trường

  float DoC = dht.readTemperature(); //Đọc nhiệt độ C

  float DoF = dht.readTemperature(true); //Đọc nhiệt độ F

  
  if (isnan(DoAm) || isnan(DoC) || isnan(DoF)) // Kiểm tra tín hiệu trả về từ cảm biến. 
  {
    Serial.println("Không có giá trị trả về từ cảm biến DHT");
    return;
  }

  Serial.print("Độ ẩm: ");
  Serial.print(DoAm);
  //Serial.print(DoAm,0); Không lấy số sau dấu ,
  
  Serial.print("%  Nhiệt độ Celsius | Fahrenheit : ");
  Serial.print(DoC,0); // Lấy 1 số sau dấu ,
  Serial.print("°C | ");
  Serial.print(DoF,0);
  Serial.println("°F");

// In ra màn hình LCD độ ẩm, nhiệt độ °C
  lcd.setCursor(0,0);
  lcd.print("DO AM:");
  lcd.setCursor(0,1);
  lcd.print("NHIET DO C:");
  lcd.setCursor(7,0); //con trỏ vị trí số 7, hiện ô số 8
  lcd.print(DoAm,0);
  lcd.setCursor(10,0); //Con trở ở vị trí 10, hiện ô 11
  lcd.print("%");

  lcd.setCursor(12,1); //con trỏ vị trí số 12, hiện ô số 13
  lcd.print(DoC,0);
  //lcd.setCursor(13,1);
  //lcd.print("°C");
  delay(3000);
  
  lcd.clear(); // Xóa màn hình.

  // In ra màn hình LCD độ ẩm, nhiệt độ °F
  lcd.setCursor(0,0);
  lcd.print("DO AM:");
  lcd.setCursor(0,1);
  lcd.print("NHIET DO F:");
  lcd.setCursor(7,0); //con trỏ vị trí số 7, hiện ô số 8
  lcd.print(DoAm,0);
  lcd.setCursor(10,0); //Con trở ở vị trí 10, hiện ô 11
  lcd.print("%");

  lcd.setCursor(12,1); //con trỏ vị trí số 12, hiện ô số 13
  lcd.print(DoF,0);
  //lcd.setCursor(13,1);
  //lcd.print("°F");
  
  delay(3000);

  int flameState = digitalRead(flamePin); // Đọc trạng thái của cảm biến lửa
  int analog = analogRead(mq2Pin);  // doc trang thai cua mq2
  delay(1000);
  Serial.println(analog);

  if ((flameState == LOW)||(analog>value)) { // Nếu phát hiện lửa (D0 = LOW) hoặc phát hiện mq2State> 350
      Serial.println("Flame detected! Sending SMS and making a call...");
      digitalWrite(ledPin, HIGH); // Bật LED
      digitalWrite(buzzer, HIGH);
      make_call(); // Thực hiện cuộc gọi
      delay(5000); 
      send_sms("BAO DONG!!"); // Gửi tin nhắn
      delay(10000); // Chờ 5 giây trước khi gọi để tin nhắn có thể gửi trước

  } else {
      digitalWrite(ledPin, LOW); // Tắt LED
      digitalWrite(buzzer, LOW);
      Serial.println("No flame detected.");
    }

    delay(500); // Chờ 500ms trước khi kiểm tra lại
}
