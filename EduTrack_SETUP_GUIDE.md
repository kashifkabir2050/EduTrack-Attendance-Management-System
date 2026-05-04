# EduTrack — Attendance Management System
## Setup Guide

---

## 1. Firebase Project Setup

### Create Firebase Project
1. Go to https://console.firebase.google.com
2. Click **Add project** → Enter name (e.g. "EduTrack School")
3. Enable Google Analytics (optional) → Create project

### Enable Authentication
1. Firebase Console → **Authentication** → **Get started**
2. Enable **Email/Password** provider

### Enable Firestore Database
1. Firebase Console → **Firestore Database** → **Create database**
2. Start in **production mode**
3. Choose a region close to you

### Enable Storage
1. Firebase Console → **Storage** → **Get started**
2. Accept default rules for now

---

## 2. Get Your Firebase Config

1. Firebase Console → Project Settings (gear icon) → **General**
2. Scroll to **Your apps** → Click **</>** (Web app)
3. Register app name → Copy the `firebaseConfig` object

Open `index.html`, find this section:
```javascript
const FIREBASE_CONFIG = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```
Replace with your actual values.

---

## 3. Create First Admin User

1. Firebase Console → **Authentication** → **Users** → **Add user**
2. Enter email and password for your admin account
3. Copy the **User UID** shown after creation

4. Firebase Console → **Firestore** → **Start collection**
   - Collection ID: `users`
   - Document ID: (paste the UID from step 3)
   - Fields:
     - `name` (string): "Admin Name"
     - `email` (string): admin@yourschool.com
     - `role` (string): `admin`

---

## 4. Firestore Security Rules

Go to Firestore → **Rules** and paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Users can read their own profile
    match /users/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow read, write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Students — authenticated users can read; admins/teachers can write
    match /students/{doc} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
    
    // Attendance — authenticated users
    match /attendance/{date}/records/{doc} {
      allow read, write: if request.auth != null;
    }
  }
}
```

---

## 5. Storage Rules

Firebase Console → Storage → **Rules**:
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /students/{allPaths=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.resource.size < 5 * 1024 * 1024;
    }
  }
}
```

---

## 6. Deploy the App

**Option A: Firebase Hosting (Recommended)**
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Choose your project, set public dir to current folder
firebase deploy
```

**Option B: Any Static Host**
- Upload `index.html` to Netlify, Vercel, GitHub Pages, etc.

**Option C: Local Testing**
```bash
npx serve .
# or
python -m http.server 8080
```

---

## 7. Application Features

| Feature | Description |
|---------|-------------|
| 🔐 Login | Email/password auth via Firebase |
| 🎓 Students | Add/Edit/Delete with photo upload |
| ✅ Attendance | P (Present), A (Absent), AA (Auth Absent) |
| 📊 Dashboard | Live stats + weekly & class charts |
| 📋 Daily Report | Date + class filter; P+AA = Present |
| 📅 Monthly Report | Per-student monthly summary with chart |
| ⚙️ Admin Panel | Create teacher/staff/admin accounts |

---

## 8. Mobile App Integration (Face Recognition)

For the mobile attendance via photo matching:

1. **Use Firebase ML Kit** (Android/iOS) for on-device face recognition
2. Store student reference photos in Firebase Storage
3. On match → call Firestore to write attendance record:
```javascript
// Mobile app writes to same Firestore path
await db.collection('attendance')
  .doc(dateString)
  .collection('records')
  .doc(studentId)
  .set({ status: 'P', method: 'face', ... });
```
4. The web app will reflect this attendance immediately

---

## Grades Available
KG1, KG2, Grade 1–12

## Sections Available
A, B, C, D, E, F, AB, BB, CB, DB, AG, BG, CG, DG

---

*EduTrack — Built with Firebase + Vanilla JS*
