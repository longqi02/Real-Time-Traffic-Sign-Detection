# Places API Debug Guide

## Quick Diagnostic Steps

### 1. Generate SHA-1 Fingerprint
```bash
# Windows (PowerShell)
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android

# Copy the SHA-1 fingerprint (40 characters, format: XX:XX:XX:XX...)
```

### 2. Google Cloud Console Checklist
- Go to [Google Cloud Console](https://console.cloud.google.com/)
- **Enable APIs**: 
  ✅ Maps SDK for Android
  ✅ Places API 
  ✅ Geocoding API
  ✅ Directions API

### 3. API Key Configuration
1. Go to "Credentials" → Click your API key
2. **Application restrictions**: Select "Android apps"
3. **Package name**: `com.example.trafficsignapp`
4. **SHA-1**: Add your fingerprint from step 1
5. **API restrictions**: Select specific APIs (Maps, Places, Geocoding, Directions)

### 4. Billing Verification
- Ensure billing account is linked and active
- Check for any usage limits or quotas

## Testing Commands
```bash
# Test Places API directly
curl -X GET "https://maps.googleapis.com/maps/api/place/autocomplete/json?input=restaurant&key=YOUR_API_KEY"

# Should return JSON with predictions array
```

## Current Configuration
- API Key: `AIzaSyB8rqEkKgFA1aMbIqUBtJj8Wk4E605RjHQ`
- Package: `com.example.trafficsignapp`
- SHA-1: **NEEDS TO BE ADDED TO RESTRICTIONS**