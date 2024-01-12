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

