# Design a chat system like whats app

## Requirement
### Clarifying questions
1. Does the system support group chat?
2. For group chat, what is the group member limit?
3. Do we support only mobile app, or both mobile app and web app?
4. Doest it support sending media files, like image or video?
5. How long shall we store the messages?
6. What's the scale of the system, such as how many daily users?

### Functional requirements
1. Chat 
Support one-on-one and group chat
2.Acknowledgment:  
The system should support message delivery acknowledgment, such as sent, delivered, and read.
3. Sharing:  
The system should support sharing of media files, such as images, videos, and audio.
4. Chat storage:  
The system must support the persistent storage of chat messages when a user is offline until the successful delivery of messages.
5. Push notifications:  
The system should be able to notify offline users of new messages once their status becomes online.

### Non-functional requirements
1. Availability
2. Scalibility
3. Low latency
4. Security

### Resource estmiation
Let's assume 100 billion messages a day, and each message is of 100Bytes on aveerage.
1. Storage estimation
For a day: 100 billion * 100 Bytes = 10 TB / day

## High-level design
<img width="812" alt="chat_high_level" src="https://github.com/toextendmylimits/system_design/assets/10056698/fc489c9f-7420-4c90-84d2-1317c05d9dee">

### API
1. Send message  
The sendMessage API is as follows:

sendMessage(message_ID, sender_ID, reciever_ID, type, text=none, media_object=none, document=none)

2. Get message  

getMessage(user_Id)
Using this API call, users can fetch all unread messages when they come online after being offline for some time.

3. Upload media or document file#  
The uploadFile API is as follows:
We can upload media files via the uploadFile API by making a POST request to the /v1/media API endpoint. A successful response returns an ID that’s forwarded to the receiver.

4. Download a document or media file  
The downloadFile API is as follows:

downloadFile(user_id, file_id)

## Detailed design
How does the system support billions of users?
1. Connection with a WebSocket server
<img width="839" alt="chat_websocket" src="https://github.com/toextendmylimits/system_design/assets/10056698/18f5921a-981b-4c34-9237-d8b3fd5f1053">
In WhatsApp, each active device is connected with a WebSocket server via WebSocket protocol. A WebSocket server keeps the connection open with all the active (online) users. Since one server isn’t enough to handle billions of devices, there should be enough servers to handle billions of users. The responsibility of each of these servers is to provide a port to every online user. The mapping between servers, ports, and users is stored in the WebSocket manager that resides on top of a cluster of the data store. In this case, that’s Redis.

2. Send or receive messages

