# Edric's Coaching Plan - React Native Week 2 Coaching

## Schedule

| Session                          | Duration   |
| -------------------------------- | ---------- |
| 1. Recap                         | 30 minutes |
| 2. Project Briefing & Discussion | 2.5 hour     |

# Recap

## 0. Follow up

1. **Is it possible to revoke the app's permission programmatically?**

   - From what I check, this is not possible, you have to do it manually from the settings.

2. **How do you store temporary data in React Native?**

   - AsyncStorage
   - SQLite
   - Keychain

## 1. Navigation Fundamentals

### 1.1 Navigator Types

#### Stack Navigator

The Stack Navigator provides a way for your app to transition between screens where each new screen is placed on top of a stack. This creates a card-style navigation pattern where screens slide in from the right (on iOS) or fade/slide up (on Android). Think of it like a stack of cards – you can add new cards on top (push) and remove them (pop).

#### Tab Navigator

Tab navigation provides an easy way to switch between different routes by tapping on tabs, usually displayed at the bottom or top of the screen. It maintains the state of each tab route and ensures quick navigation between frequently accessed screens. Common examples include the bottom tabs in Instagram or Facebook apps.

#### Drawer Navigator

A drawer navigator provides a side menu that can be pulled out from the edge of the screen. This pattern is commonly used for apps with many top-level navigation destinations. Think of apps like Gmail where you can access different sections by pulling out the side menu.

### 1.2 Navigation Hooks

#### useFocusEffect and useCallback

The `useFocusEffect` hook is used to run side effects when a screen comes into focus. However, to optimize performance and prevent unnecessary re-renders, it should be used with `useCallback`. Here's a detailed explanation:

```javascript
import { useFocusEffect } from "@react-navigation/native";
import { useCallback } from "react";

function ProfileScreen() {
  // BAD: This will create a new callback every render
  useFocusEffect(() => {
    // This effect runs on every render
    fetchUserData();
  });

  // GOOD: Using useCallback to memoize the callback
  useFocusEffect(
    useCallback(() => {
      console.log("Screen focused");

      // Example: Fetch user data when screen focuses
      const loadUserProfile = async () => {
        try {
          const userData = await fetchUserProfile(userId);
          setUserData(userData);
        } catch (error) {
          console.error("Failed to load profile:", error);
        }
      };

      loadUserProfile();

      // Cleanup function when screen unfocuses
      return () => {
        console.log("Screen unfocused");
        // Cancel any subscriptions/pending requests
      };
    }, [userId]) // Dependencies array - effect will re-run if these values change
  );

  return <View>...</View>;
}
```

Key Points about useCallback with useFocusEffect:

1. **Purpose**:

   - `useCallback` memoizes the callback function, preventing unnecessary re-renders
   - Only recreates the function when dependencies change
   - Essential for performance optimization

2. **Syntax Structure**:

   ```javascript
   useFocusEffect(
     useCallback(
       () => {
         // Setup code
         return () => {
           // Cleanup code
         };
       },
       [
         /* dependencies */
       ]
     )
   );
   ```

3. **Common Use Cases**:
   - Fetching data when screen focuses
   - Setting up subscriptions or event listeners
   - Starting/stopping animations
   - Managing real-time connections

### 1.3 Navigation Object Methods

The navigation object provides several methods for programmatic navigation:

- `navigation.navigate('RouteName')`: Navigate to a route
- `navigation.navigate('RouteName', { paramName: value })`: Navigate to a route with parameters

  ```javascript
  // Example: Passing user data to profile screen
  navigation.navigate("Profile", { userId: 123, username: "john_doe" });

  // Example: Receiving parameters in the Profile screen
  function ProfileScreen({ route }) {
    const { userId, username } = route.params;

    return (
      <View>
        <Text>User ID: {userId}</Text>
        <Text>Username: {username}</Text>
      </View>
    );
  }
  ```

- `navigation.push('RouteName')`: Push a new route onto the stack
- `navigation.goBack()`: Go back one screen
- `navigation.popToTop()`: Go back to the first screen
- `navigation.replace('RouteName')`: Replace current screen
- `navigation.reset()`: Reset entire navigation state

## 2. External Libraries Integration

### 2.1 Configuring app.json

The app.json file is used to configure your Expo project.

```json
{
  "expo": {
    "name": "Your App Name",
    "version": "1.0.0",
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera."
        }
      ],
      [
        "expo-media-library",
        {
          "photosPermission": "Allow $(PRODUCT_NAME) to access your photos.",
          "savePhotosPermission": "Allow $(PRODUCT_NAME) to save photos."
        }
      ]
    ]
  }
}
```

### 2.2 Camera Implementation

```javascript
import { Camera } from "expo-camera";
import { useRef, useState } from "react";

function CameraScreen() {
  const [permission, requestPermission] = Camera.useCameraPermissions();
  const cameraRef = useRef(null);

  const takePicture = async () => {
    if (cameraRef.current) {
      const photo = await cameraRef.current.takePictureAsync();
      console.log(photo);
    }
  };

  if (!permission?.granted) {
    return (
      <View>
        <Text>We need your permission to show the camera</Text>
        <Button onPress={requestPermission} title="Grant Permission" />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Camera style={styles.camera} ref={cameraRef}>
        <View style={styles.buttonContainer}>
          <Button title="Take Picture" onPress={takePicture} />
        </View>
      </Camera>
    </View>
  );
}
```

The `useRef` hook is used here to maintain a reference to the Camera component. This is crucial for several reasons:

- **Direct Access**: By attaching the ref to the Camera component using the `ref={cameraRef}` prop, we get direct access to the Camera instance and its methods. This is essential for operations like `takePictureAsync()`.

- **Current Property**: The reference is accessed through the `.current` property, which initially starts as `null` (as we defined it). Once the Camera component mounts, React automatically sets the `.current` property to the Camera instance.

- **Method Access**: In our `takePicture` function, we first check if `cameraRef.current` exists before attempting to take a picture. This is a safety check to ensure the Camera component is properly mounted and accessible.

### 2.3 Image Picker Implementation

```javascript
import * as ImagePicker from "expo-image-picker";
import * as MediaLibrary from "expo-media-library";

function ImagePickerScreen() {
  const pickImage = async () => {
    const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();

    if (status !== "granted") {
      alert("Sorry, we need camera roll permissions to make this work!");
      return;
    }

    const result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      allowsEditing: true,
      aspect: [4, 3],
      quality: 1,
    });

    if (!result.canceled) {
      // Handle selected image
      console.log(result.assets[0]);

      // Save to media library
      await MediaLibrary.saveToLibraryAsync(result.assets[0].uri);
    }
  };

  return (
    <View>
      <Button title="Pick an image from camera roll" onPress={pickImage} />
    </View>
  );
}
```
# React Core Concepts

## 3. React Fundamentals

### 3.1 useEffect Deep Dive

The useEffect hook is used for handling side effects in functional components. Understanding cleanup functions is crucial for preventing memory leaks and ensuring proper resource management.

#### Cleanup Function Purpose

The cleanup function runs before the component unmounts and before re-running the effect (if dependencies change). It's essential for:
- Unsubscribing from subscriptions
- Canceling network requests
- Clearing intervals/timeouts
- Removing event listeners
- Closing WebSocket connections

```javascript
import { useEffect } from 'react';

function ChatRoom({ roomId }) {
  useEffect(() => {
    // Setup websocket connection
    const ws = new WebSocket(`ws://chat.example.com/${roomId}`);
    
    ws.addEventListener('message', handleMessage);
    
    // Cleanup function
    return () => {
      ws.removeEventListener('message', handleMessage);
      ws.close();
    };
  }, [roomId]); // Re-run effect if roomId changes

  return <div>Chat Room</div>;
}
```

Common Use Cases for Cleanup:

1. **Event Listeners**:
```javascript
function WindowResizeTracker() {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <div>Window width: {windowWidth}</div>;
}
```

2. **Intervals and Timeouts**:
```javascript
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);

    return () => {
      clearInterval(intervalId); // Cleanup to prevent memory leaks
    };
  }, []);

  return <div>Count: {count}</div>;
}
```

3. **API Subscriptions**:
```javascript
function UserStatus({ userId }) {
  const [isOnline, setIsOnline] = useState(false);

  useEffect(() => {
    let isMounted = true; // Flag to prevent setting state on unmounted component

    const subscribeToUserStatus = async () => {
      const unsubscribe = await api.subscribeToStatus(userId, (status) => {
        if (isMounted) {
          setIsOnline(status.isOnline);
        }
      });

      return unsubscribe;
    };

    const unsubscribe = subscribeToUserStatus();

    return () => {
      isMounted = false;
      if (unsubscribe) {
        unsubscribe();
      }
    };
  }, [userId]);

  return <div>{isOnline ? '🟢 Online' : '⚫️ Offline'}</div>;
}
```

### 3.2 useContext

useContext provides a way to pass data through the component tree without manually passing props at every level. It's particularly useful for sharing global state or theme data.

#### Basic Implementation

```javascript
import { createContext, useContext, useState } from 'react';

// Create context with default value
const ThemeContext = createContext('light');

// Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer Component
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      style={{
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff'
      }}
    >
      Current theme: {theme}
    </button>
  );
}

// Usage
function App() {
  return (
    <ThemeProvider>
      <div>
        <ThemedButton />
        {/* Other components can access theme context */}
      </div>
    </ThemeProvider>
  );
}
```

Common Use Cases:
1. Theme Management
2. User Authentication State
3. Language/Localization
4. Feature Flags
5. Global UI State (loading, error states)

### 3.3 Custom Hooks

Custom hooks allow you to extract component logic into reusable functions. They should start with "use" and can call other hooks.

#### Example: useLocalStorage Hook

```javascript
function useLocalStorage(key, initialValue) {
  // State to store our value
  // Pass initial state function to useState so logic is only executed once
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  // Return a wrapped version of useState's setter function
  const setValue = value => {
    try {
      // Allow value to be a function so we have same API as useState
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
function App() {
  const [name, setName] = useLocalStorage('name', 'Bob');

  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={e => setName(e.target.value)}
      />
    </div>
  );
}
```

#### Example: useFetch Hook

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, {
          signal: abortController.signal
        });
        const json = await response.json();
        setData(json);
        setLoading(false);
      } catch (error) {
        if (error.name === 'AbortError') {
          console.log('Fetch aborted');
        } else {
          setError(error);
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      abortController.abort();
    };
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return <div>User: {data.name}</div>;
}
```

### 3.4 useRef

useRef is a hook that provides a mutable ref object whose .current property is initialized with the passed argument. The ref object persists for the full lifetime of the component.

Key Characteristics:
- Doesn't trigger re-renders when value changes
- Persists between renders
- Can store any mutable value
- Commonly used to access DOM elements directly

#### Common Use Cases

1. **Accessing DOM Elements**:
```javascript
function TextInputWithFocus() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}
```

2. **Instance Variables (Mutable Values)**:
```javascript
function StopWatch() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);

  const start = () => {
    if (intervalRef.current !== null) return;
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current === null) return;
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  useEffect(() => {
    return () => {
      if (intervalRef.current !== null) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <div>
      <h1>{count}s</h1>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```