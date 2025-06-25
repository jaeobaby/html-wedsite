git clone https://github.com/yourusername/streamfusion.git
cd streamfusion
npm install

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "yourapp.firebaseapp.com",
  projectId: "yourapp",
  storageBucket: "yourapp.appspot.com",
  messagingSenderId: "XXXX",
  appId: "XXXXXX",
};

npx expo start

eas build --platform android
eas build --platform ios

git init
git add .
git commit -m "Initial Commit"
git branch -M main
git remote add origin https://github.com/yourusername/streamfusion.git
git push -u origin main

{
  "name": "streamfusion",
  "version": "1.0.0",
  "main": "node_modules/expo/AppEntry.js",
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web"
  },
  "dependencies": {
    "axios": "^1.7.2",
    "expo": "~50.0.17",
    "expo-status-bar": "~1.10.0",
    "firebase": "^10.8.0",
    "react": "18.2.0",
    "react-native": "0.73.7",
    "react-native-webview": "13.11.1",
    "@react-navigation/bottom-tabs": "^6.5.17",
    "@react-navigation/native": "^6.1.12",
    "@react-navigation/native-stack": "^6.9.22"
  },
  "devDependencies": {
    "@babel/core": "^7.20.0"
  },
  "private": true
}

import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "yourapp.firebaseapp.com",
  projectId: "yourapp",
  storageBucket: "yourapp.appspot.com",
  messagingSenderId: "XXXX",
  appId: "XXXXXX",
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);

import React, { useEffect, useState } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { Ionicons } from '@expo/vector-icons';
import { authListener } from './utils/authManager';

import LoginScreen from './screens/LoginScreen';
import DiscoverScreen from './screens/DiscoverScreen';
import RecommendationsScreen from './screens/RecommendationsScreen';
import QueueScreen from './screens/QueueScreen';
import WatchedHistoryScreen from './screens/WatchedHistoryScreen';
import SearchScreen from './screens/SearchScreen';
import DetailsScreen from './screens/DetailsScreen';

const Tab = createBottomTabNavigator();
const Stack = createNativeStackNavigator();

function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarActiveTintColor: '#00FFFF',
        tabBarInactiveTintColor: 'gray',
        tabBarStyle: { backgroundColor: '#000' },
        headerStyle: { backgroundColor: '#000' },
        headerTitleStyle: { color: '#00FFFF' },
        tabBarIcon: ({ color, size }) => {
          let icon;
          if (route.name === 'Discover') icon = 'compass';
          else if (route.name === 'Queue') icon = 'list';
          else if (route.name === 'Recommendations') icon = 'sparkles';
          else if (route.name === 'History') icon = 'time';
          else if (route.name === 'Search') icon = 'search';
          return <Ionicons name={icon} size={size} color={color} />;
        }
      })}
    >
      <Tab.Screen name="Discover" component={DiscoverScreen} />
      <Tab.Screen name="Queue" component={QueueScreen} />
      <Tab.Screen name="Recommendations" component={RecommendationsScreen} />
      <Tab.Screen name="History" component={WatchedHistoryScreen} />
      <Tab.Screen name="Search" component={SearchScreen} />
    </Tab.Navigator>
  );
}

export default function App() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const unsubscribe = authListener((u) => setUser(u));
    return unsubscribe;
  }, []);

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {user ? (
          <>
            <Stack.Screen name="Main" component={MainTabs} />
            <Stack.Screen name="Details" component={DetailsScreen} />
          </>
        ) : (
          <Stack.Screen name="Login" component={LoginScreen} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}

export const API_KEY = 'YOUR_TMDB_API_KEY';
export const BASE_URL = 'https://api.themoviedb.org/3';

import { auth } from '../firebaseConfig';
import {
  createUserWithEmailAndPassword,
  signInWithEmailAndPassword,
  signOut,
  onAuthStateChanged,
} from 'firebase/auth';

export const signup = (email, password) =>
  createUserWithEmailAndPassword(auth, email, password);

export const login = (email, password) =>
  signInWithEmailAndPassword(auth, email, password);

export const logout = () => signOut(auth);

export const authListener = (callback) =>
  onAuthStateChanged(auth, callback);

import AsyncStorage from '@react-native-async-storage/async-storage';

const QUEUE_KEY = 'QUEUE_DATA';

export const loadQueue = async () => {
  const json = await AsyncStorage.getItem(QUEUE_KEY);
  return json != null ? JSON.parse(json) : [];
};

export const saveQueue = async (queue) => {
  await AsyncStorage.setItem(QUEUE_KEY, JSON.stringify(queue));
};

export const addToQueue = async (item) => {
  const queue = await loadQueue();
  const exists = queue.find((i) => i.id === item.id);
  if (!exists) {
    queue.unshift(item);
    await saveQueue(queue);
  }
};

export const removeFromQueue = async (itemId) => {
  const queue = await loadQueue();
  const newQueue = queue.filter((i) => i.id !== itemId);
  await saveQueue(newQueue);
};

export const clearQueue = async () => {
  await saveQueue([]);
};

import AsyncStorage from '@react-native-async-storage/async-storage';

const HISTORY_KEY = 'WATCHED_HISTORY';

export const loadHistory = async () => {
  const json = await AsyncStorage.getItem(HISTORY_KEY);
  return json != null ? JSON.parse(json) : [];
};

export const saveHistory = async (history) => {
  await AsyncStorage.setItem(HISTORY_KEY, JSON.stringify(history));
};

export const addToHistory = async (item) => {
  const history = await loadHistory();
  const exists = history.find((i) => i.id === item.id);
  if (!exists) {
    history.unshift(item);
    await saveHistory(history);
  }
};

export const clearHistory = async () => {
  await saveHistory([]);
};

import axios from 'axios';
import { API_KEY, BASE_URL } from './tmdb';

export const fetchTrending = async (type = 'all') => {
  const url = `${BASE_URL}/trending/${type}/week?api_key=${API_KEY}`;
  const response = await axios.get(url);
  return response.data.results;
};

export const fetchPopular = async (type = 'movie') => {
  const url = `${BASE_URL}/${type}/popular?api_key=${API_KEY}`;
  const response = await axios.get(url);
  return response.data.results;
};

export const fetchTopRated = async (type = 'movie') => {
  const url = `${BASE_URL}/${type}/top_rated?api_key=${API_KEY}`;
  const response = await axios.get(url);
  return response.data.results;
};

export const fetchByGenre = async (genreId, type = 'movie') => {
  const url = `${BASE_URL}/discover/${type}?api_key=${API_KEY}&with_genres=${genreId}`;
  const response = await axios.get(url);
  return response.data.results;
};

export const fetchGenres = async (type = 'movie') => {
  const url = `${BASE_URL}/genre/${type}/list?api_key=${API_KEY}`;
  const response = await axios.get(url);
  return response.data.genres;
};

import axios from 'axios';
import { API_KEY, BASE_URL } from './tmdb';
import { loadHistory } from './historyManager';
import { loadQueue } from './queueManager';

export const fetchRecommendationsForItem = async (id, type = 'movie') => {
  const url = `${BASE_URL}/${type}/${id}/recommendations?api_key=${API_KEY}`;
  const response = await axios.get(url);
  return response.data.results;
};

export const fetchDiscover = async (genres = [], type = 'movie') => {
  const url = `${BASE_URL}/discover/${type}?api_key=${API_KEY}&with_genres=${genres.join(',')}`;
  const response = await axios.get(url);
  return response.data.results;
};

export const getAIRecommendations = async () => {
  const history = await loadHistory();
  const queue = await loadQueue();

  const allItems = [...history, ...queue];
  if (allItems.length === 0) return [];

  const genreCounts = {};
  const typeCounts = { movie: 0, tv: 0 };

  for (const item of allItems) {
    if (item.genre_ids) {
      item.genre_ids.forEach((g) => {
        genreCounts[g] = (genreCounts[g] || 0) + 1;
      });
    }
    const t = item.media_type || item.type || 'movie';
    typeCounts[t] = (typeCounts[t] || 0) + 1;
  }

  const topGenres = Object.entries(genreCounts)
    .sort((a, b) => b[1] - a[1])
    .map(([id]) => id)
    .slice(0, 3);

  const mostWatchedType = typeCounts.movie >= typeCounts.tv ? 'movie' : 'tv';

  const discover = await fetchDiscover(topGenres, mostWatchedType);

  const recentItems = allItems.slice(0, 3);
  let recs = [];
  for (const item of recentItems) {
    const itemRecs = await fetchRecommendationsForItem(item.id, item.media_type || 'movie');
    recs = [...recs, ...itemRecs];
  }

  const seen = new Set();
  const unique = recs.concat(discover).filter((i) => {
    if (seen.has(i.id)) return false;
    seen.add(i.id);
    return true;
  });

  return unique;
};

import React, { useState, useEffect } from 'react';
import { View, TextInput, Button, Text, StyleSheet, Alert } from 'react-native';
import { login, signup, authListener } from '../utils/authManager';

export default function LoginScreen({ navigation }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  useEffect(() => {
    const unsubscribe = authListener((user) => {
      if (user) {
        navigation.replace('Main');
      }
    });
    return unsubscribe;
  }, []);

  const handleLogin = async () => {
    try {
      await login(email, password);
    } catch (e) {
      Alert.alert('Login Error', e.message);
    }
  };

  const handleSignup = async () => {
    try {
      await signup(email, password);
    } catch (e) {
      Alert.alert('Signup Error', e.message);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>StreamFusion</Text>
      <TextInput
        placeholder="Email"
        placeholderTextColor="#aaa"
        value={email}
        onChangeText={setEmail}
        style={styles.input}
      />
      <TextInput
        placeholder="Password"
        placeholderTextColor="#aaa"
        value={password}
        onChangeText={setPassword}
        style={styles.input}
        secureTextEntry
      />
      <Button title="Login" onPress={handleLogin} />
      <View style={{ height: 10 }} />
      <Button title="Sign Up" onPress={handleSignup} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#000', justifyContent: 'center', padding: 20 },
  input: {
    borderWidth: 1,
    borderColor: '#00FFFF',
    borderRadius: 8,
    padding: 10,
    marginBottom: 15,
    color: '#fff',
  },
  title: { color: '#00FFFF', fontSize: 28, marginBottom: 30, textAlign: 'center' },
});

import React, { useEffect, useState } from 'react';
import { View, Text, FlatList, Image, TouchableOpacity, ScrollView, StyleSheet, ActivityIndicator } from 'react-native';
import { fetchTrending, fetchPopular, fetchTopRated, fetchGenres, fetchByGenre } from '../utils/discoverAPI';

export default function DiscoverScreen({ navigation }) {
  const [trending, setTrending] = useState([]);
  const [popular, setPopular] = useState([]);
  const [topRated, setTopRated] = useState([]);
  const [genres, setGenres] = useState([]);
  const [genreResults, setGenreResults] = useState({});
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    setLoading(true);
    const [trend, pop, top, gen] = await Promise.all([
      fetchTrending(),
      fetchPopular(),
      fetchTopRated(),
      fetchGenres(),
    ]);

    setTrending(trend);
    setPopular(pop);
    setTopRated(top);
    setGenres(gen);

    const genreFetches = gen.slice(0, 4).map(async (g) => {
      const items = await fetchByGenre(g.id);
      return { name: g.name, items };
    });

    const genreData = await Promise.all(genreFetches);
    const genreObj = {};
    genreData.forEach((g) => {
      genreObj[g.name] = g.items;
    });

    setGenreResults(genreObj);
    setLoading(false);
  };

  const renderItem = ({ item }) => (
    <TouchableOpacity
      style={styles.card}
      onPress={() =>
        navigation.navigate('Details', {
          id: item.id,
          type: item.media_type || 'movie',
        })
      }
    >
      <Image
        source={{
          uri: item.poster_path
            ? `https://image.tmdb.org/t/p/w500${item.poster_path}`
            : 'https://via.placeholder.com/120x180.png?text=No+Image',
        }}
        style={styles.poster}
      />
    </TouchableOpacity>
  );

  const renderSection = (title, data) => (
    <View style={styles.section}>
      <Text style={styles.sectionTitle}>{title}</Text>
      <FlatList
        data={data}
        keyExtractor={(item) => item.id.toString()}
        renderItem={renderItem}
        horizontal
        showsHorizontalScrollIndicator={false}
      />
    </View>
  );

  if (loading) {
    return (
      <View style={styles.loader}>
        <ActivityIndicator size="large" color="#00FFFF" />
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.header}>Discover</Text>
      {renderSection('Trending Now', trending)}
      {renderSection('Popular', popular)}
      {renderSection('Top Rated', topRated)}
      {Object.keys(genreResults).map((genre) =>
        renderSection(genre, genreResults[genre])
      )}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#000', padding: 20 },
  header: {
    color: '#00FFFF',
    fontSize: 26,
    fontWeight: 'bold',
    marginBottom: 15,
  },
  section: { marginBottom: 25 },
  sectionTitle: {
    color: '#fff',
    fontSize: 20,
    fontWeight: '600',
    marginBottom: 8,
  },
  card: {
    marginRight: 10,
  },
  poster: {
    width: 120,
    height: 180,
    borderRadius: 8,
    backgroundColor: '#222',
  },
  loader: { flex: 1, justifyContent: 'center', alignItems: 'center' },
});

