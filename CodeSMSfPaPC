#include <ESP8266WiFi.h>
#include <DHT.h>

// WiFi credentials
const char* ssid = "ABCDEF";
const char* password = "abc123";

// DHT sensor setup (Temperature and Humidity Sensor)
#define DHTPIN D2          
#define DHTTYPE DHT11      
DHT dht(DHTPIN, DHTTYPE);

// Soil Moisture Sensor setup
#define MOISTURE_PIN A0    

// PIR Motion Sensor setup
#define PIR_PIN D5         

// Photoresistor (LDR) setup
#define LDR_PIN A0         

// LED control setup 
#define LED_PIN D6          // LED connected to GPIO D6 (replace with any available GPIO pin)

WiFiServer server(80);

// Variable to hold the LED state (0 = off, 1 = on)
int ledState = LOW;

void setup() {
  Serial.begin(115200);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  dht.begin();
  pinMode(PIR_PIN, INPUT);
  pinMode(LED_PIN, OUTPUT);  // Set the LED pin as an output
  
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    String currentLine = "";
    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        currentLine += c;
        if (c == '\n') {
          if (currentLine.startsWith("GET /")) {
            // Check for fan toggle button press
            if (currentLine.indexOf("GET /toggle_led") >= 0) {
              ledState = (ledState == LOW) ? HIGH : LOW; // Toggle the LED state
              digitalWrite(LED_PIN, ledState);  // Toggle the LED state
            }
            sendDashboard(client);
            break;
          }
        }
      }
    }
    client.stop();
  }
}

// Function to send the HTML Dashboard
void sendDashboard(WiFiClient& client) {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  int soilMoisture = analogRead(MOISTURE_PIN);
  int lightLevel = analogRead(LDR_PIN);
  int motionDetected = digitalRead(PIR_PIN);

  // Start of the HTTP response
  client.print("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n");
  client.print("<html><head>");

  // Meta tag for auto-refresh every 5 seconds
  client.print("<meta http-equiv='refresh' content='5'>");

  // CSS for styling
  client.print("<style>");
  client.print("body {");
  client.print("font-family: Arial, sans-serif;");
  client.print("text-align: center;");
  client.print("color: white;");
  client.print("background: url('https://i.imgur.com/N8HW82H.png') no-repeat center center fixed;");
  client.print("background-size: cover;");
  client.print("margin: 0;");
  client.print("padding: 10px 0;");
  client.print("height: 100vh;");
  client.print("}");
  client.print(".content {");
  client.print("background: rgba(0, 0, 0, 0.6);");
  client.print("padding: 30px;");
  client.print("border-radius: 15px;");
  client.print("display: inline-block;");
  client.print("margin-top: 100px;");
  client.print("width: 80%;");
  client.print("max-width: 600px;");
  client.print("box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.5);");
  client.print("}");
  client.print("h1 {");
  client.print("margin-bottom: 20px;");
  client.print("font-size: 28px;");
  client.print("font-weight: bold;");
  client.print("}");
  client.print("p {");
  client.print("margin: 10px 0;");
  client.print("font-size: 20px;");
  client.print("line-height: 1.5;");
  client.print("}");
  client.print("button {");
  client.print("padding: 10px 20px;");
  client.print("font-size: 18px;");
  client.print("cursor: pointer;");
  client.print("border: none;");
  client.print("background-color: #4CAF50;");
  client.print("color: white;");
  client.print("border-radius: 5px;");
  client.print("margin-top: 20px;");
  client.print("}");
  client.print("button:hover {");
  client.print("background-color: #45a049;");
  client.print("}");
  client.print("</style>");
  client.print("</head><body>");

  // HTML content (dashboard)
  client.print("<div class='content'>");
  client.print("<h1>Smart Pet and Plant Care System</h1>");
  client.print("<p><strong>Temperature:</strong> " + String(temperature) + "&deg;C</p>");
  client.print("<p><strong>Humidity:</strong> " + String(humidity) + "%</p>");
  client.print("<p><strong>Soil Moisture:</strong> " + String(soilMoisture) + "</p>");
  client.print("<p><strong>Light Level:</strong> " + String(lightLevel) + "</p>");
  client.print("<p><strong>Motion Detected:</strong> " + String(motionDetected == HIGH ? "Yes" : "No") + "</p>");

  // Add the toggle LED button
  client.print("<p><a href='/toggle_led'><button>Toggle LED</button></a></p>");
  
  client.print("<p><i>Page refreshes every 5 seconds</i></p>");
  client.print("</div>");

  // Closing the body and html tags
  client.print("</body></html>");

}
