# EverFarm Project

## Introduction

**EverFarm** is an initiative to assist smallholder farmers in adopting sustainable agricultural practices through data-driven decision-making and efficient resource management. The project addresses challenges like lack of real-time data, inefficient resource management, difficulty in early detection of crop health issues, limited ability to adapt to climate change, and financial vulnerability to extreme weather events.

### Challenges and Solutions

#### Sample Use Case A: Lack of Data-Driven Decision Making

- **Solution: Crop yield prediction**
  - Machine learning models analyze historical data, weather forecasts, and real-time farm data to predict crop yields, optimize planting, resource allocation, and risk mitigation.

- **Solution: IoT-based farm monitoring systems & Real-time farm data collection and analysis**
  - Sensor networks capture real-time farm data (soil moisture, temperature, light) for analysis. This data provides insights into soil health, irrigation needs, and crop growth, enabling data-driven decisions and efficient resource management.

#### Sample Use Case B: Inefficient Resource Management

- **Solution: Water management for precision drip irrigation**
  - AI analyzes historical water usage and real-time sensor data (soil moisture, weather) to optimize drip irrigation schedules for individual zones within a field. This ensures crops receive the precise amount of water needed, minimizing waste and maximizing water use efficiency.

- **Solution: Remote monitoring and control of irrigation systems**
  - Mobile apps or web dashboards allow farmers to remotely monitor water flow, pressure, and valve status in their irrigation systems. This enables them to adjust schedules, identify issues (leaks, blockages), and control water usage from anywhere with an internet connection.

#### Sample Use Case C: Difficulty in Early Detection of Crop Health Issues

- **Solution: Crop health diagnostics**
  - AI analyzes drone images, sensor data, or field observations to detect crop stress, pests, and diseases early. This enables targeted interventions and reduces reliance on harsh chemicals.

#### Sample Use Case D: Limited Ability to Adapt to Climate Change

- **Solution: Decision support systems for climate adaptation**
  - AI analyzes historical climate data, weather forecasts, and soil conditions to recommend crop selection, planting times, and resource allocation based on predicted weather patterns, helping farmers adapt to a changing climate.

#### Sample Use Case E: Financial Vulnerability to Extreme Weather Events

- **Solution: Blockchain-powered climate risk insurance**
  - This platform collects and analyzes weather data. Based on pre-defined weather events (drought, floods), it leverages blockchain technology to automate insurance payouts to farmers, providing financial security and mitigating losses from unpredictable weather.

## Hardware Components

- **Raspberry Pi 4 (or 3 Model B+)**
- **DHT22 (Temperature and Humidity Sensor)**
- **YL-69 (Soil Moisture Sensor)**
- **BH1750 (Light Sensor)**
- **YF-S201 (Water Flow Sensor)**
- **MCP3008 (Analog to Digital Converter for YL-69)**

## Software Setup

### Installing Required Libraries

```bash
sudo apt-get update
sudo apt-get install python3-pip
sudo pip3 install Adafruit_DHT RPi.GPIO smbus2 bh1750 py-spidev firebase-admin
```

### Setting Up Firebase

1. **Create a Firebase Project:**
   - Go to the [Firebase Console](https://console.firebase.google.com/).
   - Click on "Add Project" and follow the setup instructions.

2. **Add Firebase to Your App:**
   - Go to the Project Overview and click on the web icon (“Add app”).
   - Register your app and follow the steps to get the Firebase configuration.

3. **Enable Firebase Realtime Database:**
   - In the Firebase Console, navigate to "Database" and click on "Create Database" under the Realtime Database section.
   - Select a location and set the database to start in test mode (for development purposes).

### Setting Up Firebase Admin SDK on Raspberry Pi

1. **Install the Firebase Admin SDK:**
   ```bash
   pip3 install firebase-admin
   ```

2. **Initialize Firebase in Your Python Script:**
   - Download the service account key from Firebase Console (Project Settings > Service Accounts > Generate New Private Key).
   - Save the JSON file to your Raspberry Pi.

### Python Script for Data Collection and Cloud Storage

```python
import Adafruit_DHT
import spidev
import time
import smbus2
import RPi.GPIO as GPIO
import firebase_admin
from firebase_admin import credentials, db

# Firebase setup
cred = credentials.Certificate('path/to/your/serviceAccountKey.json')
firebase_admin.initialize_app(cred, {
    'databaseURL': 'https://your-database-name.firebaseio.com/'
})

# Initialize DHT22
sensor = Adafruit_DHT.DHT22
pin = 4

# Initialize YL-69
spi = spidev.SpiDev()
spi.open(0, 0)
spi.max_speed_hz = 1350000

def read_adc(channel):
    adc = spi.xfer2([1, (8 + channel) << 4, 0])
    data = ((adc[1] & 3) << 8) + adc[2]
    return data

# Initialize BH1750
bus = smbus2.SMBus(1)
addr = 0x23
cont_hres_mode = 0x10

def read_light():
    data = bus.read_i2c_block_data(addr, cont_hres_mode, 2)
    return ((data[0] << 8) + data[1]) / 1.2

# Initialize YF-S201
flow_sensor = 17
pulse_count = 0

def count_pulse(channel):
    global pulse_count
    pulse_count += 1

GPIO.setmode(GPIO.BCM)
GPIO.setup(flow_sensor, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.add_event_detect(flow_sensor, GPIO.FALLING, callback=count_pulse)

while True:
    humidity, temperature = Adafruit_DHT.read_retry(sensor, pin)
    moisture_level = read_adc(0)
    light_level = read_light()

    pulse_count = 0
    time.sleep(1)
    flow_rate = pulse_count / 7.5  # Calculate flow rate in liters/minute

    if humidity is not None and temperature is not None:
        data = {
            'temperature': temperature,
            'humidity': humidity,
            'soil_moisture': moisture_level,
            'light_level': light_level,
            'flow_rate': flow_rate
        }
        ref = db.reference('sensor_data')
        ref.push(data)
        print('Data uploaded to Firebase:', data)
    else:
        print('Failed to get reading. Try again!')

    time.sleep(60)  # Wait for a minute before next reading
```

## Roadmap

### Phase 1: Planning and Research
1. **Define Objectives**
2. **Gather Requirements**
3. **Learn Basics**

### Phase 2: Hardware Setup
1. **Assemble Components**
2. **Sensor Connections**

### Phase 3: Software Setup
1. **Set Up Raspberry Pi**
2. **Install Necessary Libraries**

### Phase 4: Sensor Data Collection
1. **Write Python Scripts**

### Phase 5: Data Storage and Analysis
1. **Set Up a Database**
2. **Data Analysis**

### Phase 6: Remote Monitoring and Control
1. **Develop Web Dashboard**
2. **Develop Mobile App**

### Phase 7: Testing and Deployment
1. **Test System**
2. **Deploy to Field**
3. **Iterate and Improve**

### Phase 8: Presentation and Documentation
1. **Prepare Presentation**
2. **Document the Project**
