# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build and Run
```bash
# Build the project
./gradlew build

# Install debug APK to device
./gradlew installDebug

# Run tests
./gradlew test

# Clean build
./gradlew clean

# Check for dependency updates
./gradlew dependencyUpdates
```

### Key Configuration Files
- `app/build.gradle.kts`: Main app build configuration with dependencies
- `build.gradle.kts`: Root project configuration  
- `gradle/libs.versions.toml`: Version catalog for dependency management
- `gradle.properties`: Gradle build properties

## Architecture Overview

### Core Technologies
- **Android SDK**: Target 35 (Android 14), Min 28 (Android 9)
- **Jetpack Compose**: Modern UI framework with Material3 design
- **Kotlin**: Primary language with coroutines for async operations
- **Room Database**: Local persistence with KSP for annotation processing
- **YOLOv11**: Custom traffic sign detection via PyTorch Mobile
- **Google Maps/Places**: Route planning and location services
- **Firebase**: Authentication and messaging services

### Application Structure

#### Detection Pipeline (`YoloV11Detector.kt`)
- Loads `best.torchscript` model from assets
- Processes 640x640 input images with configurable confidence thresholds
- Applies non-maximum suppression and quality filtering
- Configuration centralized in `DetectorConfig.kt` (confidence: 0.45, max detections: 5)

#### Database Layer (`data/`)
- **AppDatabase**: Room database with singleton pattern via `TrafficSignApp`
- **DetectedSignEntity**: Stores detections with location data and timestamps
- **Route/RouteDAO**: Route planning with detection cross-references
- **UserDao**: User authentication and profile data

#### UI Architecture (`ui/theme/`)
- **MainScreen**: Tab-based navigation (Detection, History, Route Planning)
- **StartDetectionScreen**: Real-time camera preview with YOLO processing
- **DetectionHistoryScreen**: Searchable list of saved detections
- **RoutePlannerScreen**: Google Maps integration with Places API
- **ViewModels**: State management with LiveData/StateFlow

#### Core Services
- **CameraPreview/FrameAnalyzer**: Camera2 API with YUV→RGB conversion
- **DetectionStateManager**: Manages detection persistence and temporal consistency
- **AppFirebaseMessagingService**: Push notification handling

### Key Business Logic

#### Detection Quality Control
- Minimum confidence: 0.45 (adjustable in `DetectorConfig`)
- Consecutive frame validation: 2+ detections before saving
- Geometric filtering: aspect ratio and minimum size validation
- Location tracking: GPS coordinates saved with each detection

#### Route Planning Integration
- Google Places API for destination search
- Directions API for route calculation
- Option to avoid routes with specific sign types
- Current location centering with fallback handling

### Google Services Setup Requirements

#### API Keys Required
1. **Google Maps API Key**: For map display and route planning
2. **Places API Key**: For location search functionality
3. **Directions API Key**: For route calculation

#### Configuration Files
- `AndroidManifest.xml`: Contains Google Maps and Places API key metadata
- `app/src/main/res/values/strings.xml`: Maps API key string resource
- `google-services.json`: Firebase configuration (place in `app/` directory)

#### SHA-1 Fingerprint
Required for API key restrictions. Generate with:
```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

### Development Workflow

#### Model and Assets
- YOLOv11 model: `app/src/main/assets/best.torchscript` (69 traffic sign classes)
- Label mapping: `app/src/main/assets/labels.txt`
- Test images: Various traffic sign samples in `assets/` for validation

#### Debug and Testing
- Debug mode: Enable via `YoloV11Detector.enableDebugMode()` for lower confidence threshold
- Self-test: Validates model loading and inference with sample images
- Performance metrics: Inference timing and detection statistics available

#### Location Services
- Requires `ACCESS_FINE_LOCATION` and `ACCESS_COARSE_LOCATION` permissions
- Location updates every 5 seconds when detection is active
- Minimum accuracy requirement: 20 meters for reliable positioning

### Performance Considerations
- Frame rate limited to 8 FPS for stability
- Top-K pruning (300) before NMS to reduce computation
- Bitmap reuse and memory management for camera frames
- Background processing threads (2) for model inference
- Automatic garbage collection every 100 frames

### Dependencies Management
Uses Gradle version catalog (`gradle/libs.versions.toml`) for consistent versioning. Key dependencies include:
- PyTorch Mobile 1.13.1 for YOLO inference
- CameraX 1.4.2 for camera operations  
- Room 2.6.1 with KSP for database
- Compose BOM 2024.09.00 for UI consistency
- Google Play Services (Maps, Location, Places)