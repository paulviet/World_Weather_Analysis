```python
# Import the dependencies.
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# Import the requests library.
import requests
import time
```


```python
# Import the datetime module from the datetime library.
from datetime import datetime

# Import the API key and lat-lon to City Name CitiPy
import sys
sys.path.append("../")
from config import weather_api_key
from citipy import citipy

# Import linear regression from the SciPy stats module.
from scipy.stats import linregress
```


```python
# Starting URL for Weather Map API Call.
#setting value for number of calls, (to check code with smaller set and avoid hitting API limitations)
NUM_CALLS=2000

params = {"units": "Imperial",
    "APPID": weather_api_key,
    "q": "Boston"} #q is city
#params
#base_url = "https://jsonplaceholder.typicode.com/" #Dummy placeholder, exceeded limit for a 60 calls a minute
base_url = "http://api.openweathermap.org/data/2.5/weather"
```


```python
# Create a set of random latitude and longitude combinations.
lats = np.random.uniform(low=-90.000, high=90.000, size=NUM_CALLS)
lngs = np.random.uniform(low=-180.000, high=180.000, size=NUM_CALLS)
lat_lngs = zip(lats, lngs)
lat_lngs
```




    <zip at 0x1d1c6988208>



Latitude and longitude


```python
# Add the latitudes and longitudes to a list.
coordinates = list(lat_lngs)
#coordinates
# Create a list for holding the cities.
cities = []
failed_cities = []
# Identify the nearest city for each latitude and longitude combination.
for coordinate in coordinates:
    city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name

    # If the city is unique, then we will add it to the cities list.
    if city not in cities:
        cities.append(city)
# Print the city count to confirm sufficient count.
len(cities)
```




    761




```python
# Create an empty list to hold the weather data.
city_data = []
skipped_cities = []
# Print the beginning of the logging.
print("Beginning Data Retrieval     ")
print("-----------------------------")

# Create counters.
record_count = 1
set_count = 1
```

    Beginning Data Retrieval     
    -----------------------------
    


```python
# Loop through all the cities in the list.
for i, city in enumerate(cities):
    if (i == 0):
        print(f"Processing Record {record_count} of Set {set_count} | {city}")

    # Group cities in sets of 50 for logging purposes.
    if (i % 50 == 0 and i >= 50):
        set_count += 1
        record_count = 1
        print(f"Processing Record {record_count} of Set {set_count} | {city}")
        time.sleep(60) #Need to sleep 60 seconds so calls don't stop
    # Create endpoint URL with each city.
    #city_url = base_url + "&q=" + city.replace(" ","+") needing + is deprecated, using dictionary of params
    params['q'] = city
    
    # Log the URL, record, and set numbers and the city.
    #print(f"Processing Record {record_count} of Set {set_count} | {city}")
    # Add 1 to the record count.
    record_count += 1
    # Run an API request for each of the cities.
    try:
        # Parse the JSON and retrieve data.
        city_weather = requests.get(base_url, params).json()
        # Parse out the needed data.
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
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
            "Date": city_date})

    # If an error is experienced, skip the city.
    except:
        #print(f"{city.title()} not found. Skipping...")
        skipped_cities.append({"City": city.title(),
            "Coords": coordinates[(set_count-1)*10+record_count-1]})
        pass #We are catching errors, commenting out pass!

    # Indicate that Data Loading is complete.
    # print("-----------------------------")
    # print("Data Retrieval Complete      ")
    # print("-----------------------------")
```

    Processing Record 1 of Set 1 | hilo
    Processing Record 1 of Set 2 | havre-saint-pierre
    Processing Record 1 of Set 3 | mys shmidta
    Processing Record 1 of Set 4 | huntsville
    Processing Record 1 of Set 5 | tonj
    Processing Record 1 of Set 6 | iquique
    Processing Record 1 of Set 7 | troitsko-pechorsk
    Processing Record 1 of Set 8 | gravelbourg
    Processing Record 1 of Set 9 | sinnamary
    Processing Record 1 of Set 10 | malanje
    Processing Record 1 of Set 11 | kashi
    Processing Record 1 of Set 12 | haibowan
    Processing Record 1 of Set 13 | eskasem
    Processing Record 1 of Set 14 | khonuu
    Processing Record 1 of Set 15 | geresk
    Processing Record 1 of Set 16 | enkoping
    


```python
# Convert the array of dictionaries to a Pandas DataFrame.
city_data_df = pd.DataFrame(city_data)
city_data_df.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Max Temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Country</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hilo</td>
      <td>19.7297</td>
      <td>-155.0900</td>
      <td>73.40</td>
      <td>73</td>
      <td>90</td>
      <td>5.75</td>
      <td>US</td>
      <td>2021-03-28 21:25:59</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Georgetown</td>
      <td>5.4112</td>
      <td>100.3354</td>
      <td>80.01</td>
      <td>94</td>
      <td>20</td>
      <td>4.27</td>
      <td>MY</td>
      <td>2021-03-28 21:29:33</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Saint-Pierre</td>
      <td>-21.3393</td>
      <td>55.4781</td>
      <td>77.00</td>
      <td>78</td>
      <td>0</td>
      <td>6.91</td>
      <td>RE</td>
      <td>2021-03-28 21:29:33</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bilibino</td>
      <td>68.0546</td>
      <td>166.4372</td>
      <td>-15.12</td>
      <td>92</td>
      <td>59</td>
      <td>1.32</td>
      <td>RU</td>
      <td>2021-03-28 21:26:10</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Borovskoy</td>
      <td>53.8000</td>
      <td>64.1500</td>
      <td>20.71</td>
      <td>90</td>
      <td>32</td>
      <td>11.27</td>
      <td>KZ</td>
      <td>2021-03-28 21:29:33</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Isangel</td>
      <td>-19.5500</td>
      <td>169.2667</td>
      <td>78.80</td>
      <td>89</td>
      <td>40</td>
      <td>6.08</td>
      <td>VU</td>
      <td>2021-03-28 21:19:51</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Hithadhoo</td>
      <td>-0.6000</td>
      <td>73.0833</td>
      <td>80.04</td>
      <td>82</td>
      <td>100</td>
      <td>31.34</td>
      <td>MV</td>
      <td>2021-03-28 21:29:33</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Arman</td>
      <td>59.7000</td>
      <td>150.1667</td>
      <td>1.11</td>
      <td>94</td>
      <td>43</td>
      <td>4.94</td>
      <td>RU</td>
      <td>2021-03-28 21:29:34</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Honavar</td>
      <td>14.2833</td>
      <td>74.4500</td>
      <td>80.49</td>
      <td>85</td>
      <td>88</td>
      <td>1.63</td>
      <td>IN</td>
      <td>2021-03-28 21:29:34</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Port Alfred</td>
      <td>-33.5906</td>
      <td>26.8910</td>
      <td>65.66</td>
      <td>75</td>
      <td>100</td>
      <td>19.24</td>
      <td>ZA</td>
      <td>2021-03-28 21:29:34</td>
    </tr>
  </tbody>
</table>
</div>




```python
new_column_order = ["City", "Cloudiness", "Country", "Date", "Humidity", "Lat", "Lng", "Max Temp", "Wind Speed"]
city_data_df = city_data_df[new_column_order]
city_data_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Cloudiness</th>
      <th>Country</th>
      <th>Date</th>
      <th>Humidity</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Max Temp</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hilo</td>
      <td>90</td>
      <td>US</td>
      <td>2021-03-28 21:25:59</td>
      <td>73</td>
      <td>19.7297</td>
      <td>-155.0900</td>
      <td>73.40</td>
      <td>5.75</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Georgetown</td>
      <td>20</td>
      <td>MY</td>
      <td>2021-03-28 21:29:33</td>
      <td>94</td>
      <td>5.4112</td>
      <td>100.3354</td>
      <td>80.01</td>
      <td>4.27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Saint-Pierre</td>
      <td>0</td>
      <td>RE</td>
      <td>2021-03-28 21:29:33</td>
      <td>78</td>
      <td>-21.3393</td>
      <td>55.4781</td>
      <td>77.00</td>
      <td>6.91</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bilibino</td>
      <td>59</td>
      <td>RU</td>
      <td>2021-03-28 21:26:10</td>
      <td>92</td>
      <td>68.0546</td>
      <td>166.4372</td>
      <td>-15.12</td>
      <td>1.32</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Borovskoy</td>
      <td>32</td>
      <td>KZ</td>
      <td>2021-03-28 21:29:33</td>
      <td>90</td>
      <td>53.8000</td>
      <td>64.1500</td>
      <td>20.71</td>
      <td>11.27</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>703</th>
      <td>Coxim</td>
      <td>4</td>
      <td>BR</td>
      <td>2021-03-28 21:47:06</td>
      <td>35</td>
      <td>-18.5067</td>
      <td>-54.7600</td>
      <td>82.54</td>
      <td>6.40</td>
    </tr>
    <tr>
      <th>704</th>
      <td>Yeovil</td>
      <td>90</td>
      <td>GB</td>
      <td>2021-03-28 21:47:06</td>
      <td>81</td>
      <td>50.9416</td>
      <td>-2.6321</td>
      <td>50.00</td>
      <td>13.80</td>
    </tr>
    <tr>
      <th>705</th>
      <td>Beringovskiy</td>
      <td>100</td>
      <td>RU</td>
      <td>2021-03-28 21:47:06</td>
      <td>91</td>
      <td>63.0500</td>
      <td>179.3167</td>
      <td>13.01</td>
      <td>17.67</td>
    </tr>
    <tr>
      <th>706</th>
      <td>Amapa</td>
      <td>100</td>
      <td>BR</td>
      <td>2021-03-28 21:47:06</td>
      <td>98</td>
      <td>1.0000</td>
      <td>-52.0000</td>
      <td>72.68</td>
      <td>2.62</td>
    </tr>
    <tr>
      <th>707</th>
      <td>Salta</td>
      <td>90</td>
      <td>AR</td>
      <td>2021-03-28 21:43:04</td>
      <td>59</td>
      <td>-24.7859</td>
      <td>-65.4117</td>
      <td>64.40</td>
      <td>8.05</td>
    </tr>
  </tbody>
</table>
<p>708 rows × 9 columns</p>
</div>




```python
# Create the output file (CSV).
output_data_file = "WeatherPy_Database.csv"
# Export the City_Data into a CSV.
city_data_df.to_csv(output_data_file, index_label="City_ID")
```


```python
#write error's out here
output_data_file = "WeatherPy_DB_skipped.csv"
skipped_cities_df = pd.DataFrame(skipped_cities)
# Export the City_Data into a CSV.
skipped_cities_df.to_csv(output_data_file, index_label="City_ID")
```


```python

```
