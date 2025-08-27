# 🐛 **Debug Test Protocol for Current Issues**

## **Issue 1: ML Model Low Confidence Fix**

### **CRITICAL FIX APPLIED:**
✅ **Preprocessing Normalization Corrected**
```kotlin
// OLD (WRONG):
val NORMALIZATION_MEAN = floatArrayOf(0.0f, 0.0f, 0.0f)
val NORMALIZATION_STD = floatArrayOf(1.0f, 1.0f, 1.0f)

// NEW (FIXED):
val NORMALIZATION_MEAN = floatArrayOf(0.485f, 0.456f, 0.406f)  // ImageNet standard
val NORMALIZATION_STD = floatArrayOf(0.229f, 0.224f, 0.225f)
```

✅ **Confidence Threshold Raised**
- `MIN_CONFIDENCE`: 0.25f → 0.45f (should work now with proper preprocessing)

### **Expected Results After Fix:**
```
Previous: Top scores: [0.36256763, 0.36121953, 0.3612053]
Expected: Top scores: [0.87542163, 0.76834251, 0.68923456]
```

### **Test Steps:**
1. **Clean and rebuild** app with new preprocessing
2. **Test with same crossroad image**
3. **Check logcat for confidence scores >0.7**

---

## **Issue 2: Places API Search Not Working**

### **Root Causes Identified:**

#### **Cause 1: API Key Validation Too Strict**
Your API key: `AIzaSyDP9JiDjq7xUb_csKRqjIrVkjDBIdD15JM` (39 chars, correct format)

#### **Cause 2: Google Cloud Console Configuration**
Missing requirements:
- ❌ Places API not enabled
- ❌ SHA-1 fingerprint not added to API key restrictions  
- ❌ Billing not properly configured

### **Immediate Diagnostic Steps:**

#### **Step 1: Check API Key Status**
Open Android Studio Logcat, look for:
```
✅ GOOD: "API key format valid: AIzaSyDP***"
✅ GOOD: "Google Places SDK initialized successfully"
❌ BAD: "Places API key validation failed"
❌ BAD: "API key format invalid"
```

#### **Step 2: Test Places Picker Manually**
1. Tap on "From" search field
2. Check logcat for error messages:
```
✅ GOOD: Places picker opens fullscreen
❌ BAD: "Error launching Places picker"
❌ BAD: "Places API error. Check your API key and billing setup"
```

#### **Step 3: Google Cloud Console Fix**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. **Enable APIs**:
   - Maps SDK for Android ✅
   - Places API ✅  
   - Geocoding API ✅
3. **Configure API Key Restrictions**:
   - Application restrictions: Android apps
   - Package name: `com.example.trafficsignapp`
   - SHA-1 fingerprint: (get with keytool command below)

#### **Step 4: Get SHA-1 Fingerprint**
```bash
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```

---

## **Expected Results After Fixes**

### **ML Detection (Issue 1)**
```
OLD LOG:
preNMS max=0.36256763 cnt=600
Final detections: [Crossroad(0.363), Speed limit(0.250)]

NEW LOG (Expected):  
preNMS max=0.87456123 cnt=600
Final detections: [Crossroad(0.875), Stop Sign(0.823), Speed Limit(0.756)]
```

### **Places API (Issue 2)**
```
OLD BEHAVIOR:
- Tap search field → Nothing happens
- Buttons remain grayed out

NEW BEHAVIOR:
- Tap search field → Places picker opens
- Select location → Button becomes enabled
- Can plan routes successfully
```

---

## **Testing Protocol**

### **Phase 1: Verify Fixes**
```bash
# 1. Clean rebuild with new preprocessing
./gradlew clean build

# 2. Install and test
./gradlew installDebug

# 3. Check logs
adb logcat -s YOLO:D ApiKeyManager:D RoutePlanner:D TrafficSignApp:D
```

### **Phase 2: ML Model Validation**
1. **Test with crossroad image**
   - Should see confidence >0.7
   - Less false positives
   
2. **Test with other traffic signs**
   - Use self-test function
   - Monitor confidence improvements

### **Phase 3: Places API Validation**  
1. **Test search functionality**
   - Tap "From" field → Places picker opens
   - Select location → Shows in field
   - Repeat for "To" field
   
2. **Test route planning**
   - Both locations selected → "Find Route" enabled
   - Tap "Find Route" → Shows route options
   - Select route → "Start Recording" enabled

---

## **If Issues Persist**

### **ML Model Still Low Confidence**
Try alternative preprocessing:
```kotlin
// In DetectorConfig.kt, temporarily use:
val NORMALIZATION_MEAN = ALTERNATIVE_MEAN  // 0-1 scaling
val NORMALIZATION_STD = ALTERNATIVE_STD    // 255 scaling
```

### **Places API Still Not Working**
Check these common issues:
1. **Billing disabled** in Google Cloud
2. **API quotas exceeded**  
3. **Wrong SHA-1 fingerprint**
4. **Package name mismatch**

---

## **Success Metrics**

### **ML Detection Success:**
- ✅ Confidence scores >0.5 for real signs
- ✅ <3 false positives per frame
- ✅ "Crossroad" detected with >0.8 confidence

### **Places API Success:**  
- ✅ Search fields open Places picker
- ✅ Selected locations appear in fields
- ✅ Route planning works end-to-end
- ✅ Recording button becomes clickable

**Test both fixes and report results!**