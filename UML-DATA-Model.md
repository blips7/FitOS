# FitOS: Detailed UML Data Model

## Entity Definitions

### Tenant
- **Attributes**:
  - `tenantId`: UUID (PK)
  - `name`: VARCHAR
  - `domain`: VARCHAR
  - `createdAt`: TIMESTAMP
  - `isActive`: BOOLEAN
  - `tier`: VARCHAR
  - `healthKitConfig`: JSONB
- **Relationships**: 
  - 1 Tenant → Many TenantConfig
  - 1 Tenant → Many User
  - 1 Tenant → Many ApiKey

### TenantConfig
- **Attributes**:
  - `configId`: UUID (PK)
  - `tenantId`: UUID (FK)
  - `category`: VARCHAR
  - `settings`: JSONB
  - `healthKitMappings`: JSONB
- **Relationships**: 
  - Many TenantConfig → 1 Tenant

### ApiKey
- **Attributes**:
  - `keyId`: UUID (PK)
  - `tenantId`: UUID (FK)
  - `keyValue`: VARCHAR
  - `expiresAt`: TIMESTAMP
  - `permissions`: VARCHAR[]
- **Relationships**: 
  - Many ApiKey → 1 Tenant

### User
- **Attributes**:
  - `userId`: UUID (PK)
  - `tenantId`: UUID (FK)
  - `username`: VARCHAR
  - `email`: VARCHAR UNIQUE
  - `passwordHash`: VARCHAR
  - `height`: REAL (HKQuantityTypeIdentifierHeight)
  - `weight`: REAL (HKQuantityTypeIdentifierBodyMass)
  - `goal`: ENUM('Lose Weight', 'Build Muscle', 'Maintain')
  - `roles`: VARCHAR[]
  - `healthKitEnabled`: BOOLEAN
  - `lastHealthKitSync`: TIMESTAMP
- **Relationships**: 
  - Many User → 1 Tenant
  - 1 User → Many Workout
  - 1 User → Many Meal
  - 1 User → Many Diet
  - 1 User → Many HealthKitPermission
  - 1 User → Many BodyMeasurement
  - 1 User → Many HealthKitSyncLog

### HealthKitPermission
- **Attributes**:
  - `permissionId`: UUID (PK)
  - `userId`: UUID (FK)
  - `healthKitTypeId`: VARCHAR
  - `read`: BOOLEAN
  - `write`: BOOLEAN
  - `lastUpdated`: TIMESTAMP
- **Relationships**: 
  - Many HealthKitPermission → 1 User

### Workout
- **Attributes**:
  - `workoutId`: UUID (PK)
  - `userId`: UUID (FK)
  - `tenantId`: UUID (FK)
  - `startDate`: TIMESTAMP
  - `endDate`: TIMESTAMP
  - `duration`: INTEGER (HKQuantityTypeIdentifierAppleExerciseTime)
  - `healthKitWorkoutType`: VARCHAR (HKWorkoutTypeIdentifier)
  - `activeEnergyBurned`: REAL (HKQuantityTypeIdentifierActiveEnergyBurned)
  - `reps`: INTEGER
  - `feedback`: TEXT
  - `videoUrl`: VARCHAR (stored in Azure Blob)
  - `healthKitMetadata`: JSONB
  - `healthKitUUID`: VARCHAR
  - `syncedToHealthKit`: BOOLEAN
  - `syncTimestamp`: TIMESTAMP
- **Relationships**: 
  - Many Workout → 1 User
  - Many Workout → 1 Tenant
  - 1 Workout → Many WorkoutEvent

### WorkoutEvent
- **Attributes**:
  - `eventId`: UUID (PK)
  - `workoutId`: UUID (FK)
  - `eventType`: VARCHAR
  - `timestamp`: TIMESTAMP
  - `data`: JSONB
  - `healthKitData`: JSONB
- **Relationships**: 
  - Many WorkoutEvent → 1 Workout

### Meal
- **Attributes**:
  - `mealId`: UUID (PK)
  - `userId`: UUID (FK)
  - `tenantId`: UUID (FK)
  - `date`: TIMESTAMP
  - `imageUrl`: VARCHAR (stored in Azure Blob)
  - `calories`: REAL (HKQuantityTypeIdentifierDietaryEnergyConsumed)
  - `nutrients`: JSONB
  - `healthKitCorrelation`: JSONB
  - `healthKitUUID`: VARCHAR
  - `syncedToHealthKit`: BOOLEAN
  - `syncTimestamp`: TIMESTAMP
- **Relationships**: 
  - Many Meal → 1 User
  - Many Meal → 1 Tenant
  - 1 Meal → Many MealItem

### MealItem
- **Attributes**:
  - `itemId`: UUID (PK)
  - `mealId`: UUID (FK)
  - `foodName`: VARCHAR
  - `quantity`: REAL
  - `unit`: VARCHAR
  - `calories`: REAL
  - `nutrients`: JSONB
  - `healthKitFoodType`: VARCHAR
- **Relationships**: 
  - Many MealItem → 1 Meal

### Diet
- **Attributes**:
  - `dietId`: UUID (PK)
  - `userId`: UUID (FK)
  - `tenantId`: UUID (FK)
  - `startDate`: DATE
  - `calorieGoal`: REAL (HKQuantityTypeIdentifierDietaryEnergyConsumed)
  - `nutritionPlan`: JSONB
  - `metadata`: JSONB
- **Relationships**: 
  - Many Diet → 1 User
  - Many Diet → 1 Tenant

### BodyMeasurement
- **Attributes**:
  - `measurementId`: UUID (PK)
  - `userId`: UUID (FK)
  - `date`: TIMESTAMP
  - `type`: VARCHAR (HKQuantityTypeIdentifier)
  - `value`: REAL
  - `unit`: VARCHAR
  - `healthKitMetadata`: JSONB
  - `healthKitUUID`: VARCHAR
  - `syncedToHealthKit`: BOOLEAN
  - `syncTimestamp`: TIMESTAMP
- **Relationships**: 
  - Many BodyMeasurement → 1 User

### TrainingContent
- **Attributes**:
  - `contentId`: UUID (PK)
  - `tenantId`: UUID (FK)
  - `title`: VARCHAR
  - `url`: VARCHAR (stored in Azure Blob)
  - `category`: ENUM('Cardio', 'Strength', 'Flexibility')
  - `metadata`: JSONB
  - `accessTiers`: VARCHAR[]
  - `healthKitWorkoutTypes`: JSONB
- **Relationships**: 
  - Many TrainingContent → 1 Tenant
  - 1 TrainingContent → Many TenantContent

### TenantContent
- **Attributes**:
  - `id`: UUID (PK)
  - `tenantId`: UUID (FK)
  - `contentId`: UUID (FK)
  - `isCustomized`: BOOLEAN
  - `settings`: JSONB
- **Relationships**: 
  - Many TenantContent → 1 TrainingContent
  - Many TenantContent → 1 Tenant

### HealthKitSyncLog
- **Attributes**:
  - `logId`: UUID (PK)
  - `userId`: UUID (FK)
  - `syncDate`: TIMESTAMP
  - `operation`: VARCHAR
  - `objectType`: VARCHAR
  - `objectId`: VARCHAR
  - `status`: VARCHAR
  - `errorMessage`: VARCHAR
- **Relationships**: 
  - Many HealthKitSyncLog → 1 User

## Entity Relationship Diagram

```
+-----------+      +---------------+      +----------+
| Tenant    |1    *| TenantConfig  |      | ApiKey   |
+-----------+<-----+---------------+      +----------+
| tenantId  |      | configId      |      | keyId    |
| name      |      | tenantId      |      | tenantId |
| domain    |      | category      |      | keyValue |
| createdAt |      | settings      |      | expiresAt|
| isActive  |      | healthKitMap  |      | perms    |
| tier      |      +---------------+      +----------+
| healthKit |                                  ^
+-----------+                                  |
     ^1                                        |1
     |                                         |
     |*                                        |*
+----------+     +-------------------+    +---------+
| User     |1   *| HealthKitPermission |  | Workout |
+----------+<----+-------------------+   +---------+
| userId   |     | permissionId      |   | workoutId|
| tenantId |     | userId            |   | userId   |
| username |     | healthKitTypeId   |   | tenantId |
| email    |     | read              |   | startDate|
| password |     | write             |   | endDate  |
| height   |     | lastUpdated       |   | duration |
| weight   |     +-------------------+   | type     |
| goal     |                             | feedback |
| roles    |1                           *| videoUrl |
| healthKit|<--------------------------->+---------+
+----------+                                  |1
     |1                                       |
     |                                        |*
     v*                                  +-----------+
 +-------+                              | WorkoutEvent|
 | Meal  |                              +-----------+
 +-------+                              | eventId   |
 | mealId|                              | workoutId |
 | userId|                              | eventType |
 |tenantId|                             | timestamp |
 | date  |                              | data      |
 |imageUrl|                             +-----------+
 |calories|
 |nutrients|1
 |healthKit|<--------------------+
 +-------+                       |
     |1                          |*
     |                     +----------+
     v*                    | MealItem  |
 +-------+                 +----------+
 | Diet  |                 | itemId    |
 +-------+                 | mealId    |
 | dietId|                 | foodName  |
 | userId|                 | quantity  |
 |tenantId|                | unit      |
 |startDate|               | calories  |
 |calorieGoal|             | nutrients |
 |nutritionPlan|           +----------+
 +-------+

 +-----------------+      +----------------+
 | BodyMeasurement |      | HealthKitSyncLog |
 +-----------------+      +----------------+
 | measurementId   |      | logId          |
 | userId          |      | userId         |
 | date            |      | syncDate       |
 | type            |      | operation      |
 | value           |      | objectType     |
 | unit            |      | objectId       |
 | healthKitMetadata|     | status         |
 | healthKitUUID   |      | errorMessage   |
 | syncedToHealthKit|     +----------------+
 | syncTimestamp   |
 +-----------------+

 +-----------------+      +---------------+
 | TrainingContent |1    *| TenantContent |
 +-----------------+<-----+---------------+
 | contentId       |      | id            |
 | tenantId        |      | tenantId      |
 | title           |      | contentId     |
 | url             |      | isCustomized  |
 | category        |      | settings      |
 | metadata        |      +---------------+
 | accessTiers     |
 | healthKitWorkouts|
 +-----------------+
```
