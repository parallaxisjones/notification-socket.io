# Notification-socket.io
Notification-socket.io is a ready to use push notification server supporting multi-session and authentication. It was built using [Node.js](https://nodejs.org) and [Socket.IO](http://socket.io/). The main purpose of notification-socket.io is to provide an **easy**, **stable** and **secure** solution which can be used to send push notifications to a client application.

## Version
1.0.0

## Features
* support for old browsers which do not support WebSockets
* REST JSON API
* authentication to API and push notification channels
* supports multi-session where the same user can be connected to the notification server from many applications and tabs at the same time
* easy to run
* easy to use

## How to run it?
### Docker
We prefer the Docker solution so the following command must be run:

```
docker run -d -p 3000:3000 \ 
  --name notification-socket.io \ 
  -e "AUTH_TOKEN=PUT_AUTH_TOKEN_HERE" \ 
  netbulls/notification-socket.io:1.0.0
```


> IMPORTANT:
> CHANGE_PUT_AUTH_TOKEN_HERE - has to be changed into a secure token which will be used as an 
> authentication token to notification-socket.io API from you backend application.


### Manual installation
* install Node.js and npm
* fetch the git repository
```
git clone https://github.com/netbulls/notification­socket.io.git
```
* install app dependencies using npm
```
npm install
```
* set evn variable with the authentication token - this token will be used to secure access to REST API
```
export AUTH_TOKEN=your_secure_token
```
* run the server
```
npm start
```

## Is it working?
Open the ```/api/status/info``` page, e.g. ```http//localhost:3000/api/status/info``` and the application should display the name and current version of the server.

## How to use it?
We recommend that it is used in the following way but of course it can be modified :)

### Register a client app

![Diagram of register process](https://raw.githubusercontent.com/netbulls/notification-socket.io/master/doc/images/register.png)

> IMPORTANT:
> each request from your backend to notification-socket.io has to be authenticated by a 
> HEADER X-AUTH-TOKEN with the same value as the env variable AUTH_TOKEN of notification-socket.io


1. Client app asks your backend about the connection data of the push notification channel (in most cases it will be executed after a successful authentication)
2. Your backend app generates a random `connectionId` - it should be unique and secure - we recommend UUID
3. Your backend app registers the user in notification-socket.io executing the method:

   ```PUT /api/{userId}/register?connectionId={connectionId} with X-AUTH-TOKEN header```

   > IMPORTANT
   > where userId is the unique id of your user in your application we will use the 
   > userId in the future to send one push notification to all clients of this user
   

4. Your backend app returns `connectionId` and `url` to notification-socket.io to client app
5. Client app connects and authenticates to notification-socket.io (using socket.io lib):

   ```javascript
   var socket = io.connect(notificationSocketIo.url);
   socket.on('connect', function() {
       socket.emit('register', userId, notificationSocketIo.connectionId);
   });
   ```
6. Register client apps which are to receive push notifications. Use socket.io to listen to messages with an identifier `message`:

   ```javascript
   socket.on('message', function(msg) {
        //handle your message
   });
   ```

### Send a push notification

![Diagram of sending push notification](https://raw.githubusercontent.com/netbulls/notification-socket.io/master/doc/images/send.png)
   
1. When your backend application wants to send a push notification to a client just execute: 

  ```POST /api/{userId}/push with X-AUTH-TOKEN header``` 
  
  and send the content of the push notification message, format of message {"message": message_json}, e.g.

   ```
   {
     "message": {
       "title":"Title of message",
       "body": "Body of message",
       "params": {}
     }
   }
   ```
2. Client app will receive a push notification in callback registered in step 6
   
### Disconnect   

1. To stop receiving push notifications your client app has to execute:

   ```
   socket.emit('disconnect')
   ```

## Security
### Access to api
There is a simple mechanism using `X-AUTH-TOKEN` header to authenticate your application in notification-socket.io server. Each request from your backend `e.g. /api/{userId}/register` and `/api/{userId}/push` requires this header. The token has to have the same value as the env variable `AUTH_TOKEN` of notification-socket.io.

### Access to the push notification channel
The first thing a client application has to do after connecting is to register in notification-socket.io. In order to do this it has to use the `connectionId` generated by the backend. Only registered applications will receive push notifications.

## Monitoring
You can execute `GET /api/status/ping` to check if the application is live.

You can execute `GET /api/status/info` to obtain the application's version.

## What's next?
**If you are looking for a push notification solution supporting GCM, APNS, and MPNS please try our [product](https://hub.docker.com/r/netbulls/notification-server/). Our notification-server offers unified secure APIs which you can use to send push notifications to all platforms using native channels which means you don't have to pay for anything – it is completely free.**
