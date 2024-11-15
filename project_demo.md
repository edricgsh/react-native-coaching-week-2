# React Native Location Tracker

## Part 1: Basic Setup and Navigation

### 1.1 Initial Project Setup

```bash
npx create-expo-app LocationTracker --template blank
cd LocationTracker
```

### 1.2 Navigation Dependencies

```bash
npm install @react-navigation/native
npm install @react-navigation/drawer
npm install @react-navigation/bottom-tabs
npm install react-native-reanimated
npm install react-native-gesture-handler
```

### 1.3 Basic Project Structure

```
LocationTracker/
├── App.js
├── index.js
├── src/
│   ├── screens/
│   │   ├── HomeScreen.js
│   │   ├── LocationScreen.js
│   │   └── LocationShowingScreen.js
│   └── styles/
│       └── styles.js
```

### Styles

```javascript
// src/styles/styles.js
import { StyleSheet } from "react-native";

export const locationScreenStyles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  buttonContainer: {
    marginTop: 20,
    marginBottom: 20,
  },
  historyContainer: {
    marginTop: 20,
  },
  historyTitle: {
    fontSize: 18,
    fontWeight: "bold",
    marginBottom: 10,
  },
  historyItem: {
    backgroundColor: "#f0f0f0",
    padding: 10,
    borderRadius: 5,
    marginBottom: 10,
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
  },
  locationInfo: {
    flex: 1,
  },
  saveButton: {
    marginLeft: 10,
  },
});

export const locationShowingScreenStyles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
  title: {
    fontSize: 20,
    fontWeight: "bold",
    marginBottom: 16,
    textAlign: "center",
  },
  locationCard: {
    backgroundColor: "#f0f0f0",
    padding: 16,
    borderRadius: 8,
    marginBottom: 12,
    shadowColor: "#000",
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
    elevation: 5,
  },
  timestamp: {
    fontWeight: "bold",
    marginBottom: 8,
  },
  noData: {
    textAlign: "center",
    marginTop: 20,
    fontSize: 16,
    color: "#666",
  },
});

export const homeScreenStyles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    padding: 20,
  },
  title: {
    fontSize: 24,
    marginBottom: 20,
    textAlign: "center",
  },
});
```

#### Basic HomeScreen.js

```javascript
import React from "react";
import { View, Text, Button } from "react-native";
import { homeScreenStyles as styles } from "../styles/styles";

const HomeScreen = ({ navigation }) => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Location Tracker</Text>
      <Button
        title="Go to Locations"
        onPress={() => navigation.navigate("Locations")}
      />
    </View>
  );
};

export default HomeScreen;
```

### 1.4 Navigation Setup

#### App.js (Navigation Only)

```javascript
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
import { createDrawerNavigator } from "@react-navigation/drawer";
import { NavigationContainer } from "@react-navigation/native";
import { Ionicons } from "@expo/vector-icons";
import React from "react";
import HomeScreen from "./src/screens/HomeScreen";
import LocationScreen from "./src/screens/LocationScreen";
import LocationShowingScreen from "./src/screens/LocationShowingScreen";

const Drawer = createDrawerNavigator();
const Tab = createBottomTabNavigator();

const LocationTabs = () => {
  return (
    <Tab.Navigator screenOptions={{ headerShown: false }}>
      <Tab.Screen name="Location" component={LocationScreen} />
      <Tab.Screen name="LocationShowing" component={LocationShowingScreen} />
    </Tab.Navigator>
  );
};

const App = () => {
  return (
    <NavigationContainer>
      <Drawer.Navigator>
        <Drawer.Screen name="Home" component={HomeScreen} />
        <Drawer.Screen name="Locations" component={LocationTabs} />
      </Drawer.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

## Part 2: Location Services

### 2.1 Location Setup

#### Install Location Dependencies

```bash
npm install expo-location
```

#### Update app.json

```json
{
  "expo": {
    // ... other configs
    "plugins": [
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow $(PRODUCT_NAME) to use your location."
        }
      ]
    ]
  }
}
```

### 2.2 Location Screen Implementation

```javascript
import * as Location from "expo-location";
import React, { useCallback, useState } from "react";
import { Alert, ScrollView, Text, View, Button } from "react-native";
import { useFocusEffect } from "@react-navigation/native";
import { locationScreenStyles as styles } from "../styles/styles";

const LocationScreen = () => {
  // State to store the history of locations
  const [locationHistory, setLocationHistory] = useState([]);

  // Use useFocusEffect to call getLocation when the screen comes into focus
  useFocusEffect(
    useCallback(() => {
      getLocation();
    }, [])
  );

  // Function to get the current location
  const getLocation = async () => {
    try {
      // Request permission to access location
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") {
        Alert.alert("Permission Denied", "Please allow location access");
        return;
      }

      // Get the current position
      const location = await Location.getCurrentPositionAsync({});
      // Create a location entry object
      const locationEntry = {
        latitude: location.coords.latitude,
        longitude: location.coords.longitude,
        timestamp: new Date().toLocaleString(),
      };

      // Update the location history state
      setLocationHistory((prev) => [...prev, locationEntry]);
    } catch (error) {
      Alert.alert("Error", "Failed to get location");
      console.error(error);
    }
  };

  return (
    <ScrollView style={styles.container}>
      <View style={styles.buttonContainer}>
        {/* Button to trigger getting current location */}
        <Button title="Get Current Location" onPress={getLocation} />
      </View>
      <View style={styles.historyContainer}>
        <Text style={styles.historyTitle}>Location History:</Text>
        {/* Map through the location history and display each entry */}
        {locationHistory.map((entry, index) => (
          <View key={index} style={styles.historyItem}>
            <Text>Time: {entry.timestamp}</Text>
            <Text>Latitude: {entry.latitude}</Text>
            <Text>Longitude: {entry.longitude}</Text>
          </View>
        ))}
      </View>
    </ScrollView>
  );
};

export default LocationScreen;
```

## Part 3: Persistent Storage

### 3.1 Storage Setup

#### Install AsyncStorage

```bash
npm install @react-native-async-storage/async-storage
```

### 3.2 Enhanced Location Screen with Storage

```javascript
// Add these imports to LocationScreen.js
import AsyncStorage from "@react-native-async-storage/async-storage";

// Add this function to LocationScreen component
const saveLocation = async (entry) => {
  try {
    const savedLocations = await AsyncStorage.getItem("savedLocations");
    let locations = savedLocations ? JSON.parse(savedLocations) : [];
    locations.push(entry);
    await AsyncStorage.setItem("savedLocations", JSON.stringify(locations));
    Alert.alert("Success", "Location saved");
  } catch (error) {
    Alert.alert("Error", "Failed to save location");
    console.error(error);
  }
};

// Update the history rendering to include save button
{
  locationHistory.map((entry, index) => (
    <View key={index} style={styles.historyItem}>
      <View style={styles.locationInfo}>
        <Text>Time: {entry.timestamp}</Text>
        <Text>Latitude: {entry.latitude}</Text>
        <Text>Longitude: {entry.longitude}</Text>
      </View>
      <Button
        title="Save"
        onPress={() => saveLocation(entry)}
        style={styles.saveButton}
      />
    </View>
  ));
}
```

### 3.3 Location Showing Screen (For Viewing Saved Locations)

```javascript
import React, { useState } from "react";
import { View, Text, ScrollView, Alert, Button } from "react-native";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { useFocusEffect } from "@react-navigation/native";
import { locationShowingScreenStyles as styles } from "../styles/styles";

const LocationShowingScreen = () => {
  // State to hold the saved locations
  const [savedLocations, setSavedLocations] = useState([]);

  // Use useFocusEffect to fetch saved locations when the screen comes into focus
  useFocusEffect(
    React.useCallback(() => {
      fetchSavedLocations();
    }, [])
  );

  // Function to fetch saved locations from AsyncStorage
  const fetchSavedLocations = async () => {
    try {
      const locations = await AsyncStorage.getItem("savedLocations");
      if (locations) {
        // If locations exist, parse the JSON string and update state
        setSavedLocations(JSON.parse(locations));
      }
    } catch (error) {
      // Handle errors when fetching locations
      Alert.alert("Error", "Failed to load locations");
      console.error(error);
    }
  };

  // Function to clear all saved locations
  const clearAllLocations = async () => {
    try {
      // Remove the savedLocations item from AsyncStorage
      await AsyncStorage.removeItem("savedLocations");
      // Clear the state
      setSavedLocations([]);
      Alert.alert("Success", "All locations cleared");
    } catch (error) {
      // Handle errors when clearing locations
      Alert.alert("Error", "Failed to clear locations");
      console.error(error);
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Saved Locations</Text>
      {/* Button to clear all saved locations */}
      <Button
        title="Clear All Locations"
        onPress={clearAllLocations}
        color="red"
      />
      {/* Conditional rendering based on whether there are saved locations */}
      {savedLocations.length === 0 ? (
        <Text style={styles.noData}>No saved locations found</Text>
      ) : (
        // Map through saved locations and render each one
        savedLocations.map((location, index) => (
          <View key={index} style={styles.locationCard}>
            <Text style={styles.timestamp}>Time: {location.timestamp}</Text>
            <Text>Latitude: {location.latitude}</Text>
            <Text>Longitude: {location.longitude}</Text>
          </View>
        ))
      )}
    </ScrollView>
  );
};

export default LocationShowingScreen;
```
