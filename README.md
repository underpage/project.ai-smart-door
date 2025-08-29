# AI Door System


**시스템 구성**
```
┌────────────────┐    ┌─────────────────┐     ┌─────────────────┐    ┌──────────────────┐
│   1. Camera    │    │   2. Server     │     │  3. Dashboard   │    │  4. ML Training  │
│   (Hardware)   │    │    (Backend)    │     │    (Frontend)   │    │   (ML Pipeline)  │
├────────────────┤    ├─────────────────┤     ├─────────────────┤    ├──────────────────┤
│ • ESP32-CAM    │ -> │ • Stream API    │ <-> │ • React         │    │ • Data Pipeline  │
│                │    │ • YOLO Engine   │     │ • Real-time UI  │    │ • Model Training │
└────────────────┘    │ • Detection     │     │ • Event Monitor │    │ • Validation     │
                      │ • Notification  │     │ • Settings UI   │    │ • Model Export   │
                      │ • Analytics     │     └─────────────────┘    └──────────────────┘
                      │ • Database      │
                      │ • MinIO Client  │
                      └─────────────────┘

                      ┌─────────────────┐              
                      │   5. MinIO      │ 
                      │    (Storage)    │
                      ├─────────────────┤
                      │ • images/       │  
                      │ • models/       │  
                      │ • datasets/     │ 
                      └─────────────────┘
```