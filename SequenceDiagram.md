# Sequence Diagram: Workout Recording with Feedback

## Overview
This sequence diagram illustrates the flow of data and interactions between components during a workout recording session with real-time feedback.

## Actors and Components
- **User**: The end user of the iOS application
- **SwiftUI App (iOS)**: The client application built with SwiftUI
- **Azure API Gateway**: Centralized API management layer
- **Azure App Service**: Hosts the backend workout service
- **AWS AI Service**: Lambda functions and SageMaker for AI analysis
- **PostgreSQL**: Database for persistent storage

## Sequence Flow

```mermaid
sequenceDiagram
    participant User
    participant iOS as SwiftUI App (iOS)
    participant Gateway as Azure API Gateway
    participant Backend as Azure App Service
    participant AI as AWS AI Service
    participant DB as PostgreSQL

    Note over User, DB: Workout Recording Session
    
    User->>iOS: Starts workout, enables camera
    iOS->>iOS: Initialize AVFoundation, CoreMotion
    iOS->>Gateway: Request auth token with tenant context
    Gateway->>Backend: Validate credentials
    Backend->>Gateway: Return JWT with tenant claims
    Gateway->>iOS: Auth token with tenant context
    
    iOS->>iOS: Start camera preview
    User->>iOS: Begin exercise
    
    Note over iOS, AI: Real-time Analysis Loop
    
    loop Every 500ms during workout
        iOS->>iOS: Capture video frame & sensor data
        iOS->>Gateway: Stream data (POST /workouts/streaming)
        Note right of Gateway: Include X-Tenant-ID header
        Gateway->>Backend: Forward request with tenant context
        Backend->>AI: Send frame for analysis
        AI->>AI: Process frame (pose estimation)
        AI->>Backend: Return feedback (reps, form analysis)
        Backend->>Gateway: Return real-time feedback
        Gateway->>iOS: Feedback response
        iOS->>iOS: Update UI with feedback
        iOS->>User: Display feedback overlay
    end
    
    User->>iOS: Complete workout
    iOS->>Gateway: Submit final workout data
    Gateway->>Backend: Process complete workout
    Backend->>DB: Insert workout record with tenant_id
    Backend->>DB: Store feedback details
    
    opt HealthKit Enabled
        Backend->>iOS: Return workout with HealthKit metadata
        iOS->>iOS: Sync workout to HealthKit
        iOS->>Backend: Confirm HealthKit sync status
        Backend->>DB: Update HealthKit sync status
    end
    
    Backend->>Gateway: Return workout summary
    Gateway->>iOS: Workout summary response
    iOS->>User: Display workout results screen
```

## Key Implementation Notes

1. **Tenant Isolation**:
   - All requests include tenant context via the X-Tenant-ID header
   - Backend services enforce tenant isolation for all data operations
   - Database queries filter by tenant_id

2. **Real-time Processing**:
   - Video frames are streamed at regular intervals (optimized for performance)
   - Initial processing occurs on device for immediate feedback
   - Cloud AI provides enhanced analysis with lower latency requirements

3. **HealthKit Integration**:
   - Workout data is mapped to HealthKit workout types
   - Synchronization occurs after workout completion
   - Sync status is tracked for troubleshooting

4. **Error Handling** (not shown):
   - Network disruptions handled with local buffering
   - AI service failures gracefully degrade to basic functionality
   - All errors logged with correlation IDs for troubleshooting
