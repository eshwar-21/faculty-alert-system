# faculty-alert-system

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);
#include <SoftwareSerial.h>
// Define SoftwareSerial for SIM800L
#define GSM_RX_PIN D5 // GPIO14
#define GSM_TX_PIN D6 // GPIO12

SoftwareSerial gsmSerial(GSM_RX_PIN, GSM_TX_PIN); // RX, TX
#include <EEPROM.h>
#include "RTClib.h"
RTC_DS3231 rtc;
char daysOfTheWeek[7][12] = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};

#define mode_button D7


//RoninDMD P10(WIDTH, HEIGHT);
String Message = "Welcome to ASCET";

const int EEPROM_SIZE = 2048; // Size of EEPROM
const int PERIOD_SIZE = 50;  // Size of each period data
const int NUM_PERIODS = 10;   // Number of periods to store per year
const int START_ADDRESS_YEAR_A = 0;            // Starting address for 2nd year in EEPROM
const int START_ADDRESS_YEAR_B = PERIOD_SIZE * NUM_PERIODS; // Starting address for 3rd year
const int START_ADDRESS_YEAR_C = PERIOD_SIZE * NUM_PERIODS * 2; // Starting address for 4th year

// Struct to hold period data
struct Period {
  char subject[20];
  char faculty[20];
  char mobile[15]; // New field for mobile number
};
int Index = 0;


void setup()
{
  Serial.begin(9600);
  gsmSerial.begin(9600);     // GSM module baud rate
  EEPROM.begin(EEPROM_SIZE);
  pinMode(mode_button, INPUT_PULLUP);
  lcd.begin();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(Message);
  delay(5000);

  if (! rtc.begin())
  {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Couldn't find RTC");
    while (1) delay(10);
  }

  if (rtc.lostPower())
  {
    //Serial.println("RTC lost power, let's set the time!");
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
    //rtc.adjust(DateTime(2014, 1, 21, 9, 30, 0));
  }
  //rtc.adjust(DateTime(2024, 3, 25, 9, 35, 0));
  //Serial.println("WELCOME");
  //if (digitalRead(mode_button) == HIGH)
  //{
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("GSM Initializing...");
    if (initializeGSM())
    {
      lcd.clear();
      lcd.print("GSM ready");
      // Serial.println("GSM module ready.");
      delay(2000);

      //lcd.setCursor(0, 1);
      //lcd.print("Sending SMS");
      // sendSMS("9704900473", "Device Activated");
    }
    else
    {
      lcd.clear();
      lcd.print("GSM failed.");
      delay(2000);
    }
  //}
}
String Sub;
String Name;
String Number;
int currentTime;
int startTime;
int endTime;
bool period_1 = true;
bool period_2 = true;
bool period_3 = true;
bool period_4 = true;
bool period_5 = true;
bool period_6 = true;
bool period_7 = true;
bool period_8 = true;
bool Lunch_time = true;
bool Scroll_status = false;

int A_1st = 0; int B_1st = 0; int C_1st = 0;
int A_2nd = 0; int B_2nd = 0; int C_2nd = 0;
int A_3rd = 0; int B_3rd = 0; int C_3rd = 0;
int A_4th = 0; int B_4th = 0; int C_4th = 0;
int A_5th = 0; int B_5th = 0; int C_5th = 0;
int A_6th = 0; int B_6th = 0; int C_6th = 0;
int A_7th = 0; int B_7th = 0; int C_7th = 0;
int A = 1;
int B = 2;
int C = 3;
String time_unit = "";

void loop()
{
  int year, period;
  Serial.println(digitalRead(mode_button));
  if (digitalRead(mode_button) == HIGH)
  {
    DateTime now = rtc.now();
    int unit_hh = now.hour();
    int unit_mm = now.minute();
    char hh_buffer[3];
    char mm_buffer[3];
    sprintf(hh_buffer, "%02d", now.hour());
    sprintf(mm_buffer, "%02d", now.minute());
    if (unit_hh >= 12)
    {
      time_unit = "PM";
    }
    else
    {
      time_unit = "AM";
    }
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Time:" + String(hh_buffer) + ":" + String(mm_buffer) +  " " + time_unit);
    lcd.setCursor(0, 1);
    lcd.print("Day :" + String(daysOfTheWeek[now.dayOfTheWeek()]));
    delay(2000);
    if (String(daysOfTheWeek[now.dayOfTheWeek()]) != "Sunday")
    {
      convert_time(8, 50, 9, 40); //1st period
      if (currentTime >= startTime && currentTime <= endTime)  // Check if the current time falls within the specified range
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("1st-Period");
        readSchedule(A, A_1st); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_1 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_1st); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_1 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_1 = false;
      }
      convert_time(9, 40, 10, 30); //2nd period
      if (currentTime >= startTime && currentTime <= endTime) // Check if the current time falls within the specified range
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("2nd-Period");
        readSchedule(A, A_2nd); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_2 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_2nd); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_2 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_2 = false;
      }
      convert_time(10, 30, 10, 50); //Break
      // Check if the current time falls within the specified range
      if (currentTime >= startTime && currentTime <= endTime)
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("      ");
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("    Break     ");
        delay(2000);
      }
      convert_time(10, 50, 11, 40); //3rd period
      if (currentTime >= startTime && currentTime <= endTime) // Check if the current time falls within the specified range
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("3rd-Period");
        readSchedule(A, A_3rd); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_3 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_3rd); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_3 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_3 = false;
      }
      convert_time(11, 40, 12, 30); //4th period
      if (currentTime >= startTime && currentTime <= endTime) // Check if the current time falls within the specified range
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("4th-Period");
        readSchedule(A, A_4th); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_4 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_4th); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_4 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_4 = false;
      }
      convert_time(12, 30, 13, 20); //Lunch
      if (currentTime >= startTime && currentTime <= endTime)       // Check if the current time falls within the specified range
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("      ");
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Lunch Break     ");
        delay(2000);
      }
      convert_time(13, 20, 14, 10); //5th period
      if (currentTime >= startTime && currentTime <= endTime) // Check if the current time falls within the specified range
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("5th-Period");
        readSchedule(A, A_5th); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_5 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_5th); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_5 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_5 = false;
      }
      convert_time(14, 10, 15, 00); //6th Period
      // Check if the current time falls within the specified range
      if (currentTime >= startTime && currentTime <= endTime)
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("6th-Period");
        //Serial.println("6th Period");
        readSchedule(A, A_6th); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_6 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_6th); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_6 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_6 = false;
      }
      convert_time(15, 00, 16, 50); //7th period
      // Check if the current time falls within the specified range
      if (currentTime >= startTime && currentTime <= endTime)
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("7th-Period");
        readSchedule(A, A_7th); //YEAR,PERIOD NUMBER
        delay(2000);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("A-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_7 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-A\nSubject:" + Sub);
        readSchedule(B, B_7th); //YEAR,PERIOD NUMBER
        delay(100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("B-Sub:" + Sub);
        lcd.setCursor(0, 1);
        lcd.print("Faculty:" + Name);
        delay(2000);
        if (period_7 == true)
          sendSMS(Number, "Dear Sir/Madam\n You have a class to Sec-B\nSubject:" + Sub);
        period_7 = false;
      }
      convert_time(16, 50, 23, 59); //Thank you
      // Check if the current time falls within the specified range
      if (currentTime >= startTime && currentTime <= endTime)
      {
        period_1 = true;
        period_2 = true;
        period_3 = true;
        period_4 = true;
        period_5 = true;
        period_6 = true;
        period_7 = true;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("  Thank you!    ");
        delay(2000);
      }
      convert_time(01, 00, 8, 50); //Welcome
      // Check if the current time falls within the specified range
      if (currentTime >= startTime && currentTime <= endTime)
      {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("   Welcome     ");
        delay(500);
      }
    }
    else
    {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print(" Happy Sunday!  ");
      delay(500);
    }
  }
  else
  {
    Serial.println("Waiting For Ur Input...");
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Open Serial monitor");
    lcd.setCursor(0, 1);
    lcd.print("U - Update shedule");
    delay(500);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("T - Update Time/Date");
    lcd.setCursor(0, 1);
    lcd.print("E - Exit");
    delay(500);
    if (Serial.available() > 0)
    {
      char cmd = Serial.read();
      if (cmd == 'u' || cmd == 'U')
      {
        while (1)
        {
ay:
          Serial.readString();
          Serial.println("Enter Sec (1-A, 2-B, or 3-C):");
          while (!Serial.available()) {}
          year = Serial.parseInt();
          Serial.println(year);
          if (year < 1 || year > 3) {
            //return;
            if (year == 0)
            {
              break;
            }
            else
            {
              Serial.println("Invalid year!");
              goto ay;
            }
          }

ap:
          Serial.readString();
          Serial.println("Enter period number (1 to 10):");
          while (!Serial.available()) {}
          period = Serial.parseInt();
          if (period < 1 || period > 10) {
            if (period == 0)
            {
              break;
            }
            else
            {
              Serial.println("Invalid period number!");
              //return;
              goto ap;
            }
          }
          // Check if the user wants to view or update the schedule
vu:
          Serial.readString();
          Serial.println("Do you want to view (v) or update (u) the schedule?");
          while (!Serial.available()) {}
          char choice = Serial.read();
          if (choice == 'v' || choice == 'V') {
            readSchedule(year, period);
          }
          else if (choice == 'u' || choice == 'U') {
            updateSchedule(year, period);
          }
          else if (choice == 'E' || choice == 'e')
          {
            break;
          }
          else
          {
            Serial.println("Invalid choice!");
            goto vu;
          }
        }
      }
      if (cmd == 'T' || cmd == 't')
      {
        Serial.readString();
        Serial.println("Enter hours");
        while (!Serial.available()) {}
        int hh = Serial.parseInt();
        Serial.readString();
        Serial.println("Enter minutes");
        while (!Serial.available()) {}
        int mm = Serial.parseInt();
        Serial.readString();
        Serial.println("Enter Date");
        while (!Serial.available()) {}
        int dd = Serial.parseInt();
        Serial.readString();
        Serial.println("Enter Month");
        while (!Serial.available()) {}
        int mn = Serial.parseInt();
        Serial.readString();
        Serial.println("Enter Year");
        while (!Serial.available()) {}
        int yy = Serial.parseInt();
        rtc.adjust(DateTime(yy, mn, dd, hh, mm, 0)); delay(100);
        Serial.println("Time and Date Updated");
      }
    }
  }
}

void convert_time(int sh, int sm, int eh, int em)
{
  DateTime now = rtc.now();
  Serial.println("Time:" + String(now.hour()) + "/" + String(now.minute()) + "/" + String(now.second()));
  delay(500);
  // Convert current time to minutes for easier comparison
  currentTime = now.hour() * 60 + now.minute();

  // Convert the boundary times (9:30 AM and 10:20 AM) to minutes
  startTime = sh * 60 + sm; // 9:30 AM
  endTime = eh * 60 + em;  // 10:20 AM

  if (String(daysOfTheWeek[now.dayOfTheWeek()]) == "Monday")
  {
    A_1st = 1;  B_1st = 1;  C_1st = 1;
    A_2nd = 1;  B_2nd = 1;  C_2nd = 1;
    A_3rd = 1;  B_3rd = 7;  C_3rd = 1;
    A_4th = 2;  B_4th = 7;  C_4th = 1;
    A_5th = 3;  B_5th = 2;  C_5th = 1;
    A_6th = 5;  B_6th = 3;  C_6th = 1;
  }

  if (String(daysOfTheWeek[now.dayOfTheWeek()]) == "Tuesday")
  {
    A_1st = 4;  B_1st = 4;  C_1st = 1;
    A_2nd = 3;  B_2nd = 7;  C_2nd = 1;
    A_3rd = 1;  B_3rd = 7;  C_3rd = 1;
    A_4th = 2;  B_4th = 7;  C_4th = 1;
    A_5th = 3;  B_5th = 2;  C_5th = 1;
    A_6th = 5;  B_6th = 3;  C_6th = 1;
  }

  if (String(daysOfTheWeek[now.dayOfTheWeek()]) == "Wednesday")
  {
    A_1st = 1;  B_1st = 4;  C_1st = 1;
    A_2nd = 3;  B_2nd = 7;  C_2nd = 1;
    A_3rd = 1;  B_3rd = 7;  C_3rd = 1;
    A_4th = 2;  B_4th = 7;  C_4th = 1;
    A_5th = 3;  B_5th = 2;  C_5th = 1;
    A_6th = 5;  B_6th = 3;  C_6th = 1;
  }

  if ((daysOfTheWeek[now.dayOfTheWeek()]) == "Thursday")
  {
    A_1st = 4;  B_1st = 4;  C_1st = 1;
    A_2nd = 3;  B_2nd = 7;  C_2nd = 1;
    A_3rd = 1;  B_3rd = 7;  C_3rd = 1;
    A_4th = 2;  B_4th = 7;  C_4th = 1;
    A_5th = 3;  B_5th = 2;  C_5th = 1;
    A_6th = 5;  B_6th = 3;  C_6th = 1;
  }

  if (String(daysOfTheWeek[now.dayOfTheWeek()]) == "Friday")
  {
    A_1st = 4;  B_1st = 4;  C_1st = 1;
    A_2nd = 3;  B_2nd = 7;  C_2nd = 1;
    A_3rd = 1;  B_3rd = 7;  C_3rd = 1;
    A_4th = 2;  B_4th = 7;  C_4th = 1;
    A_5th = 3;  B_5th = 2;  C_5th = 1;
    A_6th = 5;  B_6th = 3;  C_6th = 1;
  }
  if (String(daysOfTheWeek[now.dayOfTheWeek()]) == "Saturday")
  {
    A_1st = 1;  B_1st = 1;  C_1st = 1;
    A_2nd = 3;  B_2nd = 7;  C_2nd = 1;
    A_3rd = 1;  B_3rd = 7;  C_3rd = 1;
    A_4th = 2;  B_4th = 7;  C_4th = 1;
    A_5th = 3;  B_5th = 2;  C_5th = 1;
    A_6th = 5;  B_6th = 3;  C_6th = 1;
  }
}




void readSchedule(int year, int period) {
  int address;
  switch (year) {
    case 1:
      address = START_ADDRESS_YEAR_A;
      break;
    case 2:
      address = START_ADDRESS_YEAR_B;
      break;
    case 3:
      address = START_ADDRESS_YEAR_C;
      break;
    default:
      Serial.println("Invalid year!");
      return;
  }

  // Calculate the EEPROM address for the specified period
  address += (period - 1) * PERIOD_SIZE;

  // Create a Period struct to hold the schedule data
  Period currentPeriod;

  // Read the schedule data from EEPROM
  EEPROM.get(address, currentPeriod);

  // Print the schedule data to Serial Monitor
  //Serial.println("Schedule for Year " + String(year) + ", Period " + String(period) + ":");

  // Print subject until null terminator

  //Serial.print("Subject: ");
  //Serial.println(currentPeriod.subject);

  // Print faculty until null terminator

  //Serial.print("Faculty: ");
  //Serial.println(currentPeriod.faculty);

  //Serial.print("Mobile: ");
  //Serial.println(currentPeriod.mobile);

  Sub = currentPeriod.subject;
  Index  = Sub.indexOf('$');
  Sub = Sub.substring(0, Index);
  //Sub.trim();
  Serial.println("Subject:" + Sub);

  Name = currentPeriod.faculty;
  Index = Name.indexOf('$');
  Name = Name.substring(0, Index);
  //Name.trim();
  Serial.println("Name:" + Name);

  Number = currentPeriod.mobile;
  Number = Number.substring(0, 10);
  Serial.println("Number:" + Number);
}

void updateSchedule(int year, int period) {
  int address;
  switch (year) {
    case 1:
      address = START_ADDRESS_YEAR_A;
      break;
    case 2:
      address = START_ADDRESS_YEAR_B;
      break;
    case 3:
      address = START_ADDRESS_YEAR_C;
      break;
    default:
      Serial.println("Invalid year!");
      return;
  }

  // Calculate the EEPROM address for the specified period
  address += (period - 1) * PERIOD_SIZE;

  // Clear the EEPROM location for the specified period
  Period clearPeriod;
  EEPROM.put(address, clearPeriod);

  // Create a Period struct to hold the updated schedule data
  Period updatedPeriod;

  // Prompt user to enter subject and faculty
  Serial.readString();
  Serial.println("Enter subject name:");
  while (!Serial.available()) {}
  Serial.readBytesUntil('\n', updatedPeriod.subject, sizeof(updatedPeriod.subject) - 1); // Read subject name

  Serial.readString();
  Serial.println("Enter faculty name:");
  while (!Serial.available()) {}
  Serial.readBytesUntil('\n', updatedPeriod.faculty, sizeof(updatedPeriod.faculty) - 1); // Read faculty name

  Serial.readString();
  Serial.println("Enter faculty mobile number:"); // Prompt for mobile number
  while (!Serial.available()) {}
  Serial.readBytesUntil('\n', updatedPeriod.mobile, sizeof(updatedPeriod.mobile) - 1); // Read mobile number


  // Write the updated schedule data to EEPROM
  EEPROM.put(address, updatedPeriod);
  EEPROM.commit();

  // Notify user that schedule has been updated
  Serial.println("Schedule updated successfully!");
}
void clear_lcd()
{
  for (int l = 0; l < 20; l++)
  {
    lcd.setCursor(l, 2);
    lcd.print(" ");
    lcd.setCursor(l, 3);
    lcd.print(" ");
  }
}



/*****GSM*****/
bool initializeGSM()
{
  lcd.setCursor(0, 1);
  lcd.print("ATE0-");
  gsmSerial.println("ATE0");
  delay(1000);
  if (gsmSerial.find("OK"))
  {
    lcd.setCursor(5, 1);
    lcd.print("OK    ");
  }
  else
  {
    lcd.setCursor(5, 1);
    lcd.print("Error");
  }

  lcd.setCursor(0, 1);
  lcd.print("AT-");
  gsmSerial.println("AT");
  delay(1000);
  if (gsmSerial.find("OK"))
  {
    lcd.setCursor(3, 1);
    lcd.print("OK    ");
  }
  else
  {
    lcd.setCursor(3, 1);
    lcd.print("Error");
  }

  lcd.setCursor(0, 1);
  lcd.print("AT+CMGF-");
  gsmSerial.println("AT+CMGF=1");
  delay(1000);
  if (gsmSerial.find("OK"))
  {
    lcd.setCursor(8, 1);
    lcd.print("OK    ");
  }
  else
  {
    lcd.setCursor(8, 1);
    lcd.print("Error");
  }

  lcd.setCursor(0, 1);
  lcd.print("AT+CREG?-");
  gsmSerial.println("AT+CREG?");
  delay(1000);
  if (gsmSerial.find("+CREG: 0,1") || gsmSerial.find("+CREG: 0,5"))
  {
    lcd.setCursor(9, 1);
    lcd.print("ok        ");
    return true;
  }
  else
  {
    lcd.setCursor(9, 1);
    lcd.print("Error        ");
    return false;
  }
}

// Function to send SMS
void sendSMS(String num, String message)
{
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Sending SMS...");
  lcd.setCursor(0, 1);
  lcd.print(num);
  Serial.println("AT+CMGF=1");
  delay(1000);
  gsmSerial.println("AT+CMGS=\"" + num + "\"");
  delay(1000);
  gsmSerial.print(message);
  gsmSerial.write(26); // CTRL+Z
  delay(3000);
  
}

void read_sms()
{
  if (gsmSerial.available())
  {
    String response = gsmSerial.readString();
    response.trim();
    //Serial.println(response); // Print the raw response for debugging
    // Check if the response contains the +CMT indicator 
    
   if (response.startsWith("+CMTI:"))
    {
      gsmSerial.println("AT+CMGR=1");
      while (!gsmSerial.available() > 0);
      response = gsmSerial.readString();
      response.trim();
      //Serial.println(response);

  // Extract the phone number
      int startIndex = response.indexOf(',') + 5;
      int endIndex = response.indexOf('"', startIndex);
      String incomingNumber = response.substring(startIndex, endIndex);

  // Extract the message (second line)
      int messageStartIndex = response.indexOf('\n') + 1;
      int messageEndIndex = response.indexOf('$', messageStartIndex);
      //messageStartIndex = response.indexOf('\n', messageStartIndex) + 1;
      //endIndex = response.indexOf('\n', messageStartIndex);
      String message = response.substring(messageStartIndex, messageEndIndex);
      message.trim(); // Remove any extra whitespace or newlines

   // Print the extracted phone number and message
   //Serial.println("Phone Number: " + incomingNumber);
   //Serial.println("Message: " + message);


   // Display on LCD
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("From:" + incomingNumber);
      lcd.setCursor(0, 1);
      lcd.print("Msg: " + message);

   gsmSerial.println("AT+CMGD=1"); delay(3000);
    }
  }
}
