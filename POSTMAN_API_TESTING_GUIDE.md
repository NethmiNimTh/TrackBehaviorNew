# TrackBehavior API - Postman Testing Guide

## Base URL
```
http://localhost:5000
```

Or if using your network IP:
```
http://172.20.10.7:5000
```

---

## Testing Order (Follow this sequence)

### 1. Health Check
### 2. Register User
### 3. Login User
### 4. Check Device
### 5. Add Device
### 6. Grant Location Permission
### 7. Get User Devices
### 8. Get University Layout
### 9. Get All Device Locations
### 10. Get ML Status
### 11. Start ML Training
### 12. Get Device Patterns
### 13. System Status

---

## API Endpoints with Postman Setup

### 1. Health Check (No Auth Required)

**Endpoint:** `GET /api/health`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/health`
- Headers: None required
- Body: None

**Expected Response (200 OK):**
```json
{
    "status": "healthy",
    "timestamp": "2025-01-02T10:30:00.123456",
    "database": "connected",
    "ml_models": 0,
    "active_users": 0
}
```

**What it tests:**
- Server is running
- MongoDB connection is working
- ML models directory exists

---

### 2. Register User (No Auth Required)

**Endpoint:** `POST /api/register`

**Postman Setup:**
- Method: `POST`
- URL: `http://localhost:5000/api/register`
- Headers:
  ```
  Content-Type: application/json
  ```
- Body (raw JSON):
  ```json
  {
      "email": "test@demo.com",
      "password": "test123"
  }
  ```

**Expected Response (201 Created):**
```json
{
    "message": "User registered successfully",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
        "_id": "507f1f77bcf86cd799439011",
        "email": "test@demo.com",
        "created_at": "2025-01-02T10:30:00.123456",
        "devices": [],
        "location_permission": false,
        "last_login": "2025-01-02T10:30:00.123456"
    }
}
```

**IMPORTANT:**
- Copy the `token` value from the response
- Save it as a Postman variable or environment variable
- You'll need this token for all subsequent requests

**To save token in Postman:**
1. Go to Tests tab in Postman
2. Add this script:
   ```javascript
   var jsonData = pm.response.json();
   pm.environment.set("auth_token", jsonData.token);
   ```

---

### 3. Login User (No Auth Required)

**Endpoint:** `POST /api/login`

**Postman Setup:**
- Method: `POST`
- URL: `http://localhost:5000/api/login`
- Headers:
  ```
  Content-Type: application/json
  ```
- Body (raw JSON):
  ```json
  {
      "email": "test@demo.com",
      "password": "test123"
  }
  ```

**Expected Response (200 OK):**
```json
{
    "message": "Login successful",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
        "_id": "507f1f77bcf86cd799439011",
        "email": "test@demo.com",
        "created_at": "2025-01-02T10:30:00.123456",
        "devices": [],
        "location_permission": false,
        "last_login": "2025-01-02T10:35:00.123456"
    },
    "device_exists": false,
    "device_id": "abc123def456..."
}
```

**Save the token again using the Tests script above**

---

### 4. Check Device (Requires Auth)

**Endpoint:** `GET /api/check-device`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/check-device`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  Content-Type: application/json
  ```
- Body: None

**Expected Response (200 OK):**
```json
{
    "device_id": "abc123def456789...",
    "device_exists": false,
    "user_has_device": false,
    "device_status": "not_registered",
    "device_owner": null,
    "os": "Windows",
    "location_permission": false
}
```

**What it tests:**
- JWT authentication working
- Device fingerprint generation
- Device registration status

---

### 5. Add Device (Requires Auth)

**Endpoint:** `POST /api/add-device`

**Postman Setup:**
- Method: `POST`
- URL: `http://localhost:5000/api/add-device`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  Content-Type: application/json
  ```
- Body (raw JSON):
  ```json
  {
      "device_name": "My Test Device"
  }
  ```

**Expected Response (201 Created):**
```json
{
    "message": "Device added successfully",
    "device": {
        "_id": "507f1f77bcf86cd799439012",
        "device_id": "abc123def456789...",
        "device_name": "My Test Device",
        "user_email": "test@demo.com",
        "added_at": "2025-01-02T10:40:00.123456",
        "os": "Windows",
        "browser": "Chrome",
        "user_agent": "Mozilla/5.0...",
        "last_seen": "2025-01-02T10:40:00.123456",
        "location_tracking": false,
        "current_section": "Outside Campus"
    },
    "device_status": {
        "device_id": "abc123def456789...",
        "device_exists": true,
        "user_has_device": true,
        "device_status": "registered_to_me",
        "device_owner": "test@demo.com",
        "os": "Windows"
    },
    "ml_training_started": false
}
```

**Note:** ML training requires 2+ devices, so `ml_training_started` will be false

**To add second device:**
- You need a different browser/device fingerprint
- In Postman, you can simulate this by changing the User-Agent header
- Or use a different Postman workspace/agent

---

### 6. Grant Location Permission (Requires Auth)

**Endpoint:** `POST /api/grant-location-permission`

**Postman Setup:**
- Method: `POST`
- URL: `http://localhost:5000/api/grant-location-permission`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  Content-Type: application/json
  ```
- Body (raw JSON):
  ```json
  {
      "device_id": "abc123def456789...",
      "latitude": 37.7749,
      "longitude": -122.4194
  }
  ```

**Note:** Replace `device_id` with the actual device ID from the previous response

**Expected Response (200 OK):**
```json
{
    "message": "Location permission granted successfully",
    "location_permission": true,
    "device_id": "abc123def456789...",
    "university": {
        "_id": "507f1f77bcf86cd799439013",
        "user_email": "test@demo.com",
        "center": {
            "lat": 37.7749,
            "lon": -122.4194
        },
        "sections": [
            {
                "name": "Main Building",
                "color": "#e74c3c",
                "bounds": {
                    "min_lat": 37.774738,
                    "max_lat": 37.775008,
                    "min_lon": -122.419562,
                    "max_lon": -122.419238
                },
                "center": {
                    "lat": 37.774873,
                    "lon": -122.4194
                },
                "size_meters": 12.0
            },
            // ... 5 more sections
        ],
        "created_at": "2025-01-02T10:45:00.123456",
        "total_size_meters": 36.0
    }
}
```

**What it creates:**
- Virtual university campus at the specified location
- 6 sections (12x12 meters each)
- Enables location tracking for the device

---

### 7. Get User Devices (Requires Auth)

**Endpoint:** `GET /api/user-devices`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/user-devices`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  ```
- Body: None

**Expected Response (200 OK):**
```json
{
    "devices": [
        {
            "device_id": "abc123def456789...",
            "device_name": "My Test Device",
            "os": "Windows",
            "location_tracking": true,
            "last_seen": "2025-01-02T10:45:00.123456",
            "current_section": "Outside Campus"
        }
    ]
}
```

---

### 8. Get University Layout (Requires Auth)

**Endpoint:** `GET /api/university-layout`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/university-layout`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  ```
- Body: None

**Expected Response (200 OK):**
```json
{
    "university": {
        "_id": "507f1f77bcf86cd799439013",
        "user_email": "test@demo.com",
        "center": {
            "lat": 37.7749,
            "lon": -122.4194
        },
        "sections": [
            {
                "name": "Main Building",
                "color": "#e74c3c",
                "bounds": {...},
                "center": {...},
                "size_meters": 12.0
            },
            // ... 5 more sections
        ],
        "created_at": "2025-01-02T10:45:00.123456",
        "total_size_meters": 36.0
    }
}
```

---

### 9. Get All Device Locations (Requires Auth)

**Endpoint:** `GET /api/all-devices-locations`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/all-devices-locations`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  ```
- Body: None

**Expected Response (200 OK):**
```json
{
    "locations": [
        {
            "device_id": "abc123def456789...",
            "device_name": "My Test Device",
            "os": "Windows",
            "latitude": 37.7749,
            "longitude": -122.4194,
            "accuracy": 10.5,
            "timestamp": "2025-01-02T10:45:00.123456",
            "is_online": true,
            "current_section": "Main Building"
        }
    ]
}
```

**Note:** Will be empty if no location updates have been sent yet

---

### 10. Get ML Status (Requires Auth)

**Endpoint:** `GET /api/ml-status`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/ml-status`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  ```
- Body: None

**Expected Response - Not Started (200 OK):**
```json
{
    "is_training": false,
    "is_trained": false,
    "training_samples": 0,
    "device_count": 1,
    "can_start_training": false,
    "message": "Add 1 more device(s) to start ML training"
}
```

**Expected Response - Training (200 OK):**
```json
{
    "is_training": true,
    "is_trained": false,
    "training_samples": 15,
    "training_started": "2025-01-02T10:50:00.123456",
    "training_completed": null,
    "model_info": {},
    "message": "",
    "elapsed_minutes": 2.5,
    "remaining_minutes": 2.5,
    "progress_percentage": 50
}
```

**Expected Response - Trained (200 OK):**
```json
{
    "is_training": false,
    "is_trained": true,
    "training_samples": 30,
    "training_started": "2025-01-02T10:50:00.123456",
    "training_completed": "2025-01-02T10:55:00.123456",
    "model_info": {
        "status": "Trained",
        "training_start_time": "2025-01-02T10:50:00.123456",
        "anomaly_threshold": -0.523,
        "normal_patterns_count": 3,
        "model_type": "Isolation Forest + KMeans"
    },
    "message": ""
}
```

---

### 11. Start ML Training (Requires Auth)

**Endpoint:** `POST /api/start-ml-training`

**Postman Setup:**
- Method: `POST`
- URL: `http://localhost:5000/api/start-ml-training`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  Content-Type: application/json
  ```
- Body: None (empty JSON `{}`)

**Expected Response - Success (200 OK):**
```json
{
    "success": true,
    "message": "ML training started successfully. Will train for 5 minutes.",
    "estimated_completion": "2025-01-02T10:55:00.123456"
}
```

**Expected Response - Error (400 Bad Request):**
```json
{
    "success": false,
    "message": "Need 2+ devices to start ML training. Currently have 1."
}
```

**Note:** Requires 2+ devices to work

---

### 12. Get Device Patterns (Requires Auth)

**Endpoint:** `GET /api/device-patterns`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/device-patterns`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  ```
- Body: None

**Expected Response (200 OK):**
```json
{
    "patterns": {
        "abc123def456789...": {
            "user_email": "test@demo.com",
            "device_id": "abc123def456789...",
            "section_visits": {
                "1": 5,
                "2": 3,
                "4": 2
            },
            "section_percentages": {
                "1": 50.0,
                "2": 30.0,
                "4": 20.0
            },
            "typical_sections": [1, 2, 4],
            "movement_patterns": [
                {
                    "speed": 0.5,
                    "timestamp": "2025-01-02T10:45:00.123456",
                    "section_id": 1
                }
            ],
            "companion_devices": ["def456ghi789..."],
            "created_at": "2025-01-02T10:45:00.123456",
            "updated_at": "2025-01-02T10:50:00.123456"
        }
    },
    "device_count": 1
}
```

---

### 13. System Status (Requires Auth)

**Endpoint:** `GET /api/system-status`

**Postman Setup:**
- Method: `GET`
- URL: `http://localhost:5000/api/system-status`
- Headers:
  ```
  Authorization: Bearer {{auth_token}}
  ```
- Body: None

**Expected Response (200 OK):**
```json
{
    "status": "online",
    "users": 1,
    "devices": 1,
    "active_locations": 1,
    "behavior_records": 15,
    "trained_ml_models": 0,
    "active_ml_models": 0,
    "timestamp": "2025-01-02T10:55:00.123456"
}
```

---

## Testing WebSocket Endpoints

WebSocket endpoints cannot be tested directly in Postman, but you can:

1. Use a WebSocket client like:
   - **Postman WebSocket** (newer versions support it)
   - **wscat** command-line tool
   - **Socket.io client** in browser console

### WebSocket Events

**Connection URL:**
```
ws://localhost:5000/socket.io/?EIO=4&transport=websocket
```

**Events to emit:**
- `connect` - Connect to server
- `join_room` - Join user's room
- `update_location` - Send location update

**Events to listen for:**
- `location_update` - Real-time location updates
- `ml_status_update` - ML training status changes
- `ml_training_progress` - Training progress updates
- `ml_training_complete` - Training completed
- `anomaly_alert` - Anomaly detected
- `individual_anomaly` - Individual device anomaly
- `location_rejected` - Location rejected

### Example: Join Room (using Socket.io client)
```javascript
socket.emit('join_room', {
    user_email: 'test@demo.com'
});
```

### Example: Update Location
```javascript
socket.emit('update_location', {
    device_id: 'abc123def456789...',
    latitude: 37.7749,
    longitude: -122.4194,
    accuracy: 10.5,
    user_email: 'test@demo.com'
});
```

---

## Complete Testing Workflow

### Step 1: Basic Setup (5 min)
1. ✅ Health Check
2. ✅ Register User (save token)
3. ✅ Check Device
4. ✅ Add Device #1

### Step 2: Location Setup (3 min)
5. ✅ Grant Location Permission (creates university)
6. ✅ Get University Layout
7. ✅ Get User Devices

### Step 3: Add Second Device (5 min)
**Note:** You need a different device fingerprint

**Option A: Change User-Agent in Postman**
- In request Headers, add:
  ```
  User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)
  ```
- Repeat: Check Device → Add Device

**Option B: Use different Postman workspace**

8. ✅ Add Device #2
9. ✅ Get User Devices (should show 2 devices)
10. ✅ Get ML Status (should show can_start_training: true)

### Step 4: ML Training (10 min)
11. ✅ Start ML Training
12. ✅ Get ML Status (check training progress)
13. Wait 5 minutes or send 30 location updates
14. ✅ Get ML Status (should show is_trained: true)
15. ✅ Get Device Patterns
16. ✅ System Status

---

## Error Codes Reference

| Code | Error | Cause |
|------|-------|-------|
| 200 | OK | Success |
| 201 | Created | Resource created |
| 400 | Bad Request | Missing/invalid parameters |
| 401 | Unauthorized | Missing/invalid token |
| 404 | Not Found | Resource not found |
| 500 | Internal Server Error | Server/database error |

---

## Common Testing Issues

### Issue: "Token is missing" (401)
**Solution:** Add Authorization header: `Bearer {{auth_token}}`

### Issue: "Token is invalid" (401)
**Solution:**
- Re-login to get new token
- Check token format (must be `Bearer <token>`)
- Token expires after 7 days

### Issue: "Device already registered" (400)
**Solution:** Use different User-Agent header or different Postman workspace

### Issue: "Need 2+ devices to start ML training" (400)
**Solution:** Add second device with different fingerprint

### Issue: Connection refused
**Solution:**
- Check backend is running: `python backend/app.py`
- Verify port 5000 is not in use
- Check firewall settings

---

## Postman Collection Setup

### Environment Variables
Create a Postman environment with:

| Variable | Value |
|----------|-------|
| base_url | http://localhost:5000 |
| auth_token | (auto-set by Tests script) |
| device_id | (auto-set after Add Device) |
| user_email | test@demo.com |

### Pre-request Script (Global)
```javascript
// Optional: Auto-refresh token if expired
if (pm.environment.get("auth_token")) {
    pm.request.headers.add({
        key: 'Authorization',
        value: 'Bearer ' + pm.environment.get("auth_token")
    });
}
```

---

## Quick Test Commands (cURL alternatives)

### Register
```bash
curl -X POST http://localhost:5000/api/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@demo.com","password":"test123"}'
```

### Login
```bash
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@demo.com","password":"test123"}'
```

### Health Check
```bash
curl http://localhost:5000/api/health
```

---

Good luck with testing! 🚀
