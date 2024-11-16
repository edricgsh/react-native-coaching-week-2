# Edric's Coaching Plan - React Native Week 2 Coaching

## Schedule

| Session                          | Duration   |
| -------------------------------- | ---------- |
| 1. Recap                         | 30 minutes |
| 2. Project Demo                  | 1.5 hours  |
| 3. Project Briefing & Discussion | 1 hour     |

# Recap

## 0. Follow up

1. **Is it possible to revoke the app's permission programmatically?**

   - From what I check, this is not possible, you have to do it manually from the settings.

2. **How do you store temporary data in React Native?**

   - AsyncStorage
   - SQLite
   - Keychain

3. **Is there StrictMode in ReactNative?**
   - I saw sources saying that it is. But when I try to do it locally, the `useEffect` did not run twice like how the React project should work 

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
