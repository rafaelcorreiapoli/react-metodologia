# Definitions
## SFC

```
SFC
├── SFC       [1...n]
└── Container [2...n]
```

Responsabilities:
 - Visual
 - Grouping

## Container
```
Container
└── SFC      [1]
```

Responsabilities:
 - Provide data
 - Provide behaviour

## Screen
```
Screen
├── SFC       [1...n]
└── Container [1...n]
```

Responsabilities:
 - Interface with router
 - Obtain route data
 - Navigation capabilities

## Router
```
Router
└── Screen     [1]
```

Responsabilities:
 - Manages navigation

## Module
- Screens
- SFCs
- Containers
- Models
- Logic
- Services
- GraphQL

# Examples
## Good  👍
Simple case
```
Router
└── Screen
    └── Container
        └── SFC
```

### 1.
Screen with 2 Containers
```
Router
└── Screen
    ├── Container
    |   └── SFC
    └── Container
        └── SFC
```

### 2.
Screen with 2 Containers and 1 SFC

SFC with multiple SFC

SFC with multiple Container
```
Router
└── Screen
    ├── Container
    |   └── SFC
    |       ├── SFC
    |       ├── SFC
    |       └── SFC
    └── Container
    |   └── SFC
    └── SFC
        ├── Container
        |   └── SFC
        └── Container
            └── SFC
 
```
## Bad 👎
### 1.
Container directly rendered in Router (without screen)
```
Router
└── Container
```

### 2.
Container rendering another Container
```
Router
└── Screen
    └── Container
        └── Container
```

### 3.
SFC rendering just one Container
```
Router
└── Screen
    └── SFC
        └── Container
             └── SFC
```

### 4.
Container rendering multiple SFC
```
Router
└── Screen
    └── Container
        ├── SFC
        └── SFC
```

# Folder structure
```
Modules
└── Mdule1
    ├── Screens
    |   └── Screen1
    |       ├── index.tsx
    |       └── styles.ts
    ├── Components
    |   └── Component1
    |       ├── index.tsx
    |       └── styles.ts
    ├── Container
    |   └── Container1
    |       └── index.tsx
    ├── Model
    |   └── MyModel.ts
    ├── Navigation
    |   └── Navigator1.ts
    ├── Logic
    |   └── my-business-logic.ts
    ├── Services
    |   └── api.ts
    |   └── local-db.ts
    └── GraphQL
        ├── Mutations
        |   └── my-mutation.ts
        └── Queries
            └── my-query.ts
```
