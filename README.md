

import React, { useState } from "react";
import { View, Text, TextInput, Button, StyleSheet, FlatList, TouchableOpacity } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { Provider, useDispatch, useSelector } from "react-redux";
import { configureStore, createSlice } from "@reduxjs/toolkit";

/* ===========================
   REDUX SETUP
=========================== */

// Auth Slice
const authSlice = createSlice({
  name: "auth",
  initialState: { user: null },
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload;
    },
    logout: (state) => {
      state.user = null;
    },
  },
});

// Booking Slice
const bookingSlice = createSlice({
  name: "booking",
  initialState: { bookings: [] },
  reducers: {
    addBooking: (state, action) => {
      state.bookings.push(action.payload);
    },
  },
});

const { setUser } = authSlice.actions;
const { addBooking } = bookingSlice.actions;

const store = configureStore({
  reducer: {
    auth: authSlice.reducer,
    booking: bookingSlice.reducer,
  },
});

/* ===========================
   SCREENS
=========================== */

function LoginScreen({ navigation }) {
  const [email, setEmail] = useState("");
  const dispatch = useDispatch();

  const handleLogin = () => {
    if (email.trim() === "") return;
    dispatch(setUser({ email }));
    navigation.replace("Home");
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Smart Service App</Text>
      <TextInput
        placeholder="Enter Email"
        value={email}
        onChangeText={setEmail}
        style={styles.input}
      />
      <Button title="Login" onPress={handleLogin} />
    </View>
  );
}

function HomeScreen({ navigation }) {
  const services = ["Plumber", "Electrician", "Cleaning", "Carpenter"];

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Select Service</Text>
      {services.map((service, index) => (
        <TouchableOpacity
          key={index}
          style={styles.card}
          onPress={() => navigation.navigate("Book Service", { service })}
        >
          <Text style={styles.cardText}>{service}</Text>
        </TouchableOpacity>
      ))}
      <Button title="View Booking History" onPress={() => navigation.navigate("History")} />
    </View>
  );
}

function BookingScreen({ route, navigation }) {
  const { service } = route.params;
  const dispatch = useDispatch();

  const handleBooking = () => {
    dispatch(
      addBooking({
        service,
        status: "Pending",
        date: new Date().toLocaleString(),
      })
    );
    navigation.navigate("History");
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Booking: {service}</Text>
      <Button title="Confirm Booking" onPress={handleBooking} />
    </View>
  );
}

function BookingHistoryScreen() {
  const bookings = useSelector((state) => state.booking.bookings);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Booking History</Text>
      <FlatList
        data={bookings}
        keyExtractor={(item, index) => index.toString()}
        renderItem={({ item }) => (
          <View style={styles.historyCard}>
            <Text>Service: {item.service}</Text>
            <Text>Status: {item.status}</Text>
            <Text>Date: {item.date}</Text>
          </View>
        )}
      />
    </View>
  );
}

/* ===========================
   NAVIGATION
=========================== */

const Stack = createNativeStackNavigator();

function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Login" component={LoginScreen} />
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Book Service" component={BookingScreen} />
        <Stack.Screen name="History" component={BookingHistoryScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

/* ===========================
   MAIN APP
=========================== */

export default function App() {
  return (
    <Provider store={store}>
      <AppNavigator />
    </Provider>
  );
}

/* ===========================
   STYLES
=========================== */

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: "center",
  },
  title: {
    fontSize: 22,
    fontWeight: "bold",
    marginBottom: 20,
    textAlign: "center",
  },
  input: {
    borderWidth: 1,
    padding: 10,
    marginBottom: 20,
    borderRadius: 8,
  },
  card: {
    padding: 15,
    borderWidth: 1,
    borderRadius: 10,
    marginBottom: 15,
    alignItems: "center",
  },
  cardText: {
    fontSize: 18,
  },
  historyCard: {
    padding: 15,
    borderWidth: 1,
    borderRadius: 10,
    marginBottom: 10,
  },
});
