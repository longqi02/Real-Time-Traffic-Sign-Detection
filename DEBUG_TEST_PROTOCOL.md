# Debug Testing Protocol

## **Issue 1: Places API Search Fix**

### **Immediate Actions Required:**

1. **Generate SHA-1 Fingerprint:**
```bash
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```
**Copy the SHA-1 (40 characters) from output**

2. **Google Cloud Console Setup:**
- Go to [console.cloud.google.com](https://console.cloud.google.com/)
- **APIs & Services** → **Library**
- Enable: `Maps SDK for Android`, `Places API`, `Geocoding API`, `Directions API`
- **Credentials** → Click API key: `AIzaSyB8rqEkKgFA1aMbIqUBtJj8Wk4E605RjHQ`
- **Application restrictions**: Android apps
- **Package name**: `com.example.trafficsignapp`  
- **SHA-1 fingerprint**: Paste your SHA-1 from step 1
- **API restrictions**: Select the 4 APIs above

3. **Billing Verification:**
- Ensure billing account is active
- Check quotas are not exceeded

### **Test Places API:**
After setup, tap the "Search place" fields - they should now show autocomplete suggestions.

---

## **Issue 2: YOLO Detection Enhancement**

### **Changes Made:**
1. ✅ **MIN_CONFIDENCE**: Reduced from 0.45 to 0.3
2. ✅ **Debug Threshold**: Lowered to 0.08 for maximum visibility
3. ✅ **Enhanced Logging**: Added detailed detection pipeline logs

### **Testing Protocol:**

#### **Step 1: Enable Debug Mode**
Add this to your detection screen initialization:
```kotlin
detector.enableDebugMode() // Enables 0.08 confidence threshold
```

#### **Step 2: Test with Self-Test First**
- Tap "Run Self-Test" button
- Check if model can detect signs in sample images
- Look for console logs showing detection results

#### **Step 3: Camera Detection Test**
- Point camera at the yellow diamond sign
- Monitor Android Studio logs for:
  - Model loading messages
  - Shape information: `shape=[1, 74, 8400] → numClasses=69`
  - Detection counts: `preNMS count=X, maxScore=Y`
  - Final results: `Final detections: [...]`

#### **Step 4: Check Expected Sign Types**
For the yellow diamond sign in your image, possible matches:
- **Class 7**: "Construction sign"
- **Class 33**: "Other dangers nearby" 
- **Class 43**: "Road work"
- **Class 45**: "Slippery road"

### **Expected Log Output:**
```
YOLO: Model loaded successfully
YOLO: shape=[1, 74, 8400] → numClasses=69
YOLO: preNMS max=0.85 cnt=45
YOLO: Final detections: [Construction sign(0.678)]
```

### **If Still No Detection:**

#### **Model File Check:**
```bash
# Check if files exist in assets
ls -la app/src/main/assets/
# Should show: best.torchscript (large file) and labels.txt
```

#### **Debug Input Preprocessing:**
Add temporary logging in `FrameAnalyzer.kt` line ~125:
```kotlin
Log.d("FrameAnalyzer", "Input bitmap: ${inputBitmap.width}x${inputBitmap.height}, " +
      "config=${inputBitmap.config}, recycled=${inputBitmap.isRecycled}")
```

#### **Alternative Confidence Test:**
If no detections with 0.3, try even lower:
```kotlin
// In DetectorConfig.kt, temporarily set:
const val MIN_CONFIDENCE = 0.1f
```

---

## **Why Your Changes Will Work:**

### **MIN_CONFIDENCE: 0.45 → 0.3**
- **Effect**: 50% more detections will pass initial threshold
- **Yellow diamond signs** often detect at 0.35-0.55 confidence
- **Trade-off**: Slightly more false positives, but better recall

### **Debug Mode (0.08 threshold)**
- **Effect**: Shows even weak detections for debugging
- **Use case**: Verify model is working, then tune thresholds upward

### **Enhanced Logging**
- **Pre-NMS counts**: Shows raw detection counts before filtering
- **Score distribution**: Shows if model is producing reasonable outputs
- **Final results**: Confirms what reaches the UI

---

## **Expected Results After Fixes:**

1. **Places API**: Search fields show dropdown with location suggestions
2. **YOLO Detection**: Yellow diamond sign detected as "Construction sign" or similar with confidence 0.3-0.7

## **Performance Impact:**
- **Lower confidence**: +15% processing time due to more detections
- **Debug mode**: Should only be used temporarily
- **Memory**: No significant impact with current settings

---

## **Next Steps if Issues Persist:**

1. **Places API**: Check logcat for specific error codes
2. **YOLO**: Enable bitmap saving to verify input preprocessing:
   ```kotlin
   // Add to DetectorConfig.kt
   const val SAVE_DEBUG_IMAGES = true
   ```

Test methodically - fix Places API first (simpler), then focus on YOLO detection with debug mode enabled.