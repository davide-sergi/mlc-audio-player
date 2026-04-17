# MLC Audio Player - Exercise Guide

Learn to build an audio player from scratch through interactive exercises. Each exercise builds on the previous one, progressively adding features to your Python audio player application.

## Prerequisites

- Python 3.8+
- `pip` package manager
- A code editor (VS Code, PyCharm, etc.)

### Setup Instructions

Create a new Python file in your local workspace:

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

Install required dependencies:

```bash
pip install pygame  # For audio playback
pip install pyaudio  # For audio I/O
pip install numpy   # For audio processing (optional)
```

---

## Exercise 1: Initialize Your First Audio Player

**Objective**: Set up the basic structure of your audio player and load an audio file.

**Tasks**:
1. Create a Python script called `audio_player.py`
2. Import necessary libraries
3. Create a class to manage audio playback
4. Initialize the player with an audio file path

**Code Snippet** - Copy this into your `audio_player.py`:

```python
import pygame
import os

class AudioPlayer:
    def __init__(self, audio_file):
        """Initialize the audio player with an audio file."""
        pygame.mixer.init()
        self.audio_file = audio_file
        self.is_playing = False
        
        # TODO: Add your code here to load the audio file
        # Hint: Use pygame.mixer.Sound() to load the file
        
    def load_audio(self):
        """Load the audio file."""
        # TODO: Implement audio loading logic
        pass
    
    def display_info(self):
        """Display information about the loaded audio."""
        print(f"Loaded: {self.audio_file}")
        # TODO: Add code to display audio properties (duration, format, etc.)

# Test your implementation
if __name__ == "__main__":
    # TODO: Create an instance of AudioPlayer with an audio file path
    # player = AudioPlayer("path/to/your/audio.mp3")
    pass
```

**Expected Outcome**: Your player should initialize without errors and be ready to load audio.

---

## Exercise 2: Add Basic Playback Controls

**Objective**: Implement play, pause, and stop functionality.

**Tasks**:
1. Add a `play()` method to start audio playback
2. Add a `pause()` method to pause playback
3. Add a `stop()` method to stop and reset playback
4. Add a `get_status()` method to return current state

**Code Snippet** - Add these methods to your `AudioPlayer` class:

```python
def play(self):
    """Start or resume audio playback."""
    # TODO: Implement play logic
    # Hint: Check if already playing, then use pygame.mixer.Sound.play()
    pass

def pause(self):
    """Pause the audio playback."""
    # TODO: Implement pause logic
    pass

def stop(self):
    """Stop audio playback and reset."""
    # TODO: Implement stop logic
    self.is_playing = False

def get_status(self):
    """Return the current playback status."""
    return {
        "playing": self.is_playing,
        "file": self.audio_file
    }

# Add to your test section:
if __name__ == "__main__":
    # TODO: Test play, pause, and stop methods
    pass
```

**Expected Outcome**: You should be able to play, pause, and stop audio from your terminal.

---

## Exercise 3: Add Volume Control

**Objective**: Implement volume adjustment functionality.

**Tasks**:
1. Add a `set_volume(level)` method (0.0 to 1.0)
2. Add a `get_volume()` method to check current volume
3. Add `increase_volume()` and `decrease_volume()` convenience methods
4. Validate volume levels

**Code Snippet** - Add these methods to your `AudioPlayer` class:

```python
def set_volume(self, level):
    """Set volume level (0.0 to 1.0)."""
    # TODO: Add validation for level (0.0-1.0)
    # TODO: Apply volume using pygame.mixer
    pass

def get_volume(self):
    """Get current volume level."""
    # TODO: Return current volume
    pass

def increase_volume(self, amount=0.1):
    """Increase volume by the specified amount."""
    # TODO: Get current volume, add amount, set it (with bounds checking)
    pass

def decrease_volume(self, amount=0.1):
    """Decrease volume by the specified amount."""
    # TODO: Get current volume, subtract amount, set it (with bounds checking)
    pass
```

**Expected Outcome**: Volume changes should be reflected in audio playback.

---

## Exercise 4: Implement Playlist Management

**Objective**: Create a playlist system to manage multiple audio files.

**Tasks**:
1. Add a `add_to_playlist(file)` method
2. Add a `next_track()` method to play next in playlist
3. Add a `previous_track()` method to play previous track
4. Display current track and playlist position

**Code Snippet** - Extend your `AudioPlayer` class:

```python
class AudioPlayer:
    def __init__(self, audio_file=None):
        pygame.mixer.init()
        self.playlist = []
        self.current_index = 0
        self.is_playing = False
        
        if audio_file:
            self.add_to_playlist(audio_file)
    
    def add_to_playlist(self, file):
        """Add a file to the playlist."""
        # TODO: Validate file exists
        # TODO: Add to playlist
        pass
    
    def next_track(self):
        """Play the next track in the playlist."""
        # TODO: Increment current_index
        # TODO: Check bounds
        # TODO: Load and play new track
        pass
    
    def previous_track(self):
        """Play the previous track in the playlist."""
        # TODO: Decrement current_index
        # TODO: Check bounds
        # TODO: Load and play previous track
        pass
    
    def get_playlist_info(self):
        """Get information about current playlist."""
        return {
            "total_tracks": len(self.playlist),
            "current_index": self.current_index,
            "current_track": self.playlist[self.current_index] if self.playlist else None
        }
```

**Expected Outcome**: You should be able to maintain and navigate through a playlist.

---

## Exercise 5: Add Playback Position Control

**Objective**: Implement seeking and progress tracking.

**Tasks**:
1. Add a `get_position()` method to get current playback position
2. Add a `seek(position)` method to jump to a specific time
3. Add a `get_duration()` method to get total audio duration
4. Display progress information

**Code Snippet** - Add these methods to your `AudioPlayer` class:

```python
def get_position(self):
    """Get current playback position in seconds."""
    # TODO: Return current position from mixer
    pass

def get_duration(self):
    """Get total audio duration in seconds."""
    # TODO: Return duration of loaded audio
    pass

def seek(self, position):
    """Seek to a specific position in seconds."""
    # TODO: Validate position is within duration
    # TODO: Seek to position (may need to stop and replay from position)
    pass

def get_progress_percentage(self):
    """Get playback progress as a percentage."""
    # TODO: Calculate (position / duration) * 100
    pass

def display_progress(self):
    """Display formatted progress information."""
    # TODO: Show current position and total duration
    # Format: "2:34 / 5:12" or similar
    pass
```

**Expected Outcome**: You can seek to specific positions and track playback progress.

---

## Exercise 6: Add Event Handling (Bonus)

**Objective**: Implement event callbacks for better control.

**Tasks**:
1. Add callback methods for play, pause, stop events
2. Add callback for track completion
3. Implement a simple event system

**Code Snippet** - Add event handling to your `AudioPlayer` class:

```python
class AudioPlayer:
    def __init__(self):
        # ... existing code ...
        self.callbacks = {
            "on_play": [],
            "on_pause": [],
            "on_stop": [],
            "on_track_end": []
        }
    
    def register_callback(self, event, callback):
        """Register a callback for an event."""
        # TODO: Validate event type
        # TODO: Add callback to the list
        pass
    
    def trigger_event(self, event, data=None):
        """Trigger all callbacks for an event."""
        # TODO: Iterate through callbacks for the event
        # TODO: Call each callback with data
        pass
    
    def play(self):
        """Start playback and trigger on_play event."""
        # TODO: Existing play logic
        # TODO: Call self.trigger_event("on_play", {...})
        pass
```

**Expected Outcome**: Events trigger callbacks when player state changes.

---

## How to Work Through These Exercises

1. **Create your script** with the provided skeleton code
2. **Read each TODO comment** carefully
3. **Implement the logic** based on the hints provided
4. **Test your implementation** with sample audio files
5. **Move to the next exercise** once current one works

## Resources

- [Pygame Documentation](https://www.pygame.org/docs/)
- [Python Audio Processing](https://realpython.com/python-audio-processing/)
- [PySerial Documentation](https://pyserial.readthedocs.io/) - if you need serial communication

## Tips

- Start with MP3 or WAV files for testing
- Use small audio files to test quickly
- Print debug information to understand what's happening
- Test each method individually before combining them

Good luck! 🎵