# IoT System Examples

## Example: Complete IoT Platform

```javascript
const mqtt = require('mqtt');
const { InfluxDB, Point } = require('@influxdata/influxdb-client');

class IoTPlatform {
  constructor() {
    this.mqttClient = mqtt.connect('mqtt://localhost:1883');
    this.influx = new InfluxDB({url: 'http://localhost:8086', token: process.env.INFLUX_TOKEN});
    this.writeApi = this.influx.getWriteApi('org', 'bucket');

    this.setupMQTT();
  }

  setupMQTT() {
    this.mqttClient.on('connect', () => {
      this.mqttClient.subscribe('devices/+/telemetry');
      this.mqttClient.subscribe('devices/+/status');
    });

    this.mqttClient.on('message', async (topic, message) => {
      const parts = topic.split('/');
      const deviceId = parts[1];
      const type = parts[2];

      const data = JSON.parse(message.toString());

      if (type === 'telemetry') {
        await this.handleTelemetry(deviceId, data);
      } else if (type === 'status') {
        await this.handleStatus(deviceId, data);
      }
    });
  }

  async handleTelemetry(deviceId, data) {
    const point = new Point('telemetry')
      .tag('deviceId', deviceId)
      .floatField('temperature', data.temperature)
      .floatField('humidity', data.humidity)
      .timestamp(new Date());

    this.writeApi.writePoint(point);
    await this.writeApi.flush();

    // Check alerts
    if (data.temperature > 80) {
      await this.triggerAlert(deviceId, 'high_temperature');
    }
  }

  async sendCommand(deviceId, command, params) {
    this.mqttClient.publish(
      `devices/${deviceId}/commands`,
      JSON.stringify({ command, params })
    );
  }
}
```

## Example: Device Client (Arduino/ESP32)

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "WiFi_SSID";
const char* password = "WiFi_Pass";
const char* mqtt_server = "mqtt.example.com";
const char* device_id = "device_001";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Send telemetry every 10 seconds
  static unsigned long lastMsg = 0;
  unsigned long now = millis();
  if (now - lastMsg > 10000) {
    lastMsg = now;
    sendTelemetry();
  }
}

void sendTelemetry() {
  float temperature = readTemperature();
  float humidity = readHumidity();

  String payload = "{\"temperature\":" + String(temperature) +
                   ",\"humidity\":" + String(humidity) + "}";

  client.publish("devices/device_001/telemetry", payload.c_str());
}

void callback(char* topic, byte* payload, unsigned int length) {
  // Handle incoming commands
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }

  // Parse and execute command
  handleCommand(message);
}
```
