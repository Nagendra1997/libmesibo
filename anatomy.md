## Anatomy of a Mesibo Application

Mesibo is designed to be a cross-platform API, maintaining the same API signature and performance across all platforms. Two components form any Mesibo Application:
- Sending the data, implemented by a functional interface
- Recieving the data, implemented by a callback interface

## Sending the data

To send data from one endpoint to another is remarkably simple using Mesibo. Based on the type of data you need to send there is a wide range of API functions you can use. You can also interact with your database in which case you will be sending a message to the database ie-a query


For example,
- to send a text message you can use `sendMessage()`
- to send a file, such as an image you can use `sendFile()`
- to read messages from database you can use `read()`

## Recieving the data

When you send data/query you will get a response. This response is delivered through a callback interface. 

### What is a callback?
A callback is a function that gets triggered based on a signal/event. The event is described by a set of parameters and whenever the call back function is triggered these parameters are passed on to the client.

### Mesibo Listeners
Mesibo invokes various Listeners for various events. These listener functions have a callback interface. Mesibo provides a class of Listeners that can be invoked to get real-time notification of events. You can override the notify class behavior as per your requirements.
Following are some of the individual [listeners](https://mesibo.com/documentation/api/listeners/):

- mesibo_onConnectionStatus
- mesibo_onMessageStatus
- mesibo_onMessage
- mesibo_onFile
- mesibo_onLocation
- mesibo_onActivity

### onConnectionStatus 
Invoked when the connection status is changed. There are different connection status codes corresponding to which you can get to know you whether your connection is online/offline.

### onMessageStatus
Invoked when the status of the outgoing or sent message is changed. You will receive the status of the sent message here.
For example, When you send a message, you expect a delivery status :

For example,
When you send a message, you expect a delievery status : 
- sent from device
- recieved by recipient
- read by recipient

So, when you send a message using the sendMessage API function, you need a way to know the status of your message. To help you with this there is a callback function called onMessageStatus which will let you know the status of your message. The message parameters will have a variable called status, which changes as it goes through the steps sent--> Recieved--> Delievered. A change in message status is an event and this event will trigger the function onMessageStatus.

### onMessage
Invoked on receiving a new message or reading database messages. This callback function will notify you when a message is received. The message parameters of this function will contain the peer which will inform you who sent the message and the data parameter will contain the message data.

### Initialising Mesibo

Before you run Mesibo you need to initialise it. The initialisation involves following steps:
1. Set your app credentials for authentication: access token and app ID / app name.
2. Connecting your listener class
3. Set your database
4. Start Mesibo
 

