# mqtt-wx-bridge1
const WebSocket = require('ws'); // 用于 WebSocket 服务
const mqtt = require('mqtt'); // 用于 MQTT 连接

// 配置 WebSocket 服务端口和 MQTT Broker
const WS_PORT = process.env.WS_PORT || 8084;
const MQTT_BROKER_URL = process.env.MQTT_BROKER_URL || 'mqtt://broker.emqx.io';
const MQTT_TOPIC = process.env.MQTT_TOPIC || 'sensor/data';

// 初始化 MQTT 客户端
const mqttClient = mqtt.connect(MQTT_BROKER_URL);

// 当 MQTT 客户端连接成功时
mqttClient.on('connect', () => {
  console.log('Connected to MQTT broker');
  mqttClient.subscribe(MQTT_TOPIC, (err) => {
    if (err) {
      console.error('MQTT subscription error:', err);
    } else {
      console.log('Subscribed to topic:', MQTT_TOPIC);
    }
  });
});

// 当收到 MQTT 消息时，将消息转发给 WebSocket 客户端
mqttClient.on('message', (topic, message) => {
  console.log(`Received message on topic ${topic}: ${message}`);
  if (wss && wss.clients) {
    wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message.toString()); // 转发消息
      }
    });
  }
});

// 创建 WebSocket 服务器
const wss = new WebSocket.Server({ port: WS_PORT });

// 当 WebSocket 客户端连接时
wss.on('connection', (ws) => {
  console.log('New WebSocket client connected');

  // 处理来自 WebSocket 客户端的消息
  ws.on('message', (message) => {
    console.log(`Received message from WebSocket client: ${message}`);
    // 可在此发送消息到 MQTT Broker
    mqttClient.publish(MQTT_TOPIC, message);
  });

  // 向 WebSocket 客户端发送欢迎消息
  ws.send('Connected to WebSocket server');
});

console.log(`WebSocket server is running at ws://localhost:${WS_PORT}`);
