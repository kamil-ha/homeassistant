# Home Assistant Sonos Occupancy Automation

This Home Assistant configuration automatically manages Sonos music playback based on room occupancy, ensuring music follows you throughout your home.

## Features

- **Automatic Room Following**: Music automatically plays in occupied rooms and stops in unoccupied ones
- **Smart Pausing**: Music pauses when all rooms are unoccupied for 5 minutes
- **Auto-Resume**: Music resumes when you re-enter a room (if paused within the last hour)
- **Debouncing**: 30-second delay before removing unoccupied rooms to prevent flickering
- **Multi-Room Support**: Seamlessly manages 4 rooms (kitchen, living room, bedroom, bathroom)

## Room Configuration

The automation is configured for the following rooms:
- Kitchen
- Living Room
- Bedroom
- Bathroom

## Prerequisites

Before using this configuration, you need:

1. **Home Assistant** installed and running
2. **Sonos Integration** configured in Home Assistant
3. **Occupancy Sensors** for each room

## Required Entities

### Sonos Media Players

You must have the following Sonos media player entities configured:
- `media_player.sonos_kitchen`
- `media_player.sonos_livingroom`
- `media_player.sonos_bedroom`
- `media_player.sonos_bathroom`

### Occupancy Sensors

You must have binary occupancy sensors for each room:
- `binary_sensor.kitchen_occupancy`
- `binary_sensor.livingroom_occupancy`
- `binary_sensor.bedroom_occupancy`
- `binary_sensor.bathroom_occupancy`

These sensors should be in the `on` state when a room is occupied and `off` when unoccupied.

## Setup Instructions

### 1. Install Sonos Integration

1. Go to **Settings** > **Devices & Services** in Home Assistant
2. Click **Add Integration**
3. Search for "Sonos" and follow the setup wizard
4. Your Sonos speakers should be automatically discovered

### 2. Configure Occupancy Sensors

You can use various types of sensors for occupancy detection:

#### Option A: Motion Sensors
Configure motion sensors (PIR sensors, smart sensors, etc.) for each room.

#### Option B: Presence Detection
Use smartphone presence detection, Bluetooth trackers, or other presence detection methods.

#### Option C: Manual Binary Sensors (for testing)
Create manual switches in `configuration.yaml`:

```yaml
input_boolean:
  kitchen_occupancy:
    name: "Kitchen Occupied"
    icon: mdi:home-account
  livingroom_occupancy:
    name: "Living Room Occupied"
    icon: mdi:home-account
  bedroom_occupancy:
    name: "Bedroom Occupied"
    icon: mdi:home-account
  bathroom_occupancy:
    name: "Bathroom Occupied"
    icon: mdi:home-account

# Then create template binary sensors
template:
  - binary_sensor:
      - name: "Kitchen Occupancy"
        state: "{{ is_state('input_boolean.kitchen_occupancy', 'on') }}"
      - name: "Living Room Occupancy"
        state: "{{ is_state('input_boolean.livingroom_occupancy', 'on') }}"
      - name: "Bedroom Occupancy"
        state: "{{ is_state('input_boolean.bedroom_occupancy', 'on') }}"
      - name: "Bathroom Occupancy"
        state: "{{ is_state('input_boolean.bathroom_occupancy', 'on') }}"
```

### 3. Deploy Configuration Files

1. Copy all YAML files to your Home Assistant configuration directory
2. Restart Home Assistant
3. Check for any configuration errors in **Settings** > **System** > **Logs**

### 4. Verify Setup

1. Go to **Developer Tools** > **States**
2. Verify all required entities exist:
   - All 4 `media_player.sonos_*` entities
   - All 4 `binary_sensor.*_occupancy` entities
   - `input_boolean.sonos_music_playing`
   - `binary_sensor.any_room_occupied`

## Usage

### Automatic Operation

Once configured, the automation works automatically:

1. **Start playing music** on any Sonos speaker (using Sonos app, voice assistant, or Home Assistant)
2. **Move between rooms** - music will automatically follow you
3. **Leave all rooms** - music will pause after 5 minutes
4. **Return to a room** - music will resume if it was paused recently

### Manual Control

You can manually trigger the music to start in occupied rooms:

1. Go to **Developer Tools** > **Services**
2. Call the service `script.sonos_start_music_occupied_rooms`

This will:
- Start music on the kitchen Sonos (default master)
- Join all occupied rooms to the music group

## How It Works

### Automation Flow

1. **Room Becomes Occupied**:
   - Automation detects occupancy sensor change
   - Joins that room's Sonos speaker to the group
   - Music starts playing in that room

2. **Room Becomes Unoccupied**:
   - Waits 30 seconds (debounce)
   - Unjoins that room's Sonos speaker from the group
   - Music stops in that room

3. **All Rooms Unoccupied**:
   - Waits 5 minutes
   - Pauses music on all speakers
   - Sets tracking boolean to off

4. **First Room Becomes Occupied (after all were empty)**:
   - Resumes music if it was paused within the last hour
   - Joins occupied rooms to the group

### Smart Features

- **Master Speaker Selection**: The first occupied room becomes the master, or kitchen is used as default
- **State Tracking**: `input_boolean.sonos_music_playing` tracks whether music should be playing
- **Debouncing**: Prevents rapid on/off switching as you move between rooms
- **Auto-Resume Logic**: Only resumes if music was recently paused (not from days ago)

## Customization

### Adjust Timing

Edit `automations.yaml` to change:

- **Unjoin delay**: Change `for: seconds: 30` under the unjoin trigger
- **Pause delay**: Change `for: minutes: 5` under the pause trigger
- **Resume window**: Change `3600` (1 hour in seconds) in the resume condition

### Change Default Master Speaker

Edit `scripts.yaml` in the `sonos_join_occupied_rooms` script to change the master speaker priority order.

### Add/Remove Rooms

To add or remove rooms:

1. Update entity lists in `configuration.yaml`
2. Add/remove triggers in `automations.yaml`
3. Add/remove conditions in `scripts.yaml`

## Troubleshooting

### Music doesn't follow to occupied rooms

- Check that occupancy sensors are working correctly
- Verify `input_boolean.sonos_music_playing` is `on`
- Check automation traces in **Settings** > **Automations & Scenes**

### Music doesn't stop in unoccupied rooms

- Verify occupancy sensors show `off` for unoccupied rooms
- Check the 30-second delay hasn't been interrupted
- Review automation logs

### Music doesn't resume when entering a room

- Check if more than 1 hour has passed since pause
- Verify `binary_sensor.any_room_occupied` is working
- Ensure music was actually paused by the automation (not manually stopped)

## File Structure

```
homeassistant/
├── configuration.yaml      # Main configuration with helpers and templates
├── automations.yaml       # Occupancy-based music automations
├── scripts.yaml           # Helper scripts for Sonos management
├── groups.yaml            # (create if needed)
├── scenes.yaml            # (create if needed)
└── README.md              # This file
```

## Contributing

Feel free to customize this automation for your specific needs. Some ideas:

- Add volume control based on time of day
- Include bedroom only during certain hours
- Add bathroom music only when showering (using humidity sensor)
- Create scenes for different music sources per room

## License

This configuration is provided as-is for personal use in Home Assistant.
