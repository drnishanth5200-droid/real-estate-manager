 import React, { useState, useEffect } from 'react';
import { Text, View, TextInput, Button, FlatList, Image, TouchableOpacity, ScrollView, StyleSheet, Alert } from 'react-native';
import * as ImagePicker from 'expo-image-picker';

import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

// Mock simple user authentication
const mockUsers = [{ email: 'test@example.com', password: '123456' }];

function LoginScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const login = () => {
    const found = mockUsers.find(u => u.email === email && u.password === password);
    if (found) {
      navigation.replace('PropertyList');
    } else {
      Alert.alert('Login Failed', 'Invalid email or password');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>
      <TextInput placeholder="Email" style={styles.input} value={email} onChangeText={setEmail} keyboardType="email-address" />
      <TextInput placeholder="Password" style={styles.input} value={password} onChangeText={setPassword} secureTextEntry />
      <Button title="Login" onPress={login} />
      <TouchableOpacity onPress={() => navigation.navigate('Register')} style={{ marginTop: 15 }}>
        <Text style={{ color: 'blue' }}>Don't have an account? Register</Text>
      </TouchableOpacity>
    </View>
  );
}

function RegisterScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const register = () => {
    if (!email || !password) {
      Alert.alert('Error', 'Please fill all fields');
      return;
    }
    const exists = mockUsers.find(u => u.email === email);
    if (exists) {
      Alert.alert('Error', 'Email already registered');
      return;
    }
    mockUsers.push({ email, password });
    Alert.alert('Success', 'Registration successful. Please login.');
    navigation.goBack();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Register</Text>
      <TextInput placeholder="Email" style={styles.input} value={email} onChangeText={setEmail} keyboardType="email-address" />
      <TextInput placeholder="Password" style={styles.input} value={password} onChangeText={setPassword} secureTextEntry />
      <Button title="Register" onPress={register} />
    </View>
  );
}

function AddPropertyScreen({ navigation, route }) {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [price, setPrice] = useState('');
  const [address, setAddress] = useState('');
  const [contactNumber, setContactNumber] = useState('');
  const [images, setImages] = useState([]);

  const pickImages = async () => {
    const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
    if (status !== 'granted') {
      alert('Sorry, we need camera roll permissions to select images!');
      return;
    }
    let result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      allowsMultipleSelection: true,
      quality: 0.5,
      base64: false,
    });

    if (!result.canceled) {
      if (result.assets) {
        setImages([...images, ...result.assets.map(a => a.uri)]);
      } else {
        setImages([...images, result.uri]);
      }
    }
  };

  const submitProperty = () => {
    if (!title || !price || !address || !contactNumber) {
      Alert.alert('Error', 'Please fill required fields');
      return;
    }
    const newProperty = {
      id: Date.now().toString(),
      title,
      description,
      price,
      address,
      contactNumber,
      images,
    };
    route.params.addProperty(newProperty);
    navigation.goBack();
  };

  return (
    <ScrollView contentContainerStyle={styles.container}>
      <Text style={styles.title}>Add Property</Text>
      <TextInput placeholder="Title*" value={title} onChangeText={setTitle} style={styles.input} />
      <TextInput placeholder="Description" value={description} onChangeText={setDescription} style={[styles.input, {height: 80}]} multiline />
      <TextInput placeholder="Price*" value={price} onChangeText={setPrice} keyboardType="numeric" style={styles.input} />
      <TextInput placeholder="Address*" value={address} onChangeText={setAddress} style={styles.input} />
      <TextInput placeholder="Contact Number*" value={contactNumber} onChangeText={setContactNumber} keyboardType="phone-pad" style={styles.input} />

      <Button title="Pick Images" onPress={pickImages} />
      <View style={{ flexDirection: 'row', marginVertical: 10, flexWrap: 'wrap' }}>
        {images.map((uri, i) => <Image key={i} source={{ uri }} style={styles.thumbnail} />)}
      </View>

      <Button title="Submit Property" onPress={submitProperty} />
    </ScrollView>
  );
}

function PropertyListScreen({ navigation }) {
  const [properties, setProperties] = useState([]);

  // Receive new property from AddPropertyScreen
  const addProperty = (property) => {
    setProperties([property, ...properties]);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Property Listings</Text>
      <Button title="Add Property" onPress={() => navigation.navigate('AddProperty', { addProperty })} />
      <FlatList
        data={properties}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.card}>
            <Text style={styles.propertyTitle}>{item.title} - ₹{item.price}</Text>
            <Text>{item.address}</Text>
            <Text>Contact: {item.contactNumber}</Text>
            <ScrollView horizontal>
              {item.images.map((uri, i) => (
                <Image key={i} source={{ uri }} style={styles.thumbnail} />
              ))}
            </ScrollView>
          </View>
        )}
        ListEmptyComponent={<Text>No properties yet. Add one!</Text>}
      />
    </View>
  );
}

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Login">
        <Stack.Screen options={{ title: 'Login' }} name="Login" component={LoginScreen} />
        <Stack.Screen options={{ title: 'Register' }} name="Register" component={RegisterScreen} />
        <Stack.Screen options={{ title: 'Properties' }} name="PropertyList" component={PropertyListScreen} />
        <Stack.Screen options={{ title: 'Add Property' }} name="AddProperty" component={AddPropertyScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    flexGrow: 1,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    marginBottom: 15,
    fontWeight: 'bold'
  },
  input: {
    borderColor: '#aaa',
    borderWidth: 1,
    marginBottom: 10,
    padding: 8,
    borderRadius: 4,
  },
  card: {
    borderWidth: 1,
    borderColor: '#ddd',
    marginVertical: 8,
    padding: 10,
    borderRadius: 5,
    backgroundColor: '#f9f9f9',
  },
  propertyTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 4,
  },
  thumbnail: {
    width: 100,
    height: 75,
    marginRight: 10,
    marginTop: 5,
    borderRadius: 4,
  },
});
