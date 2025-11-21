# SafeGuard Architecture Documentation

## System Overview

SafeGuard is a distributed safety application with the following main components:

```
┌────────────────────────────────────────────────────────────────────┐
│                      FRONTEND (React + Vite)                       │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    User Interface Layer                      │ │
│  │  ┌─────────────┐ ┌────────────┐ ┌──────────┐ ┌────────────┐ │ │
│  │  │Home Screen  │ │Emergency   │ │Tracking  │ │Police      │ │ │
│  │  │Safety ON/OFF│ │Contacts    │ │Screen    │ │Dashboard   │ │ │
│  │  └─────────────┘ └────────────┘ └──────────┘ └────────────┘ │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                  Service Layer                               │ │
│  │                                                               │ │
│  │  Audio Detection Service ──┐                                 │ │
│  │  ├─ Web Audio API          │                                 │ │
│  │  ├─ Mel Spectrogram        │                                 │ │
│  │  ├─ Feature Extraction     │                                 │ │
│  │  └─ Model Inference        │                                 │ │
│  │                             │                                 │ │
│  │  Location Service          │─────→ Emergency Alert Service  │ │
│  │  ├─ Geolocation API        │                                 │ │
│  │  ├─ Continuous Tracking    │                                 │ │
│  │  └─ Distance Calculation   │                                 │ │
│  │                             │                                 │ │
│  │  Sensor Services           │                                 │ │
│  │  ├─ Shake Detection        │                                 │ │
│  │  ├─ Keyword Detection      │                                 │ │
│  │  └─ Multi-sensor Fusion    │                                 │ │
│  │                             │                                 │ │
│  │  State Management          │                                 │ │
│  │  ├─ Redux / Context        │                                 │ │
│  │  └─ Local Storage          │                                 │ │
│  │                             │                                 │ │
│  │  API Service               │                                 │ │
│  │  ├─ REST Endpoints         │                                 │ │
│  │  ├─ WebSocket Connections  │                                 │ │
│  │  └─ Error Handling         │                                 │ │
│  │                             │                                 │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │            Local Data Storage (IndexedDB/LocalStorage)       │ │
│  │  ├─ Emergency Contacts                                       │ │
│  │  ├─ Alert History                                            │ │
│  │  ├─ Location History                                         │ │
│  │  ├─ User Preferences                                         │ │
│  │  └─ Offline Cache                                            │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────────┐
│                      BACKEND SERVICES                              │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────�� │
│  │                     Firebase                                 │ │
│  │  ┌─────────────────┐ ┌──────────────┐ ┌────────────────────┐│ │
│  │  │ Authentication  │ │  Firestore   │ │ Cloud Functions    ││ │
│  │  │ ├─ Email/Pass   │ │ ├─ Users     │ │ ├─ Alert Handler   ││ │
│  │  │ ├─ Social Login │ │ ├─ Alerts    │ │ ├─ Notification    ││ │
│  │  │ └─ Session Mgmt │ │ └─ Contacts  │ │ └─ Analytics       ││ │
│  │  └─────────────────┘ └──────────────┘ └────────────────────┘│ │
│  │                                                               │ │
│  │  ┌──────────────────────────────────────────────────────────┐│ │
│  │  │           Real-time Database (Realtime DB)             ││ │
│  │  │  ├─ Active Alerts Feed                                  ││ │
│  │  │  ├─ Location Updates                                    ││ │
│  │  │  └─ Presence Status                                     ││ │
│  │  └──────────────────────────────────────────────────────────┘│ │
│  │                                                               │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                  External Services                           │ │
│  │  ├─ SMS Gateway (Twilio)                                     │ │
│  │  ├─ Push Notifications (FCM)                                 │ │
│  │  ├─ Email Service (SendGrid)                                 │ │
│  │  └─ Maps API (Google Maps / OpenStreetMap)                   │ │
│  │                                                               │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

## Component Diagram

### Frontend Components

```
App.tsx
├── Layout
│   ├── Header
│   └── Navigation
│
├── Pages
│   ├── Index (Home/Safety Screen)
│   │   ├── SafetyToggle
│   │   ├── AudioLevelIndicator
│   │   ├── LocationStatus
│   │   ├── QuickActionCards
│   │   ├── HowItWorksSection
│   │   └── SafetyTipsSection
│   │
│   ├── EmergencyAlert
│   │   ├── AlertHeader
│   │   ├── CountdownTimer
│   │   ├── LocationDisplay
│   │   ├── CancelButton
│   │   └── StatusIndicator
│   │
│   ├── EmergencyContacts
│   │   ├── AddContactForm
│   │   ├── ContactsList
│   │   ��── ContactCard
│   │
│   ├── LiveTracking
│   │   ├── MapContainer
│   │   ├── LocationDetails
│   │   ├── AccuracyIndicator
│   │   └── LocationHistory
│   │
│   └── PoliceDashboard
│       ├── AlertStats
│       ├── FilterTabs
│       ├── AlertsList
│       └── AlertDetails
│
└── Shared Components
    ├── Button
    ├── Card
    ├── Modal
    └── Notification
```

## Data Flow Diagram

### Emergency Alert Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                     Audio Detection                             │
│                                                                 │
│  Microphone Input → Web Audio API → Mel Spectrogram Extraction │
│                                            ↓                     │
│                                    Feature Normalization        │
│                                            ↓                     │
│                                  ML Model Inference              │
│                                            ↓                     │
│                                    Prediction (Confidence)       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                   Threshold Check (>0.7)
                        ↙         ↖
                   YES            NO → Continue Listening
                      ↓
┌─────────────────────────────────────────────────────────────────┐
│              Alert Trigger (or Manual)                           │
│                                                                 │
│  Alert Creation → Location Acquisition → Alert Validation       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                    5-Second Countdown
                   ↙                ↖
              CANCEL          SEND ALERT
                ↓                    ↓
          Discard Alert      ┌─────────────────────┐
                             │  Multi-notification │
                             │                     │
                             ├─ Firestore Update   │
                             ├─ Contact SMS/Push   │
                             ├─ Police API         │
                             ├─ Real-time Updates  │
                             └─ Analytics Logging  │
```

## Service Interaction Diagram

```
┌──────────────────────────┐
│  AudioDetectionService   │
│                          │
│  • startListening()      │──┐
│  • stopListening()       │  │
│  • getAudioLevel()       │  │
│  • isActive()            │  │
└──────────────────────────┘  │
                              │
┌──────────────────────────┐  │  ┌─────────────────────────┐
│ LocationService          │  ├─→│ EmergencyAlertService   │
│                          │  │  │                         │
│ • getCurrentLocation()   │  │  │ • createAlert()         │
│ • startWatching()        │  │  │ • sendToContacts()      │
│ • stopWatching()         │  │  │ • sendToPolice()        │
│ • getLastLocation()      │  │  │ • cancelAlert()         │
└──────────────────────────┘  │  │ • updateAlertStatus()   │
                              │  └─────────────────────────┘
┌──────────────────────────┐  │          ↓
│  ShakeDetectionService   │  │  ┌──────────────────┐
│                          │──┤  │  API Service     │
│  • startListening()      │  │  │                  │
│  • stopListening()       │  │  │ • sendAlert()    │
│  • isActive()            │  │  │ • updateStatus() │
└──────────────────────────┘  │  │ • getAlerts()    │
                              │  └──────────────────┘
┌──────────────────────────┐  │
│KeywordDetectionService   │  │
│                          │  │
│ • startListening()       │──┘
│ • stopListening()        │
│ • isActive()             │
└──────────────────────────┘
```

## State Management

### Redux Store Structure (if using Redux)

```
{
  auth: {
    user: User | null
    isAuthenticated: boolean
    authLoading: boolean
  }
  
  safety: {
    isActive: boolean
    audioLevel: number
    isListening: boolean
    lastDetection: DetectionResult | null
  }
  
  location: {
    current: LocationData | null
    history: LocationData[]
    isTracking: boolean
    accuracy: number
  }
  
  alerts: {
    activeAlerts: Alert[]
    alertHistory: Alert[]
    selectedAlert: Alert | null
    isAlertActive: boolean
  }
  
  contacts: {
    emergencyContacts: Contact[]
    isLoading: boolean
  }
  
  ui: {
    theme: 'light' | 'dark'
    notifications: Notification[]
    modals: { [key: string]: boolean }
  }
}
```

## Database Schema

### Firestore Collections

```javascript
// Users Collection
collection('users')
  ├── doc(userId)
  │   ├── email: string
  │   ├── name: string
  │   ├── phone: string
  │   ├── profileImage: string
  │   ├── emergencyContacts: Reference[]
  │   ├── createdAt: timestamp
  │   └── updatedAt: timestamp

// Alerts Collection
collection('alerts')
  ├── doc(alertId)
  │   ���── userId: Reference (to users)
  │   ├── type: string (PANIC_SCREAM | HELP_CRY | DISTRESS_SOUND)
  │   ├── severity: string (HIGH | CRITICAL)
  │   ├── source: string (AUDIO_DETECTION | SHAKE | KEYWORD | MANUAL)
  │   ├── location: GeoPoint { latitude, longitude }
  │   ├── accuracy: number
  │   ├── timestamp: timestamp
  │   ├── status: string (ACTIVE | SENT | ACKNOWLEDGED | RESPONDED | RESOLVED)
  │   ├── contactsNotified: array<Contact>
  │   ├── policeNotified: boolean
  │   ├── respondingOfficer: string
  │   ├── responseTime: timestamp
  │   └── notes: string

// Emergency Contacts Collection
collection('contacts')
  ├── doc(contactId)
  │   ├── userId: Reference (to users)
  │   ├── name: string
  │   ├── phone: string
  │   ├── email: string
  │   ├── relationship: string
  │   ├── notificationPreference: string (SMS | EMAIL | PUSH)
  │   ├── verified: boolean
  │   ├── createdAt: timestamp
  │   └── updatedAt: timestamp

// Alert Log / Analytics
collection('alertLogs')
  ├── doc(logId)
  │   ├── alertId: Reference (to alerts)
  │   ├── event: string (TRIGGERED | SENT | ACKNOWLEDGED | RESOLVED)
  │   ├── timestamp: timestamp
  │   ├── metadata: object
  │   └── details: string
```

## API Endpoints

### REST Endpoints

```
POST /api/alerts
├── Create emergency alert
└── Request: { type, location, source }
    Response: { alertId, status }

GET /api/alerts/:alertId
├── Get alert details
└── Response: { Alert }

PUT /api/alerts/:alertId/status
├── Update alert status
└── Request: { status }
    Response: { success }

POST /api/contacts
├── Add emergency contact
└── Request: { name, phone, email }
    Response: { contactId }

GET /api/contacts
├── Get all emergency contacts
└── Response: { Contact[] }

DELETE /api/contacts/:contactId
├── Delete contact
└── Response: { success }

GET /api/dashboard/alerts
├── Get police dashboard alerts (admin only)
└── Response: { Alert[] }

PUT /api/dashboard/alerts/:alertId/respond
├── Mark alert as responded (admin only)
└── Request: { officerId, notes }
    Response: { success }
```

### WebSocket Events (Real-time)

```
Client → Server:
  • "listen-alerts" → Subscribe to new alerts
  • "listen-location" → Subscribe to location updates

Server → Client:
  • "new-alert" → New emergency alert triggered
  • "alert-status-updated" → Alert status changed
  • "location-updated" → Location update received
```

## Security Considerations

### Data Protection

1. **Encryption in Transit**
   - HTTPS/TLS for all API communications
   - WebSocket Secure (WSS) for real-time updates

2. **Encryption at Rest**
   - Firebase encryption (automatic)
   - Device-level encryption for local storage

3. **Authentication**
   - Firebase Authentication
   - JWT tokens for API calls
   - Role-based access control (RBAC)

### Privacy

- Audio data never stored or transmitted
- Location data sent only to authorized contacts
- User consent required for all permissions
- GDPR compliance
- Data retention policies

## Performance Optimization

### Frontend Optimization
- Code splitting with React.lazy()
- Lazy loading of maps and heavy components
- Service Workers for offline support
- Image optimization and lazy loading
- Minification and tree-shaking

### Backend Optimization
- Firestore indexing for fast queries
- Cloud Functions deployment near users
- Caching strategies for frequently accessed data
- Rate limiting for API endpoints

### Audio Processing
- Web Workers for heavy computation
- Efficient Mel Spectrogram calculation
- Batch processing for multiple detections
- Memory optimization for mobile devices

## Deployment Architecture

```
┌─────────────────────────────────────────┐
│          Application Layer              │
│  (React SPA - Vite Build Output)        │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│        Static Hosting & CDN              │
│  (Firebase Hosting / Netlify)            │
│  ├─ Global Edge Caching                 │
│  ├─ Automatic HTTPS                     │
│  └─ DDoS Protection                     │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│       API & Backend Services             │
│  ├─ Firebase Firestore (Database)        │
│  ├─ Cloud Functions (Serverless)         │
│  ├─ Authentication                       │
│  └─ Real-time Database                   │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│      Third-party Integrations            │
│  ├─ SMS Gateway (Twilio)                 │
│  ├─ Push Notifications (FCM)             │
│  ├─ Email Service                        │
│  └─ Maps API                             │
└─────────────────────────────────────────┘
```

## Monitoring & Analytics

### Metrics to Track
- Alert detection rate and accuracy
- Average response time
- False positive rate
- User engagement metrics
- Performance metrics (latency, uptime)

### Logging
- All alert events logged
- API request/response logging
- Error tracking (Sentry)
- User action tracking

### Alerting
- Critical error notifications
- Performance degradation alerts
- High false positive rate alerts
- Service availability monitoring

---

For more information, see README.md and the inline code documentation.
