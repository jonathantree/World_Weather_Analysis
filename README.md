# World Weather Analysis

## Overview of Project
The purpose of this project was to create inteactive maps using the Google Maps, Places, and Directions APIs to provide a travel itineraries based on a user provided parameter of the temperature range they desired to spend their vacation in. 

Resources: 
API keys gor Google API and OpenWeatherMap API
Software: 
- citipy 0.0.5
- pandas 1.3.5
- requests 2.27.0
- gmaps 0.9.0
- numpy 1.20.3


## Project Flow:

### 1. Simulate coordinates dataset and retrieve the cities near those locations using the citipy API and the weather in those cities using the OpenWeatherMap API
```python
# Import the dependencies.
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
from citipy import citipy
import requests
import os
import time
from datetime import datetime

# Import the API key.
from config import weather_api_key

# Create a set of random latitude and longitude combinations.
lats = np.random.uniform(low=-90.000, high=90.000, size=2000)
lngs = np.random.uniform(low=-180.000, high=180.000, size=2000)
lat_lngs = zip(lats, lngs)
# Add the latitudes and longitudes to a list.
coordinates = list(lat_lngs)

# Create a list for holding the cities.
cities = []
# Identify the nearest city for each latitude and longitude combination.
for coordinate in coordinates:
    city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name

    # If the city is unique, then we will add it to the cities list.
    if city not in cities:
        cities.append(city)
# Print the city count to confirm sufficient count.
len(cities)
```




    734




```python
# Starting URL for Weather Map API Call.
url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=" + weather_api_key

# Create an empty list to hold the weather data.
city_data = []
# Print the beginning of the logging.
print("Beginning Data Retrieval     ")
print("-----------------------------")

# Create counters.
record_count = 1
set_count = 1

# Loop through all the cities in the list.
for i, city in enumerate(cities):

    # Group cities in sets of 50 for logging purposes.
    if (i % 50 == 0 and i >= 50):
        set_count += 1
        record_count = 1
        time.sleep(60)

    # Create endpoint URL with each city.
    city_url = url + "&q=" + city.replace(" ","+")

    # Log the URL, record, and set numbers and the city.
    print(f"Processing Record {record_count} of Set {set_count} | {city}")
    # Add 1 to the record count.
    record_count += 1
# Run an API request for each of the cities.
    try:
        # Parse the JSON and retrieve data.
        city_weather = requests.get(city_url).json()
        # Parse out the needed data.
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
        weather_desc = city_weather["weather"][0]["description"]
        # Convert the date to ISO standard.
        city_date = datetime.utcfromtimestamp(city_weather["dt"]).strftime('%Y-%m-%d %H:%M:%S')
        # Append the city information into city_data list.
        city_data.append({"City": city.title(),
                          "Lat": city_lat,
                          "Lng": city_lng,
                          "Max Temp": city_max_temp,
                          "Humidity": city_humidity,
                          "Cloudiness": city_clouds,
                          "Wind Speed": city_wind,
                          "Country": city_country,
                          "Date": city_date,
                          "Current Weather" : weather_desc
                         
                         })

# If an error is experienced, skip the city.
    except:
        print("City not found. Skipping...")
        pass


len(city_data)
```
    664

### Convert the data in the dictionary into a dataframe


```python
# Convert the array of dictionaries to a Pandas DataFrame.
city_data_df = pd.DataFrame(city_data)
city_data_df.head(10)
new_column_order = ["City",
                    "Country",
                    "Date",
                    "Lat",
                    "Lng",
                    "Max Temp",
                    "Humidity",
                    "Cloudiness",
                    "Wind Speed",
                    "Current Weather"]
city_data_df=city_data_df[new_column_order]
city_data_df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Country</th>
      <th>Date</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Max Temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Current Weather</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tuatapere</td>
      <td>NZ</td>
      <td>2022-01-27 01:15:20</td>
      <td>-46.1333</td>
      <td>167.6833</td>
      <td>58.82</td>
      <td>50</td>
      <td>20</td>
      <td>9.10</td>
      <td>few clouds</td>
    </tr>
    <tr>
      <th>1</th>
      <td>East London</td>
      <td>ZA</td>
      <td>2022-01-27 01:11:47</td>
      <td>-33.0153</td>
      <td>27.9116</td>
      <td>72.97</td>
      <td>92</td>
      <td>26</td>
      <td>2.84</td>
      <td>scattered clouds</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Dicabisagan</td>
      <td>PH</td>
      <td>2022-01-27 01:15:22</td>
      <td>17.0818</td>
      <td>122.4157</td>
      <td>76.71</td>
      <td>91</td>
      <td>100</td>
      <td>3.20</td>
      <td>moderate rain</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Punta Arenas</td>
      <td>CL</td>
      <td>2022-01-27 01:15:23</td>
      <td>-53.1500</td>
      <td>-70.9167</td>
      <td>50.11</td>
      <td>76</td>
      <td>40</td>
      <td>28.77</td>
      <td>scattered clouds</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Lagos</td>
      <td>NG</td>
      <td>2022-01-27 01:12:04</td>
      <td>6.5833</td>
      <td>3.7500</td>
      <td>71.28</td>
      <td>95</td>
      <td>0</td>
      <td>0.89</td>
      <td>clear sky</td>
    </tr>
  </tbody>
</table>
</div>

```python
# Create the output file (CSV).
output_data_file = "WeatherPy_Database.csv"
# Export the City_Data into a CSV.
city_data_df.to_csv(output_data_file, index_label="City_ID")
```
### Step 2. Use input statements to retrieve customer weather preferences, then use those preferences to identify potential travel destinations and nearby hotels using the Google Places API. Then, show those destinations on a marker layer map with pop-up markers Using the Google Maps API and `gmmaps`.


```python
# Dependencies and Setup
import pandas as pd
import requests
import gmaps
import numpy as np

# Import API key
from config import g_key

# Configure gmaps API key
gmaps.configure(api_key=g_key)
```


```python
# 1. Import the WeatherPy_database.csv file. 
city_data_df = pd.read_csv("../Weather_Database/WeatherPy_database.csv")

# 2. Prompt the user to enter minimum and maximum temperature criteria 
# Ask the customer to add a minimum and maximum temperature value.
min_temp = float(input("What is the minimum temperature you would like for your trip? "))
max_temp = float(input("What is the maximum temperature you would like for your trip? "))
```

    What is the minimum temperature you would like for your trip? 45
    What is the maximum temperature you would like for your trip? 65
    


```python
# 3. Filter the city_data_df DataFrame using the input statements to create a new DataFrame using the loc method.
preferred_cities_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & \
                                       (city_data_df["Max Temp"] >= min_temp)]
preferred_cities_df.head(10)

```
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City_ID</th>
      <th>City</th>
      <th>Country</th>
      <th>Date</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Max Temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Current Weather</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Tuatapere</td>
      <td>NZ</td>
      <td>2022-01-27 01:15:20</td>
      <td>-46.1333</td>
      <td>167.6833</td>
      <td>58.82</td>
      <td>50</td>
      <td>20</td>
      <td>9.10</td>
      <td>few clouds</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Punta Arenas</td>
      <td>CL</td>
      <td>2022-01-27 01:15:23</td>
      <td>-53.1500</td>
      <td>-70.9167</td>
      <td>50.11</td>
      <td>76</td>
      <td>40</td>
      <td>28.77</td>
      <td>scattered clouds</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>Ushuaia</td>
      <td>AR</td>
      <td>2022-01-27 01:15:25</td>
      <td>-54.8000</td>
      <td>-68.3000</td>
      <td>49.66</td>
      <td>76</td>
      <td>40</td>
      <td>27.63</td>
      <td>scattered clouds</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>San Quintin</td>
      <td>MX</td>
      <td>2022-01-27 01:15:25</td>
      <td>30.4833</td>
      <td>-115.9500</td>
      <td>59.49</td>
      <td>67</td>
      <td>100</td>
      <td>8.63</td>
      <td>overcast clouds</td>
    </tr>
    <tr>
      <th>16</th>
      <td>16</td>
      <td>Naze</td>
      <td>JP</td>
      <td>2022-01-27 01:15:33</td>
      <td>28.3667</td>
      <td>129.4833</td>
      <td>63.52</td>
      <td>69</td>
      <td>81</td>
      <td>6.58</td>
      <td>broken clouds</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>Balkanabat</td>
      <td>TM</td>
      <td>2022-01-27 01:15:34</td>
      <td>39.5108</td>
      <td>54.3671</td>
      <td>49.42</td>
      <td>33</td>
      <td>100</td>
      <td>4.85</td>
      <td>overcast clouds</td>
    </tr>
    <tr>
      <th>19</th>
      <td>19</td>
      <td>Brae</td>
      <td>GB</td>
      <td>2022-01-27 01:15:36</td>
      <td>60.3964</td>
      <td>-1.3530</td>
      <td>46.31</td>
      <td>96</td>
      <td>100</td>
      <td>43.17</td>
      <td>overcast clouds</td>
    </tr>
    <tr>
      <th>23</th>
      <td>23</td>
      <td>Ribeira Grande</td>
      <td>PT</td>
      <td>2022-01-27 01:15:40</td>
      <td>38.5167</td>
      <td>-28.7000</td>
      <td>59.61</td>
      <td>78</td>
      <td>18</td>
      <td>14.65</td>
      <td>few clouds</td>
    </tr>
    <tr>
      <th>30</th>
      <td>30</td>
      <td>Guerrero Negro</td>
      <td>MX</td>
      <td>2022-01-27 01:15:45</td>
      <td>27.9769</td>
      <td>-114.0611</td>
      <td>64.85</td>
      <td>47</td>
      <td>100</td>
      <td>8.63</td>
      <td>overcast clouds</td>
    </tr>
    <tr>
      <th>32</th>
      <td>32</td>
      <td>Cameron Park</td>
      <td>US</td>
      <td>2022-01-27 01:13:13</td>
      <td>38.6688</td>
      <td>-120.9872</td>
      <td>58.93</td>
      <td>54</td>
      <td>0</td>
      <td>1.79</td>
      <td>clear sky</td>
    </tr>
  </tbody>
</table>
</div>

```python
# 4a. Determine if there are any empty rows.
print(preferred_cities_df.count())
```

    City_ID            125
    City               125
    Country            124
    Date               125
    Lat                125
    Lng                125
    Max Temp           125
    Humidity           125
    Cloudiness         125
    Wind Speed         125
    Current Weather    125
    dtype: int64
    


```python
# 4b. Drop any empty rows and create a new DataFrame that doesn’t have empty rows.
clean_df=preferred_cities_df.dropna()

# 5a. Create DataFrame called hotel_df to store hotel names along with city, country, max temp, and coordinates.
hotel_df = clean_df[["City", "Country", "Max Temp", "Current Weather", "Lat", "Lng"]].copy()

# 5b. Create a new column "Hotel Name"
hotel_df["Hotel Name"] = np.nan

# 6a. Set parameters to search for hotels with 5000 meters.
params = {
    "radius": 5000,
    "type": "lodging",
    "key": g_key
}

# 6b. Iterate through the hotel DataFrame.
for index, row in hotel_df.iterrows():

    # 6c. Get latitude and longitude from DataFrame
    lat = row["Lat"]
    lng = row["Lng"]
    
    # 6d. Set up the base URL for the Google Directions API to get JSON data.
    params["location"] = f"{lat},{lng}"
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"   

    # 6e. Make request and retrieve the JSON data from the search. 
    hotels = requests.get(base_url, params=params).json()   
    
    # 6f. Get the first hotel from the results and store the name, if a hotel isn't found skip the city.
    try:
        hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
    except (IndexError):
        print("Hotel not found... skipping.")   
        
```

    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    Hotel not found... skipping.
    


```python

# 7. Drop the rows where there is no Hotel Name.
clean_hotel_df=hotel_df.dropna()

# 8a. Create the output File (CSV)
output_data_file="WeatherPy_vacation.csv"
# 8b. Export the City_Data into a csv
clean_hotel_df.to_csv(output_data_file, index_label="City_ID")
```


```python
clean_hotel_df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Country</th>
      <th>Max Temp</th>
      <th>Current Weather</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Hotel Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tuatapere</td>
      <td>NZ</td>
      <td>58.82</td>
      <td>few clouds</td>
      <td>-46.1333</td>
      <td>167.6833</td>
      <td>Ron and Tony's Bed &amp; Breakfast</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Punta Arenas</td>
      <td>CL</td>
      <td>50.11</td>
      <td>scattered clouds</td>
      <td>-53.1500</td>
      <td>-70.9167</td>
      <td>Hotel Hain</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Ushuaia</td>
      <td>AR</td>
      <td>49.66</td>
      <td>scattered clouds</td>
      <td>-54.8000</td>
      <td>-68.3000</td>
      <td>Albatross Hotel</td>
    </tr>
    <tr>
      <th>7</th>
      <td>San Quintin</td>
      <td>MX</td>
      <td>59.49</td>
      <td>overcast clouds</td>
      <td>30.4833</td>
      <td>-115.9500</td>
      <td>Don Eddie's Landing</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Naze</td>
      <td>JP</td>
      <td>63.52</td>
      <td>broken clouds</td>
      <td>28.3667</td>
      <td>129.4833</td>
      <td>Shishime Hotel</td>
    </tr>
  </tbody>
</table>
</div>

```python
# 9. Using the template add city name, the country code, the weather description and maximum temperature for the city.
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Weather Description</dt><dd>{Current Weather} and {Max Temp} °F</dd>
</dl>
"""

# 10a. Get the data from each row and add it to the formatting template and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in clean_hotel_df.iterrows()]

# 10b. Get the latitude and longitude from each row and store in a new DataFrame.
locations = clean_hotel_df[["Lat", "Lng"]]

# 11a. Add a marker layer for each city to the map. 
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)

# 11b. Display the figure
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
fig.add_layer(marker_layer)
fig
```
#### *The resulting figure is an interactive map of all of the cities that fir the temperature criteria*
![png](/Vacation_Search/WeatherPy_vacation_map.png)

### Step 3. Use the Google Directions API to create a travel itinerary that shows the route between four cities chosen from the customer’s possible travel destinations. Then, create a marker layer map with a pop-up marker for each city on the itinerary.

```python
# Dependencies and Setup
import pandas as pd
import requests
import gmaps

# Import API key
from config import g_key

# Configure gmaps
gmaps.configure(api_key=g_key)
```


```python
# 1. Read the WeatherPy_vacation.csv into a DataFrame.
vacation_df = pd.read_csv("../Vacation_Search/WeatherPy_vacation.csv")
vacation_df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City_ID</th>
      <th>City</th>
      <th>Country</th>
      <th>Max Temp</th>
      <th>Current Weather</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Hotel Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Tuatapere</td>
      <td>NZ</td>
      <td>58.82</td>
      <td>few clouds</td>
      <td>-46.1333</td>
      <td>167.6833</td>
      <td>Ron and Tony's Bed &amp; Breakfast</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Punta Arenas</td>
      <td>CL</td>
      <td>50.11</td>
      <td>scattered clouds</td>
      <td>-53.1500</td>
      <td>-70.9167</td>
      <td>Hotel Hain</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>Ushuaia</td>
      <td>AR</td>
      <td>49.66</td>
      <td>scattered clouds</td>
      <td>-54.8000</td>
      <td>-68.3000</td>
      <td>Albatross Hotel</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7</td>
      <td>San Quintin</td>
      <td>MX</td>
      <td>59.49</td>
      <td>overcast clouds</td>
      <td>30.4833</td>
      <td>-115.9500</td>
      <td>Don Eddie's Landing</td>
    </tr>
    <tr>
      <th>4</th>
      <td>16</td>
      <td>Naze</td>
      <td>JP</td>
      <td>63.52</td>
      <td>broken clouds</td>
      <td>28.3667</td>
      <td>129.4833</td>
      <td>Shishime Hotel</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 2. Using the template add the city name, the country code, the weather description and maximum temperature for the city.
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Weather Description</dt><dd>{Current Weather} and {Max Temp} °F</dd>
</dl>
"""

# 3a. Get the data from each row and add it to the formatting template and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in vacation_df.iterrows()]

# 3b. Get the latitude and longitude from each row and store in a new DataFrame.
locations = vacation_df[["Lat", "Lng"]]

# 4a. Add a marker layer for each city to the map.
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)

# 4b. Display the figure
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
fig.add_layer(marker_layer)
fig

```

##### Map not displayed because it is just the same map as above but it is used within the notebook to interactively pick city names from and drop them in to the variable definitions below


```python
# From the map above pick 4 cities and create a vacation itinerary route to travel between the four cities. 
# 5. Create DataFrames for each city by filtering the 'vacation_df' using the loc method. 
# Hint: The starting and ending city should be the same city.

vacation_start = vacation_df.loc[vacation_df["City"] == "Wanaka"]
vacation_end = vacation_df.loc[vacation_df["City"] == "Wanaka"]
vacation_stop1 = vacation_df.loc[vacation_df["City"] == "Te Anau"]
vacation_stop2 = vacation_df.loc[vacation_df["City"] == "Bluff"] 
vacation_stop3 = vacation_df.loc[vacation_df["City"] == "Dunedin"] 


# 6. Get the latitude-longitude pairs as tuples from each city DataFrame using the to_numpy function and list indexing.
start = vacation_start["Lat"].to_numpy()[0], vacation_start["Lng"].to_numpy()[0]
end = vacation_end["Lat"].to_numpy()[0], vacation_end["Lng"].to_numpy()[0]
stop1 = vacation_stop1["Lat"].to_numpy()[0], vacation_stop1["Lng"].to_numpy()[0]
stop2 = vacation_stop2["Lat"].to_numpy()[0], vacation_stop2["Lng"].to_numpy()[0]
stop3 = vacation_stop3["Lat"].to_numpy()[0], vacation_stop3["Lng"].to_numpy()[0]

# 7. Create a direction layer map using the start and end latitude-longitude pairs,
# and stop1, stop2, and stop3 as the waypoints. The travel_mode should be "DRIVING", "BICYCLING", or "WALKING".
fig = gmaps.figure()
city_itinerary = gmaps.directions_layer(
        start, end, waypoints=[stop1, stop2, stop3], travel_mode='DRIVING')

fig.add_layer(city_itinerary)
fig

```

#### *Map providing the itinerary trip plan by driving*
![png](/Vacation_Itinerary/WeatherPy_travel_map.png)



```python
# 8. To create a marker layer map between the four cities.
#  Combine the four city DataFrames into one DataFrame using the concat() function.
itinerary_df = pd.concat([
    vacation_start,
    vacation_stop1,
    vacation_stop2,
    vacation_stop3
],ignore_index=True)
itinerary_df
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City_ID</th>
      <th>City</th>
      <th>Country</th>
      <th>Max Temp</th>
      <th>Current Weather</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Hotel Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>401</td>
      <td>Wanaka</td>
      <td>NZ</td>
      <td>63.10</td>
      <td>clear sky</td>
      <td>-44.7000</td>
      <td>169.1500</td>
      <td>Wanaka Homestead Lodge and Cottages</td>
    </tr>
    <tr>
      <th>1</th>
      <td>113</td>
      <td>Te Anau</td>
      <td>NZ</td>
      <td>60.73</td>
      <td>scattered clouds</td>
      <td>-45.4167</td>
      <td>167.7167</td>
      <td>Kingsgate Hotel Te Anau</td>
    </tr>
    <tr>
      <th>2</th>
      <td>91</td>
      <td>Bluff</td>
      <td>NZ</td>
      <td>58.80</td>
      <td>scattered clouds</td>
      <td>-46.6000</td>
      <td>168.3333</td>
      <td>Bluff Homestead - Guesthouse &amp; Campervan Park</td>
    </tr>
    <tr>
      <th>3</th>
      <td>95</td>
      <td>Dunedin</td>
      <td>NZ</td>
      <td>58.80</td>
      <td>broken clouds</td>
      <td>-45.8742</td>
      <td>170.5036</td>
      <td>Scenic Hotel Southern Cross</td>
    </tr>
  </tbody>
</table>
</div>

```python
# 9 Using the template add city name, the country code, the weather description and maximum temperature for the city. 
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Weather Description</dt><dd>{Current Weather} and {Max Temp} °F</dd>
</dl>
"""

# 10a Get the data from each row and add it to the formatting template and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in itinerary_df.iterrows()]

# 10b. Get the latitude and longitude from each row and store in a new DataFrame.
locations = itinerary_df[["Lat", "Lng"]]

# 11a. Add a marker layer for each city to the map.
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)

# 11b. Display the figure
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
fig.add_layer(marker_layer)
fig

```

#### *Map showing the hotels at each waypoint along the itinerary*
![png](/Vacation_Itinerary/WeatherPy_map_markers.png)





