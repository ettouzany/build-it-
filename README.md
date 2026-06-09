For a Unity implementation, I would build it as a data-driven architecture so new models can be added without releasing a new app.

High-Level Architecture
Client (Unity)
│
├── Authentication
├── Catalog System
├── Build System
├── Progression System
├── Inventory System
├── Collection Room
├── Monetization
├── Analytics
└── Cloud Save
        │
        ▼
Firebase
├── Authentication
├── Firestore
├── Cloud Storage
├── Remote Config
├── Analytics
├── Cloud Functions
└── Crashlytics
Unity Folder Structure
Assets
│
├── _Project
│   ├── Art
│   │   ├── Models
│   │   ├── Materials
│   │   ├── Textures
│   │   └── UI
│   │
│   ├── Audio
│   │   ├── Music
│   │   ├── SFX
│   │   └── ASMR
│   │
│   ├── Prefabs
│   │   ├── Bricks
│   │   ├── UI
│   │   ├── Models
│   │   └── Collection
│   │
│   ├── Scenes
│   │   ├── Boot
│   │   ├── MainMenu
│   │   ├── BuildScene
│   │   ├── CollectionRoom
│   │   └── Loading
│   │
│   ├── Addressables
│   │
│   ├── Data
│   │   ├── ScriptableObjects
│   │   └── LocalConfigs
│   │
│   └── Scripts
│
├── ThirdParty
└── Packages
Scripts Structure
Scripts
│
├── Core
│   ├── GameManager.cs
│   ├── SceneLoader.cs
│   ├── EventBus.cs
│   ├── SaveManager.cs
│   └── Bootstrapper.cs
│
├── Authentication
│   ├── AuthManager.cs
│   ├── FirebaseAuthService.cs
│   └── GuestLogin.cs
│
├── Build
│   ├── BuildManager.cs
│   ├── BlueprintManager.cs
│   ├── BrickSpawner.cs
│   ├── BrickPlacementValidator.cs
│   ├── SnapSystem.cs
│   ├── StepController.cs
│   ├── ModelCompletionChecker.cs
│   └── BuildCameraController.cs
│
├── Inventory
│   ├── InventoryManager.cs
│   ├── BrickInventory.cs
│   └── RewardManager.cs
│
├── Collection
│   ├── CollectionRoomManager.cs
│   ├── DioramaSpawner.cs
│   └── DecorationManager.cs
│
├── Progression
│   ├── XPManager.cs
│   ├── UnlockManager.cs
│   ├── LevelManager.cs
│   └── AchievementManager.cs
│
├── Firebase
│   ├── FirestoreManager.cs
│   ├── StorageManager.cs
│   ├── RemoteConfigManager.cs
│   └── CloudFunctionsManager.cs
│
├── Monetization
│   ├── AdsManager.cs
│   ├── IAPManager.cs
│   └── BattlePassManager.cs
│
├── UI
│   ├── HomeScreenUI.cs
│   ├── BuildUI.cs
│   ├── RewardPopup.cs
│   ├── CollectionUI.cs
│   └── LoadingUI.cs
│
└── Analytics
    ├── AnalyticsManager.cs
    └── EventTracker.cs
Data Models
Brick Definition
[CreateAssetMenu]
public class BrickDefinition : ScriptableObject
{
    public string Id;
    public string Name;
    public GameObject Prefab;
    public Sprite Icon;
    public Vector3 Size;
}
Blueprint Model
[Serializable]
public class BlueprintData
{
    public string BlueprintId;
    public string Name;
    public string Category;
    public int Difficulty;
    public List<BuildStepData> Steps;
}
Build Step
[Serializable]
public class BuildStepData
{
    public int StepIndex;
    public string BrickId;

    public Vector3 Position;
    public Quaternion Rotation;

    public string HintText;
}
Collectible Model
[Serializable]
public class CollectibleModel
{
    public string ModelId;
    public string Name;
    public string Category;
    public string Thumbnail;
    public string BlueprintId;
    public int RewardCoins;
}
Build Flow
Load Blueprint
       │
       ▼
Spawn Step #1
       │
       ▼
Player Selects Brick
       │
       ▼
Snap Validation
       │
       ▼
Correct?
 ├─ No → Shake + ASMR Fail
 └─ Yes
       │
       ▼
Play Snap Sound
       │
       ▼
Advance Step
       │
       ▼
Model Complete?
 ├─ No
 └─ Yes
       │
       ▼
Reward Player
       │
       ▼
Unlock New Blueprint
Firestore Schema
users
users
{
  "uid": "123",
  "displayName": "Player",
  "coins": 1500,
  "gems": 200,
  "level": 12,
  "xp": 5400,
  "createdAt": "timestamp",
  "lastLogin": "timestamp"
}
user_progress
user_progress
{
  "uid": "123",
  "completedModels": [
      "car_001",
      "house_002"
  ],
  "currentBlueprint": "train_004",
  "currentStep": 15
}
user_inventory
user_inventory
{
  "uid": "123",
  "ownedModels": [
      "car_001",
      "house_002"
  ],
  "decorations": [
      "tree_01",
      "lamp_03"
  ]
}
blueprints
blueprints
{
  "id": "car_001",
  "name": "Sports Car",
  "difficulty": 2,
  "addressableUrl": "..."
}
models
models
{
  "id": "car_001",
  "thumbnail": "...",
  "rewardCoins": 150,
  "rewardXP": 25,
  "unlockLevel": 3
}
Cloud Storage Structure
storage
│
├── blueprints
│   ├── car_001.json
│   ├── house_001.json
│   └── train_001.json
│
├── thumbnails
│
├── addressables
│
├── audio
│
└── events
Addressables Strategy

Instead of shipping 500+ models inside the APK:

Client
    ↓
Catalog
    ↓
Download Selected Blueprint
    ↓
Download Required Prefabs
    ↓
Cache Locally

This allows weekly content updates without publishing a new version, which aligns with the "new sets every week" model used by similar brick-building games.

Collection Room System

The collection room becomes the long-term retention feature.

CollectionRoom
│
├── Vehicles
├── Buildings
├── Nature
├── Fantasy
└── Seasonal

Completed models are instantiated as decorative dioramas.

public class CollectionRoomManager : MonoBehaviour
{
    public void LoadOwnedModels()
    {
        // Fetch from Firestore
        // Spawn owned dioramas
    }
}
Recommended MVP (3 Months)
Phase 1
Authentication
20 blueprints
Build system
Snap placement
Progress saving
Phase 2
Collection room
XP and levels
Achievements
Daily rewards
Phase 3
Firebase content delivery
Ads
Battle pass
Events
Seasonal blueprints

This architecture comfortably scales from a small MVP with 20 models to a live-service game with hundreds of downloadable blueprints, collection rooms, seasonal events, battle passes, and cloud-synced progression.