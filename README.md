# hivemq-
or an already finished project that provides you with all the code in this guide, clone this example repository. You can also copy the code below into your own project. If you cloned the repository from GitHub, the required dependencies are listed in the pubspec.yaml and you need to run dart pub get to install them. Otherwise, follow these instructions to create your own dependency file.

Create a new project folder mqtt-dart-hivemq-cloud in your preferred IDE. Create a subfolder bin with a main.dart file inside. Add the code shown in the next section to this dart file. To install the required dependencies, you need to create your own pubspec.yaml in the root folder. That mean it should be outside the bin folder. Add the following code to the pubspec.yaml:

```python
name: mqtt
version: 1.0.1
environment:
  sdk: '>=2.1.0 <3.0.0'
dependencies:
  mqtt_client: ^7.2.1
  ini: '>=1.0.0'
  test: '>=0.12.0'

```
Use the command line tool of your choice to navigate to this directory. Then execute the following command in order to install the MQTT.Dart library. ( https://pub.dev/packages/mqtt_client )

```bash
dart pub get
```

Connect MQTT clients
Add the following content to your main.dart.

Use this code to connect to your HiveMQ Cloud Instance via the MQTT.Dart library. The host name of your cluster is already inserted into the serverURI. Your username is also inserted automatically. To fully verify your credentials, replace the variable '<your_password>' with the value you entered when creating your credentials. As HiveMQ Cloud does not support insecure connections, TLS is required. The code below enables TLS by using:
```python
client.secure = true;
client.securityContext = SecurityContext.defaultContext;
```
This enables a secure connection and sets the default security context. The default port used for secure MQTT connections is 8883.
```python
/*
* Copyright 2021 HiveMQ GmbH
*
* Licensed under the Apache License, Version 2.0 (the "License");
* you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
*
*       http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/

import 'dart:io';
import 'package:mqtt_client/mqtt_client.dart';
import 'package:mqtt_client/mqtt_server_client.dart';

main() {
  MQTTClientWrapper newclient = new MQTTClientWrapper();
  newclient.prepareMqttClient();
}

// connection states for easy identification
enum MqttCurrentConnectionState {
  IDLE,
  CONNECTING,
  CONNECTED,
  DISCONNECTED,
  ERROR_WHEN_CONNECTING
}

enum MqttSubscriptionState {
  IDLE,
  SUBSCRIBED
}

class MQTTClientWrapper {

  MqttServerClient client;

  MqttCurrentConnectionState connectionState = MqttCurrentConnectionState.IDLE;
  MqttSubscriptionState subscriptionState = MqttSubscriptionState.IDLE;

  // using async tasks, so the connection won't hinder the code flow
  void prepareMqttClient() async {
    _setupMqttClient();
    await _connectClient();
    _subscribeToTopic('Dart/Mqtt_client/testtopic');
    _publishMessage('Hello');
  }

  // waiting for the connection, if an error occurs, print it and disconnect
  Future<void> _connectClient() async {
    try {
      print('client connecting....');
      connectionState = MqttCurrentConnectionState.CONNECTING;
      await client.connect('<your_username>', '<your_password>');
    } on Exception catch (e) {
      print('client exception - $e');
      connectionState = MqttCurrentConnectionState.ERROR_WHEN_CONNECTING;
      client.disconnect();
    }

    // when connected, print a confirmation, else print an error
    if (client.connectionStatus.state == MqttConnectionState.connected) {
      connectionState = MqttCurrentConnectionState.CONNECTED;
      print('client connected');
    } else {
      print(
          'ERROR client connection failed - disconnecting, status is ${client.connectionStatus}');
      connectionState = MqttCurrentConnectionState.ERROR_WHEN_CONNECTING;
      client.disconnect();
    }
  }

  void _setupMqttClient() {
    client = MqttServerClient.withPort('<your_cluster_url>', <your_name>', 8883);
    // the next 2 lines are necessary to connect with tls, which is used by HiveMQ Cloud
    client.secure = true;
    client.securityContext = SecurityContext.defaultContext;
    client.keepAlivePeriod = 20;
    client.onDisconnected = _onDisconnected;
    client.onConnected = _onConnected;
    client.onSubscribed = _onSubscribed;
  }

  void _subscribeToTopic(String topicName) {
    print('Subscribing to the $topicName topic');
    client.subscribe(topicName, MqttQos.atMostOnce);

    // print the message when it is received
    client.updates.listen((List<MqttReceivedMessage<MqttMessage>> c) {
      final MqttPublishMessage recMess = c[0].payload;
      var message = MqttPublishPayload.bytesToStringAsString(recMess.payload.message);

      print('YOU GOT A NEW MESSAGE:');
      print(message);
    });
  }

  void _publishMessage(String message) {
    final MqttClientPayloadBuilder builder = MqttClientPayloadBuilder();
    builder.addString(message);

    print('Publishing message "$message" to topic ${'Dart/Mqtt_client/testtopic'}');
    client.publishMessage('Dart/Mqtt_client/testtopic', MqttQos.exactlyOnce, builder.payload);
  }

  // callbacks for different events
  void _onSubscribed(String topic) {
    print('Subscription confirmed for topic $topic');
    subscriptionState = MqttSubscriptionState.SUBSCRIBED;
  }

  void _onDisconnected() {
    print('OnDisconnected client callback - Client disconnection');
    connectionState = MqttCurrentConnectionState.DISCONNECTED;
  }

  void _onConnected() {
    connectionState = MqttCurrentConnectionState.CONNECTED;
    print('OnConnected client callback - Client connection was sucessful');
  }

}
```
Run this example by executing this command in your terminal:
```bash
dart main.dart
```
Publish and Subscribe with your MQTT client
This code creates an MQTT client that is publishing and subscribing to a topic on your HiveMQ Cloud cluster. The ```prepareMqttClient()``` method is called after the class ```MQTTClientWrapper``` is created. It calls the methods for setup, connection, subscribing and publishing.

First it sets up the client by setting the host name, client name and port. The TLS options and callbacks are also set here. You can optionally use other callbacks as well. Afterwards it connects to the client using your username and password. Then it subscribes to the topic ```'Dart/Mqtt_client/testtopic'``` with QoS = 0. Also, a listener is registered to notify you when a message is received. Then it publishes a message with the payload ```'Hello' and QoS = 2``` to this topic. You can also print the content of the incoming message, process it, etc.

Next steps
Get familiar with the MQTT.Dart API and build your first application. Further information on MQTT.Dart can be found in our MQTT Client Library Encyclopedia (```https://www.hivemq.com/blog/mqtt-client-library-mqtt-dart/``` ). Learn more about MQTT by visiting the MQTT Essentials guide ```https://www.hivemq.com/mqtt/``` , that explains the core of MQTT concepts, its features and other essential information.
