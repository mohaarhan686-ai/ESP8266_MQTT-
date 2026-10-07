# 🔐 ESP8266 & ESP32 MQTT Communication with HiveMQ Cloud

A professional IoT communication project demonstrating **secure MQTT communication between ESP8266/ESP32 devices and HiveMQ Cloud** using **MQTT over TLS on port 8883**.

The project implements both **MQTT publishing and subscribing**, allowing an ESP32 to send sensor/data messages to the cloud and receive commands remotely.

---

## 📌 Project Overview

This project demonstrates how to connect an **ESP8266 and ESP32** to **HiveMQ Cloud** using:

* Wi-Fi connectivity
* MQTT protocol
* HiveMQ Cloud MQTT broker
* TLS/SSL encrypted connection
* Username/password authentication
* MQTT Publish/Subscribe communication
* Remote GPIO/LED control

### Communication Flow

```text
                    Internet
                       │
                       │ MQTT over TLS
                       │ Port 8883
                       ▼
              ┌──────────────────┐
              │   HiveMQ Cloud    │
              │   MQTT Broker     │
              └────────┬─────────┘
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
      ┌─────────────┐     ┌─────────────┐
      │   ESP8266   │     │    ESP32    │
      │             │     │             │
      │ Publisher   │     │ Pub/Sub     │
      └─────────────┘     └──────┬──────┘
                                 │
                                 ▼
                              GPIO 2
                                 │
                                LED
```

---

# 🧩 Hardware

### Supported Boards

* ESP8266 NodeMCU
* ESP32 Development Board

### Required Hardware

| Component       | Quantity |
| --------------- | -------: |
| ESP8266 NodeMCU |        1 |
| ESP32 Board     |        1 |
| LED             |        1 |
| 220Ω Resistor   |        1 |
| USB Cable       |        1 |
| Wi-Fi Network   |        1 |

The ESP32 code uses **GPIO 2** for LED control.

---

# 💻 Software & Libraries

## Arduino IDE

The project can be developed using the Arduino IDE.

### Required Libraries

```text
ESP8266WiFi
WiFi
WiFiClientSecure
PubSubClient
```

### Library Roles

| Library            | Purpose                        |
| ------------------ | ------------------------------ |
| `ESP8266WiFi`      | Wi-Fi connectivity for ESP8266 |
| `WiFi`             | Wi-Fi connectivity for ESP32   |
| `WiFiClientSecure` | Secure TCP/TLS connection      |
| `PubSubClient`     | MQTT client implementation     |

---

# ☁️ HiveMQ Cloud

This project uses **HiveMQ Cloud** as the MQTT broker.

HiveMQ acts as the central MQTT server through which devices exchange messages.

```text
ESP8266 ───────► HiveMQ Cloud ◄─────── ESP32
                    │
                    │
              MQTT Broker
```

The broker handles:

* Client connections
* Authentication
* MQTT topics
* Message publishing
* Message subscriptions
* Message routing

---

# 🔐 MQTT over TLS

The project uses:

```text
MQTT Port: 8883
```

Port `8883` is commonly used for **MQTT over TLS/SSL**.

The connection architecture is:

```text
Application
     │
     ▼
    MQTT
     │
     ▼
   TLS/SSL
     │
     ▼
    TCP
     │
     ▼
  Internet
```

This provides encrypted communication between the IoT device and HiveMQ Cloud.

---

# ⚠️ Current TLS Configuration

For initial testing, both programs use:

```cpp
secureClient.setInsecure();
```

or:

```cpp
espClient.setInsecure();
```

This disables certificate verification.

### Important

`setInsecure()` is acceptable for **testing and development**, but it is **not recommended for production deployments**.

For a production IoT system, the HiveMQ CA certificate should be installed and verified properly.

---

# 📡 ESP8266 MQTT Client

The ESP8266 program connects to Wi-Fi and then establishes an MQTT connection with HiveMQ Cloud.

### Main Configuration

```cpp
const char* mqtt_server = "YOUR_HIVEMQ_HOST";
const int mqtt_port = 8883;

const char* mqtt_user = "YOUR_MQTT_USERNAME";
const char* mqtt_pass = "YOUR_MQTT_PASSWORD";
```

### MQTT Client

```cpp
WiFiClientSecure espClient;
PubSubClient client(espClient);
```

Here:

* `WiFiClientSecure` provides the secure network connection.
* `PubSubClient` handles MQTT communication.

### MQTT Connection

```cpp
client.connect("ESP8266-Test", mqtt_user, mqtt_pass)
```

The ESP8266 connects using:

* Client ID: `ESP8266-Test`
* MQTT username
* MQTT password

After successful connection, it publishes:

```text
Topic:
test/hello

Message:
Hello from ESP8266
```

---

# 📤 ESP8266 Publishing

The ESP8266 performs a simple MQTT publish:

```cpp
client.publish("test/hello", "Hello from ESP8266");
```

Conceptually:

```text
ESP8266
   │
   │ Publish
   ▼
test/hello
   │
   ▼
HiveMQ Cloud
```

Any MQTT client subscribed to `test/hello` can receive this message.

---

# 📡 ESP32 MQTT Client

The ESP32 implementation is more advanced.

It supports:

* Wi-Fi connection
* MQTT connection
* MQTT authentication
* Publishing
* Subscribing
* MQTT callback
* Remote LED control
* Automatic reconnection

---

# 📤 ESP32 Publisher

The ESP32 publishes data every **5 seconds**.

Topic:

```text
home/esp32/data
```

Message:

```text
Hello From ESP32
```

The publishing interval is controlled using:

```cpp
if (millis() - lastPublish > 5000)
```

This avoids using a blocking `delay()` for periodic publishing.

---

# 📥 ESP32 Subscriber

The ESP32 subscribes to:

```text
home/esp32/cmd
```

When a message arrives, the MQTT callback function is executed:

```cpp
void mqttCallback(char* topic, byte* payload, unsigned int length)
```

The incoming MQTT payload is converted into a `String`.

---

# 💡 Remote LED Control

The ESP32 listens for commands on:

```text
home/esp32/cmd
```

### Turn LED ON

Publish:

```text
ON
```

The ESP32 executes:

```cpp
digitalWrite(2, HIGH);
```

### Turn LED OFF

Publish:

```text
OFF
```

The ESP32 executes:

```cpp
digitalWrite(2, LOW);
```

### Command Flow

```text
MQTT Client
     │
     │ "ON"
     ▼
HiveMQ Cloud
     │
     ▼
home/esp32/cmd
     │
     ▼
ESP32
     │
     ▼
GPIO 2 HIGH
     │
     ▼
LED ON
```

---

# 🔄 Complete ESP32 Communication

The ESP32 simultaneously performs both publishing and subscribing.

```text
                 HiveMQ Cloud
                MQTT Broker
                /          \
               /            \
              ▼              ▼
     home/esp32/data    home/esp32/cmd
              ▲              │
              │              │
              │              ▼
           ESP32          ESP32
           Publish       Subscribe
              │              │
              ▼              ▼
       "Hello From ESP32"   ON/OFF
```

---

# 🏷️ MQTT Topics

| Direction | Topic             | Purpose                |
| --------- | ----------------- | ---------------------- |
| Publish   | `test/hello`      | ESP8266 test message   |
| Publish   | `home/esp32/data` | ESP32 data/message     |
| Subscribe | `home/esp32/cmd`  | ESP32 control commands |

---

# 🔑 MQTT Authentication

HiveMQ Cloud requires authentication credentials.

The ESP32/ESP8266 sends:

```text
MQTT Username
       +
MQTT Password
       +
Client ID
```

during the MQTT connection process.

Example:

```cpp
mqtt.connect(
    "ESP32_Client",
    MQTT_USER,
    MQTT_PASS
);
```

---

# 🌐 Wi-Fi Connection

Both devices first connect to the configured Wi-Fi network.

Example:

```cpp
WiFi.begin(WIFI_SSID, WIFI_PASS);
```

The program waits until the connection is established:

```cpp
while (WiFi.status() != WL_CONNECTED)
{
    delay(500);
    Serial.print(".");
}
```

After connection, the device prints its local IP address.

Example:

```text
WiFi Connected
IP : 192.168.1.105
```

---

# 🔁 Automatic MQTT Reconnection

The ESP32 continuously checks whether MQTT is connected.

```cpp
if (!mqtt.connected())
    connectMQTT();
```

If the connection fails, the program waits 3 seconds and tries again:

```cpp
delay(3000);
```

This makes the system more reliable against temporary network or broker disconnections.

---

# 📊 MQTT Connection States

If the connection fails, the program prints:

```text
Failed, State = X
```

`PubSubClient` uses different return codes to indicate the reason for failure.

Common examples include:

| State | Meaning                                                            |
| ----: | ------------------------------------------------------------------ |
|   `0` | Connected                                                          |
|  `-2` | MQTT connect failed                                                |
|  `-4` | Connection timeout                                                 |
|  `-1` | Connection refused / protocol-related failure depending on context |

Always verify the broker hostname, port, credentials, Wi-Fi and TLS configuration when troubleshooting.

---

# 🖥️ Serial Monitor

Set the Arduino Serial Monitor to:

```text
115200 baud
```

A successful ESP32 connection should look similar to:

```text
Connecting WiFi.....
WiFi Connected
IP : 192.168.1.xxx

Connecting MQTT...Connected
Subscribed : home/esp32/cmd

Published : Hello From ESP32
```

When a command is received:

```text
-------------------------
Topic : home/esp32/cmd
Message : ON
LED ON
```

---

# 🧪 Testing with an MQTT Client

You can test the system using any MQTT client that supports:

```text
Host       : Your HiveMQ Cloud host
Port       : 8883
Protocol   : MQTT
Security   : TLS
Username   : Your HiveMQ username
Password   : Your HiveMQ password
```

### Test ESP8266

Subscribe to:

```text
test/hello
```

Expected message:

```text
Hello from ESP8266
```

### Test ESP32 Data

Subscribe to:

```text
home/esp32/data
```

Expected message:

```text
Hello From ESP32
```

### Test ESP32 LED

Publish:

```text
Topic: home/esp32/cmd
Message: ON
```

The ESP32 LED should turn ON.

Then publish:

```text
Topic: home/esp32/cmd
Message: OFF
```

The LED should turn OFF.

---

# 🔒 Security Recommendations

Do **not** upload real credentials to GitHub.

Avoid committing:

```cpp
const char* WIFI_PASS = "real-password";
const char* MQTT_USER = "real-user";
const char* MQTT_PASS = "real-password";
```

Instead use placeholders:

```cpp
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";

const char* MQTT_HOST = "YOUR_HIVEMQ_HOST";
const char* MQTT_USER = "YOUR_MQTT_USERNAME";
const char* MQTT_PASS = "YOUR_MQTT_PASSWORD";
```

### Recommended Production Improvements

* Use proper CA certificate validation.
* Avoid `setInsecure()`.
* Store credentials outside the main source file.
* Use unique MQTT client IDs.
* Use access-control rules on the broker.
* Use separate credentials for different devices.
* Avoid exposing MQTT credentials in public repositories.

---

# 📁 Suggested Project Structure

```text
ESP-MQTT-HiveMQ/
│
├── ESP8266/
│   └── ESP8266_HiveMQ_MQTT.ino
│
├── ESP32/
│   └── ESP32_HiveMQ_MQTT.ino
│
├── README.md
│
└── .gitignore
```

---

# 🚀 Future Improvements

This project can be extended into a complete IoT platform.

### Sensors

```text
ESP32
  │
  ├── Temperature
  ├── Humidity
  ├── Motion
  ├── Distance
  └── Gas Sensors
          │
          ▼
      MQTT Broker
          │
          ▼
      Cloud Dashboard
```

### Possible Additions

* DHT22/BME280 sensor monitoring
* Relay control
* Motor control
* Smart-home automation
* Real-time dashboards
* Node-RED integration
* MQTT retained messages
* Last Will and Testament
* QoS configuration
* OTA firmware updates
* Device authentication
* Proper TLS certificate validation
* Database/cloud data logging

---

# 🧠 Technologies Used

```text
ESP8266
ESP32
Arduino
C/C++
Wi-Fi
MQTT
HiveMQ Cloud
TLS/SSL
PubSubClient
IoT
```

---

# 📜 License

This project is intended for **educational, research and IoT development purposes**.

You may modify and extend the project according to your requirements.

---

# 👨‍💻 Author

**Mohd Arhan**

Mechanical & Robotics Engineering
IoT • Robotics • Embedded Systems • Automation

---

## ⭐ Project Highlights

```text
✓ ESP8266 MQTT Client
✓ ESP32 MQTT Client
✓ HiveMQ Cloud
✓ MQTT over TLS
✓ Username/Password Authentication
✓ MQTT Publish
✓ MQTT Subscribe
✓ Callback Handling
✓ Remote GPIO Control
✓ Automatic MQTT Reconnection
✓ IoT Cloud Communication
```

**Built for learning, experimentation and real-world IoT development.**
