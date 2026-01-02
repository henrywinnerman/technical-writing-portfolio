# Building a Weather App with React and OpenWeatherMap API

A hands-on tutorial for fetching and displaying real-time weather data in React.

## What You'll Build

By the end of this tutorial, you'll have a fully functional weather app that:
- Fetches real-time weather data from an API
- Searches for weather by city name
- Displays current temperature, conditions, and details
- Shows a 5-day forecast
- Handles loading and error states gracefully

**Live demo preview:**

```
┌─────────────────────────────────────────────────────────────┐
│                    🌤️ Weather App                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 🔍  Enter city name...                    [Search]  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│           ☀️                                                │
│          Lagos, NG                                          │
│           32°C                                              │
│        Clear Sky                                            │
│                                                             │
│  ┌─────────┬─────────┬─────────┬─────────┐                 │
│  │ 💧 78%  │ 💨 12km/h│ 👁️ 10km │ 🌡️ 34°  │                 │
│  │Humidity │  Wind   │Visibility│ Feels   │                 │
│  └─────────┴─────────┴─────────┴─────────┘                 │
│                                                             │
│  5-Day Forecast                                             │
│  ┌─────┬─────┬─────┬─────┬─────┐                           │
│  │ Mon │ Tue │ Wed │ Thu │ Fri │                           │
│  │ ☀️  │ ⛅  │ 🌧️  │ ⛅  │ ☀️  │                           │
│  │ 31° │ 29° │ 27° │ 28° │ 30° │                           │
│  └─────┴─────┴─────┴─────┴─────┘                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

Before starting, you'll need:
- **Node.js** (version 14 or higher) installed
- Basic knowledge of React (components, props, state)
- Basic understanding of JavaScript (async/await, fetch)
- A code editor (VS Code recommended)

**Time to complete:** 45-60 minutes

---

## Table of Contents

1. [Getting Your API Key](#step-1-getting-your-api-key)
2. [Setting Up the Project](#step-2-setting-up-the-project)
3. [Understanding the API Response](#step-3-understanding-the-api-response)
4. [Building the Search Component](#step-4-building-the-search-component)
5. [Creating the Weather Display](#step-5-creating-the-weather-display)
6. [Fetching Weather Data](#step-6-fetching-weather-data)
7. [Adding the 5-Day Forecast](#step-7-adding-the-5-day-forecast)
8. [Handling Loading and Errors](#step-8-handling-loading-and-errors)
9. [Styling the App](#step-9-styling-the-app)
10. [Adding Extra Features](#step-10-adding-extra-features)

---

## Step 1: Getting Your API Key

We'll use the OpenWeatherMap API — it's free and reliable.

### 1.1 Create an Account

1. Go to [OpenWeatherMap](https://openweathermap.org/)
2. Click **Sign In** → **Create an Account**
3. Fill in your details and verify your email

### 1.2 Get Your API Key

1. Log in to your account
2. Go to **API Keys** (in the dropdown under your username)
3. You'll see a default key, or click **Generate** to create a new one
4. Copy your API key

Your key looks like this:
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

> **Note:** New API keys may take up to 2 hours to activate. If you get "Invalid API key" errors, wait and try again.

### 1.3 Understanding the Free Tier

| Feature | Free Tier Limit |
|---------|-----------------|
| API Calls | 1,000 calls/day |
| Current Weather | ✅ Included |
| 5-Day Forecast | ✅ Included |
| Historical Data | ❌ Paid only |

This is plenty for development and personal projects.

---

## Step 2: Setting Up the Project

Let's create a new React application.

### 2.1 Create the Project

Open your terminal and run:

```bash
npx create-react-app weather-app
cd weather-app
```

Or with Vite (faster):

```bash
npm create vite@latest weather-app -- --template react
cd weather-app
npm install
```

### 2.2 Project Structure

We'll organize our code like this:

```
weather-app/
├── src/
│   ├── components/
│   │   ├── SearchBar.jsx
│   │   ├── WeatherCard.jsx
│   │   ├── WeatherDetails.jsx
│   │   └── Forecast.jsx
│   ├── App.jsx
│   ├── App.css
│   └── index.js
├── .env
└── package.json
```

### 2.3 Store Your API Key Safely

Create a `.env` file in your project root:

```bash
REACT_APP_WEATHER_API_KEY=your_api_key_here
```

For Vite projects, use:
```bash
VITE_WEATHER_API_KEY=your_api_key_here
```

> **Important:** Add `.env` to your `.gitignore` file to keep your API key private.

### 2.4 Create the Components Folder

```bash
mkdir src/components
```

### 2.5 Start the Development Server

```bash
npm start
```

Visit `http://localhost:3000` to see your app.

---

## Step 3: Understanding the API Response

Before coding, let's understand what data we'll receive.

### 3.1 Current Weather Endpoint

**URL:**
```
https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric
```

**Example Response:**

```json
{
  "coord": { "lon": 3.3958, "lat": 6.4541 },
  "weather": [
    {
      "id": 801,
      "main": "Clouds",
      "description": "few clouds",
      "icon": "02d"
    }
  ],
  "main": {
    "temp": 31.5,
    "feels_like": 34.2,
    "temp_min": 30.1,
    "temp_max": 32.8,
    "pressure": 1012,
    "humidity": 70
  },
  "visibility": 10000,
  "wind": { "speed": 3.6, "deg": 220 },
  "clouds": { "all": 20 },
  "dt": 1704067200,
  "sys": {
    "country": "NG",
    "sunrise": 1704088800,
    "sunset": 1704130800
  },
  "timezone": 3600,
  "id": 2332459,
  "name": "Lagos",
  "cod": 200
}
```

### 3.2 Key Data Points

| Field | Path | Description |
|-------|------|-------------|
| City Name | `name` | City name |
| Country | `sys.country` | Country code |
| Temperature | `main.temp` | Current temp (°C with `units=metric`) |
| Feels Like | `main.feels_like` | "Feels like" temperature |
| Description | `weather[0].description` | Weather description |
| Icon Code | `weather[0].icon` | Icon identifier |
| Humidity | `main.humidity` | Humidity percentage |
| Wind Speed | `wind.speed` | Wind speed (m/s) |
| Visibility | `visibility` | Visibility in meters |

### 3.3 Weather Icons

OpenWeatherMap provides icons. Use this URL pattern:

```
https://openweathermap.org/img/wn/{icon_code}@2x.png
```

Example: `https://openweathermap.org/img/wn/02d@2x.png`

---

## Step 4: Building the Search Component

Let's create a search bar for entering city names.

### 4.1 Create SearchBar.jsx

Create `src/components/SearchBar.jsx`:

```jsx
import React, { useState } from 'react';

const SearchBar = ({ onSearch }) => {
  const [city, setCity] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (city.trim()) {
      onSearch(city.trim());
      setCity('');
    }
  };

  return (
    <form className="search-bar" onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="Enter city name..."
        value={city}
        onChange={(e) => setCity(e.target.value)}
        className="search-input"
      />
      <button type="submit" className="search-button">
        Search
      </button>
    </form>
  );
};

export default SearchBar;
```

### 4.2 Understanding the Component

| Part | Purpose |
|------|---------|
| `useState` | Tracks the input value |
| `onSearch` prop | Function passed from parent to handle the search |
| `handleSubmit` | Prevents page reload, validates input, calls parent |
| `trim()` | Removes whitespace from input |

---

## Step 5: Creating the Weather Display

Now let's display the current weather.

### 5.1 Create WeatherCard.jsx

Create `src/components/WeatherCard.jsx`:

```jsx
import React from 'react';

const WeatherCard = ({ weather }) => {
  if (!weather) return null;

  const {
    name,
    sys: { country },
    main: { temp, feels_like, humidity },
    weather: weatherInfo,
    wind: { speed },
    visibility,
  } = weather;

  const { description, icon } = weatherInfo[0];
  const iconUrl = `https://openweathermap.org/img/wn/${icon}@4x.png`;

  return (
    <div className="weather-card">
      {/* Main Weather Info */}
      <div className="weather-main">
        <img src={iconUrl} alt={description} className="weather-icon" />
        <div className="weather-info">
          <h2 className="city-name">
            {name}, {country}
          </h2>
          <p className="temperature">{Math.round(temp)}°C</p>
          <p className="description">{description}</p>
        </div>
      </div>

      {/* Weather Details */}
      <div className="weather-details">
        <div className="detail-item">
          <span className="detail-icon">💧</span>
          <span className="detail-value">{humidity}%</span>
          <span className="detail-label">Humidity</span>
        </div>
        <div className="detail-item">
          <span className="detail-icon">💨</span>
          <span className="detail-value">{speed} m/s</span>
          <span className="detail-label">Wind</span>
        </div>
        <div className="detail-item">
          <span className="detail-icon">👁️</span>
          <span className="detail-value">{(visibility / 1000).toFixed(1)} km</span>
          <span className="detail-label">Visibility</span>
        </div>
        <div className="detail-item">
          <span className="detail-icon">🌡️</span>
          <span className="detail-value">{Math.round(feels_like)}°C</span>
          <span className="detail-label">Feels Like</span>
        </div>
      </div>
    </div>
  );
};

export default WeatherCard;
```

### 5.2 Key Concepts

**Destructuring:** We extract nested values from the API response for cleaner code.

```jsx
// Instead of:
weather.main.temp
weather.weather[0].description

// We use:
const { main: { temp }, weather: weatherInfo } = weather;
const { description } = weatherInfo[0];
```

**Conditional Rendering:** `if (!weather) return null;` prevents errors when no data is loaded yet.

---

## Step 6: Fetching Weather Data

Now let's connect everything in App.jsx.

### 6.1 Update App.jsx

Replace `src/App.jsx`:

```jsx
import React, { useState } from 'react';
import SearchBar from './components/SearchBar';
import WeatherCard from './components/WeatherCard';
import './App.css';

function App() {
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const API_KEY = process.env.REACT_APP_WEATHER_API_KEY;
  // For Vite: const API_KEY = import.meta.env.VITE_WEATHER_API_KEY;

  const fetchWeather = async (city) => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
      );

      if (!response.ok) {
        throw new Error(
          response.status === 404
            ? 'City not found. Please check the spelling.'
            : 'Failed to fetch weather data.'
        );
      }

      const data = await response.json();
      setWeather(data);
    } catch (err) {
      setError(err.message);
      setWeather(null);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="app">
      <h1 className="app-title">🌤️ Weather App</h1>
      
      <SearchBar onSearch={fetchWeather} />

      {loading && <p className="loading">Loading...</p>}
      
      {error && <p className="error">{error}</p>}
      
      {weather && !loading && <WeatherCard weather={weather} />}
    </div>
  );
}

export default App;
```

### 6.2 Understanding the Fetch Logic

```jsx
const fetchWeather = async (city) => {
  setLoading(true);     // Show loading state
  setError(null);       // Clear previous errors

  try {
    const response = await fetch(url);
    
    if (!response.ok) {
      throw new Error('...');  // Handle HTTP errors
    }

    const data = await response.json();
    setWeather(data);   // Store the weather data
    
  } catch (err) {
    setError(err.message);  // Store error message
    setWeather(null);       // Clear old weather data
    
  } finally {
    setLoading(false);  // Always hide loading
  }
};
```

**Flow:**
1. User searches for "Lagos"
2. `fetchWeather("Lagos")` is called
3. Loading spinner appears
4. API request is made
5. On success: weather data is stored and displayed
6. On error: error message is shown
7. Loading spinner disappears

---

## Step 7: Adding the 5-Day Forecast

Let's add a forecast section using a different API endpoint.

### 7.1 Create Forecast.jsx

Create `src/components/Forecast.jsx`:

```jsx
import React from 'react';

const Forecast = ({ forecast }) => {
  if (!forecast || !forecast.list) return null;

  // Get one forecast per day (API returns 8 forecasts per day, every 3 hours)
  const dailyForecasts = forecast.list.filter((item, index) => index % 8 === 0).slice(0, 5);

  const getDayName = (dateString) => {
    const date = new Date(dateString);
    return date.toLocaleDateString('en-US', { weekday: 'short' });
  };

  return (
    <div className="forecast">
      <h3 className="forecast-title">5-Day Forecast</h3>
      <div className="forecast-list">
        {dailyForecasts.map((day, index) => {
          const iconUrl = `https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`;
          
          return (
            <div key={index} className="forecast-item">
              <p className="forecast-day">{getDayName(day.dt_txt)}</p>
              <img src={iconUrl} alt={day.weather[0].description} />
              <p className="forecast-temp">{Math.round(day.main.temp)}°C</p>
            </div>
          );
        })}
      </div>
    </div>
  );
};

export default Forecast;
```

### 7.2 Update App.jsx to Fetch Forecast

Add forecast state and fetching:

```jsx
import React, { useState } from 'react';
import SearchBar from './components/SearchBar';
import WeatherCard from './components/WeatherCard';
import Forecast from './components/Forecast';
import './App.css';

function App() {
  const [weather, setWeather] = useState(null);
  const [forecast, setForecast] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const API_KEY = process.env.REACT_APP_WEATHER_API_KEY;

  const fetchWeather = async (city) => {
    setLoading(true);
    setError(null);

    try {
      // Fetch current weather
      const weatherResponse = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
      );

      if (!weatherResponse.ok) {
        throw new Error(
          weatherResponse.status === 404
            ? 'City not found. Please check the spelling.'
            : 'Failed to fetch weather data.'
        );
      }

      const weatherData = await weatherResponse.json();
      setWeather(weatherData);

      // Fetch 5-day forecast
      const forecastResponse = await fetch(
        `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${API_KEY}&units=metric`
      );

      if (forecastResponse.ok) {
        const forecastData = await forecastResponse.json();
        setForecast(forecastData);
      }

    } catch (err) {
      setError(err.message);
      setWeather(null);
      setForecast(null);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="app">
      <h1 className="app-title">🌤️ Weather App</h1>
      
      <SearchBar onSearch={fetchWeather} />

      {loading && <p className="loading">Loading...</p>}
      
      {error && <p className="error">{error}</p>}
      
      {weather && !loading && (
        <>
          <WeatherCard weather={weather} />
          <Forecast forecast={forecast} />
        </>
      )}
    </div>
  );
}

export default App;
```

---

## Step 8: Handling Loading and Errors

Good UX requires proper feedback during loading and errors.

### 8.1 Create a Loading Spinner

Add to your CSS (we'll cover this in Step 9), but here's a simple loading component:

```jsx
// In App.jsx, replace the loading paragraph:

{loading && (
  <div className="loading">
    <div className="spinner"></div>
    <p>Fetching weather data...</p>
  </div>
)}
```

### 8.2 Error States to Handle

| Error | Cause | User Message |
|-------|-------|--------------|
| 404 | City not found | "City not found. Please check the spelling." |
| 401 | Invalid API key | "API configuration error. Please try again later." |
| Network Error | No internet | "Network error. Please check your connection." |
| 429 | Rate limit | "Too many requests. Please wait a moment." |

### 8.3 Enhanced Error Handling

```jsx
const fetchWeather = async (city) => {
  setLoading(true);
  setError(null);

  try {
    const response = await fetch(url);

    if (!response.ok) {
      const errorMessages = {
        404: 'City not found. Please check the spelling.',
        401: 'API key error. Please check configuration.',
        429: 'Too many requests. Please wait a moment.',
      };
      
      throw new Error(errorMessages[response.status] || 'Failed to fetch weather data.');
    }

    const data = await response.json();
    setWeather(data);
    
  } catch (err) {
    // Handle network errors
    if (err.name === 'TypeError') {
      setError('Network error. Please check your internet connection.');
    } else {
      setError(err.message);
    }
    setWeather(null);
  } finally {
    setLoading(false);
  }
};
```

---

## Step 9: Styling the App

Let's make it look professional with CSS.

### 9.1 Replace App.css

Replace `src/App.css`:

```css
/* Base Styles */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
}

.app {
  max-width: 600px;
  margin: 0 auto;
}

.app-title {
  text-align: center;
  color: white;
  font-size: 2rem;
  margin-bottom: 24px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

/* Search Bar */
.search-bar {
  display: flex;
  gap: 8px;
  margin-bottom: 24px;
}

.search-input {
  flex: 1;
  padding: 14px 18px;
  font-size: 16px;
  border: none;
  border-radius: 12px;
  outline: none;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.search-input:focus {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.search-button {
  padding: 14px 24px;
  background: #fff;
  border: none;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  color: #667eea;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.search-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

/* Weather Card */
.weather-card {
  background: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
  margin-bottom: 20px;
}

.weather-main {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  margin-bottom: 30px;
}

.weather-icon {
  width: 120px;
  height: 120px;
}

.weather-info {
  text-align: left;
}

.city-name {
  font-size: 1.5rem;
  color: #333;
  margin-bottom: 8px;
}

.temperature {
  font-size: 3.5rem;
  font-weight: 700;
  color: #667eea;
  line-height: 1;
  margin-bottom: 8px;
}

.description {
  font-size: 1.2rem;
  color: #666;
  text-transform: capitalize;
}

/* Weather Details */
.weather-details {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.detail-item {
  text-align: center;
}

.detail-icon {
  font-size: 1.5rem;
  display: block;
  margin-bottom: 5px;
}

.detail-value {
  font-size: 1.1rem;
  font-weight: 600;
  color: #333;
  display: block;
}

.detail-label {
  font-size: 0.85rem;
  color: #888;
}

/* Forecast */
.forecast {
  background: white;
  border-radius: 20px;
  padding: 25px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
}

.forecast-title {
  font-size: 1.2rem;
  color: #333;
  margin-bottom: 20px;
  text-align: center;
}

.forecast-list {
  display: flex;
  justify-content: space-between;
}

.forecast-item {
  text-align: center;
  padding: 10px;
}

.forecast-day {
  font-weight: 600;
  color: #555;
  margin-bottom: 5px;
}

.forecast-item img {
  width: 50px;
  height: 50px;
}

.forecast-temp {
  font-weight: 600;
  color: #667eea;
}

/* Loading & Error States */
.loading {
  text-align: center;
  color: white;
  padding: 40px;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  margin: 0 auto 15px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.error {
  background: #fee2e2;
  color: #dc2626;
  padding: 15px 20px;
  border-radius: 12px;
  text-align: center;
  margin-bottom: 20px;
}

/* Responsive Design */
@media (max-width: 500px) {
  .weather-main {
    flex-direction: column;
    text-align: center;
  }

  .weather-info {
    text-align: center;
  }

  .weather-details {
    grid-template-columns: repeat(2, 1fr);
  }

  .forecast-list {
    flex-wrap: wrap;
    gap: 10px;
    justify-content: center;
  }

  .forecast-item {
    width: calc(33% - 10px);
  }
}
```

---

## Step 10: Adding Extra Features

Enhance your app with these optional improvements.

### 10.1 Default City on Load

Load weather for a default city when the app starts:

```jsx
// In App.jsx, add useEffect:
import React, { useState, useEffect } from 'react';

// Inside the App function:
useEffect(() => {
  fetchWeather('Lagos'); // Default city
}, []);
```

### 10.2 Temperature Unit Toggle

Add Celsius/Fahrenheit switching:

```jsx
const [unit, setUnit] = useState('metric'); // 'metric' or 'imperial'

// In the fetch URL:
`...&units=${unit}`

// Add a toggle button:
<button onClick={() => setUnit(unit === 'metric' ? 'imperial' : 'metric')}>
  Switch to {unit === 'metric' ? '°F' : '°C'}
</button>
```

### 10.3 Geolocation

Get weather for user's current location:

```jsx
const getLocationWeather = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        fetchWeatherByCoords(latitude, longitude);
      },
      (error) => {
        setError('Unable to get your location.');
      }
    );
  }
};

const fetchWeatherByCoords = async (lat, lon) => {
  const response = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`
  );
  // ... handle response
};
```

### 10.4 Recent Searches

Store and display recent city searches:

```jsx
const [recentSearches, setRecentSearches] = useState([]);

// When searching:
const handleSearch = (city) => {
  fetchWeather(city);
  setRecentSearches(prev => {
    const updated = [city, ...prev.filter(c => c !== city)].slice(0, 5);
    return updated;
  });
};

// Display as clickable chips:
<div className="recent-searches">
  {recentSearches.map(city => (
    <button key={city} onClick={() => fetchWeather(city)}>
      {city}
    </button>
  ))}
</div>
```

---

## Complete Project Structure

Your final project should look like this:

```
weather-app/
├── src/
│   ├── components/
│   │   ├── SearchBar.jsx
│   │   ├── WeatherCard.jsx
│   │   └── Forecast.jsx
│   ├── App.jsx
│   ├── App.css
│   └── index.js
├── .env
├── .gitignore
└── package.json
```

---

## Troubleshooting

### "Invalid API key"

- New keys take up to 2 hours to activate
- Check for typos in your `.env` file
- Restart your development server after adding `.env`

### "City not found"

- Check spelling
- Try adding country code: "Lagos,NG"
- Some small cities may not be in the database

### Environment variable not working

- File must be named exactly `.env` (not `.env.local` for Create React App)
- Variable must start with `REACT_APP_` (Create React App) or `VITE_` (Vite)
- Restart the development server

### CORS errors

- This shouldn't happen with OpenWeatherMap
- If it does, you may need a proxy in development

---

## Summary

You've built a complete weather app that:

✅ Fetches real-time data from an external API
✅ Handles user input with controlled components
✅ Manages loading and error states
✅ Displays current weather and 5-day forecast
✅ Uses responsive CSS for mobile devices
✅ Securely stores API keys with environment variables

**Skills practiced:**
- React hooks (useState, useEffect)
- Async/await and fetch API
- Conditional rendering
- Component composition
- CSS Grid and Flexbox
- Error handling

---

## Resources

- [OpenWeatherMap API Docs](https://openweathermap.org/api)
- [React Documentation](https://react.dev/)
- [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

---

*Last updated: January 2026*
