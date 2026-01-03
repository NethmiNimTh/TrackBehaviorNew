# TrackBehavior - Complete Demo Guide for Presentation

## Overview
This demo shows a **Multi-Device Location Tracking System with ML-Powered Anomaly Detection** that learns normal device behavior patterns and alerts users when unusual activity is detected.

---

## Demo Flow (15-20 minutes)

### Phase 1: System Setup & Introduction (3 min)

**What to show:**
- Explain the problem: How to detect if someone is using your device in an unauthorized location
- Solution: ML system that learns normal patterns of multiple devices and detects anomalies

**Key Points:**
- Uses Isolation Forest + KMeans for anomaly detection
- Real-time tracking with WebSocket communication
- Smart GPS filtering (rejects low accuracy, anchors to high accuracy points)

---

### Phase 2: Start the Application (2 min)

#### Terminal 1: Start Backend
```bash
cd backend
python app.py
```

**Expected Output:**
```
🚀 Starting server on port 5000
📍 Location validation settings:
   - High accuracy threshold: < 3.0m (anchor points)
   - Maximum acceptable accuracy: < 50.0m
   - Maximum position drift: 3.0m (for medium accuracy locations)
🏛️ University system enabled - 12x12 meter sections
🤖 ENHANCED ML Anomaly Detection: Active
   - Training: 5 minutes or 30 samples
   - Features: Distance, sections, speed, time, patterns
   - Detection: Pair anomalies + individual device anomalies
```

#### Terminal 2: Start Frontend
```bash
cd frontend
npm start
```

**Browser opens at:** http://localhost:3000

---

### Phase 3: User Registration & First Device (3 min)

**Step 1: Register User**
1. Click "Register"
2. Enter email: `demo@test.com`
3. Enter password: `demo123`
4. Click Register

**What happens internally:**
- User created in MongoDB
- JWT token generated
- Device fingerprint created (based on browser + OS)
- Redirects to Dashboard

**Step 2: Add First Device**
1. Device form appears automatically
2. Enter device name: `My Laptop`
3. Click "Add Device"

**Show in terminal:**
```
📱 Device added: device_id (first 8 chars)
OS: Windows/Mac
Browser: Chrome/Safari
```

**Explain:**
- System needs minimum 2 devices to start ML training
- Each device gets unique fingerprint based on browser, OS, system info

---

### Phase 4: Add Second Device (4 min)

**Option A: Use Different Browser**
1. Open app in different browser (Chrome → Firefox, or Chrome → Edge)
2. Login with same credentials: `demo@test.com` / `demo123`
3. Add device name: `My Phone`

**Option B: Use Incognito/Private Window**
1. Open incognito window
2. Navigate to http://localhost:3000
3. Login and add device

**What happens internally:**
- Second device registered
- System automatically starts ML training!
- Status changes to "Training in Progress"

**Show in terminal:**
```
🚀 Starting ML training for demo@test.com
⏳ ML Training for demo@test.com: 0/30 samples
```

---

### Phase 5: Location Permission & Campus Setup (3 min)

**On each device/browser:**
1. Click "Grant Location Permission"
2. Browser asks for location access → Allow
3. System creates virtual "university campus" at your current location

**Show in terminal:**
```
🏛️ University created at [latitude], [longitude]
🏛️ Generated university with 6 sections (12x12m each)
```

**Explain the 6 sections:**
- Main Building (Red)
- Library (Blue)
- New Building (Green)
- Canteen (Orange)
- Sports Complex (Purple)
- Admin Block (Teal)

**Map appears showing:**
- 6 colored rectangular sections (each 12x12 meters)
- Device markers with live positions
- Real-time updates via WebSocket

---

### Phase 6: ML Training Process (5 min)

**What's happening:**
- System collects behavior data every location update
- Tracks: distance between devices, sections, movement speed, time patterns
- Needs 30 samples OR 5 minutes (whichever comes first)

**Show in Dashboard:**
- Training status indicator
- Sample count: "Collecting behavior patterns: X/30 samples"
- Progress bar showing elapsed time

**Show in terminal:**
```
📌 Device abc12345... in section: Main Building
📌 Device def67890... in section: Library
⏳ ML Training for demo@test.com: 5/30 samples
```

**Explain features being tracked:**
1. Distance between devices
2. Section IDs (which building each device is in)
3. Movement speeds (meters/second)
4. Time of day patterns
5. Same section indicator
6. Moving together indicator
7. Speed differences

**Move devices around (simulate by moving/walking):**
- Watch sections change in real-time
- See sample count increase
- Explain how it's learning YOUR normal patterns

**When training completes:**
```
🤖 Training ML model for demo@test.com with 30 samples...
✅ ML Model trained successfully for demo@test.com
💾 Model saved to models/demo@test.com_model.pkl
```

**Dashboard shows:**
- "Security system activated!" notification
- ML Status: Trained ✓
- Training samples: 30

---

### Phase 7: Anomaly Detection Demo (5 min)

**Now the ML model is active and monitoring!**

#### Test 1: Normal Behavior
**Action:** Keep both devices in same/nearby sections
**Expected:** No alerts (green status)
**Explain:** "Model learned these patterns are normal for me"

#### Test 2: Distance Anomaly
**Action:** Move one device far from the other (simulate by asking someone to walk away with one device)
**Expected:**
```
Terminal:
🚨 ANOMALY DETECTED for demo@test.com!
   Score: -0.753 (threshold: -0.500)
   Device 1: Main Building
   Device 2: Sports Complex
   Distance: 85.3m
   Confidence: 0.75
```

**Dashboard shows:**
- 🚨 Red anomaly alert popup
- "Unusual device behavior detected!"
- Shows which devices, distance, confidence score

#### Test 3: One Device Leaves Campus
**Action:** Move one device outside all sections (or simulate by changing section to "Outside Campus")
**Expected:**
```
Terminal:
🚨 ANOMALY DETECTED
   Reason: Device left campus while companion stayed
```

**Dashboard shows:**
- Individual device anomaly alert
- "Device left campus while companion stayed"

#### Test 4: High Speed Movement (if possible)
**Action:** Rapid movement detected (>5 m/s = 18 km/h)
**Expected:**
- "High speed: 6.2 m/s" alert

---

### Phase 8: Show the ML Internals (3 min)

**Open browser console/Network tab:**
- Show WebSocket connection active
- Real-time messages flowing: `location_update`, `ml_status_update`, `anomaly_alert`

**Show in code (quick walkthrough):**

1. **ML Model (ml_model.py:93-121):**
   ```python
   self.model = IsolationForest(
       contamination=0.15,  # Expects 15% anomalies
       n_estimators=150,
       random_state=42
   )
   ```

2. **Feature Extraction (ml_model.py:24-51):**
   - Shows 16 features being extracted from behavior data

3. **Anomaly Prediction (ml_model.py:123-172):**
   - Score calculation
   - Threshold comparison
   - Confidence computation

4. **Saved Model:**
   ```bash
   ls backend/models/
   # Shows: demo@test.com_model.pkl
   ```

**Explain:**
- Model persists to disk
- On server restart, loads existing models
- Continues monitoring without retraining

---

## Technical Highlights for Q&A

### Architecture
- **Backend:** Flask + SocketIO (Python)
- **Frontend:** React with real-time WebSocket
- **Database:** MongoDB (Atlas cloud)
- **ML:** scikit-learn (Isolation Forest + KMeans)

### ML Pipeline
1. **Data Collection:** Real-time device location + behavior patterns
2. **Feature Engineering:** 16 features extracted from device pairs
3. **Training:** Isolation Forest (unsupervised anomaly detection)
4. **Clustering:** KMeans for pattern recognition (3 clusters)
5. **Prediction:** Real-time anomaly scoring with confidence intervals
6. **Persistence:** Models saved as pickle files

### Smart GPS Filtering
- **High accuracy (<3m):** Accepted as anchor points
- **Medium accuracy (3-50m):** Constrained to 3m drift from anchor
- **Low accuracy (>50m):** Rejected completely
- Prevents GPS noise from triggering false alarms

### Use Cases
1. **Device Theft Detection:** Alert if device is in unusual location
2. **Account Sharing Detection:** Detect if multiple people using same account
3. **Family Safety:** Monitor if devices are together as expected
4. **Corporate Security:** Detect unauthorized device usage patterns

---

## Common Demo Issues & Solutions

### Issue: "Only 1 device registered"
**Solution:** Open second browser/incognito window and login with same account

### Issue: "Location not updating"
**Solution:**
- Check browser location permissions
- Try refreshing location manually
- Check network connectivity

### Issue: "ML training stuck at 0 samples"
**Solution:**
- Make sure 2+ devices are active
- Both devices need location permission granted
- Move devices to trigger location updates

### Issue: "No anomalies detected"
**Solution:**
- Training needs to complete first (30 samples or 5 min)
- Try more extreme scenarios (large distance, one device outside campus)
- Check anomaly threshold in terminal output

---

## Presentation Tips

1. **Start with the problem:** "How do we know if someone else is using our device?"

2. **Show the solution:** "ML learns normal patterns and alerts on anomalies"

3. **Live demo:** Always better than slides - show real-time tracking

4. **Explain the ML:** "Unsupervised learning - no labeled data needed"

5. **Highlight features:**
   - Real-time WebSocket communication
   - Smart GPS filtering (production-ready)
   - Model persistence (survives restarts)
   - Confidence scores (explainable AI)

6. **End with use cases:** Theft detection, account security, family safety

---

## Quick Command Reference

### Start Backend
```bash
cd backend
python app.py
```

### Start Frontend
```bash
cd frontend
npm start
```

### Check MongoDB
```bash
# Show collections
mongosh "<your-mongo-uri>"
show collections
db.users.find()
db.devices.find()
db.device_behaviors.find().limit(5)
```

### Check Trained Models
```bash
ls -lh backend/models/
python -c "import pickle; print(pickle.load(open('backend/models/demo@test.com_model.pkl', 'rb')))"
```

### Test API Endpoints
```bash
# Health check
curl http://localhost:5000/api/health

# System status (requires auth token)
curl -H "Authorization: Bearer <token>" http://localhost:5000/api/system-status
```

---

## Demo Checklist

- [ ] MongoDB connection working
- [ ] Backend server running on port 5000
- [ ] Frontend running on port 3000
- [ ] Can register new user
- [ ] Can add first device
- [ ] Can open second browser for second device
- [ ] Both devices can get location permission
- [ ] Map shows both devices in real-time
- [ ] ML training starts automatically
- [ ] Can see sample count increasing
- [ ] Training completes (30 samples or 5 min)
- [ ] Can trigger anomaly by separating devices
- [ ] Anomaly alert appears in dashboard
- [ ] Can see alerts in terminal logs

---

Good luck with your presentation! 🚀
