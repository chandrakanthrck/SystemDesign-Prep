# System Design: Messenger Application

## Problem Statement

Design a real-time messaging application that allows users to send and receive messages, handle group chats, support rich media (images, videos), and provide features like notifications, message read receipts, and message history.

### Requirements

#### Functional Requirements:
1. **User Registration and Authentication**: Users should be able to register, log in, and authenticate via email, phone, or third-party services (e.g., Google, Facebook).
2. **One-to-One Chat**: Users can send and receive messages in real-time with individual contacts.
3. **Group Chat**: Users can create and participate in group chats.
4. **Rich Media Support**: Users can send images, videos, and files.
5. **Read Receipts**: Show whether the recipient has seen the message.
6. **Notifications**: Users should receive notifications for new messages.
7. **Message History**: Store and retrieve message history for users.
8. **Typing Indicators**: Show when the other user is typing a message.

#### Non-Functional Requirements:
1. **Scalability**: The system should support millions of users, handling large volumes of real-time messages.
2. **Low Latency**: Messages should be delivered and displayed with minimal delay.
3. **High Availability**: Ensure that the system is available 24/7 without downtime.
4. **Data Security**: Encrypt messages both at rest and in transit to protect user privacy.
5. **Real-Time Messaging**: Ensure instant message delivery with acknowledgment for sent and received messages.
6. **Reliability**: Handle message delivery failures and retries.

---

## Components of the System

1. **User Service**: Manages user registration, authentication, and profile management.
2. **Message Service**: Handles message delivery, storing messages, and message read receipts.
3. **Group Service**: Manages group creation, user participation, and group chat functionality.
4. **Notification Service**: Sends push notifications for new messages, mentions, or group chat updates.
5. **Media Service**: Handles the uploading and retrieval of media (images, videos, files) shared in chats.
6. **Real-Time Messaging**: Handles real-time message delivery using protocols like WebSockets.
7. **Database Service**: Stores user profiles, messages, group details, and other metadata.
8. **Cache (Optional)**: Caches frequently accessed data like user profiles and message metadata to improve performance.

---

## High-Level Design

1. **User Interface (UI)**: Users send and receive messages in real time, create groups, and access chat history. The UI includes typing indicators, read receipts, and notifications for new messages.
2. **API Layer**: Exposes APIs for sending messages (`POST /sendMessage`), creating groups (`POST /createGroup`), retrieving messages (`GET /messages`), and sending media (`POST /sendMedia`).
3. **Service Layer**:
    - **User Service**: Handles registration, authentication, and user profile management.
    - **Message Service**: Manages message creation, delivery, and retrieval.
    - **Group Service**: Handles group chat operations.
    - **Notification Service**: Sends push notifications to users for new messages.
    - **Media Service**: Uploads and retrieves images, videos, and files shared in chats.
4. **Storage Layer**:
    - **Database**: Stores users, messages, media metadata, group details, and message delivery status.
    - **NoSQL (Optional)**: Stores chat histories and real-time data.
    - **Blob Storage**: Stores media files such as images, videos, and documents.

---

## Detailed Design

### 1. **User Registration and Authentication**

Users should be able to register using an email, phone number, or third-party services like Google or Facebook. The system should handle user authentication, including login and logout.

#### Database Schema for Users:
| Field          | Type          | Description                             |
|----------------|---------------|-----------------------------------------|
| `user_id`      | String (PK)   | Unique identifier for the user.         |
| `username`     | String        | User's display name.                    |
| `email`        | String        | User's email address.                   |
| `phone_number` | String        | User's phone number.                    |
| `profile_pic`  | Blob          | User's profile picture.                 |
| `status`       | String        | User's status message (e.g., online, busy). |
| `last_seen`    | Timestamp     | Timestamp of the user's last activity.  |

---

### 2. **One-to-One Chat**

Users can send real-time messages to one another. Messages are delivered instantly, and both users can view message history.

#### Message Flow:
1. User A sends a message to User B.
2. The system stores the message in the database.
3. User B is notified of the new message in real-time via WebSockets or push notifications.
4. The message is displayed in User B’s chat interface.

#### Database Schema for Messages:
| Field          | Type          | Description                             |
|----------------|---------------|-----------------------------------------|
| `message_id`   | String (PK)   | Unique identifier for the message.      |
| `sender_id`    | String (FK)   | ID of the user who sent the message.    |
| `recipient_id` | String (FK)   | ID of the user who received the message.|
| `content`      | Text          | The message content (text, links, etc.).|
| `timestamp`    | Timestamp     | Time when the message was sent.         |
| `read_status`  | Boolean       | Whether the message has been read.      |
| `media_url`    | String        | URL of any media attached to the message. |

---

### 3. **Group Chat**

Users can create group chats, invite other users, and send messages to the group. Messages in group chats are visible to all participants.

#### Group Creation Flow:
1. A user creates a new group and adds participants.
2. The system assigns a unique group ID and stores the group metadata.
3. All users in the group are notified and can start sending messages.

#### Database Schema for Groups:
| Field          | Type          | Description                             |
|----------------|---------------|-----------------------------------------|
| `group_id`     | String (PK)   | Unique identifier for the group.        |
| `group_name`   | String        | Name of the group.                      |
| `created_by`   | String (FK)   | ID of the user who created the group.   |
| `members`      | Array[String] | List of user IDs who are members of the group. |
| `group_icon`   | Blob          | Group profile picture or icon.          |
| `created_at`   | Timestamp     | Timestamp of group creation.            |

---

### 4. **Rich Media Support**

Users can send images, videos, and other files as part of their chats. Media files are uploaded to blob storage, and URLs to the files are included in the message.

#### Media Flow:
1. The user uploads an image or video to the media service.
2. The media service stores the file in blob storage (e.g., Amazon S3).
3. The media service returns a URL to the file, which is included in the message.
4. The recipient retrieves the media using the URL.

#### Database Schema for Media:
| Field          | Type          | Description                             |
|----------------|---------------|-----------------------------------------|
| `media_id`     | String (PK)   | Unique identifier for the media file.   |
| `uploaded_by`  | String (FK)   | ID of the user who uploaded the media.  |
| `media_url`    | String        | URL of the media file in blob storage.  |
| `media_type`   | String        | Type of media (image, video, file).     |
| `timestamp`    | Timestamp     | Time when the media was uploaded.       |

---

### 5. **Real-Time Messaging (WebSockets)**

Real-time messaging is implemented using **WebSockets**, which maintain an open connection between the client and server for instant message delivery.

#### Real-Time Messaging Process:
1. User A sends a message.
2. The message is sent over a WebSocket connection to the server.
3. The server delivers the message to User B in real-time using the WebSocket connection.
4. If User B is offline, the server stores the message and delivers it when they come online.

---

### 6. **Message Read Receipts**

Users should be able to see whether their messages have been read by the recipient. This can be implemented using read receipts.

#### Read Receipt Flow:
1. When User B reads a message, the system updates the `read_status` field for the message.
2. User A is notified that their message has been read.

#### Read Receipts Table:
| Field          | Type          | Description                             |
|----------------|---------------|-----------------------------------------|
| `message_id`   | String (PK)   | ID of the message.                      |
| `read_by`      | String (FK)   | ID of the user who read the message.    |
| `read_at`      | Timestamp     | Time when the message was read.         |

---

### 7. **Notifications**

Users should receive notifications for new messages, mentions, or group updates. Notifications can be sent via push notifications or in-app alerts.

#### Types of Notifications:
1. **New Message**: Notify users when they receive a new message.
2. **Mention**: Notify users when they are mentioned in a group chat.
3. **Group Updates**: Notify users of group creation, invitations, or when someone leaves a group.

---

### 8. **Message History**

Users can access their previous messages in one-on-one or group chats. Message history is stored in a database and retrieved when needed.

#### Message History Process:
1. When a user opens a chat, the system retrieves the most recent messages from the database.
2. The system supports pagination to load more messages as the user scrolls up.
3. Messages older than a certain time (e.g., 6 months) can be archived.

---

## Key Challenges

### 1. **Scalability**
- As the user base grows, the system must scale to handle millions of users, messages, and media files.
    - **Solution**: Use horizontal scaling, sharding, and partitioning for user data and messages. Implement a distributed NoSQL database for chat history.

### 2. **Real-Time Messaging**
- Real-time messaging requires low-latency message delivery.
    - **Solution**: Use WebSockets for instant message delivery and implement a message queue (e.g., Kafka) for handling high-throughput messaging.

### 3. **Data Privacy and Security**
- Messaging apps handle sensitive user data, including private conversations and media.
    - **Solution**: Encrypt all messages and media files both in transit (e.g., using HTTPS) and at rest (e.g., AES encryption).

### 4. **Reliability**
- Messages must be delivered reliably, even if users are temporarily offline.
    - **Solution**: Implement message persistence and retry mechanisms to ensure that no messages are lost.

---

## Advanced Features (Optional)

### 1. **End-to-End Encryption**
- Implement end-to-end encryption for all messages, ensuring that only the sender and recipient can read the messages, not even the server.

### 2. **Video and Voice Calling**
- Add support for voice and video calls within the chat interface.

### 3. **Message Reactions**
- Allow users to react to messages with emojis, similar to reactions in social apps.

### 4. **Status Updates**
- Enable users to post status updates (e.g., images, videos) that are visible to their contacts for 24 hours.

---

## Data Flow

### 1. **User Registration**
- User signs up with an email or phone number.
- The system creates a new user profile and stores it in the database.

### 2. **Sending a Message**
- User A sends a message to User B.
- The message is sent over WebSockets and stored in the database.
- User B receives the message in real time.

### 3. **Media Sharing**
- User uploads an image or video to the media service.
- The media is stored in blob storage, and the media URL is sent as part of the message.

---

## Scaling the System

### 1. **Database Sharding**
- Use database sharding to distribute user data and messages across multiple servers to ensure scalability as the user base grows.

### 2. **Caching Frequently Accessed Data**
- Use caching (e.g., Redis) to store frequently accessed data like user profiles and recent messages to reduce load on the database.

### 3. **Load Balancing**
- Use load balancers to distribute incoming traffic across multiple application servers to handle large volumes of requests efficiently.

### 4. **Message Queues**
- Use message queues (e.g., Kafka) to handle high-throughput events like message delivery, notifications, and updates.

---

## Conclusion

Designing a messenger application involves handling real-time communication, ensuring scalability, and maintaining data privacy. By leveraging real-time messaging protocols, caching, database sharding, and security best practices, the system can handle large-scale user activity with low latency and high reliability.

---

## Further Reading

- [Designing Real-Time Messaging Systems](https://example-link.com/real-time-messaging)
- [WebSocket Scalability for Chat Apps](https://example-link.com/websocket-scalability)
- [End-to-End Encryption for Messaging](https://example-link.com/end-to-end-encryption)