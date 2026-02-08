# WuzAPI Complete Reference

This documentation covers all available endpoints in the WuzAPI.

## Authentication

The API supports two authentication methods:
1.  **User Token**: For regular endpoints. Use the `token` header or query parameter (depending on configuration, usually `Authorization: <token>` or `token` in header).
    *   Header: `Authorization: <your-user-token>`
    *   Or Header: `token: <your-user-token>`
2.  **Admin Token**: For `/admin/*` endpoints. Use the `Authorization` header.
    *   Header: `Authorization: <WUZAPI_ADMIN_TOKEN>`

## Common Response Structure
Most endpoints return a JSON response with:
```json
{
  "code": 200,
  "success": true,
  "message": "...",
  "data": { ... }
}
```

---

## 1. Admin Endpoints (User Management)
**Requires Admin Token**

### List Users
`GET /admin/users`
Returns a list of all registered users and their configurations.
*   **Query Params**:
    *   `id` (optional): Filter by specific User ID.

### Add User
`POST /admin/users`
Creates a new user instance.
*   **Body**:
    ```json
    {
      "name": "My Business",
      "token": "secret-token-123",
      "webhook": "https://myapp.com/webhook",
      "events": "Message, ReadReceipt",
      "history": 100,
      "proxyConfig": {
        "enabled": false,
        "proxyURL": ""
      },
      "s3Config": {
        "enabled": false
      },
      "hmacKey": "32-char-secret-key-..."
    }
    ```

### Edit User
`PATCH /admin/users/{id}`
Updates an existing user.
*   **Body**: (Same fields as Add User, all optional)

### Delete User
`DELETE /admin/users/{id}`
Soft deletes a user (removes from DB).

### Delete User Complete
`DELETE /admin/users/{id}/full`
Completely removes user, including session files, media, and S3 objects.

---

## 2. Session Management

### Connect / Start Session
`POST /session/connect`
Starts the WhatsApp client.
*   **Body**:
    ```json
    {
      "subscribe": ["Message", "Call"], 
      "immediate": true 
    }
    ```

### Disconnect
`POST /session/disconnect`
Stops the WhatsApp client but keeps the session (no logout).

### Logout
`POST /session/logout`
Logs out and clears the session. Requires re-scanning QR code.

### Get Status
`GET /session/status`
Returns connection status (`LoggedIn`, `Connected`, etc.).

### Get QR Code
`GET /session/qr`
Returns the QR code (as image or JSON depending on Accept header) for pairing.

### Pair Code (Phone Number)
`POST /session/pairphone`
Initiates pairing via 8-digit code.
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "show_code": true
    }
    ```

### Sync History
`GET /session/history`
Requests history sync from WhatsApp.
*   **Query Params**:
    *   `count`: Number of messages.
    *   `chat_jid`: Specific chat to sync.

---

## 3. Chat & Message Sending

### Send Text
`POST /chat/send/text`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "body": "Hello World",
      "reply_to_id": "optional-msg-id"
    }
    ```

### Send Image
`POST /chat/send/image`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "image": "data:image/jpeg;base64,..." or "https://...",
      "caption": "Check this!",
      "view_once": false
    }
    ```

### Send Video
`POST /chat/send/video`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "video": "data:video/mp4;base64,..." or "https://...",
      "caption": "Watch this"
    }
    ```

### Send Audio / Voice
`POST /chat/send/audio`
`POST /chat/send/voice` (PTT - Push To Talk waveform)
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "audio": "data:audio/mp3;base64,..." or "https://..."
    }
    ```

### Send Document/File
`POST /chat/send/doc`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "document": "data:application/pdf;base64,...",
      "fileName": "invoice.pdf",
      "caption": "Here is your invoice"
    }
    ```

### Send Sticker
`POST /chat/send/sticker`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "sticker": "data:image/webp;base64,...",
      "pack_name": "My Pack",
      "pack_publisher": "Me",
      "keep_scale": true
    }
    ```

### Send Location
`POST /chat/send/location`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "latitude": -23.5505,
      "longitude": -46.6333,
      "address": "São Paulo, SP",
      "name": "My Office"
    }
    ```

### Send Contact
`POST /chat/send/contact`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "name": "John Doe",
      "vcard": "BEGIN:VCARD..."
    }
    ```

### Send List
`POST /chat/send/list`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "title": "Menu",
      "description": "Choose an option",
      "buttonText": "Open Menu",
      "sections": [
        {
          "title": "Section 1",
          "rows": [
            {"rowId": "opt1", "title": "Option 1", "description": "Desc 1"},
            {"rowId": "opt2", "title": "Option 2"}
          ]
        }
      ]
    }
    ```

### Send Poll
`POST /chat/send/poll`
*   **Body**:
    ```json
    {
      "phone": "1203630XXXXX@g.us",
      "name": "Lunch?",
      "options": ["Pizza", "Sushi", "Burger"],
      "selectable_count": 1
    }
    ```

### Send Buttons
`POST /chat/send/buttons`
> **WARNING**: Not implemented / Does not work reliably on multi-device.

---

## 4. Message Operations

### Edit Message
`POST /chat/send/edit`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "message_id": "ABC123XYZ",
      "message": "Corrected text"
    }
    ```

### Delete Message (Revoke)
`POST /chat/delete`
*   **Body**:
    ```json
    {
      "phone": "5511999999999",
      "message_id": "ABC123XYZ"
    }
    ```

### Archive Chat
`POST /chat/archive`
*   **Body**:
    ```json
    {
      "jid": "5511999999999@s.whatsapp.net",
      "archive": true
    }
    ```

### Request Unavailable Message
`POST /chat/request-unavailable-message`
Requests a re-send of a message that failed to decrypt.
*   **Body**:
    ```json
    {
      "chat": "...",
      "sender": "...",
      "id": "..."
    }
    ```

---

## 5. Group Management

### Create Group
`POST /group/create`
*   **Body**:
    ```json
    {
      "name": "New Group",
      "participants": ["5511900000001", "5511900000002"]
    }
    ```

### Update Participants
`POST /group/participants`
*   **Body**:
    ```json
    {
      "groupjid": "123456789@g.us",
      "phone": ["5511900000003"],
      "action": "add" // or "remove", "promote", "demote"
    }
    ```

### Group Settings
*   **Set Name**: `POST /group/subject` -> `{"groupjid": "...", "name": "..."}`
*   **Set Description**: `POST /group/description` -> `{"groupjid": "...", "topic": "..."}`
*   **Set Photo**: `POST /group/photo` -> `{"groupjid": "...", "image": "data:..."}`
*   **Lock/Unlock**: `POST /group/lock` -> `{"groupjid": "...", "locked": true}`
*   **Announce/No-Announce**: `POST /group/announce` -> `{"groupjid": "...", "announce": true}`
*   **Get Invite Info**: `POST /group/invite-info` -> `{"code": "..."}`
*   **Leave**: `POST /group/leave` -> `{"groupjid": "..."}`

---

## 6. Configuration & Misc

### S3 Configuration
`POST /session/s3`
Configures S3 for media storage.
*   **Body**:
    ```json
    {
      "enabled": true,
      "endpoint": "s3.amazonaws.com",
      "bucket": "my-bucket",
      "access_key": "...",
      "secret_key": "..."
    }
    ```

### HMAC Configuration
`POST /session/hmac`
Sets HMAC key for webhook signature verification.
*   **Body**:
    ```json
    {
      "hmac_key": "..."
    }
    ```

### Proxy Configuration
`POST /session/proxy`
*   **Body**:
    ```json
    {
      "enable": true,
      "proxy_url": "http://user:pass@host:port"
    }
    ```

### Get User LID
`GET /user/lid/{jid}`
Returns the Local ID (LID) for a phone number.

### Reject Call
`POST /call/reject`
*   **Body**:
    ```json
    {
      "call_from": "...",
      "call_id": "..."
    }
    ```

### List Newsletters
`GET /newsletter/list`
Returns subscribed newsletters/channels.
