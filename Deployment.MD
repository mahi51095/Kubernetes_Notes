Head Office (Control Plane)
├── Project Manager = Deployment
│   └── Sends instructions to Team Lead = ReplicaSet
│       └── Ensures developers = Pods are working
└── (Reception Desk = Service) routes clients to developers

Factory Floor (Data Plane)
├── Developers = Pods (do real work)
├── Tools = Containers
└── Team Lead ensures headcount
