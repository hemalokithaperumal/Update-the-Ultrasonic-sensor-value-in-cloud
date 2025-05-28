# Update-the-Ultrasonic-sensor-value-in-Things Mate

# AIM:
To upload the Ultrasonic sensor value in the Things mate using Arduino controller.

# Apparatus required:
Arduino Controller  </br>
Indoor gateway</br>
LoRaWAN shield </br>
HC-SR04 Ultrasonic sensor module </br>
Power supply </br>
Connecting wires </br>
Bread board </br>

# PROCEDURE:

### Procedure for gateway setup
	Go to the link http://172.31.255.254:8000 </br>
	Type the user name password </br>
	Go to LoRa and set the frequency plan 865 Mhz </br>
	In LoRa WAN configurations enter the Gate EUI and server address </br>
	Enable the Internet </br>
	Select the Wifi access point </br>
	Type the ssid and password in wifi LAN setting </br>
	Select the internet and IoT service and provide the details. </br>
	Check all the green colour tick marks in gate way and proceed </br>
### Procedure for gateway registration in The thingsMate LoRaWAN Management </br>
	Login https://iot.saveetha.in:4433 and provide the user id and password </br>
	Go to overview and select application server </br>
	Go to gateway and add new gateway </br>
	Enter the gateway id, name, EUI and select frequency plan </br>
	Open Arduino IDE and type the program for the given application </br>
	Compile the program if no error uploads the program in the controller </br>
	Go to gateways in things mate and check the live data </br>
	Create channel by giving channel name and ID </br>
	Add end device and enter the frequency plan, DevEUI, AppEUI, APP key, Select LoRaWAN version </br>
	Enter payload formatters </br>
	Go to the option query and give the new query name </br>
	Go to the option Dashboard and verify the output.</br>  

# THEORY:

### What is IoT?

Internet of Things (IoT) describes an emerging trend where a large number of embedded devices (things) are connected to the Internet. These connected devices communicate with people and other things and often provide sensor data to cloud storage and cloud computing resources where the data is processed and analyzed to gain important insights. Cheap cloud computing power and increased device connectivity is enabling this trend.IoT solutions are built for many vertical applications such as environmental monitoring and control, health monitoring, vehicle fleet monitoring, industrial monitoring and control, and home automation

![image](https://user-images.githubusercontent.com/71547910/235334044-c01d4261-d46f-4f62-b07f-72a7b6fce5d5.png)

### What is LoRaWAN

The LoRaWAN® specification is a Low Power, Wide Area (LPWA) networking protocol designed to wirelessly connect battery operated ‘things’ to the internet in regional, national or global networks, and targets key Internet of Things (IoT) requirements such as bi-directional communication, end-to-end security, mobility and localization services.LoRaWAN® network architecture is deployed in a star-of-stars topology in which gateways relay messages between end-devices and a central network server. The gateways are connected to the network server via standard IP connections and act as a transparent bridge, simply converting RF packets to IP packets and vice versa. The wireless communication takes advantage of the Long Range characteristics of the LoRaÒ physical layer, allowing a single-hop link between the end-device and one or many gateways. All modes are capable of bi-directional communication, and there is support for multicast addressing groups to make efficient use of spectrum during tasks such as Firmware Over-The-Air (FOTA) upgrades or other mass distribution messages.

The specification defines the device-to-infrastructure (LoRa®) physical layer parameters & (LoRaWAN®) protocol and so provides seamless interoperability between manufacturers, as demonstrated via the device certification program.While the specification defines the technical implementation, it does not define any commercial model or type of deployment (public, shared, private, enterprise) and so offers the industry the freedom to innovate and differentiate how it is used.The LoRaWAN® specification is developed and maintained by the LoRa Alliance®: an open association of collaborating members.

![image](https://github.com/anishkumar-Embedded/Update-the-Ultrasonic-sensor-value-in-cloud/assets/71547910/c63c4edd-3b95-4a69-b9c2-4862afb335c3)

### Characteristics of LoRaWAN technology
Long range communication up to 10 miles in line of sight.
Long battery duration of up to 10 years. For enhanced battery life, you can operate your devices in class A or class B mode, which requires increased downlink latency.
Low cost for devices and maintenance.
License-free radio spectrum but region-specific regulations apply.
Low power but has a limited payload size of 51 bytes to 241 bytes depending on the data rate. The data rate can be 0,3 Kbit/s – 27 Kbit/s data rate with a 222 maximal payload size.

### The Things Mate - IoT Cloud Platform

IoT cloud platforms play a pivotal role in the development and deployment of Internet of Things (IoT) applications, connecting devices and enabling seamless data management and analysis. These platforms typically offer a comprehensive suite of services, including device provisioning, secure connectivity, data storage, and advanced analytics.Leading IoT cloud platforms, such as AWS IoT, Azure IoT, and Google Cloud IoT, provide scalable and reliable infrastructure to accommodate diverse IoT deployments. They facilitate device management, allowing users to monitor, update, and control connected devices remotely. Security features are integral, ensuring data integrity and safeguarding against potential threats.

Analytics capabilities enable organizations to derive meaningful insights from the vast amounts of data generated by IoT devices. Machine learning and artificial intelligence integrations further enhance predictive analytics, enabling proactive decision-making.These platforms often offer APIs for seamless integration with other cloud services, supporting a wide range of industries and applications, from smart homes to industrial automation. As the IoT landscape evolves, cloud platforms continue to innovate, contributing to the growth and sophistication of IoT ecosystems worldwide. Choosing the right IoT cloud platform involves considering factors such as scalability, security, and compatibility with specific use cases.

# PROGRAM:
```
#include <SoftwareSerial.h>

// Pin configuration for ultrasonic sensor
#define TRIG_PIN 7
#define ECHO_PIN 8

SoftwareSerial ss(10, 11); // RX, TX to LA66

String inputString = "";
bool stringComplete = false;

long old_time = millis();
long new_time;
long uplink_interval = 30000;

bool time_to_at_recvb = false;
bool get_LA66_data_status = false;
bool network_joined_status = false;

char rxbuff[128];
uint8_t rxbuff_index = 0;
char Downlink_Data;

uint16_t Distance = 0;
uint8_t LoRa_Uplink_Buffer[2]; // 2 bytes for Distance

void setup() {
  Serial.begin(9600);
  ss.begin(9600);
  ss.listen();

  pinMode(6, OUTPUT); // onboard LED
  inputString.reserve(200);

  pinMode(TRIG_PIN, OUTPUT);  // Initialize the TRIG_PIN
  pinMode(ECHO_PIN, INPUT);   // Initialize the ECHO_PIN

  ss.println("ATZ");   // Reset LA66
}

void loop() {
  new_time = millis();
  if ((new_time - old_time >= uplink_interval) && network_joined_status) {
    old_time = new_time;
    get_LA66_data_status = false;
    sensorread();

    // Fill buffer
    LoRa_Uplink_Buffer[0] = highByte(Distance);
    LoRa_Uplink_Buffer[1] = lowByte(Distance);

    // Build HEX payload string
    String payload = "";
    for (int i = 0; i < 2; i++) {
      if (LoRa_Uplink_Buffer[i] < 0x10) payload += "0";
      payload += String(LoRa_Uplink_Buffer[i], HEX);
    }

    ss.println("AT+SENDB=0,4,2," + payload);
    Serial.println("Sent Payload: " + payload);
  }

  if (time_to_at_recvb) {
    time_to_at_recvb = false;
    get_LA66_data_status = true;
    delay(1000);
    ss.println("AT+CFG");
  }

  while (ss.available()) {
    char inChar = (char)ss.read();
    inputString += inChar;
    rxbuff[rxbuff_index++] = inChar;
    if (rxbuff_index > 127) break;

    if (inChar == '\n' || inChar == '\r') {
      stringComplete = true;
      rxbuff[rxbuff_index] = '\0';

      if (strncmp(rxbuff, "JOINED", 6) == 0) network_joined_status = true;
      if (strncmp(rxbuff, "Dragino LA66 Device", 19) == 0) network_joined_status = false;

      if (strncmp(rxbuff, "Run AT+RECVB=? to see detail", 28) == 0) {
        time_to_at_recvb = true;
        stringComplete = false;
        inputString = "\0";
      }

      if (strncmp(rxbuff, "AT+RECVB=", 9) == 0) {
        stringComplete = false;
        inputString = "\0";
        Serial.print("\nGet downlink data (FPort & Payload): ");
        Serial.println(&rxbuff[12]);
        Downlink_Data = rxbuff[12];
        if (Downlink_Data == '1') 
        { 
        Serial.println("LED_ON"); 
        digitalWrite(6, HIGH); 
        }
        else if (Downlink_Data == '2') 
        { 
          Serial.println("LED_OFF"); 
          digitalWrite(6, LOW); 
        }
      }

      rxbuff_index = 0;
      if (get_LA66_data_status) {
        stringComplete = false;
        inputString = "\0";
      }
    }
  }

  while (Serial.available()) {
    char inChar = (char)Serial.read();
    inputString += inChar;
    if (inChar == '\n' || inChar == '\r') {
      ss.print(inputString);
      inputString = "\0";
    }
  }

  if (stringComplete) {
    Serial.print(inputString);
    inputString = "\0";
    stringComplete = false;
  }
}

void sensorread() {
  // Trigger the ultrasonic pulse
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Measure the pulse duration
  long duration = pulseIn(ECHO_PIN, HIGH);

  // Calculate distance in cm (speed of sound is 34300 cm/s)
  Distance = duration * 0.0344 / 2; // Divide by 2 because the pulse travels to the object and back

  // Print readable values
  Serial.print("Distance (cm): ");
  Serial.println(Distance);
}
```


# OUTPUT:

![image](https://github.com/user-attachments/assets/79cf76af-9d38-4278-9683-a55097f9efeb)

![Screenshot 2025-05-24 143627](https://github.com/user-attachments/assets/0221d2b8-a3cb-494d-889b-d6b946bad09f)



![image](https://github.com/user-attachments/assets/85385a95-3e80-4f76-bf5d-d1d4824219f2)


![image](https://github.com/user-attachments/assets/fca8e7ec-1468-46d6-a849-676069b4daf8)

# RESULT:

Thus the Ultrasonic sensor value is uploaded in the Things mate using Arduino controller.

