#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <SoftwareSerial.h>
#include <LiquidCrystal.h> // Include the LiquidCrystal library for non-I2C LCD



// Define the pins for the RFID module

// Define pins for the LCD
LiquidCrystal lcd(12, 14, 2, 5, 4, 0);

const char* ssid = "Gsgs";
const char* password = "36503650";

const int BUZZER_PIN = 15;

ESP8266WebServer server(80);
String page = "";
char input[13]; // Increased array size by 1 for null termination
int count = 0;
int current_weight = 0;
int upper_limit = 0;
int lower_limit = 0;
float tolerance = 0.25;
int p1 = 0; // For sugar
int p2 = 0; // For Soap
double total = 0;
int count_prod = 0;
int sugar_weight = 250;
int soap_weight = 100;
int total_weight = 0;
int buttonPin = 16; // GPIO15, change as needed
bool removeItemRequested = false;
//unsigned long lastButtonPress = 0;

// HardwareSerial RFID(3); // RX, TX
#define EM18_RX_PIN 13  // GPIO4 (D2 on NodeMCU)
#define EM18_TX_PIN 5  // GPIO5 (D1 on NodeMCU, not used, but needed for SoftwareSerial declaration)

SoftwareSerial RFID(EM18_RX_PIN, EM18_TX_PIN); // RX, TX

void setup() {
  Serial.begin(9600);
  RFID.begin(9600); // Initialize SoftwareSerial for the EM-18 reader
  pinMode(buttonPin, INPUT_PULLUP); // Button pin as input with internal pull-up resistor
  pinMode(BUZZER_PIN, OUTPUT);
  lcd.begin(16, 2);
  lcd.setCursor(0, 0);
  lcd.print("Welcome TO");
  lcd.setCursor(0, 1);
  lcd.print("INTELLI BASKET");
  delay(2000);
  lcd.clear();

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  // Serial.println("");
  // Serial.println("WiFi connected");

  // Serial.println(WiFi.localIP());
  lcd.setCursor(0, 0);
  lcd.print("WiFi Connected");
  lcd.setCursor(0, 1);
  lcd.print(WiFi.localIP());
  delay(2000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(" PLZ SCAN ITEMS");
  lcd.setCursor(0, 1);
  lcd.print("    TO CART");

  server.on("/", [](){
    page = "<html><head><title>Smart Shopping Basket</title></head><style type=\"text/css\">";
    page += "table{border-collapse: collapse;}th {background-color: #FFC0CB ;color: white;}table,td {border: 4px solid black;font-size: x-large;";
    page += "text-align:center;border-style: groove;border-color: rgb(176,224,230);}</style><body><center>";
    page += "<h1>Welcome To INTELLI-BASKET</h1><br><br><table style=\"width: 1200px;height: 450px;\"><tr>";
    page += "<th>ITEMS</th><th>QUANTITY</th><th>COST</th></tr><tr><td>Sugar</td><td>" + String(p1) + "</td><td>" + String(p1 * 35.00) + "</td></tr>";
    page += "<tr><td>Soap</td><td>" + String(p2) + "</td><td>" + String(p2 * 25.00) + "</td></tr>";
    page += "</table><br><input type=\"button\" name=\"Pay Online Now\" value=\"Pay Online Now\" style=\"width: 200px;height: 50px\"></center></body></html>";
    page += "<meta http-equiv=\"refresh\" content=\"2\">";
    server.send(200, "text/html", page);
  });
  server.begin();
}

void loop() {
  if (RFID.available() ) {
    count = 0;
    while (RFID.available() && count < 12) {
      input[count] = RFID.read();
      count++;
      delay(5);
      // Serial.print("read");
    }
    
    input[count] = '\0'; // Null terminate the input array
    if(digitalRead(buttonPin) == HIGH){
    if (count == 12 ) {
      // Serial.println("RFID tag read: " + String(input));
      if ((strncmp(input, "5000DB03FD75", 12) == 0)) {
        // Serial.print("5000DAC31950");
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Sugar Added");
        lcd.setCursor(0, 1);
        lcd.print("Price(Rs): 35.00");
        p1++;
        total_weight = total_weight + sugar_weight;
        delay(2000);
        total += 35.00;
        count_prod++;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Total Price:");
        lcd.setCursor(0, 1);
        lcd.print(total);
      } else if ((strncmp(input, "5000DB468449", 12) == 0)) { // Soap RFID tag
      // Serial.print("5000DAD01C46");      
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Soap Added");
        lcd.setCursor(0, 1);
        lcd.print("Price(Rs): 25.00");
        p2++;
        total_weight = total_weight + soap_weight;
        delay(2000);
        total += 25.00;
        count_prod++;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Total Price:");
        lcd.setCursor(0, 1);
        lcd.print(total);
      } else if ((strncmp(input, "5000DADBE3B2", 12) == 0)) { // Third RFID tag
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Go for Checkout");
        delay(1000);
        while (!Serial) {
          ; // Wait for the serial port to connect. Needed for native USB
        }
        if (Serial.available() > 0) {
          // Read the incoming string:
          String incomingString = Serial.readString();
          
          // Clear the LCD screen
          lcd.clear();
          
          current_weight = incomingString.toFloat();
          lower_limit = total_weight - (total_weight * tolerance); // 25% as a decimal is 0.25
          upper_limit = total_weight + (total_weight * tolerance); // 25% as a decimal is 0.25

          if (current_weight >= lower_limit && current_weight <= upper_limit){
            lcd.clear();
            lcd.setCursor(0, 0);
            lcd.print("Ready to");
            lcd.setCursor(0, 1);
            lcd.print("CheckOut");
            delay(3000);
            lcd.clear();
            lcd.setCursor(0, 0);
            lcd.print("Total price:");
            lcd.setCursor(0, 1);
            lcd.print(total);
          }
          else if (current_weight < lower_limit){
            lcd.clear();
            lcd.setCursor(0, 0);
            lcd.print("Item missing");
            lcd.setCursor(0, 1);
            lcd.print("in cart.");
             int toneFrequency[] = {1000, 1500, 2000, 1000, 1500, 2000, 1000, 1500, 2000}; // You can adjust these frequencies
  
              // Duration of each tone (in milliseconds)
              int toneDuration = 100; // Adjust as needed
              
              // Loop through each tone and play it
              for (int i = 0; i < 9; i++) {
                tone(BUZZER_PIN, toneFrequency[i]); // Start playing the tone
                delay(toneDuration); // Wait for the duration of the tone
                noTone(BUZZER_PIN); // Stop playing the tone
                delay(50); // Add a short delay between tones
              }
          }
          else if (current_weight > upper_limit){
            lcd.clear();
            lcd.setCursor(0, 0);
            lcd.print("Item unscanned");
            lcd.setCursor(0, 1);
            lcd.print("in cart.");
             int toneFrequency[] = {1000, 1500, 2000, 1000, 1500, 2000, 1000, 1500, 2000}; // You can adjust these frequencies
  
              // Duration of each tone (in milliseconds)
              int toneDuration = 100; // Adjust as needed
              
              // Loop through each tone and play it
              for (int i = 0; i < 9; i++) {
                tone(BUZZER_PIN, toneFrequency[i]); // Start playing the tone
                delay(toneDuration); // Wait for the duration of the tone
                noTone(BUZZER_PIN); // Stop playing the tone
                delay(50); // Add a short delay between tones
              }
          }
        }
        
        // if (total_weight <= scale.get_units(10)-scale.get_units(10)*)
      } else {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print(" PLZ SCAN ITEMS");
        lcd.setCursor(0, 1);
        lcd.print("   TO CART");
      }
    } else {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print(" PLZ SCAN ITEMS");
      lcd.setCursor(0, 1);
      lcd.print("   TO CART");
    }
  
    }
  // Check if the button is pressed


else if(digitalRead(buttonPin) == LOW) {

  if (digitalRead(buttonPin) == LOW) {
    removeItemRequested = true;
    delay(200); // Debouncing delay
  }

  // Handle removing item when removeItemRequested is true
if (removeItemRequested) {
  // Remove the scanned item based on RFID tag
  if (strncmp(input, "5000DB03FD75", 12) == 0 && p1 > 0) {

    p1--;
    total_weight = total_weight - sugar_weight;
    total -= 35.00;
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Sugar Removed");
    lcd.setCursor(0, 1);
    lcd.print("Total Price:");
    lcd.print(total);
  } else if (strncmp(input, "5000DB468449", 12) == 0 && p2 > 0) {
    p2--;
    total_weight = total_weight - soap_weight;
    total -= 25.00;
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Soap Removed");
    lcd.setCursor(0, 1);
    lcd.print("Total Price:");
    lcd.print(total);
  }
  // count = 0;
  removeItemRequested = false;
}

}


}
server.handleClient();}

