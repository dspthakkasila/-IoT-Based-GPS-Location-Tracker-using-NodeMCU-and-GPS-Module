# -IoT-Based-GPS-Location-Tracker-using-NodeMCU-and-GPS-Module

#include <LiquidCrystal.h>
#include<DHT.h> 
#define DHTPIN A5  // Digital pin connected to the DHT sensor
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal lcd(13, 12, 11, 10, 9, 8);
#include <Servo.h>
Servo myservo;
int m11 = 2;
int m12 = 3;
int m21 = 4;
int m22 = 5;
int pump = 7;
int s1 = A4;
unsigned char rcv, count, gchr, gchr1, robos = 's';
int sti = 0;
String inputString = "";         // a string to hold incoming data
boolean stringComplete = false;  // whether the string is complete

void okcheck() {
  unsigned char rcr;
  do {
    rcr = Serial.read();
  } while (rcr != 'K');
}

void setup()
{
  Serial.begin(115200);
  dht.begin();
  lcd.begin(16, 2);
  myservo.attach(6);  // attaches the servo on pin 9 to the servo object
  myservo.write(90); 
  pinMode(s1, INPUT);
  pinMode(m11, OUTPUT);
  pinMode(m12, OUTPUT);
  pinMode(m21, OUTPUT);
  pinMode(m22, OUTPUT);
  pinMode(pump, OUTPUT);
  lcd.setCursor(0, 0);
  lcd.print(" WELCOME ");
  digitalWrite(m11, LOW);
  digitalWrite(m12, LOW);
  digitalWrite(m21, LOW);
  digitalWrite(m22, LOW);
  digitalWrite(pump, HIGH);
  delay(2000);
  lcd.clear();
  lcd.print("Wifi init");
  Serial.write("AT\r\n");
  delay(500);
  okcheck();
  Serial.write("ATE0\r\n");
  okcheck();
  Serial.write("AT+CWMODE=3\r\n");
  delay(500);
  Serial.write("AT+CIPMUX=1\r\n");
  delay(500);
  okcheck();
  Serial.write("AT+CIPSERVER=1,23\r\n");
  okcheck();
  lcd.clear();
  lcd.print("Waiting For");
  lcd.setCursor(0, 1);
  lcd.print("Connection");
  do {
    rcv = Serial.read();
  } while (rcv != 'C');


  lcd.clear();
  lcd.print("Connected");
  delay(2000);
  lcd.clear();
}

void loop() 
    {    
 
   serialEvent();
  lcd.setCursor(0, 0);
  lcd.print("STATUS::");
  if (stringComplete)
  {
    if (gchr == '1')
    {
      gchr = 'x';
      robos = 'm';
      digitalWrite(m11, HIGH);
      digitalWrite(m12, LOW);
      digitalWrite(m21, HIGH);
      digitalWrite(m22, LOW);
      Serial.write("AT+CIPSEND=0,11\r\n"); delay(100);
      Serial.print("\r\n FORWARD \r\n");  delay(100);
    }

    if (gchr == '2')
    {
      gchr = 'x';
      robos = 'm';
      digitalWrite(m11, LOW);
      digitalWrite(m12, HIGH);
      digitalWrite(m21, LOW);
      digitalWrite(m22, HIGH);
      Serial.write("AT+CIPSEND=0,12\r\n"); delay(100);
      Serial.write("\r\n BACKWARD \r\n"); delay(100);
    }

    if (gchr == '3')
    {
      gchr = 'x';
      robos = 'm';
      digitalWrite(m11, HIGH);
      digitalWrite(m12, LOW);
      digitalWrite(m21, LOW);
      digitalWrite(m22, HIGH);
      Serial.write("AT+CIPSEND=0,8\r\n"); delay(100);
      Serial.write("\r\n RIGHT \r\n"); delay(100);
    }

    if (gchr == '4')
    {
      gchr = 'x';
      robos = 'm';
      digitalWrite(m11, LOW);
      digitalWrite(m12, HIGH);
      digitalWrite(m21, HIGH);
      digitalWrite(m22, LOW);

      Serial.write("AT+CIPSEND=0,8\r\n"); delay(100);
      Serial.write("\r\n LEFT  \r\n"); delay(100);
    }

    if (gchr == '5')
    {
      gchr = 'x';
      robos = 's';
      digitalWrite(m11, LOW);
      digitalWrite(m12, LOW);
      digitalWrite(m21, LOW);
      digitalWrite(m22, LOW);
      Serial.write("AT+CIPSEND=0,8\r\n"); delay(100);
      Serial.write("\r\n STOP  \r\n"); delay(100);
    }
     
    if (gchr == '6')
    {
      gchr = 'x';
      robos = 's';
      digitalWrite(pump, LOW);
      Serial.write("AT+CIPSEND=0,16\r\n"); delay(100);
      Serial.write("\r\n charger on  \r\n"); delay(100);
    }    
    if (gchr == '7')
    {
      gchr = 'x';
      robos = 's';
      digitalWrite(pump, HIGH);
      Serial.write("AT+CIPSEND=0,17\r\n"); delay(100);
      Serial.write("\r\n charger off \r\n"); delay(100);
    }
    if (gchr == '0')
    {
      gchr = 'x';
      robos = 's';
      myservo.write(0);              // tell servo to go to position in variable 'pos'
      Serial.write("AT+CIPSEND=0,13\r\n"); delay(100);
      Serial.write("\r\n SERVO-0 \r\n"); delay(100);
    }
    if (gchr == '8')
    {
      gchr = 'x';
      robos = 's';
      myservo.write(90);              // tell servo to go to position in variable 'pos'
      Serial.write("AT+CIPSEND=0,14\r\n"); delay(100);
      Serial.write("\r\n SERVO-90 \r\n"); delay(100);
    }
    if (gchr == '9')
    {
      gchr = 'x';
      robos = 's';
      myservo.write(180);              // tell servo to go to position in variable 'pos'
      Serial.write("AT+CIPSEND=0,15\r\n"); delay(100);
      Serial.write("\r\n SERVO-180 \r\n"); delay(100);
    }  
    inputString = "";
    stringComplete = false;
  }
}
void converts(unsigned int value)
{
  unsigned int a,b,c,d,e,f,g,h;

      a=value/10000;
      b=value%10000;
      c=b/1000;
      d=b%1000;
      e=d/100;
      f=d%100;
      g=f/10;
      h=f%10;


      a=a|0x30;               
      c=c|0x30;
      e=e|0x30; 
      g=g|0x30;              
      h=h|0x30;
    
   Serial.write(a);
   Serial.write(c);
   Serial.write(e); 
   Serial.write(g);
   Serial.write(h);
}
void serialEvent()
{
  while (Serial.available())
  {
    char inChar = (char)Serial.read();
    if (inChar == '*')
    { sti = 1;
      inputString += inChar;
    }
    if (sti == 1)
    {
      inputString += inChar;
    }
    if (inChar == '#')
    { sti = 0;
      gchr = inputString[2];
      stringComplete = true;
    }
  }
}
