# slay-e-chatapp-backend


```mermaid
graph TD
    A[User] -->|Logs in| B[Authentication]
    B -->|Issues| C{JWT Token}
    C -->|Company uses for| D[Job Posting]
    D -->|Creates| E[Chat Group]
    C -->|Applicant uses for| F[Application]
    F -->|Adds to| G[Group Membership]
    C -->|Used for| H[Socket.io Connection]
    H -->|Joins| I[Chat Room]
    I -->|Sends/Receives| J[Messages]
    J -->|Stores| K[Database]
    J -->|Uploads| L[Files to S3]
    K -->|Triggers| M[Notifications]
    N[Security: JWT, Validation, Scanning]
    O[Scalability: Load Balancers, Redis, Optimization]
```

Here’s a simplified explanation using **Mermaid.js** syntax for diagrams. You can render these diagrams in tools like [Mermaid Live Editor](https://mermaid-js.github.io/mermaid-live-editor/) or VS Code with a Mermaid plugin.

---

## **Simplified Architecture**  
### **System Flow**  
```mermaid  
flowchart TD  
    A[Client] -->|1. Login & Get JWT| B(Node.js Server)  
    B -->|2. Store Token| C[(Redis)]  
    A -->|3. Connect via Socket.io| B  
    B -->|4. Create Job & Chat Group| D[(PostgreSQL)]  
    B -->|5. Upload Files| E[(AWS S3)]  
    B -->|6. Send Notifications| F[(RabbitMQ)]  
```  

---

## **Real-Time Messaging Sequence**  
```mermaid  
sequenceDiagram  
    participant Client  
    participant Node.js  
    participant PostgreSQL  
    participant S3  
    participant OtherClients  

    Client->>Node.js: Send Message (via Socket.io)  
    Node.js->>PostgreSQL: Validate Permissions  
    Node.js->>S3: Upload File (if media)  
    Node.js->>PostgreSQL: Save Message  
    Node.js->>OtherClients: Broadcast Message  
```  

---

## **Component Interactions**  
```mermaid  
graph LR  
    subgraph Backend  
        B[Node.js Server]  
        C[(PostgreSQL)]  
        D[(AWS S3)]  
        E[(Redis)]  
        F[(RabbitMQ)]  
    end  

    A[Client] -->|HTTP/Socket.io| B  
    B -->|Read/Write Data| C  
    B -->|Store Files| D  
    B -->|Sync Socket.io Rooms| E  
    B -->|Queue Notifications| F  
```  

---

## **Simplified Data Flow Explanation**  

### **1. Authentication**  
- **Client**: Logs in via REST API to get a JWT.  
- **Server**: Validates credentials, returns JWT.  
- **Socket.io**: Uses JWT to authenticate real-time connections.  

### **2. Job Creation**  
```mermaid  
flowchart LR  
    A[Company] -->|POST /jobs| B[Node.js]  
    B -->|Create Job Document| C[(PostgreSQL: Jobs)]  
    B -->|Create Chat Group| C[(PostgreSQL: ChatGroups)]  
```  

### **3. Applicant Joins Chat**  
```mermaid  
flowchart LR  
    A[Applicant] -->|POST /apply| B[Node.js]  
    B -->|Update ChatGroup Members| C[(PostgreSQL)]  
    B -->|Emit joinRoom Event| D[Socket.io]  
```  

### **4. Sending a Message**  
```mermaid  
flowchart LR  
    A[User] -->|Socket.io: sendMessage| B[Node.js]  
    B -->|Validate Sender| C[PostgreSQL]  
    B -->|Upload File if any| D[AWS S3]  
    B -->|Save Message| C  
    B -->|Broadcast to Room| E[Other Users]  
```  

---

## **Key Mermaid Diagrams Explained**  

### **1. Authentication Flow**  
```mermaid  
sequenceDiagram  
    participant Client  
    participant Node.js  
    participant PostgreSQL  

    Client->>Node.js: POST /login (email, password)  
    Node.js->>PostgreSQL: Find User  
    PostgreSQL-->>Node.js: User Data  
    Node.js-->>Client: JWT Token  
```  

### **2. File Upload Flow**  
```mermaid  
sequenceDiagram  
    participant Client  
    participant Node.js  
    participant S3  

    Client->>Node.js: POST /files (file)  
    Node.js->>S3: Generate Pre-Signed URL  
    S3-->>Node.js: Upload URL  
    Node.js-->>Client: Return URL  
    Client->>S3: Upload File Directly  
```  

### **3. Notification System**  
```mermaid  
flowchart TD  
    A[New Message] --> B{User Online?}  
    B -->|Yes| C[Send via Socket.io]  
    B -->|No| D[Queue in RabbitMQ]  
    D --> E[Send Email/Push]  
```  

---

## **How to Use These Diagrams**  
1. Copy the Mermaid code blocks.  
2. Paste them into the [Mermaid Live Editor](https://mermaid-js.github.io/mermaid-live-editor/).  
3. Render to visualize the flow.  

---

Here’s a simplified breakdown of the system’s data flow and architecture, using **text-based diagrams** and straightforward explanations. For graphical clarity, tools like [Draw.io](https://app.diagrams.net/) or [Excalidraw](https://excalidraw.com/) can visualize this flow.

---

## **Simplified Data Flow**  
### **Step 1: Authentication**  
```
[Client] → (Login API) → [JWT Token] → [Socket.io Connection]  
```  
- **What’s Happening**:  
  - Users (companies/applicants) log in via REST API to get a JWT token.  
  - The token is used to authenticate real-time Socket.io connections.  

---

### **Step 2: Job Creation & Chat Group Setup**  
```
Company → (Create Job API) → [Server] → [Database: Job + ChatGroup]  
```  
- **What’s Happening**:  
  - A company posts a job.  
  - The backend **automatically creates a chat group** linked to the job.  
  - The company user is added as the group admin.  

---

### **Step 3: Applicant Applies & Joins Chat**  
```
Applicant → (Apply to Job API) → [Server] → [Add to ChatGroup] → [Socket.io: joinRoom Event]  
```  
- **What’s Happening**:  
  - When an applicant applies, they’re added to the job’s chat group.  
  - The server triggers a Socket.io `joinRoom` event to add them to the real-time chat room.  

---

### **Step 4: Real-Time Messaging**  
```
User → (Send Message) → [Socket.io] → [Save to DB] → [Broadcast to Room]  
```  
- **What’s Happening**:  
  - Messages are sent via Socket.io, saved to the database, and broadcast to all users in the chat room.  
  - Files (images/docs) are uploaded to AWS S3 first, and their URLs are sent as messages.  

---

### **Step 5: Notifications**  
```
[Socket.io] → (Online Users)  
[Message Queue] → (Offline Users) → [Email/Push Notifications]  
```  
- **What’s Happening**:  
  - Online users receive messages instantly via Socket.io.  
  - Offline users get notifications via email/push using a message queue (e.g., RabbitMQ).  

---

## **System Diagram (Simplified)**  
```  
                       +-----------------+
                       |     Client      |
                       | (Web/Mobile App)|
                       +--------+--------+
                                |
                                | HTTP/Socket.io
                                |
                  +-------------v-------------+
                  |   Load Balancer (NGINX)  |
                  +-------------+-------------+
                                |
                                | Distributed Traffic
                                |
                  +-------------v-------------+
                  |   Node.js Servers        |
                  | (Express.js + Socket.io) |
                  +-------------+-------------+
                                |
                                | Redis (Pub/Sub)
                                |
+-----------------+    +--------v--------+    +-----------------+
|   PostgreSQL       |    |   AWS S3       |    |   RabbitMQ      |
| (Jobs, Chats,   <----+ (File Storage) +----> (Notifications) |
|  Messages)      |    +-----------------+    +-----------------+
+-----------------+  
```  

---

## **Key Components Explained**  
1. **Client**:  
   - **Companies**: Post jobs, manage chats.  
   - **Applicants**: Apply to jobs, send messages.  
   - **Real-Time Updates**: Messages appear instantly via Socket.io.  

2. **Node.js Servers**:  
   - Handle REST APIs (job creation, applications).  
   - Manage Socket.io rooms for real-time chat.  

3. **PostgreSQL**:  
   - Stores jobs, chat groups, messages, and user data.  
   - Example query: `Find all messages where chatGroupId = XYZ`.  

4. **AWS S3**:  
   - Stores files (images, resumes) securely.  
   - Returns URLs like `https://s3.amazonaws.com/job-chats/file.pdf`.  

5. **Redis**:  
   - Syncs Socket.io rooms across multiple Node.js servers.  
   - Ensures messages reach all users, even in a scaled setup.  

6. **RabbitMQ**:  
   - Queues notifications for offline users (e.g., "New message in Job Chat!").  

---

## **Example Scenario: Sending a Message**  
```  
Applicant (Client)           Node.js Server               PostgreSQL                Other Clients  
      | → "Hello Team!"           |                           |                        |  
      |                           | → Validate Permissions    |                        |  
      |                           | → Save to Messages        |                        |  
      |                           | → Broadcast via Socket.io |                        |  
      | ← "Message Received"      |                           |                        | ← "Hello Team!"  
```  

1. The applicant sends a message.  
2. The server checks if they’re part of the chat group.  
3. The message is saved to PostgreSQL.  
4. Socket.io broadcasts it to all users in the room.  

---

## **Security Simplified**  
- **JWT Tokens**: Verify users on every Socket.io message.  
- **File Scanning**: AWS Lambda scans uploaded files for malware.  
- **Rate Limiting**: Block users sending too many messages (e.g., 100/min).  

---

## **Summary**  
- **Companies** create jobs → **Chat groups auto-generated**.  
- **Applicants** apply → **Auto-join chats** → **Send messages/files**.  
- **Real-time updates** via Socket.io + **Notifications** for offline users.  
- **Scalable** with Redis/NGINX and secure with JWT/S3.
# slay-e-chatapp-backend


```mermaid
graph TD
    A[User] -->|Logs in| B[Authentication]
    B -->|Issues| C{JWT Token}
    C -->|Company uses for| D[Job Posting]
    D -->|Creates| E[Chat Group]
    C -->|Applicant uses for| F[Application]
    F -->|Adds to| G[Group Membership]
    C -->|Used for| H[Socket.io Connection]
    H -->|Joins| I[Chat Room]
    I -->|Sends/Receives| J[Messages]
    J -->|Stores| K[Database]
    J -->|Uploads| L[Files to S3]
    K -->|Triggers| M[Notifications]
    N[Security: JWT, Validation, Scanning]
    O[Scalability: Load Balancers, Redis, Optimization]
```

Here’s a simplified explanation using **Mermaid.js** syntax for diagrams. You can render these diagrams in tools like [Mermaid Live Editor](https://mermaid-js.github.io/mermaid-live-editor/) or VS Code with a Mermaid plugin.

---

## **Simplified Architecture**  
### **System Flow**  
```mermaid  
flowchart TD  
    A[Client] -->|1. Login & Get JWT| B(Node.js Server)  
    B -->|2. Store Token| C[(Redis)]  
    A -->|3. Connect via Socket.io| B  
    B -->|4. Create Job & Chat Group| D[(PostgreSQL)]  
    B -->|5. Upload Files| E[(AWS S3)]  
    B -->|6. Send Notifications| F[(RabbitMQ)]  
```  

---

## **Real-Time Messaging Sequence**  
```mermaid  
sequenceDiagram  
    participant Client  
    participant Node.js  
    participant PostgreSQL  
    participant S3  
    participant OtherClients  

    Client->>Node.js: Send Message (via Socket.io)  
    Node.js->>PostgreSQL: Validate Permissions  
    Node.js->>S3: Upload File (if media)  
    Node.js->>PostgreSQL: Save Message  
    Node.js->>OtherClients: Broadcast Message  
```  

---

## **Component Interactions**  
```mermaid  
graph LR  
    subgraph Backend  
        B[Node.js Server]  
        C[(PostgreSQL)]  
        D[(AWS S3)]  
        E[(Redis)]  
        F[(RabbitMQ)]  
    end  

    A[Client] -->|HTTP/Socket.io| B  
    B -->|Read/Write Data| C  
    B -->|Store Files| D  
    B -->|Sync Socket.io Rooms| E  
    B -->|Queue Notifications| F  
```  

---

## **Simplified Data Flow Explanation**  

### **1. Authentication**  
- **Client**: Logs in via REST API to get a JWT.  
- **Server**: Validates credentials, returns JWT.  
- **Socket.io**: Uses JWT to authenticate real-time connections.  

### **2. Job Creation**  
```mermaid  
flowchart LR  
    A[Company] -->|POST /jobs| B[Node.js]  
    B -->|Create Job Document| C[(PostgreSQL: Jobs)]  
    B -->|Create Chat Group| C[(PostgreSQL: ChatGroups)]  
```  

### **3. Applicant Joins Chat**  
```mermaid  
flowchart LR  
    A[Applicant] -->|POST /apply| B[Node.js]  
    B -->|Update ChatGroup Members| C[(PostgreSQL)]  
    B -->|Emit joinRoom Event| D[Socket.io]  
```  

### **4. Sending a Message**  
```mermaid  
flowchart LR  
    A[User] -->|Socket.io: sendMessage| B[Node.js]  
    B -->|Validate Sender| C[PostgreSQL]  
    B -->|Upload File if any| D[AWS S3]  
    B -->|Save Message| C  
    B -->|Broadcast to Room| E[Other Users]  
```  

---

## **Key Mermaid Diagrams Explained**  

### **1. Authentication Flow**  
```mermaid  
sequenceDiagram  
    participant Client  
    participant Node.js  
    participant PostgreSQL  

    Client->>Node.js: POST /login (email, password)  
    Node.js->>PostgreSQL: Find User  
    PostgreSQL-->>Node.js: User Data  
    Node.js-->>Client: JWT Token  
```  

### **2. File Upload Flow**  
```mermaid  
sequenceDiagram  
    participant Client  
    participant Node.js  
    participant S3  

    Client->>Node.js: POST /files (file)  
    Node.js->>S3: Generate Pre-Signed URL  
    S3-->>Node.js: Upload URL  
    Node.js-->>Client: Return URL  
    Client->>S3: Upload File Directly  
```  

### **3. Notification System**  
```mermaid  
flowchart TD  
    A[New Message] --> B{User Online?}  
    B -->|Yes| C[Send via Socket.io]  
    B -->|No| D[Queue in RabbitMQ]  
    D --> E[Send Email/Push]  
```  

---

## **How to Use These Diagrams**  
1. Copy the Mermaid code blocks.  
2. Paste them into the [Mermaid Live Editor](https://mermaid-js.github.io/mermaid-live-editor/).  
3. Render to visualize the flow.  

---

Here’s a simplified breakdown of the system’s data flow and architecture, using **text-based diagrams** and straightforward explanations. For graphical clarity, tools like [Draw.io](https://app.diagrams.net/) or [Excalidraw](https://excalidraw.com/) can visualize this flow.

---

## **Simplified Data Flow**  
### **Step 1: Authentication**  
```
[Client] → (Login API) → [JWT Token] → [Socket.io Connection]  
```  
- **What’s Happening**:  
  - Users (companies/applicants) log in via REST API to get a JWT token.  
  - The token is used to authenticate real-time Socket.io connections.  

---

### **Step 2: Job Creation & Chat Group Setup**  
```
Company → (Create Job API) → [Server] → [Database: Job + ChatGroup]  
```  
- **What’s Happening**:  
  - A company posts a job.  
  - The backend **automatically creates a chat group** linked to the job.  
  - The company user is added as the group admin.  

---

### **Step 3: Applicant Applies & Joins Chat**  
```
Applicant → (Apply to Job API) → [Server] → [Add to ChatGroup] → [Socket.io: joinRoom Event]  
```  
- **What’s Happening**:  
  - When an applicant applies, they’re added to the job’s chat group.  
  - The server triggers a Socket.io `joinRoom` event to add them to the real-time chat room.  

---

### **Step 4: Real-Time Messaging**  
```
User → (Send Message) → [Socket.io] → [Save to DB] → [Broadcast to Room]  
```  
- **What’s Happening**:  
  - Messages are sent via Socket.io, saved to the database, and broadcast to all users in the chat room.  
  - Files (images/docs) are uploaded to AWS S3 first, and their URLs are sent as messages.  

---

### **Step 5: Notifications**  
```
[Socket.io] → (Online Users)  
[Message Queue] → (Offline Users) → [Email/Push Notifications]  
```  
- **What’s Happening**:  
  - Online users receive messages instantly via Socket.io.  
  - Offline users get notifications via email/push using a message queue (e.g., RabbitMQ).  

---

## **System Diagram (Simplified)**  
```  
                       +-----------------+
                       |     Client      |
                       | (Web/Mobile App)|
                       +--------+--------+
                                |
                                | HTTP/Socket.io
                                |
                  +-------------v-------------+
                  |   Load Balancer (NGINX)  |
                  +-------------+-------------+
                                |
                                | Distributed Traffic
                                |
                  +-------------v-------------+
                  |   Node.js Servers        |
                  | (Express.js + Socket.io) |
                  +-------------+-------------+
                                |
                                | Redis (Pub/Sub)
                                |
+-----------------+    +--------v--------+    +-----------------+
|   PostgreSQL       |    |   AWS S3       |    |   RabbitMQ      |
| (Jobs, Chats,   <----+ (File Storage) +----> (Notifications) |
|  Messages)      |    +-----------------+    +-----------------+
+-----------------+  
```  

---

## **Key Components Explained**  
1. **Client**:  
   - **Companies**: Post jobs, manage chats.  
   - **Applicants**: Apply to jobs, send messages.  
   - **Real-Time Updates**: Messages appear instantly via Socket.io.  

2. **Node.js Servers**:  
   - Handle REST APIs (job creation, applications).  
   - Manage Socket.io rooms for real-time chat.  

3. **PostgreSQL**:  
   - Stores jobs, chat groups, messages, and user data.  
   - Example query: `Find all messages where chatGroupId = XYZ`.  

4. **AWS S3**:  
   - Stores files (images, resumes) securely.  
   - Returns URLs like `https://s3.amazonaws.com/job-chats/file.pdf`.  

5. **Redis**:  
   - Syncs Socket.io rooms across multiple Node.js servers.  
   - Ensures messages reach all users, even in a scaled setup.  

6. **RabbitMQ**:  
   - Queues notifications for offline users (e.g., "New message in Job Chat!").  

---

## **Example Scenario: Sending a Message**  
```  
Applicant (Client)           Node.js Server               PostgreSQL                Other Clients  
      | → "Hello Team!"           |                           |                        |  
      |                           | → Validate Permissions    |                        |  
      |                           | → Save to Messages        |                        |  
      |                           | → Broadcast via Socket.io |                        |  
      | ← "Message Received"      |                           |                        | ← "Hello Team!"  
```  

1. The applicant sends a message.  
2. The server checks if they’re part of the chat group.  
3. The message is saved to PostgreSQL.  
4. Socket.io broadcasts it to all users in the room.  

---

## **Security Simplified**  
- **JWT Tokens**: Verify users on every Socket.io message.  
- **File Scanning**: AWS Lambda scans uploaded files for malware.  
- **Rate Limiting**: Block users sending too many messages (e.g., 100/min).  

---

## **Summary**  
- **Companies** create jobs → **Chat groups auto-generated**.  
- **Applicants** apply → **Auto-join chats** → **Send messages/files**.  
- **Real-time updates** via Socket.io + **Notifications** for offline users.  
- **Scalable** with Redis/NGINX and secure with JWT/S3.
