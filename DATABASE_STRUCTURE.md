# Database Structure Documentation

This document describes the Firebase Firestore database collections and structure needed for the Medical ECG Project.

## Collections Overview

### 1. `users` Collection

Stores user profile information for both patients and doctors.

**Collection Path:** `users/{userId}`

**Document Structure:**
```javascript
{
  uid: string,                    // Firebase Auth UID
  fullName: string,               // User's full name
  email: string,                  // User's email address
  contact: string,                // Phone number
  role: "patient" | "doctor",    // User role
  createdAt: timestamp,           // Account creation timestamp
  verified: boolean,              // Email verification status
  
  // Patient-specific fields
  dob?: string,                   // Date of birth (YYYY-MM-DD)
  emergencyContact?: string,      // Emergency contact name & phone
  
  // Doctor-specific fields
  license?: string,               // Medical license number
  specialization?: string,        // Medical specialization (e.g., "Cardiology")
}
```

**Example Patient Document:**
```javascript
{
  uid: "abc123...",
  fullName: "John Doe",
  email: "john@example.com",
  contact: "+1234567890",
  role: "patient",
  createdAt: Timestamp,
  verified: true,
  dob: "1990-01-15",
  emergencyContact: "Jane Doe +1234567891"
}
```

**Example Doctor Document:**
```javascript
{
  uid: "xyz789...",
  fullName: "Dr. Sarah Smith",
  email: "dr.smith@hospital.com",
  contact: "+1987654321",
  role: "doctor",
  createdAt: Timestamp,
  verified: true,
  license: "LIC-12345678",
  specialization: "Cardiology"
}
```

---

### 2. `ecg_data` Collection

Stores ECG reading data for each patient. This uses a subcollection structure for organized data storage.

**Collection Path:** `ecg_data/{userId}/readings/{readingId}`

**Document Structure (in readings subcollection):**
```javascript
{
  timestamp: timestamp,           // Reading timestamp
  heartRate: number,              // Heart rate in BPM
  signalQuality: number,          // Signal quality percentage (0-100)
  leadsOff: boolean,              // Leads connection status
  motion: boolean,                // Motion detected status
  ecg: array<number>,            // ECG waveform data array (0-4095 range)
  
  // Optional additional fields
  status?: string,                // "normal" | "elevated" | "irregular"
  notes?: string,                 // Additional notes or annotations
}
```

**Alternative Structure (Direct Document):**

If you prefer a simpler structure without subcollections, you can use:

**Collection Path:** `ecg_data/{userId}`

**Document Structure:**
```javascript
{
  latest: {                       // Latest reading
    timestamp: timestamp,
    heartRate: number,
    signalQuality: number,
    leadsOff: boolean,
    motion: boolean,
    ecg: array<number>,
  },
  history: array<{                // Historical readings (last 100 or so)
    timestamp: timestamp,
    heartRate: number,
    signalQuality: number,
    leadsOff: boolean,
    motion: boolean,
    ecg: array<number>,
  }>,
  updatedAt: timestamp,           // Last update timestamp
}
```

**Example Reading Document:**
```javascript
{
  timestamp: Timestamp,
  heartRate: 72,
  signalQuality: 95,
  leadsOff: false,
  motion: false,
  ecg: [2048, 2050, 2055, 2060, 2065, ...],  // Array of ECG values
  status: "normal"
}
```

---

## Firestore Security Rules

To secure your database, use these Firestore security rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Users collection - users can read/write their own data
    match /users/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    
    // ECG data - patients can read/write their own data, doctors can read patient data
    match /ecg_data/{userId} {
      allow read: if request.auth != null && 
        (request.auth.uid == userId || 
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'doctor');
      allow write: if request.auth != null && request.auth.uid == userId;
      
      // Readings subcollection
      match /readings/{readingId} {
        allow read: if request.auth != null && 
          (request.auth.uid == userId || 
           get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'doctor');
        allow write: if request.auth != null && request.auth.uid == userId;
      }
    }
  }
}
```

---

## Indexes Required

For querying ECG data efficiently, you may need to create these composite indexes:

1. **For ECG readings queries:**
   - Collection: `ecg_data/{userId}/readings`
   - Fields: `timestamp` (descending)
   - Query type: Order by timestamp descending

**How to create indexes:**
1. Go to Firebase Console → Firestore → Indexes
2. Click "Create Index"
3. Enter the collection path and fields
4. Or let Firebase create them automatically when you run a query (follow the link in the error message)

---

## Data Insertion Examples

### Creating a User Document (Patient)

```javascript
import { doc, setDoc, serverTimestamp } from "firebase/firestore";

await setDoc(doc(db, "users", user.uid), {
  uid: user.uid,
  fullName: "John Doe",
  email: user.email,
  contact: "+1234567890",
  role: "patient",
  createdAt: serverTimestamp(),
  verified: false,
  dob: "1990-01-15",
  emergencyContact: "Jane Doe +1234567891"
});
```

### Creating an ECG Reading (Subcollection Structure)

```javascript
import { collection, addDoc, serverTimestamp } from "firebase/firestore";

await addDoc(collection(db, "ecg_data", userId, "readings"), {
  timestamp: serverTimestamp(),
  heartRate: 72,
  signalQuality: 95,
  leadsOff: false,
  motion: false,
  ecg: [2048, 2050, 2055, 2060, ...], // Array of values
  status: "normal"
});
```

### Updating Latest ECG Reading (Direct Document Structure)

```javascript
import { doc, updateDoc, serverTimestamp, arrayUnion } from "firebase/firestore";

const ecgRef = doc(db, "ecg_data", userId);
await updateDoc(ecgRef, {
  latest: {
    timestamp: serverTimestamp(),
    heartRate: 72,
    signalQuality: 95,
    leadsOff: false,
    motion: false,
    ecg: [2048, 2050, ...]
  },
  updatedAt: serverTimestamp()
});

// Add to history (limit to last 100 readings)
await updateDoc(ecgRef, {
  history: arrayUnion({
    timestamp: serverTimestamp(),
    heartRate: 72,
    // ... other fields
  })
});
```

---

## Recommendations

1. **Use Subcollection Structure** for ECG data if you expect many readings per patient (>1000). This is more scalable.

2. **Use Direct Document Structure** if you have fewer readings and want simpler queries.

3. **Implement Data Cleanup** - Consider deleting old ECG readings after a certain period (e.g., older than 1 year) to manage storage costs.

4. **Batch Operations** - When inserting multiple readings, use batch writes for better performance.

5. **Pagination** - For ECG history, implement pagination to limit the number of documents read per query.

---

## Next Steps

1. Create the `users` collection structure in Firestore
2. Decide on ECG data structure (subcollection vs direct document)
3. Set up Firestore security rules
4. Create necessary indexes
5. Test data insertion and retrieval
6. Implement data cleanup policies if needed
