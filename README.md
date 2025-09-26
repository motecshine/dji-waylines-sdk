# WPML SDK - DJI Waypoint Markup Language

A Go library for creating and managing DJI drone waypoint missions using the WPML (Waypoint Markup Language) format. This SDK allows you to programmatically create flight plans and generate KMZ files compatible with DJI drone systems.

## Features

- ✅ Create waypoint missions from simple JSON structures
- ✅ Generate KMZ files for DJI drones
- ✅ Support multiple drone and payload models
- ✅ Comprehensive validation with detailed error messages
- ✅ XML serialization with proper formatting
- ✅ Action system for photos, gimbal control, and custom actions
- ✅ Type-safe API with extensive validation

## Installation

```bash
go get github.com/motecshine/wpml
```

## Quick Start

### 1. Define Your Waypoints

Create a `Waylines` structure with your mission parameters:

```go
package main

import (
    "fmt"
    "log"
    "github.com/motecshine/wpml"
)

func main() {
    // Define waypoints
    waylines := &wpml.Waylines{
        Name:         "Sample Mission",
        Description:  "A simple waypoint mission",
        DroneModel:   wpml.DroneModelMatrice3TD,
        PayloadModel: wpml.PayloadModelM3TD,
        TemplateType: wpml.TemplateTypeWaypoint,
        GlobalHeight: 50.0,  // meters
        GlobalSpeed:  5.0,   // m/s
        HeightType:   wpml.HeightModeRelativeToStartPoint,
        Waypoints: []wpml.WaylinesWaypoint{
            {
                Latitude:  39.9042,
                Longitude: 116.4074,
                Height:    50.0,
                Speed:     5.0,
                Actions: []wpml.ActionRequest{
                    {
                        Type: "takePhoto",
                        Params: map[string]interface{}{
                            "payloadPositionIndex": 0,
                        },
                    },
                },
            },
            {
                Latitude:  39.9052,
                Longitude: 116.4084,
                Height:    60.0,
                Speed:     5.0,
            },
        },
    }

    // Convert to WPML Mission
    mission, err := wpml.ConvertWaylinesToWPMLMission(waylines)
    if err != nil {
        log.Fatal("Failed to convert waylines:", err)
    }

    // Generate KMZ file
    err = wpml.CreateKmz(mission, "mission.kmz")
    if err != nil {
        log.Fatal("Failed to create KMZ:", err)
    }

    fmt.Println("Mission created successfully: mission.kmz")
}
```

### 2. Working with JSON

You can also load waylines from JSON:

```go
func LoadFromJSON() {
    jsonData := `{
        "name": "JSON Mission",
        "drone_model": "M3TD",
        "payload_model": "M3TD",
        "template_type": "waypoint",
        "global_height": 80,
        "global_speed": 8,
        "waypoints": [
            {
                "latitude": 39.9042,
                "longitude": 116.4074,
                "height": 80,
                "actions": [
                    {
                        "type": "takePhoto",
                        "params": {"payloadPositionIndex": 0}
                    }
                ]
            }
        ]
    }`

    var waylines wpml.Waylines
    if err := json.Unmarshal([]byte(jsonData), &waylines); err != nil {
        log.Fatal("Failed to parse JSON:", err)
    }

    mission, err := wpml.ConvertWaylinesToWPMLMission(&waylines)
    if err != nil {
        log.Fatal("Failed to convert mission:", err)
    }

    // Create KMZ
    err = wpml.CreateKmz(mission, "json_mission.kmz")
    if err != nil {
        log.Fatal("Failed to create KMZ:", err)
    }
}
```

## API Reference

### Core Types

#### Waylines
The main structure for defining a waypoint mission:

```go
type Waylines struct {
    Name                     string             `json:"name"`
    Description              string             `json:"description,omitempty"`
    DroneModel               DroneModel         `json:"drone_model"`
    PayloadModel             PayloadModel       `json:"payload_model"`
    TemplateType             TemplateType       `json:"template_type"`
    GlobalHeight             float64            `json:"global_height,omitempty"`
    GlobalSpeed              float64            `json:"global_speed,omitempty"`
    HeightType               HeightMode         `json:"height_type,omitempty"`
    Waypoints                []WaylinesWaypoint `json:"waypoints"`
    // ... additional fields
}
```

#### WaylinesWaypoint
Individual waypoint definition:

```go
type WaylinesWaypoint struct {
    Latitude     float64         `json:"latitude"`
    Longitude    float64         `json:"longitude"`
    Height       float64         `json:"height"`
    Speed        float64         `json:"speed,omitempty"`
    Actions      []ActionRequest `json:"actions,omitempty"`
    // ... additional fields
}
```

### Supported Drone Models

```go
const (
    DroneModelMini3         = "Mini 3"
    DroneModelAir2S         = "Air 2S"
    DroneModelMavic3        = "Mavic 3"
    DroneModelMatrice3TD    = "M3TD"
    DroneModelMatrice3M     = "M3M"
    DroneModelMatrice3E     = "M3E"
    // ... more models
)
```

### Payload Models

```go
const (
    PayloadModelMini3       = "Mini 3"
    PayloadModelAir2S       = "Air 2S"
    PayloadModelM3TD        = "M3TD"
    PayloadModelM3M         = "M3M"
    // ... more payloads
)
```

### Actions

The SDK supports various actions that can be performed at waypoints:

#### Take Photo
```go
action := wpml.ActionRequest{
    Type: "takePhoto",
    Params: map[string]interface{}{
        "payloadPositionIndex": 0,
    },
}
```

#### Gimbal Control
```go
action := wpml.ActionRequest{
    Type: "gimbalRotate",
    Params: map[string]interface{}{
        "payloadPositionIndex": 0,
        "gimbalRotateMode": "absoluteAngle",
        "gimbalPitchRotateAngle": -45,
        "gimbalYawRotateAngle": 0,
        "gimbalRotateTimeInSeconds": 2,
    },
}
```

#### Aircraft Yaw
```go
action := wpml.ActionRequest{
    Type: "aircraftYaw",
    Params: map[string]interface{}{
        "aircraftYawRotateMode": "absoluteAngle",
        "aircraftYawRotateAngle": 90,
    },
}
```

#### Hover
```go
action := wpml.ActionRequest{
    Type: "hover",
    Params: map[string]interface{}{
        "hoverTime": 5.0, // seconds
    },
}
```

### Main Functions

#### ConvertWaylinesToWPMLMission
Converts waylines to WPML mission format:

```go
mission, err := wpml.ConvertWaylinesToWPMLMission(waylines)
if err != nil {
    log.Fatal("Conversion failed:", err)
}
```

#### CreateKmz
Creates a KMZ file from a WPML mission:

```go
err := wpml.CreateKmz(mission, "output.kmz")
if err != nil {
    log.Fatal("KMZ creation failed:", err)
}
```

#### CreateKmzBuffer
Creates a KMZ buffer in memory:

```go
buffer, err := wpml.CreateKmzBuffer(mission)
if err != nil {
    log.Fatal("Buffer creation failed:", err)
}
// Use buffer.Bytes() to get []byte
```

#### ParseKMZBuffer
Parse an existing KMZ file:

```go
kmzData, err := os.ReadFile("existing.kmz")
if err != nil {
    log.Fatal("Failed to read KMZ:", err)
}

mission, err := wpml.ParseKMZBuffer(kmzData)
if err != nil {
    log.Fatal("Failed to parse KMZ:", err)
}
```

## Validation

The SDK includes comprehensive validation for all mission parameters:

### Waylines Validation
```go
waylines := &wpml.Waylines{
    // ... your waylines data
}

// Validation is automatically called during conversion
// But you can also validate manually:
if err := waylines.Validate(); err != nil {
    log.Printf("Validation error: %v", err)
}
```

### Validation Rules

- **Name**: Required, 1-100 characters
- **Drone Model**: Must be one of supported models
- **Payload Model**: Must be one of supported payloads
- **Global Height**: 5-1500 meters
- **Global Speed**: 1-15 m/s
- **Waypoint Coordinates**: Valid latitude (-90 to 90) and longitude (-180 to 180)
- **Actions**: Must have valid type and required parameters

## Advanced Usage

### Custom Height Modes

```go
waylines.HeightType = wpml.HeightModeRelativeToStartPoint
// or
waylines.HeightType = wpml.HeightModeEGM96
// or
waylines.HeightType = wpml.HeightModeRelativeToTakeoff
```

### Finish Actions

```go
waylines.FinishAction = wpml.FinishActionGoHome
// or
waylines.FinishAction = wpml.FinishActionAutoLand
// or
waylines.FinishAction = wpml.FinishActionNoAction
```

### RTH (Return to Home) Settings

```go
waylines.GlobalRTHHeight = 120.0  // RTH altitude in meters
waylines.SafeHeight = 50.0        // Safe altitude for RTH
```

### Waypoint Turn Modes

```go
waypoint.WaypointTurnMode = "coordinateTurn"
// or
waypoint.WaypointTurnMode = "toPointAndStopWithContinuityCurvature"
```

## Error Handling

The SDK provides detailed error messages for common issues:

```go
mission, err := wpml.ConvertWaylinesToWPMLMission(waylines)
if err != nil {
    switch {
    case strings.Contains(err.Error(), "validation failed"):
        log.Printf("Validation error: %v", err)
    case strings.Contains(err.Error(), "unsupported drone model"):
        log.Printf("Drone model error: %v", err)
    case strings.Contains(err.Error(), "invalid coordinates"):
        log.Printf("Coordinate error: %v", err)
    default:
        log.Printf("Unknown error: %v", err)
    }
}
```

## Examples

### Multi-Action Waypoint

```go
waypoint := wpml.WaylinesWaypoint{
    Latitude:  39.9042,
    Longitude: 116.4074,
    Height:    80.0,
    Actions: []wpml.ActionRequest{
        {
            Type: "takePhoto",
            Params: map[string]interface{}{
                "payloadPositionIndex": 0,
            },
        },
        {
            Type: "gimbalRotate",
            Params: map[string]interface{}{
                "payloadPositionIndex": 0,
                "gimbalRotateMode": "absoluteAngle",
                "gimbalPitchRotateAngle": -90,
            },
        },
        {
            Type: "hover",
            Params: map[string]interface{}{
                "hoverTime": 3.0,
            },
        },
    },
}
```

### Survey Mission Pattern

```go
func CreateSurveyMission(centerLat, centerLon, width, height float64, altitude float64) *wpml.Waylines {
    waypoints := []wpml.WaylinesWaypoint{}
    
    // Create grid pattern
    rows := 10
    cols := 10
    
    for i := 0; i < rows; i++ {
        for j := 0; j < cols; j++ {
            lat := centerLat + (float64(i-rows/2) * height / float64(rows))
            lon := centerLon + (float64(j-cols/2) * width / float64(cols))
            
            waypoints = append(waypoints, wpml.WaylinesWaypoint{
                Latitude:  lat,
                Longitude: lon,
                Height:    altitude,
                Actions: []wpml.ActionRequest{
                    {
                        Type: "takePhoto",
                        Params: map[string]interface{}{
                            "payloadPositionIndex": 0,
                        },
                    },
                },
            })
        }
    }
    
    return &wpml.Waylines{
        Name:         "Survey Mission",
        DroneModel:   wpml.DroneModelMatrice3TD,
        PayloadModel: wpml.PayloadModelM3TD,
        TemplateType: wpml.TemplateTypeWaypoint,
        GlobalHeight: altitude,
        GlobalSpeed:  5.0,
        Waypoints:    waypoints,
    }
}
```

## Dependencies

- `github.com/nbio/xml` - XML processing
- `github.com/go-playground/validator/v10` - Validation
- Standard library packages for JSON, ZIP, and file operations

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
