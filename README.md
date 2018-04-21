
# WeatherPy
***

## Analysis

### Methodology
- 1,779 samples gathered by picking 20 random longitudinal coordinates from each latitude (90S to 90N)
- Used citipy library to find nearest city / country code for each latitude / longitude combination, keeping only unique cities
- Fed city / country code combination to OpenWeather API -- returned weather data for 1,590 cities (~89%).

### Observations
- The sample generally falls between 60S and 80N degrees latitude, probably because of the lack of human habitation at the poles (the north pole is entirely ocean, Antarctica has no cities), as well as the 'closest city' algorithm of citipy, the library used to find the sample cities by lat / long.
- Temperature displays a distinct boomerang pattern. The southern hemisphere is entering autumn, so temperatures trend higher overall than the northern hemisphere, which is now entering spring. However, there is a clear trend on both sides toward higher temperatures as the latitudes near the equator. There is a cluster of cities with max temps higher than the equator from about 15N to 20N; however, the cluster seems to be located within the Tropic of Cancer, which is generally considered equatorial.
- There seems to be a strong band of high humidity (between 80% and 100%) across the entire sample. However, there seems to be a higher relative proportion of cities immediately around the equator (between the Tropics of Cancer and Capricorn) with higher humidity. Interestingly, there seem to be two clusters of cities which have very low humidities around 25N and 25S -- perhaps because of the location of the Sahara (23N) and the Kalahari (23S) deserts? 
- Cloud cover does not seem to be correlated with latitude at all, but there does seem to be a cluster of lower wind speeds right over the equator.


```python
import openweathermapy
from citipy import citipy
import matplotlib.pyplot as plt
import pandas as pd
import random
import numpy as np
import requests
from config import api_key
from collections import defaultdict
import json
from matplotlib import style
style.use('fivethirtyeight')
import time
import datetime
```


```python
# set min/max coords for randomized search of cities
max_latitude = 90
min_latitude = -90
max_longitude = 180
min_longitude = -180

cities = []

#for latitude in range(min_latitude, max_latitude, 1):
for latitude in np.linspace(min_latitude, max_latitude, num=360, endpoint=True):
    for _ in range(1, 21):
        lng = random.uniform(min_longitude, max_longitude)
        lat = random.uniform(latitude, latitude + 1)
        city = citipy.nearest_city(lat, lng)
        city_data = f'{city.city_name},{city.country_code}'
        if city_data not in cities:
            cities.append(city_data)
            
#the free api only allows 60 calls per minute, so divide into 34 samples. This should get ~55 cities per split.
samples = np.array_split(cities, 34)

print("City Location Complete")
```

    City Location Complete



```python
# Save config information.
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "imperial"

# Build partial query URL
query_url = f"{url}appid={api_key}&units={units}&q="

# set up dictionary to hold data
city_data_dict = defaultdict(list)

divider = f'*********************************'
print(divider)
print('* BEGINNING DATA RETRIEVAL..... *')
print(divider)

for index, target_cities in enumerate(samples):
    for idx, target_city in enumerate(target_cities):
        print(f'Processing Record {idx} of Set {index} | {target_city}')
        response = requests.get(query_url + target_city)
        response_data = response.json()

        #this is not a particularly pythonic way to build an exception, but in this program, we want all the data
        #or none of it. So if we get a 200 but there is a problem with one of the fields in the API, we do not want
        #it getting in the table. Dump the json() and troubleshoot. If we get anything but a 200, let us know about it.
        #It is mostly likely a 404 message, telling us it can't find the city. We will not load any data for a city we
        #can't find.
        if response.status_code == 200:
            print(f'Data found (status code: {response.status_code}) | {target_city}')
            try:
                city_data_dict['city_name'].append(response_data['name'])
                city_data_dict['city_country_code'].append(response_data['sys']['country'])
                city_data_dict['city_id'].append(response_data['id'])
                city_data_dict['city_lat'].append(response_data['coord']['lat'])
                city_data_dict['city_lng'].append(response_data['coord']['lon'])
                city_data_dict['city_max_temp'].append(response_data['main']['temp_max'])
                city_data_dict['city_humidity'].append(response_data['main']['humidity'])
                city_data_dict['city_wind_speed'].append(response_data['wind']['speed'])
                city_data_dict['city_cloud_cover'].append(response_data['clouds']['all'])
                city_data_dict['record_dt'].append(response_data['dt'])
                print(f'Data loaded successfully | {target_city}')
                print(divider)
            except KeyError:
                print(f'We got a good response (status code: {response.status_code}), but did not receive good info. \
                | {target_city }')
                print(f'API Response: ')
                print(json.dumps(response_data))
        else:
            try:
                print(f"ERROR: Data retrieval unsuccessful for {target_city}. Code: {response_data['cod']} \
                      MSG: {response_data['message']}")
            except KeyError:
                print(f'SEVERE ERROR: Data retrieval unsuccessful for {target_city}. Code: {response.status_code}')
    print("Sleeping 60 seconds before next round to avoid usage limits")
    time.sleep(60)
    
print(divider)
print("* DATA COLLECTION COMPLETE..... ")        
              


    
```

    *********************************
    * BEGINNING DATA RETRIEVAL..... *
    *********************************
    Processing Record 0 of Set 0 | taolanaro,mg
    ERROR: Data retrieval unsuccessful for taolanaro,mg. Code: 404                       MSG: city not found
    Processing Record 1 of Set 0 | hobart,au
    Data found (status code: 200) | hobart,au
    Data loaded successfully | hobart,au
    *********************************
    Processing Record 2 of Set 0 | port alfred,za
    Data found (status code: 200) | port alfred,za
    Data loaded successfully | port alfred,za
    *********************************
    Processing Record 3 of Set 0 | rikitea,pf
    Data found (status code: 200) | rikitea,pf
    Data loaded successfully | rikitea,pf
    *********************************
    Processing Record 4 of Set 0 | hermanus,za
    Data found (status code: 200) | hermanus,za
    Data loaded successfully | hermanus,za
    *********************************
    Processing Record 5 of Set 0 | ushuaia,ar
    Data found (status code: 200) | ushuaia,ar
    Data loaded successfully | ushuaia,ar
    *********************************
    Processing Record 6 of Set 0 | bredasdorp,za
    Data found (status code: 200) | bredasdorp,za
    Data loaded successfully | bredasdorp,za
    *********************************
    Processing Record 7 of Set 0 | bluff,nz
    Data found (status code: 200) | bluff,nz
    Data loaded successfully | bluff,nz
    *********************************
    Processing Record 8 of Set 0 | new norfolk,au
    Data found (status code: 200) | new norfolk,au
    Data loaded successfully | new norfolk,au
    *********************************
    Processing Record 9 of Set 0 | punta arenas,cl
    Data found (status code: 200) | punta arenas,cl
    Data loaded successfully | punta arenas,cl
    *********************************
    Processing Record 10 of Set 0 | mataura,pf
    ERROR: Data retrieval unsuccessful for mataura,pf. Code: 404                       MSG: city not found
    Processing Record 11 of Set 0 | east london,za
    Data found (status code: 200) | east london,za
    Data loaded successfully | east london,za
    *********************************
    Processing Record 12 of Set 0 | albany,au
    Data found (status code: 200) | albany,au
    Data loaded successfully | albany,au
    *********************************
    Processing Record 13 of Set 0 | busselton,au
    Data found (status code: 200) | busselton,au
    Data loaded successfully | busselton,au
    *********************************
    Processing Record 14 of Set 0 | kruisfontein,za
    Data found (status code: 200) | kruisfontein,za
    Data loaded successfully | kruisfontein,za
    *********************************
    Processing Record 15 of Set 0 | port elizabeth,za
    Data found (status code: 200) | port elizabeth,za
    Data loaded successfully | port elizabeth,za
    *********************************
    Processing Record 16 of Set 0 | vaini,to
    Data found (status code: 200) | vaini,to
    Data loaded successfully | vaini,to
    *********************************
    Processing Record 17 of Set 0 | cape town,za
    Data found (status code: 200) | cape town,za
    Data loaded successfully | cape town,za
    *********************************
    Processing Record 18 of Set 0 | kaitangata,nz
    Data found (status code: 200) | kaitangata,nz
    Data loaded successfully | kaitangata,nz
    *********************************
    Processing Record 19 of Set 0 | avarua,ck
    Data found (status code: 200) | avarua,ck
    Data loaded successfully | avarua,ck
    *********************************
    Processing Record 20 of Set 0 | mar del plata,ar
    Data found (status code: 200) | mar del plata,ar
    Data loaded successfully | mar del plata,ar
    *********************************
    Processing Record 21 of Set 0 | saint-philippe,re
    Data found (status code: 200) | saint-philippe,re
    Data loaded successfully | saint-philippe,re
    *********************************
    Processing Record 22 of Set 0 | chuy,uy
    Data found (status code: 200) | chuy,uy
    Data loaded successfully | chuy,uy
    *********************************
    Processing Record 23 of Set 0 | tuatapere,nz
    Data found (status code: 200) | tuatapere,nz
    Data loaded successfully | tuatapere,nz
    *********************************
    Processing Record 24 of Set 0 | dunedin,nz
    Data found (status code: 200) | dunedin,nz
    Data loaded successfully | dunedin,nz
    *********************************
    Processing Record 25 of Set 0 | souillac,mu
    Data found (status code: 200) | souillac,mu
    Data loaded successfully | souillac,mu
    *********************************
    Processing Record 26 of Set 0 | mount gambier,au
    Data found (status code: 200) | mount gambier,au
    Data loaded successfully | mount gambier,au
    *********************************
    Processing Record 27 of Set 0 | castro,cl
    Data found (status code: 200) | castro,cl
    Data loaded successfully | castro,cl
    *********************************
    Processing Record 28 of Set 0 | cidreira,br
    Data found (status code: 200) | cidreira,br
    Data loaded successfully | cidreira,br
    *********************************
    Processing Record 29 of Set 0 | tsihombe,mg
    ERROR: Data retrieval unsuccessful for tsihombe,mg. Code: 404                       MSG: city not found
    Processing Record 30 of Set 0 | portland,au
    Data found (status code: 200) | portland,au
    Data loaded successfully | portland,au
    *********************************
    Processing Record 31 of Set 0 | arraial do cabo,br
    Data found (status code: 200) | arraial do cabo,br
    Data loaded successfully | arraial do cabo,br
    *********************************
    Processing Record 32 of Set 0 | esperance,au
    Data found (status code: 200) | esperance,au
    Data loaded successfully | esperance,au
    *********************************
    Processing Record 33 of Set 0 | rawson,ar
    Data found (status code: 200) | rawson,ar
    Data loaded successfully | rawson,ar
    *********************************
    Processing Record 34 of Set 0 | umzimvubu,za
    ERROR: Data retrieval unsuccessful for umzimvubu,za. Code: 404                       MSG: city not found
    Processing Record 35 of Set 0 | mahebourg,mu
    Data found (status code: 200) | mahebourg,mu
    Data loaded successfully | mahebourg,mu
    *********************************
    Processing Record 36 of Set 0 | jamestown,sh
    Data found (status code: 200) | jamestown,sh
    Data loaded successfully | jamestown,sh
    *********************************
    Processing Record 37 of Set 0 | christchurch,nz
    Data found (status code: 200) | christchurch,nz
    Data loaded successfully | christchurch,nz
    *********************************
    Processing Record 38 of Set 0 | rio gallegos,ar
    Data found (status code: 200) | rio gallegos,ar
    Data loaded successfully | rio gallegos,ar
    *********************************
    Processing Record 39 of Set 0 | rocha,uy
    Data found (status code: 200) | rocha,uy
    Data loaded successfully | rocha,uy
    *********************************
    Processing Record 40 of Set 0 | coihaique,cl
    Data found (status code: 200) | coihaique,cl
    Data loaded successfully | coihaique,cl
    *********************************
    Processing Record 41 of Set 0 | necochea,ar
    Data found (status code: 200) | necochea,ar
    Data loaded successfully | necochea,ar
    *********************************
    Processing Record 42 of Set 0 | waitati,nz
    Data found (status code: 200) | waitati,nz
    Data loaded successfully | waitati,nz
    *********************************
    Processing Record 43 of Set 0 | comodoro rivadavia,ar
    Data found (status code: 200) | comodoro rivadavia,ar
    Data loaded successfully | comodoro rivadavia,ar
    *********************************
    Processing Record 44 of Set 0 | waipawa,nz
    Data found (status code: 200) | waipawa,nz
    Data loaded successfully | waipawa,nz
    *********************************
    Processing Record 45 of Set 0 | southbridge,nz
    Data found (status code: 200) | southbridge,nz
    Data loaded successfully | southbridge,nz
    *********************************
    Processing Record 46 of Set 0 | wyndham,nz
    Data found (status code: 200) | wyndham,nz
    Data loaded successfully | wyndham,nz
    *********************************
    Processing Record 47 of Set 0 | port lincoln,au
    Data found (status code: 200) | port lincoln,au
    Data loaded successfully | port lincoln,au
    *********************************
    Processing Record 48 of Set 0 | pareora,nz
    Data found (status code: 200) | pareora,nz
    Data loaded successfully | pareora,nz
    *********************************
    Processing Record 49 of Set 0 | te anau,nz
    Data found (status code: 200) | te anau,nz
    Data loaded successfully | te anau,nz
    *********************************
    Processing Record 50 of Set 0 | burnie,au
    Data found (status code: 200) | burnie,au
    Data loaded successfully | burnie,au
    *********************************
    Processing Record 51 of Set 0 | saldanha,za
    Data found (status code: 200) | saldanha,za
    Data loaded successfully | saldanha,za
    *********************************
    Processing Record 52 of Set 0 | wanaka,nz
    Data found (status code: 200) | wanaka,nz
    Data loaded successfully | wanaka,nz
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 1 | rio grande,br
    Data found (status code: 200) | rio grande,br
    Data loaded successfully | rio grande,br
    *********************************
    Processing Record 1 of Set 1 | otane,nz
    Data found (status code: 200) | otane,nz
    Data loaded successfully | otane,nz
    *********************************
    Processing Record 2 of Set 1 | hokitika,nz
    Data found (status code: 200) | hokitika,nz
    Data loaded successfully | hokitika,nz
    *********************************
    Processing Record 3 of Set 1 | margate,za
    Data found (status code: 200) | margate,za
    Data loaded successfully | margate,za
    *********************************
    Processing Record 4 of Set 1 | san carlos de bariloche,ar
    Data found (status code: 200) | san carlos de bariloche,ar
    Data loaded successfully | san carlos de bariloche,ar
    *********************************
    Processing Record 5 of Set 1 | trelew,ar
    Data found (status code: 200) | trelew,ar
    Data loaded successfully | trelew,ar
    *********************************
    Processing Record 6 of Set 1 | ancud,cl
    Data found (status code: 200) | ancud,cl
    Data loaded successfully | ancud,cl
    *********************************
    Processing Record 7 of Set 1 | puerto montt,cl
    Data found (status code: 200) | puerto montt,cl
    Data loaded successfully | puerto montt,cl
    *********************************
    Processing Record 8 of Set 1 | murchison,nz
    Data found (status code: 200) | murchison,nz
    Data loaded successfully | murchison,nz
    *********************************
    Processing Record 9 of Set 1 | viedma,ar
    Data found (status code: 200) | viedma,ar
    Data loaded successfully | viedma,ar
    *********************************
    Processing Record 10 of Set 1 | batemans bay,au
    Data found (status code: 200) | batemans bay,au
    Data loaded successfully | batemans bay,au
    *********************************
    Processing Record 11 of Set 1 | calbuco,cl
    Data found (status code: 200) | calbuco,cl
    Data loaded successfully | calbuco,cl
    *********************************
    Processing Record 12 of Set 1 | general roca,ar
    Data found (status code: 200) | general roca,ar
    Data loaded successfully | general roca,ar
    *********************************
    Processing Record 13 of Set 1 | hastings,nz
    Data found (status code: 200) | hastings,nz
    Data loaded successfully | hastings,nz
    *********************************
    Processing Record 14 of Set 1 | bambous virieux,mu
    Data found (status code: 200) | bambous virieux,mu
    Data loaded successfully | bambous virieux,mu
    *********************************
    Processing Record 15 of Set 1 | ulladulla,au
    Data found (status code: 200) | ulladulla,au
    Data loaded successfully | ulladulla,au
    *********************************
    Processing Record 16 of Set 1 | gisborne,nz
    Data found (status code: 200) | gisborne,nz
    Data loaded successfully | gisborne,nz
    *********************************
    Processing Record 17 of Set 1 | punta alta,ar
    Data found (status code: 200) | punta alta,ar
    Data loaded successfully | punta alta,ar
    *********************************
    Processing Record 18 of Set 1 | takaka,nz
    Data found (status code: 200) | takaka,nz
    Data loaded successfully | takaka,nz
    *********************************
    Processing Record 19 of Set 1 | wonthaggi,au
    Data found (status code: 200) | wonthaggi,au
    Data loaded successfully | wonthaggi,au
    *********************************
    Processing Record 20 of Set 1 | bahia blanca,ar
    Data found (status code: 200) | bahia blanca,ar
    Data loaded successfully | bahia blanca,ar
    *********************************
    Processing Record 21 of Set 1 | valdivia,cl
    Data found (status code: 200) | valdivia,cl
    Data loaded successfully | valdivia,cl
    *********************************
    Processing Record 22 of Set 1 | wairoa,nz
    Data found (status code: 200) | wairoa,nz
    Data loaded successfully | wairoa,nz
    *********************************
    Processing Record 23 of Set 1 | lebu,cl
    Data found (status code: 200) | lebu,cl
    Data loaded successfully | lebu,cl
    *********************************
    Processing Record 24 of Set 1 | warrnambool,au
    Data found (status code: 200) | warrnambool,au
    Data loaded successfully | warrnambool,au
    *********************************
    Processing Record 25 of Set 1 | westport,nz
    Data found (status code: 200) | westport,nz
    Data loaded successfully | westport,nz
    *********************************
    Processing Record 26 of Set 1 | nueva imperial,cl
    Data found (status code: 200) | nueva imperial,cl
    Data loaded successfully | nueva imperial,cl
    *********************************
    Processing Record 27 of Set 1 | lakes entrance,au
    Data found (status code: 200) | lakes entrance,au
    Data loaded successfully | lakes entrance,au
    *********************************
    Processing Record 28 of Set 1 | santa rosa,ar
    Data found (status code: 200) | santa rosa,ar
    Data loaded successfully | santa rosa,ar
    *********************************
    Processing Record 29 of Set 1 | karamea,nz
    ERROR: Data retrieval unsuccessful for karamea,nz. Code: 404                       MSG: city not found
    Processing Record 30 of Set 1 | ahipara,nz
    Data found (status code: 200) | ahipara,nz
    Data loaded successfully | ahipara,nz
    *********************************
    Processing Record 31 of Set 1 | mulchen,cl
    Data found (status code: 200) | mulchen,cl
    Data loaded successfully | mulchen,cl
    *********************************
    Processing Record 32 of Set 1 | laguna,br
    ERROR: Data retrieval unsuccessful for laguna,br. Code: 404                       MSG: city not found
    Processing Record 33 of Set 1 | horsham,au
    Data found (status code: 200) | horsham,au
    Data loaded successfully | horsham,au
    *********************************
    Processing Record 34 of Set 1 | flinders,au
    Data found (status code: 200) | flinders,au
    Data loaded successfully | flinders,au
    *********************************
    Processing Record 35 of Set 1 | bendigo,au
    Data found (status code: 200) | bendigo,au
    Data loaded successfully | bendigo,au
    *********************************
    Processing Record 36 of Set 1 | dolores,ar
    Data found (status code: 200) | dolores,ar
    Data loaded successfully | dolores,ar
    *********************************
    Processing Record 37 of Set 1 | george,za
    Data found (status code: 200) | george,za
    Data loaded successfully | george,za
    *********************************
    Processing Record 38 of Set 1 | san clemente,cl
    Data found (status code: 200) | san clemente,cl
    Data loaded successfully | san clemente,cl
    *********************************
    Processing Record 39 of Set 1 | richards bay,za
    Data found (status code: 200) | richards bay,za
    Data loaded successfully | richards bay,za
    *********************************
    Processing Record 40 of Set 1 | olavarria,ar
    Data found (status code: 200) | olavarria,ar
    Data loaded successfully | olavarria,ar
    *********************************
    Processing Record 41 of Set 1 | san rafael,ar
    Data found (status code: 200) | san rafael,ar
    Data loaded successfully | san rafael,ar
    *********************************
    Processing Record 42 of Set 1 | luderitz,na
    Data found (status code: 200) | luderitz,na
    Data loaded successfully | luderitz,na
    *********************************
    Processing Record 43 of Set 1 | ruatoria,nz
    ERROR: Data retrieval unsuccessful for ruatoria,nz. Code: 404                       MSG: city not found
    Processing Record 44 of Set 1 | geraldton,au
    Data found (status code: 200) | geraldton,au
    Data loaded successfully | geraldton,au
    *********************************
    Processing Record 45 of Set 1 | constitucion,cl
    Data found (status code: 200) | constitucion,cl
    Data loaded successfully | constitucion,cl
    *********************************
    Processing Record 46 of Set 1 | plettenberg bay,za
    Data found (status code: 200) | plettenberg bay,za
    Data loaded successfully | plettenberg bay,za
    *********************************
    Processing Record 47 of Set 1 | beloha,mg
    Data found (status code: 200) | beloha,mg
    Data loaded successfully | beloha,mg
    *********************************
    Processing Record 48 of Set 1 | adelaide,au
    Data found (status code: 200) | adelaide,au
    Data loaded successfully | adelaide,au
    *********************************
    Processing Record 49 of Set 1 | port macquarie,au
    Data found (status code: 200) | port macquarie,au
    Data loaded successfully | port macquarie,au
    *********************************
    Processing Record 50 of Set 1 | scottsburgh,za
    ERROR: Data retrieval unsuccessful for scottsburgh,za. Code: 404                       MSG: city not found
    Processing Record 51 of Set 1 | sao joao da barra,br
    Data found (status code: 200) | sao joao da barra,br
    Data loaded successfully | sao joao da barra,br
    *********************************
    Processing Record 52 of Set 1 | cootamundra,au
    Data found (status code: 200) | cootamundra,au
    Data loaded successfully | cootamundra,au
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 2 | swellendam,za
    Data found (status code: 200) | swellendam,za
    Data loaded successfully | swellendam,za
    *********************************
    Processing Record 1 of Set 2 | san jose,uy
    ERROR: Data retrieval unsuccessful for san jose,uy. Code: 404                       MSG: city not found
    Processing Record 2 of Set 2 | rivadavia,ar
    Data found (status code: 200) | rivadavia,ar
    Data loaded successfully | rivadavia,ar
    *********************************
    Processing Record 3 of Set 2 | ceres,za
    Data found (status code: 200) | ceres,za
    Data loaded successfully | ceres,za
    *********************************
    Processing Record 4 of Set 2 | russell,nz
    Data found (status code: 200) | russell,nz
    Data loaded successfully | russell,nz
    *********************************
    Processing Record 5 of Set 2 | quilpue,cl
    Data found (status code: 200) | quilpue,cl
    Data loaded successfully | quilpue,cl
    *********************************
    Processing Record 6 of Set 2 | carnarvon,au
    Data found (status code: 200) | carnarvon,au
    Data loaded successfully | carnarvon,au
    *********************************
    Processing Record 7 of Set 2 | valparaiso,cl
    Data found (status code: 200) | valparaiso,cl
    Data loaded successfully | valparaiso,cl
    *********************************
    Processing Record 8 of Set 2 | talcahuano,cl
    Data found (status code: 200) | talcahuano,cl
    Data loaded successfully | talcahuano,cl
    *********************************
    Processing Record 9 of Set 2 | victoria,ar
    Data found (status code: 200) | victoria,ar
    Data loaded successfully | victoria,ar
    *********************************
    Processing Record 10 of Set 2 | broken hill,au
    Data found (status code: 200) | broken hill,au
    Data loaded successfully | broken hill,au
    *********************************
    Processing Record 11 of Set 2 | port augusta,au
    Data found (status code: 200) | port augusta,au
    Data loaded successfully | port augusta,au
    *********************************
    Processing Record 12 of Set 2 | griffith,au
    Data found (status code: 200) | griffith,au
    Data loaded successfully | griffith,au
    *********************************
    Processing Record 13 of Set 2 | dordrecht,za
    Data found (status code: 200) | dordrecht,za
    Data loaded successfully | dordrecht,za
    *********************************
    Processing Record 14 of Set 2 | vredendal,za
    Data found (status code: 200) | vredendal,za
    Data loaded successfully | vredendal,za
    *********************************
    Processing Record 15 of Set 2 | dom pedrito,br
    Data found (status code: 200) | dom pedrito,br
    Data loaded successfully | dom pedrito,br
    *********************************
    Processing Record 16 of Set 2 | coquimbo,cl
    Data found (status code: 200) | coquimbo,cl
    Data loaded successfully | coquimbo,cl
    *********************************
    Processing Record 17 of Set 2 | tacuarembo,uy
    Data found (status code: 200) | tacuarembo,uy
    Data loaded successfully | tacuarembo,uy
    *********************************
    Processing Record 18 of Set 2 | dubbo,au
    Data found (status code: 200) | dubbo,au
    Data loaded successfully | dubbo,au
    *********************************
    Processing Record 19 of Set 2 | tapes,br
    Data found (status code: 200) | tapes,br
    Data loaded successfully | tapes,br
    *********************************
    Processing Record 20 of Set 2 | oranjemund,na
    Data found (status code: 200) | oranjemund,na
    Data loaded successfully | oranjemund,na
    *********************************
    Processing Record 21 of Set 2 | san juan,ar
    Data found (status code: 200) | san juan,ar
    Data loaded successfully | san juan,ar
    *********************************
    Processing Record 22 of Set 2 | springbok,za
    Data found (status code: 200) | springbok,za
    Data loaded successfully | springbok,za
    *********************************
    Processing Record 23 of Set 2 | encruzilhada do sul,br
    Data found (status code: 200) | encruzilhada do sul,br
    Data loaded successfully | encruzilhada do sul,br
    *********************************
    Processing Record 24 of Set 2 | alofi,nu
    Data found (status code: 200) | alofi,nu
    Data loaded successfully | alofi,nu
    *********************************
    Processing Record 25 of Set 2 | avera,pf
    ERROR: Data retrieval unsuccessful for avera,pf. Code: 404                       MSG: city not found
    Processing Record 26 of Set 2 | byron bay,au
    Data found (status code: 200) | byron bay,au
    Data loaded successfully | byron bay,au
    *********************************
    Processing Record 27 of Set 2 | rafaela,ar
    Data found (status code: 200) | rafaela,ar
    Data loaded successfully | rafaela,ar
    *********************************
    Processing Record 28 of Set 2 | villa carlos paz,ar
    Data found (status code: 200) | villa carlos paz,ar
    Data loaded successfully | villa carlos paz,ar
    *********************************
    Processing Record 29 of Set 2 | la rioja,ar
    Data found (status code: 200) | la rioja,ar
    Data loaded successfully | la rioja,ar
    *********************************
    Processing Record 30 of Set 2 | vila velha,br
    Data found (status code: 200) | vila velha,br
    Data loaded successfully | vila velha,br
    *********************************
    Processing Record 31 of Set 2 | vicuna,cl
    Data found (status code: 200) | vicuna,cl
    Data loaded successfully | vicuna,cl
    *********************************
    Processing Record 32 of Set 2 | kaeo,nz
    Data found (status code: 200) | kaeo,nz
    Data loaded successfully | kaeo,nz
    *********************************
    Processing Record 33 of Set 2 | reconquista,ar
    Data found (status code: 200) | reconquista,ar
    Data loaded successfully | reconquista,ar
    *********************************
    Processing Record 34 of Set 2 | inhambane,mz
    Data found (status code: 200) | inhambane,mz
    Data loaded successfully | inhambane,mz
    *********************************
    Processing Record 35 of Set 2 | northam,au
    Data found (status code: 200) | northam,au
    Data loaded successfully | northam,au
    *********************************
    Processing Record 36 of Set 2 | getulio vargas,br
    ERROR: Data retrieval unsuccessful for getulio vargas,br. Code: 404                       MSG: city not found
    Processing Record 37 of Set 2 | harrismith,za
    Data found (status code: 200) | harrismith,za
    Data loaded successfully | harrismith,za
    *********************************
    Processing Record 38 of Set 2 | grand river south east,mu
    ERROR: Data retrieval unsuccessful for grand river south east,mu. Code: 404                       MSG: city not found
    Processing Record 39 of Set 2 | porto belo,br
    Data found (status code: 200) | porto belo,br
    Data loaded successfully | porto belo,br
    *********************************
    Processing Record 40 of Set 2 | pisco,pe
    Data found (status code: 200) | pisco,pe
    Data loaded successfully | pisco,pe
    *********************************
    Processing Record 41 of Set 2 | saint-joseph,re
    Data found (status code: 200) | saint-joseph,re
    Data loaded successfully | saint-joseph,re
    *********************************
    Processing Record 42 of Set 2 | santiago del estero,ar
    Data found (status code: 200) | santiago del estero,ar
    Data loaded successfully | santiago del estero,ar
    *********************************
    Processing Record 43 of Set 2 | sao sebastiao,br
    Data found (status code: 200) | sao sebastiao,br
    Data loaded successfully | sao sebastiao,br
    *********************************
    Processing Record 44 of Set 2 | vao,nc
    Data found (status code: 200) | vao,nc
    Data loaded successfully | vao,nc
    *********************************
    Processing Record 45 of Set 2 | puerto ayora,ec
    Data found (status code: 200) | puerto ayora,ec
    Data loaded successfully | puerto ayora,ec
    *********************************
    Processing Record 46 of Set 2 | marcona,pe
    ERROR: Data retrieval unsuccessful for marcona,pe. Code: 404                       MSG: city not found
    Processing Record 47 of Set 2 | dalby,au
    Data found (status code: 200) | dalby,au
    Data loaded successfully | dalby,au
    *********************************
    Processing Record 48 of Set 2 | mount isa,au
    Data found (status code: 200) | mount isa,au
    Data loaded successfully | mount isa,au
    *********************************
    Processing Record 49 of Set 2 | port hedland,au
    Data found (status code: 200) | port hedland,au
    Data loaded successfully | port hedland,au
    *********************************
    Processing Record 50 of Set 2 | koster,za
    Data found (status code: 200) | koster,za
    Data loaded successfully | koster,za
    *********************************
    Processing Record 51 of Set 2 | boksburg,za
    Data found (status code: 200) | boksburg,za
    Data loaded successfully | boksburg,za
    *********************************
    Processing Record 52 of Set 2 | bethanien,na
    Data found (status code: 200) | bethanien,na
    Data loaded successfully | bethanien,na
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 3 | ilhabela,br
    Data found (status code: 200) | ilhabela,br
    Data loaded successfully | ilhabela,br
    *********************************
    Processing Record 1 of Set 3 | manjacaze,mz
    Data found (status code: 200) | manjacaze,mz
    Data loaded successfully | manjacaze,mz
    *********************************
    Processing Record 2 of Set 3 | taltal,cl
    Data found (status code: 200) | taltal,cl
    Data loaded successfully | taltal,cl
    *********************************
    Processing Record 3 of Set 3 | diego de almagro,cl
    Data found (status code: 200) | diego de almagro,cl
    Data loaded successfully | diego de almagro,cl
    *********************************
    Processing Record 4 of Set 3 | hervey bay,au
    Data found (status code: 200) | hervey bay,au
    Data loaded successfully | hervey bay,au
    *********************************
    Processing Record 5 of Set 3 | salta,ar
    Data found (status code: 200) | salta,ar
    Data loaded successfully | salta,ar
    *********************************
    Processing Record 6 of Set 3 | pontal do parana,br
    Data found (status code: 200) | pontal do parana,br
    Data loaded successfully | pontal do parana,br
    *********************************
    Processing Record 7 of Set 3 | alice springs,au
    Data found (status code: 200) | alice springs,au
    Data loaded successfully | alice springs,au
    *********************************
    Processing Record 8 of Set 3 | toliary,mg
    ERROR: Data retrieval unsuccessful for toliary,mg. Code: 404                       MSG: city not found
    Processing Record 9 of Set 3 | campo largo,br
    Data found (status code: 200) | campo largo,br
    Data loaded successfully | campo largo,br
    *********************************
    Processing Record 10 of Set 3 | isangel,vu
    Data found (status code: 200) | isangel,vu
    Data loaded successfully | isangel,vu
    *********************************
    Processing Record 11 of Set 3 | henties bay,na
    Data found (status code: 200) | henties bay,na
    Data loaded successfully | henties bay,na
    *********************************
    Processing Record 12 of Set 3 | yulara,au
    Data found (status code: 200) | yulara,au
    Data loaded successfully | yulara,au
    *********************************
    Processing Record 13 of Set 3 | san pedro,ar
    Data found (status code: 200) | san pedro,ar
    Data loaded successfully | san pedro,ar
    *********************************
    Processing Record 14 of Set 3 | antofagasta,cl
    Data found (status code: 200) | antofagasta,cl
    Data loaded successfully | antofagasta,cl
    *********************************
    Processing Record 15 of Set 3 | tshane,bw
    Data found (status code: 200) | tshane,bw
    Data loaded successfully | tshane,bw
    *********************************
    Processing Record 16 of Set 3 | potgietersrus,za
    ERROR: Data retrieval unsuccessful for potgietersrus,za. Code: 404                       MSG: city not found
    Processing Record 17 of Set 3 | palotina,br
    Data found (status code: 200) | palotina,br
    Data loaded successfully | palotina,br
    *********************************
    Processing Record 18 of Set 3 | vangaindrano,mg
    Data found (status code: 200) | vangaindrano,mg
    Data loaded successfully | vangaindrano,mg
    *********************************
    Processing Record 19 of Set 3 | caravelas,br
    Data found (status code: 200) | caravelas,br
    Data loaded successfully | caravelas,br
    *********************************
    Processing Record 20 of Set 3 | rockhampton,au
    Data found (status code: 200) | rockhampton,au
    Data loaded successfully | rockhampton,au
    *********************************
    Processing Record 21 of Set 3 | san ramon de la nueva oran,ar
    Data found (status code: 200) | san ramon de la nueva oran,ar
    Data loaded successfully | san ramon de la nueva oran,ar
    *********************************
    Processing Record 22 of Set 3 | doctor pedro p. pena,py
    ERROR: Data retrieval unsuccessful for doctor pedro p. pena,py. Code: 404                       MSG: city not found
    Processing Record 23 of Set 3 | walvis bay,na
    Data found (status code: 200) | walvis bay,na
    Data loaded successfully | walvis bay,na
    *********************************
    Processing Record 24 of Set 3 | loanda,br
    Data found (status code: 200) | loanda,br
    Data loaded successfully | loanda,br
    *********************************
    Processing Record 25 of Set 3 | rio de janeiro,br
    Data found (status code: 200) | rio de janeiro,br
    Data loaded successfully | rio de janeiro,br
    *********************************
    Processing Record 26 of Set 3 | boatlaname,bw
    ERROR: Data retrieval unsuccessful for boatlaname,bw. Code: 404                       MSG: city not found
    Processing Record 27 of Set 3 | blackwater,au
    Data found (status code: 200) | blackwater,au
    Data loaded successfully | blackwater,au
    *********************************
    Processing Record 28 of Set 3 | marica,br
    Data found (status code: 200) | marica,br
    Data loaded successfully | marica,br
    *********************************
    Processing Record 29 of Set 3 | kalamare,bw
    Data found (status code: 200) | kalamare,bw
    Data loaded successfully | kalamare,bw
    *********************************
    Processing Record 30 of Set 3 | manakara,mg
    Data found (status code: 200) | manakara,mg
    Data loaded successfully | manakara,mg
    *********************************
    Processing Record 31 of Set 3 | koumac,nc
    Data found (status code: 200) | koumac,nc
    Data loaded successfully | koumac,nc
    *********************************
    Processing Record 32 of Set 3 | itatiaia,br
    Data found (status code: 200) | itatiaia,br
    Data loaded successfully | itatiaia,br
    *********************************
    Processing Record 33 of Set 3 | kang,bw
    Data found (status code: 200) | kang,bw
    Data loaded successfully | kang,bw
    *********************************
    Processing Record 34 of Set 3 | hualmay,pe
    Data found (status code: 200) | hualmay,pe
    Data loaded successfully | hualmay,pe
    *********************************
    Processing Record 35 of Set 3 | hithadhoo,mv
    Data found (status code: 200) | hithadhoo,mv
    Data loaded successfully | hithadhoo,mv
    *********************************
    Processing Record 36 of Set 3 | mopipi,bw
    Data found (status code: 200) | mopipi,bw
    Data loaded successfully | mopipi,bw
    *********************************
    Processing Record 37 of Set 3 | charters towers,au
    Data found (status code: 200) | charters towers,au
    Data loaded successfully | charters towers,au
    *********************************
    Processing Record 38 of Set 3 | tocopilla,cl
    Data found (status code: 200) | tocopilla,cl
    Data loaded successfully | tocopilla,cl
    *********************************
    Processing Record 39 of Set 3 | serowe,bw
    Data found (status code: 200) | serowe,bw
    Data loaded successfully | serowe,bw
    *********************************
    Processing Record 40 of Set 3 | broome,au
    Data found (status code: 200) | broome,au
    Data loaded successfully | broome,au
    *********************************
    Processing Record 41 of Set 3 | okahandja,na
    Data found (status code: 200) | okahandja,na
    Data loaded successfully | okahandja,na
    *********************************
    Processing Record 42 of Set 3 | calama,cl
    Data found (status code: 200) | calama,cl
    Data loaded successfully | calama,cl
    *********************************
    Processing Record 43 of Set 3 | yeppoon,au
    Data found (status code: 200) | yeppoon,au
    Data loaded successfully | yeppoon,au
    *********************************
    Processing Record 44 of Set 3 | georgetown,sh
    Data found (status code: 200) | georgetown,sh
    Data loaded successfully | georgetown,sh
    *********************************
    Processing Record 45 of Set 3 | nosy varika,mg
    Data found (status code: 200) | nosy varika,mg
    Data loaded successfully | nosy varika,mg
    *********************************
    Processing Record 46 of Set 3 | ghanzi,bw
    Data found (status code: 200) | ghanzi,bw
    Data loaded successfully | ghanzi,bw
    *********************************
    Processing Record 47 of Set 3 | tadine,nc
    Data found (status code: 200) | tadine,nc
    Data loaded successfully | tadine,nc
    *********************************
    Processing Record 48 of Set 3 | beira,mz
    Data found (status code: 200) | beira,mz
    Data loaded successfully | beira,mz
    *********************************
    Processing Record 49 of Set 3 | tautira,pf
    Data found (status code: 200) | tautira,pf
    Data loaded successfully | tautira,pf
    *********************************
    Processing Record 50 of Set 3 | gwanda,zw
    Data found (status code: 200) | gwanda,zw
    Data loaded successfully | gwanda,zw
    *********************************
    Processing Record 51 of Set 3 | alfredo chaves,br
    Data found (status code: 200) | alfredo chaves,br
    Data loaded successfully | alfredo chaves,br
    *********************************
    Processing Record 52 of Set 3 | ifanadiana,mg
    Data found (status code: 200) | ifanadiana,mg
    Data loaded successfully | ifanadiana,mg
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 4 | iquique,cl
    Data found (status code: 200) | iquique,cl
    Data loaded successfully | iquique,cl
    *********************************
    Processing Record 1 of Set 4 | kalakamati,bw
    Data found (status code: 200) | kalakamati,bw
    Data loaded successfully | kalakamati,bw
    *********************************
    Processing Record 2 of Set 4 | morondava,mg
    Data found (status code: 200) | morondava,mg
    Data loaded successfully | morondava,mg
    *********************************
    Processing Record 3 of Set 4 | andevoranto,mg
    ERROR: Data retrieval unsuccessful for andevoranto,mg. Code: 404                       MSG: city not found
    Processing Record 4 of Set 4 | bundaberg,au
    Data found (status code: 200) | bundaberg,au
    Data loaded successfully | bundaberg,au
    *********************************
    Processing Record 5 of Set 4 | bowen,au
    Data found (status code: 200) | bowen,au
    Data loaded successfully | bowen,au
    *********************************
    Processing Record 6 of Set 4 | labuhan,id
    Data found (status code: 200) | labuhan,id
    Data loaded successfully | labuhan,id
    *********************************
    Processing Record 7 of Set 4 | okakarara,na
    Data found (status code: 200) | okakarara,na
    Data loaded successfully | okakarara,na
    *********************************
    Processing Record 8 of Set 4 | mahanoro,mg
    Data found (status code: 200) | mahanoro,mg
    Data loaded successfully | mahanoro,mg
    *********************************
    Processing Record 9 of Set 4 | opuwo,na
    Data found (status code: 200) | opuwo,na
    Data loaded successfully | opuwo,na
    *********************************
    Processing Record 10 of Set 4 | mvuma,zw
    Data found (status code: 200) | mvuma,zw
    Data loaded successfully | mvuma,zw
    *********************************
    Processing Record 11 of Set 4 | le port,re
    Data found (status code: 200) | le port,re
    Data loaded successfully | le port,re
    *********************************
    Processing Record 12 of Set 4 | quatre cocos,mu
    Data found (status code: 200) | quatre cocos,mu
    Data loaded successfully | quatre cocos,mu
    *********************************
    Processing Record 13 of Set 4 | nokaneng,bw
    Data found (status code: 200) | nokaneng,bw
    Data loaded successfully | nokaneng,bw
    *********************************
    Processing Record 14 of Set 4 | poum,nc
    Data found (status code: 200) | poum,nc
    Data loaded successfully | poum,nc
    *********************************
    Processing Record 15 of Set 4 | palabuhanratu,id
    ERROR: Data retrieval unsuccessful for palabuhanratu,id. Code: 404                       MSG: city not found
    Processing Record 16 of Set 4 | arcos,br
    Data found (status code: 200) | arcos,br
    Data loaded successfully | arcos,br
    *********************************
    Processing Record 17 of Set 4 | we,nc
    Data found (status code: 200) | we,nc
    Data loaded successfully | we,nc
    *********************************
    Processing Record 18 of Set 4 | pangai,to
    Data found (status code: 200) | pangai,to
    Data loaded successfully | pangai,to
    *********************************
    Processing Record 19 of Set 4 | quelimane,mz
    Data found (status code: 200) | quelimane,mz
    Data loaded successfully | quelimane,mz
    *********************************
    Processing Record 20 of Set 4 | bengkulu,id
    ERROR: Data retrieval unsuccessful for bengkulu,id. Code: 404                       MSG: city not found
    Processing Record 21 of Set 4 | kununurra,au
    Data found (status code: 200) | kununurra,au
    Data loaded successfully | kununurra,au
    *********************************
    Processing Record 22 of Set 4 | arica,cl
    Data found (status code: 200) | arica,cl
    Data loaded successfully | arica,cl
    *********************************
    Processing Record 23 of Set 4 | maun,bw
    Data found (status code: 200) | maun,bw
    Data loaded successfully | maun,bw
    *********************************
    Processing Record 24 of Set 4 | teahupoo,pf
    Data found (status code: 200) | teahupoo,pf
    Data loaded successfully | teahupoo,pf
    *********************************
    Processing Record 25 of Set 4 | conselheiro pena,br
    Data found (status code: 200) | conselheiro pena,br
    Data loaded successfully | conselheiro pena,br
    *********************************
    Processing Record 26 of Set 4 | puerto suarez,bo
    Data found (status code: 200) | puerto suarez,bo
    Data loaded successfully | puerto suarez,bo
    *********************************
    Processing Record 27 of Set 4 | shakawe,bw
    Data found (status code: 200) | shakawe,bw
    Data loaded successfully | shakawe,bw
    *********************************
    Processing Record 28 of Set 4 | acari,pe
    Data found (status code: 200) | acari,pe
    Data loaded successfully | acari,pe
    *********************************
    Processing Record 29 of Set 4 | rundu,na
    Data found (status code: 200) | rundu,na
    Data loaded successfully | rundu,na
    *********************************
    Processing Record 30 of Set 4 | nova venecia,br
    Data found (status code: 200) | nova venecia,br
    Data loaded successfully | nova venecia,br
    *********************************
    Processing Record 31 of Set 4 | ambodifototra,mg
    ERROR: Data retrieval unsuccessful for ambodifototra,mg. Code: 404                       MSG: city not found
    Processing Record 32 of Set 4 | ayr,au
    Data found (status code: 200) | ayr,au
    Data loaded successfully | ayr,au
    *********************************
    Processing Record 33 of Set 4 | karratha,au
    Data found (status code: 200) | karratha,au
    Data loaded successfully | karratha,au
    *********************************
    Processing Record 34 of Set 4 | san joaquin,bo
    Data found (status code: 200) | san joaquin,bo
    Data loaded successfully | san joaquin,bo
    *********************************
    Processing Record 35 of Set 4 | cochabamba,bo
    Data found (status code: 200) | cochabamba,bo
    Data loaded successfully | cochabamba,bo
    *********************************
    Processing Record 36 of Set 4 | chimoio,mz
    Data found (status code: 200) | chimoio,mz
    Data loaded successfully | chimoio,mz
    *********************************
    Processing Record 37 of Set 4 | namibe,ao
    Data found (status code: 200) | namibe,ao
    Data loaded successfully | namibe,ao
    *********************************
    Processing Record 38 of Set 4 | eenhana,na
    Data found (status code: 200) | eenhana,na
    Data loaded successfully | eenhana,na
    *********************************
    Processing Record 39 of Set 4 | vaitape,pf
    Data found (status code: 200) | vaitape,pf
    Data loaded successfully | vaitape,pf
    *********************************
    Processing Record 40 of Set 4 | puerto quijarro,bo
    Data found (status code: 200) | puerto quijarro,bo
    Data loaded successfully | puerto quijarro,bo
    *********************************
    Processing Record 41 of Set 4 | viloco,bo
    Data found (status code: 200) | viloco,bo
    Data loaded successfully | viloco,bo
    *********************************
    Processing Record 42 of Set 4 | vavatenina,mg
    Data found (status code: 200) | vavatenina,mg
    Data loaded successfully | vavatenina,mg
    *********************************
    Processing Record 43 of Set 4 | angoche,mz
    Data found (status code: 200) | angoche,mz
    Data loaded successfully | angoche,mz
    *********************************
    Processing Record 44 of Set 4 | grand gaube,mu
    Data found (status code: 200) | grand gaube,mu
    Data loaded successfully | grand gaube,mu
    *********************************
    Processing Record 45 of Set 4 | neiafu,to
    Data found (status code: 200) | neiafu,to
    Data loaded successfully | neiafu,to
    *********************************
    Processing Record 46 of Set 4 | huarmey,pe
    Data found (status code: 200) | huarmey,pe
    Data loaded successfully | huarmey,pe
    *********************************
    Processing Record 47 of Set 4 | atuona,pf
    Data found (status code: 200) | atuona,pf
    Data loaded successfully | atuona,pf
    *********************************
    Processing Record 48 of Set 4 | alyangula,au
    Data found (status code: 200) | alyangula,au
    Data loaded successfully | alyangula,au
    *********************************
    Processing Record 49 of Set 4 | katherine,au
    Data found (status code: 200) | katherine,au
    Data loaded successfully | katherine,au
    *********************************
    Processing Record 50 of Set 4 | cap malheureux,mu
    Data found (status code: 200) | cap malheureux,mu
    Data loaded successfully | cap malheureux,mu
    *********************************
    Processing Record 51 of Set 4 | mocuba,mz
    Data found (status code: 200) | mocuba,mz
    Data loaded successfully | mocuba,mz
    *********************************
    Processing Record 52 of Set 4 | paracatu,br
    Data found (status code: 200) | paracatu,br
    Data loaded successfully | paracatu,br
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 5 | srandakan,id
    Data found (status code: 200) | srandakan,id
    Data loaded successfully | srandakan,id
    *********************************
    Processing Record 1 of Set 5 | sesheke,zm
    Data found (status code: 200) | sesheke,zm
    Data loaded successfully | sesheke,zm
    *********************************
    Processing Record 2 of Set 5 | buritizeiro,br
    Data found (status code: 200) | buritizeiro,br
    Data loaded successfully | buritizeiro,br
    *********************************
    Processing Record 3 of Set 5 | kawalu,id
    Data found (status code: 200) | kawalu,id
    Data loaded successfully | kawalu,id
    *********************************
    Processing Record 4 of Set 5 | corrales,pe
    Data found (status code: 200) | corrales,pe
    Data loaded successfully | corrales,pe
    *********************************
    Processing Record 5 of Set 5 | mahajanga,mg
    Data found (status code: 200) | mahajanga,mg
    Data loaded successfully | mahajanga,mg
    *********************************
    Processing Record 6 of Set 5 | vila,vu
    ERROR: Data retrieval unsuccessful for vila,vu. Code: 404                       MSG: city not found
    Processing Record 7 of Set 5 | porto seguro,br
    Data found (status code: 200) | porto seguro,br
    Data loaded successfully | porto seguro,br
    *********************************
    Processing Record 8 of Set 5 | halalo,wf
    ERROR: Data retrieval unsuccessful for halalo,wf. Code: 404                       MSG: city not found
    Processing Record 9 of Set 5 | fare,pf
    Data found (status code: 200) | fare,pf
    Data loaded successfully | fare,pf
    *********************************
    Processing Record 10 of Set 5 | satitoa,ws
    ERROR: Data retrieval unsuccessful for satitoa,ws. Code: 404                       MSG: city not found
    Processing Record 11 of Set 5 | lalomanu,ws
    ERROR: Data retrieval unsuccessful for lalomanu,ws. Code: 404                       MSG: city not found
    Processing Record 12 of Set 5 | faanui,pf
    Data found (status code: 200) | faanui,pf
    Data loaded successfully | faanui,pf
    *********************************
    Processing Record 13 of Set 5 | belmonte,br
    Data found (status code: 200) | belmonte,br
    Data loaded successfully | belmonte,br
    *********************************
    Processing Record 14 of Set 5 | cuamba,mz
    Data found (status code: 200) | cuamba,mz
    Data loaded successfully | cuamba,mz
    *********************************
    Processing Record 15 of Set 5 | maragogi,br
    Data found (status code: 200) | maragogi,br
    Data loaded successfully | maragogi,br
    *********************************
    Processing Record 16 of Set 5 | ngukurr,au
    ERROR: Data retrieval unsuccessful for ngukurr,au. Code: 404                       MSG: city not found
    Processing Record 17 of Set 5 | bambanglipuro,id
    Data found (status code: 200) | bambanglipuro,id
    Data loaded successfully | bambanglipuro,id
    *********************************
    Processing Record 18 of Set 5 | mareeba,au
    Data found (status code: 200) | mareeba,au
    Data loaded successfully | mareeba,au
    *********************************
    Processing Record 19 of Set 5 | lima,pe
    Data found (status code: 200) | lima,pe
    Data loaded successfully | lima,pe
    *********************************
    Processing Record 20 of Set 5 | kirakira,sb
    Data found (status code: 200) | kirakira,sb
    Data loaded successfully | kirakira,sb
    *********************************
    Processing Record 21 of Set 5 | guiratinga,br
    Data found (status code: 200) | guiratinga,br
    Data loaded successfully | guiratinga,br
    *********************************
    Processing Record 22 of Set 5 | chilca,pe
    Data found (status code: 200) | chilca,pe
    Data loaded successfully | chilca,pe
    *********************************
    Processing Record 23 of Set 5 | luganville,vu
    Data found (status code: 200) | luganville,vu
    Data loaded successfully | luganville,vu
    *********************************
    Processing Record 24 of Set 5 | asau,tv
    ERROR: Data retrieval unsuccessful for asau,tv. Code: 404                       MSG: city not found
    Processing Record 25 of Set 5 | antsohihy,mg
    Data found (status code: 200) | antsohihy,mg
    Data loaded successfully | antsohihy,mg
    *********************************
    Processing Record 26 of Set 5 | antalaha,mg
    Data found (status code: 200) | antalaha,mg
    Data loaded successfully | antalaha,mg
    *********************************
    Processing Record 27 of Set 5 | caconda,ao
    Data found (status code: 200) | caconda,ao
    Data loaded successfully | caconda,ao
    *********************************
    Processing Record 28 of Set 5 | menongue,ao
    Data found (status code: 200) | menongue,ao
    Data loaded successfully | menongue,ao
    *********************************
    Processing Record 29 of Set 5 | san vicente de canete,pe
    Data found (status code: 200) | san vicente de canete,pe
    Data loaded successfully | san vicente de canete,pe
    *********************************
    Processing Record 30 of Set 5 | port keats,au
    Data found (status code: 200) | port keats,au
    Data loaded successfully | port keats,au
    *********************************
    Processing Record 31 of Set 5 | santa rosa,bo
    Data found (status code: 200) | santa rosa,bo
    Data loaded successfully | santa rosa,bo
    *********************************
    Processing Record 32 of Set 5 | cairns,au
    Data found (status code: 200) | cairns,au
    Data loaded successfully | cairns,au
    *********************************
    Processing Record 33 of Set 5 | sambava,mg
    Data found (status code: 200) | sambava,mg
    Data loaded successfully | sambava,mg
    *********************************
    Processing Record 34 of Set 5 | sola,vu
    Data found (status code: 200) | sola,vu
    Data loaded successfully | sola,vu
    *********************************
    Processing Record 35 of Set 5 | samusu,ws
    ERROR: Data retrieval unsuccessful for samusu,ws. Code: 404                       MSG: city not found
    Processing Record 36 of Set 5 | waingapu,id
    Data found (status code: 200) | waingapu,id
    Data loaded successfully | waingapu,id
    *********************************
    Processing Record 37 of Set 5 | barra do garcas,br
    Data found (status code: 200) | barra do garcas,br
    Data loaded successfully | barra do garcas,br
    *********************************
    Processing Record 38 of Set 5 | san ramon,bo
    Data found (status code: 200) | san ramon,bo
    Data loaded successfully | san ramon,bo
    *********************************
    Processing Record 39 of Set 5 | samarai,pg
    Data found (status code: 200) | samarai,pg
    Data loaded successfully | samarai,pg
    *********************************
    Processing Record 40 of Set 5 | parana,br
    Data found (status code: 200) | parana,br
    Data loaded successfully | parana,br
    *********************************
    Processing Record 41 of Set 5 | benguela,ao
    Data found (status code: 200) | benguela,ao
    Data loaded successfully | benguela,ao
    *********************************
    Processing Record 42 of Set 5 | chicama,pe
    Data found (status code: 200) | chicama,pe
    Data loaded successfully | chicama,pe
    *********************************
    Processing Record 43 of Set 5 | salvador,br
    Data found (status code: 200) | salvador,br
    Data loaded successfully | salvador,br
    *********************************
    Processing Record 44 of Set 5 | sao jose da coroa grande,br
    Data found (status code: 200) | sao jose da coroa grande,br
    Data loaded successfully | sao jose da coroa grande,br
    *********************************
    Processing Record 45 of Set 5 | honiara,sb
    Data found (status code: 200) | honiara,sb
    Data loaded successfully | honiara,sb
    *********************************
    Processing Record 46 of Set 5 | port moresby,pg
    Data found (status code: 200) | port moresby,pg
    Data loaded successfully | port moresby,pg
    *********************************
    Processing Record 47 of Set 5 | ndola,zm
    Data found (status code: 200) | ndola,zm
    Data loaded successfully | ndola,zm
    *********************************
    Processing Record 48 of Set 5 | dzaoudzi,yt
    Data found (status code: 200) | dzaoudzi,yt
    Data loaded successfully | dzaoudzi,yt
    *********************************
    Processing Record 49 of Set 5 | paratinga,br
    Data found (status code: 200) | paratinga,br
    Data loaded successfully | paratinga,br
    *********************************
    Processing Record 50 of Set 5 | conde,br
    Data found (status code: 200) | conde,br
    Data loaded successfully | conde,br
    *********************************
    Processing Record 51 of Set 5 | san cristobal,ec
    Data found (status code: 200) | san cristobal,ec
    Data loaded successfully | san cristobal,ec
    *********************************
    Processing Record 52 of Set 5 | chambishi,zm
    Data found (status code: 200) | chambishi,zm
    Data loaded successfully | chambishi,zm
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 6 | alotau,pg
    ERROR: Data retrieval unsuccessful for alotau,pg. Code: 404                       MSG: city not found
    Processing Record 1 of Set 6 | huambo,ao
    Data found (status code: 200) | huambo,ao
    Data loaded successfully | huambo,ao
    *********************************
    Processing Record 2 of Set 6 | domoni,km
    Data found (status code: 200) | domoni,km
    Data loaded successfully | domoni,km
    *********************************
    Processing Record 3 of Set 6 | lumeje,ao
    Data found (status code: 200) | lumeje,ao
    Data loaded successfully | lumeje,ao
    *********************************
    Processing Record 4 of Set 6 | san jeronimo de tunan,pe
    ERROR: Data retrieval unsuccessful for san jeronimo de tunan,pe. Code: 404                       MSG: city not found
    Processing Record 5 of Set 6 | pangoa,pe
    Data found (status code: 200) | pangoa,pe
    Data loaded successfully | pangoa,pe
    *********************************
    Processing Record 6 of Set 6 | namikupa,tz
    Data found (status code: 200) | namikupa,tz
    Data loaded successfully | namikupa,tz
    *********************************
    Processing Record 7 of Set 6 | vaitupu,wf
    ERROR: Data retrieval unsuccessful for vaitupu,wf. Code: 404                       MSG: city not found
    Processing Record 8 of Set 6 | huaral,pe
    Data found (status code: 200) | huaral,pe
    Data loaded successfully | huaral,pe
    *********************************
    Processing Record 9 of Set 6 | guayaramerin,bo
    Data found (status code: 200) | guayaramerin,bo
    Data loaded successfully | guayaramerin,bo
    *********************************
    Processing Record 10 of Set 6 | ribeira do pombal,br
    Data found (status code: 200) | ribeira do pombal,br
    Data loaded successfully | ribeira do pombal,br
    *********************************
    Processing Record 11 of Set 6 | nhulunbuy,au
    Data found (status code: 200) | nhulunbuy,au
    Data loaded successfully | nhulunbuy,au
    *********************************
    Processing Record 12 of Set 6 | lata,sb
    ERROR: Data retrieval unsuccessful for lata,sb. Code: 404                       MSG: city not found
    Processing Record 13 of Set 6 | alta floresta,br
    Data found (status code: 200) | alta floresta,br
    Data loaded successfully | alta floresta,br
    *********************************
    Processing Record 14 of Set 6 | mansa,zm
    Data found (status code: 200) | mansa,zm
    Data loaded successfully | mansa,zm
    *********************************
    Processing Record 15 of Set 6 | sechura,pe
    Data found (status code: 200) | sechura,pe
    Data loaded successfully | sechura,pe
    *********************************
    Processing Record 16 of Set 6 | barra,br
    Data found (status code: 200) | barra,br
    Data loaded successfully | barra,br
    *********************************
    Processing Record 17 of Set 6 | kamina,cd
    Data found (status code: 200) | kamina,cd
    Data loaded successfully | kamina,cd
    *********************************
    Processing Record 18 of Set 6 | luwingu,zm
    Data found (status code: 200) | luwingu,zm
    Data loaded successfully | luwingu,zm
    *********************************
    Processing Record 19 of Set 6 | ambilobe,mg
    Data found (status code: 200) | ambilobe,mg
    Data loaded successfully | ambilobe,mg
    *********************************
    Processing Record 20 of Set 6 | samalaeulu,ws
    ERROR: Data retrieval unsuccessful for samalaeulu,ws. Code: 404                       MSG: city not found
    Processing Record 21 of Set 6 | formoso do araguaia,br
    ERROR: Data retrieval unsuccessful for formoso do araguaia,br. Code: 404                       MSG: city not found
    Processing Record 22 of Set 6 | bom jesus,br
    Data found (status code: 200) | bom jesus,br
    Data loaded successfully | bom jesus,br
    *********************************
    Processing Record 23 of Set 6 | merauke,id
    Data found (status code: 200) | merauke,id
    Data loaded successfully | merauke,id
    *********************************
    Processing Record 24 of Set 6 | chimbote,pe
    Data found (status code: 200) | chimbote,pe
    Data loaded successfully | chimbote,pe
    *********************************
    Processing Record 25 of Set 6 | pilao arcado,br
    ERROR: Data retrieval unsuccessful for pilao arcado,br. Code: 404                       MSG: city not found
    Processing Record 26 of Set 6 | morros,br
    Data found (status code: 200) | morros,br
    Data loaded successfully | morros,br
    *********************************
    Processing Record 27 of Set 6 | luanda,ao
    Data found (status code: 200) | luanda,ao
    Data loaded successfully | luanda,ao
    *********************************
    Processing Record 28 of Set 6 | saurimo,ao
    Data found (status code: 200) | saurimo,ao
    Data loaded successfully | saurimo,ao
    *********************************
    Processing Record 29 of Set 6 | madimba,tz
    Data found (status code: 200) | madimba,tz
    Data loaded successfully | madimba,tz
    *********************************
    Processing Record 30 of Set 6 | olinda,br
    Data found (status code: 200) | olinda,br
    Data loaded successfully | olinda,br
    *********************************
    Processing Record 31 of Set 6 | nguiu,au
    ERROR: Data retrieval unsuccessful for nguiu,au. Code: 404                       MSG: city not found
    Processing Record 32 of Set 6 | sumbawa,id
    ERROR: Data retrieval unsuccessful for sumbawa,id. Code: 404                       MSG: city not found
    Processing Record 33 of Set 6 | mtwara,tz
    Data found (status code: 200) | mtwara,tz
    Data loaded successfully | mtwara,tz
    *********************************
    Processing Record 34 of Set 6 | campoverde,pe
    Data found (status code: 200) | campoverde,pe
    Data loaded successfully | campoverde,pe
    *********************************
    Processing Record 35 of Set 6 | miracema do tocantins,br
    Data found (status code: 200) | miracema do tocantins,br
    Data loaded successfully | miracema do tocantins,br
    *********************************
    Processing Record 36 of Set 6 | soyo,ao
    Data found (status code: 200) | soyo,ao
    Data loaded successfully | soyo,ao
    *********************************
    Processing Record 37 of Set 6 | aripuana,br
    Data found (status code: 200) | aripuana,br
    Data loaded successfully | aripuana,br
    *********************************
    Processing Record 38 of Set 6 | bima,id
    Data found (status code: 200) | bima,id
    Data loaded successfully | bima,id
    *********************************
    Processing Record 39 of Set 6 | malanje,ao
    Data found (status code: 200) | malanje,ao
    Data loaded successfully | malanje,ao
    *********************************
    Processing Record 40 of Set 6 | ipinda,tz
    Data found (status code: 200) | ipinda,tz
    Data loaded successfully | ipinda,tz
    *********************************
    Processing Record 41 of Set 6 | conceicao do araguaia,br
    Data found (status code: 200) | conceicao do araguaia,br
    Data loaded successfully | conceicao do araguaia,br
    *********************************
    Processing Record 42 of Set 6 | atambua,id
    Data found (status code: 200) | atambua,id
    Data loaded successfully | atambua,id
    *********************************
    Processing Record 43 of Set 6 | matai,tz
    Data found (status code: 200) | matai,tz
    Data loaded successfully | matai,tz
    *********************************
    Processing Record 44 of Set 6 | gizo,sb
    Data found (status code: 200) | gizo,sb
    Data loaded successfully | gizo,sb
    *********************************
    Processing Record 45 of Set 6 | lolua,tv
    ERROR: Data retrieval unsuccessful for lolua,tv. Code: 404                       MSG: city not found
    Processing Record 46 of Set 6 | mbeya,tz
    Data found (status code: 200) | mbeya,tz
    Data loaded successfully | mbeya,tz
    *********************************
    Processing Record 47 of Set 6 | buin,pg
    ERROR: Data retrieval unsuccessful for buin,pg. Code: 404                       MSG: city not found
    Processing Record 48 of Set 6 | sao felix do xingu,br
    Data found (status code: 200) | sao felix do xingu,br
    Data loaded successfully | sao felix do xingu,br
    *********************************
    Processing Record 49 of Set 6 | mayumba,ga
    Data found (status code: 200) | mayumba,ga
    Data loaded successfully | mayumba,ga
    *********************************
    Processing Record 50 of Set 6 | ibimirim,br
    Data found (status code: 200) | ibimirim,br
    Data loaded successfully | ibimirim,br
    *********************************
    Processing Record 51 of Set 6 | jacareacanga,br
    Data found (status code: 200) | jacareacanga,br
    Data loaded successfully | jacareacanga,br
    *********************************
    Processing Record 52 of Set 6 | santa luzia,br
    Data found (status code: 200) | santa luzia,br
    Data loaded successfully | santa luzia,br
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 7 | mitsamiouli,km
    Data found (status code: 200) | mitsamiouli,km
    Data loaded successfully | mitsamiouli,km
    *********************************
    Processing Record 1 of Set 7 | singaparna,id
    Data found (status code: 200) | singaparna,id
    Data loaded successfully | singaparna,id
    *********************************
    Processing Record 2 of Set 7 | pitimbu,br
    Data found (status code: 200) | pitimbu,br
    Data loaded successfully | pitimbu,br
    *********************************
    Processing Record 3 of Set 7 | porto walter,br
    Data found (status code: 200) | porto walter,br
    Data loaded successfully | porto walter,br
    *********************************
    Processing Record 4 of Set 7 | victoria,sc
    Data found (status code: 200) | victoria,sc
    Data loaded successfully | victoria,sc
    *********************************
    Processing Record 5 of Set 7 | tual,id
    Data found (status code: 200) | tual,id
    Data loaded successfully | tual,id
    *********************************
    Processing Record 6 of Set 7 | gamba,ga
    Data found (status code: 200) | gamba,ga
    Data loaded successfully | gamba,ga
    *********************************
    Processing Record 7 of Set 7 | sumbawanga,tz
    Data found (status code: 200) | sumbawanga,tz
    Data loaded successfully | sumbawanga,tz
    *********************************
    Processing Record 8 of Set 7 | morehead,pg
    Data found (status code: 200) | morehead,pg
    Data loaded successfully | morehead,pg
    *********************************
    Processing Record 9 of Set 7 | araripina,br
    Data found (status code: 200) | araripina,br
    Data loaded successfully | araripina,br
    *********************************
    Processing Record 10 of Set 7 | tanete,id
    Data found (status code: 200) | tanete,id
    Data loaded successfully | tanete,id
    *********************************
    Processing Record 11 of Set 7 | talara,pe
    Data found (status code: 200) | talara,pe
    Data loaded successfully | talara,pe
    *********************************
    Processing Record 12 of Set 7 | kimbe,pg
    Data found (status code: 200) | kimbe,pg
    Data loaded successfully | kimbe,pg
    *********************************
    Processing Record 13 of Set 7 | sao bento,br
    ERROR: Data retrieval unsuccessful for sao bento,br. Code: 404                       MSG: city not found
    Processing Record 14 of Set 7 | canutama,br
    Data found (status code: 200) | canutama,br
    Data loaded successfully | canutama,br
    *********************************
    Processing Record 15 of Set 7 | sao geraldo do araguaia,br
    Data found (status code: 200) | sao geraldo do araguaia,br
    Data loaded successfully | sao geraldo do araguaia,br
    *********************************
    Processing Record 16 of Set 7 | elesbao veloso,br
    Data found (status code: 200) | elesbao veloso,br
    Data loaded successfully | elesbao veloso,br
    *********************************
    Processing Record 17 of Set 7 | lamas,pe
    Data found (status code: 200) | lamas,pe
    Data loaded successfully | lamas,pe
    *********************************
    Processing Record 18 of Set 7 | dibaya,cd
    ERROR: Data retrieval unsuccessful for dibaya,cd. Code: 404                       MSG: city not found
    Processing Record 19 of Set 7 | matadi,cd
    Data found (status code: 200) | matadi,cd
    Data loaded successfully | matadi,cd
    *********************************
    Processing Record 20 of Set 7 | singaraja,id
    Data found (status code: 200) | singaraja,id
    Data loaded successfully | singaraja,id
    *********************************
    Processing Record 21 of Set 7 | katobu,id
    Data found (status code: 200) | katobu,id
    Data loaded successfully | katobu,id
    *********************************
    Processing Record 22 of Set 7 | mirador,br
    Data found (status code: 200) | mirador,br
    Data loaded successfully | mirador,br
    *********************************
    Processing Record 23 of Set 7 | cabedelo,br
    Data found (status code: 200) | cabedelo,br
    Data loaded successfully | cabedelo,br
    *********************************
    Processing Record 24 of Set 7 | galesong,id
    Data found (status code: 200) | galesong,id
    Data loaded successfully | galesong,id
    *********************************
    Processing Record 25 of Set 7 | vice,pe
    Data found (status code: 200) | vice,pe
    Data loaded successfully | vice,pe
    *********************************
    Processing Record 26 of Set 7 | kabalo,cd
    Data found (status code: 200) | kabalo,cd
    Data loaded successfully | kabalo,cd
    *********************************
    Processing Record 27 of Set 7 | kananga,cd
    Data found (status code: 200) | kananga,cd
    Data loaded successfully | kananga,cd
    *********************************
    Processing Record 28 of Set 7 | kiunga,pg
    Data found (status code: 200) | kiunga,pg
    Data loaded successfully | kiunga,pg
    *********************************
    Processing Record 29 of Set 7 | mpanda,tz
    Data found (status code: 200) | mpanda,tz
    Data loaded successfully | mpanda,tz
    *********************************
    Processing Record 30 of Set 7 | gairo,tz
    Data found (status code: 200) | gairo,tz
    Data loaded successfully | gairo,tz
    *********************************
    Processing Record 31 of Set 7 | kainantu,pg
    Data found (status code: 200) | kainantu,pg
    Data loaded successfully | kainantu,pg
    *********************************
    Processing Record 32 of Set 7 | saleaula,ws
    ERROR: Data retrieval unsuccessful for saleaula,ws. Code: 404                       MSG: city not found
    Processing Record 33 of Set 7 | auki,sb
    Data found (status code: 200) | auki,sb
    Data loaded successfully | auki,sb
    *********************************
    Processing Record 34 of Set 7 | indramayu,id
    Data found (status code: 200) | indramayu,id
    Data loaded successfully | indramayu,id
    *********************************
    Processing Record 35 of Set 7 | lufilufi,ws
    Data found (status code: 200) | lufilufi,ws
    Data loaded successfully | lufilufi,ws
    *********************************
    Processing Record 36 of Set 7 | mbanza-ngungu,cd
    Data found (status code: 200) | mbanza-ngungu,cd
    Data loaded successfully | mbanza-ngungu,cd
    *********************************
    Processing Record 37 of Set 7 | itaituba,br
    Data found (status code: 200) | itaituba,br
    Data loaded successfully | itaituba,br
    *********************************
    Processing Record 38 of Set 7 | ambon,id
    Data found (status code: 200) | ambon,id
    Data loaded successfully | ambon,id
    *********************************
    Processing Record 39 of Set 7 | padang,id
    Data found (status code: 200) | padang,id
    Data loaded successfully | padang,id
    *********************************
    Processing Record 40 of Set 7 | itupiranga,br
    Data found (status code: 200) | itupiranga,br
    Data loaded successfully | itupiranga,br
    *********************************
    Processing Record 41 of Set 7 | metro,id
    Data found (status code: 200) | metro,id
    Data loaded successfully | metro,id
    *********************************
    Processing Record 42 of Set 7 | martapura,id
    Data found (status code: 200) | martapura,id
    Data loaded successfully | martapura,id
    *********************************
    Processing Record 43 of Set 7 | jutai,br
    Data found (status code: 200) | jutai,br
    Data loaded successfully | jutai,br
    *********************************
    Processing Record 44 of Set 7 | madang,pg
    Data found (status code: 200) | madang,pg
    Data loaded successfully | madang,pg
    *********************************
    Processing Record 45 of Set 7 | natal,br
    Data found (status code: 200) | natal,br
    Data loaded successfully | natal,br
    *********************************
    Processing Record 46 of Set 7 | baturaja,id
    Data found (status code: 200) | baturaja,id
    Data loaded successfully | baturaja,id
    *********************************
    Processing Record 47 of Set 7 | lagunas,pe
    Data found (status code: 200) | lagunas,pe
    Data loaded successfully | lagunas,pe
    *********************************
    Processing Record 48 of Set 7 | kalemie,cd
    Data found (status code: 200) | kalemie,cd
    Data loaded successfully | kalemie,cd
    *********************************
    Processing Record 49 of Set 7 | altamira,br
    Data found (status code: 200) | altamira,br
    Data loaded successfully | altamira,br
    *********************************
    Processing Record 50 of Set 7 | barranca,pe
    Data found (status code: 200) | barranca,pe
    Data loaded successfully | barranca,pe
    *********************************
    Processing Record 51 of Set 7 | angoram,pg
    Data found (status code: 200) | angoram,pg
    Data loaded successfully | angoram,pg
    *********************************
    Processing Record 52 of Set 7 | rabaul,pg
    Data found (status code: 200) | rabaul,pg
    Data loaded successfully | rabaul,pg
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 8 | meulaboh,id
    Data found (status code: 200) | meulaboh,id
    Data loaded successfully | meulaboh,id
    *********************************
    Processing Record 1 of Set 8 | kasongo,cd
    Data found (status code: 200) | kasongo,cd
    Data loaded successfully | kasongo,cd
    *********************************
    Processing Record 2 of Set 8 | borba,br
    Data found (status code: 200) | borba,br
    Data loaded successfully | borba,br
    *********************************
    Processing Record 3 of Set 8 | utiroa,ki
    ERROR: Data retrieval unsuccessful for utiroa,ki. Code: 404                       MSG: city not found
    Processing Record 4 of Set 8 | lamu,ke
    Data found (status code: 200) | lamu,ke
    Data loaded successfully | lamu,ke
    *********************************
    Processing Record 5 of Set 8 | kishapu,tz
    Data found (status code: 200) | kishapu,tz
    Data loaded successfully | kishapu,tz
    *********************************
    Processing Record 6 of Set 8 | mogadishu,so
    Data found (status code: 200) | mogadishu,so
    Data loaded successfully | mogadishu,so
    *********************************
    Processing Record 7 of Set 8 | lodja,cd
    Data found (status code: 200) | lodja,cd
    Data loaded successfully | lodja,cd
    *********************************
    Processing Record 8 of Set 8 | amahai,id
    Data found (status code: 200) | amahai,id
    Data loaded successfully | amahai,id
    *********************************
    Processing Record 9 of Set 8 | sibolga,id
    Data found (status code: 200) | sibolga,id
    Data loaded successfully | sibolga,id
    *********************************
    Processing Record 10 of Set 8 | manacapuru,br
    Data found (status code: 200) | manacapuru,br
    Data loaded successfully | manacapuru,br
    *********************************
    Processing Record 11 of Set 8 | banjarmasin,id
    Data found (status code: 200) | banjarmasin,id
    Data loaded successfully | banjarmasin,id
    *********************************
    Processing Record 12 of Set 8 | axim,gh
    Data found (status code: 200) | axim,gh
    Data loaded successfully | axim,gh
    *********************************
    Processing Record 13 of Set 8 | iquitos,pe
    Data found (status code: 200) | iquitos,pe
    Data loaded successfully | iquitos,pe
    *********************************
    Processing Record 14 of Set 8 | port-gentil,ga
    Data found (status code: 200) | port-gentil,ga
    Data loaded successfully | port-gentil,ga
    *********************************
    Processing Record 15 of Set 8 | mossendjo,cg
    Data found (status code: 200) | mossendjo,cg
    Data loaded successfully | mossendjo,cg
    *********************************
    Processing Record 16 of Set 8 | bandundu,cd
    Data found (status code: 200) | bandundu,cd
    Data loaded successfully | bandundu,cd
    *********************************
    Processing Record 17 of Set 8 | sibiti,cg
    Data found (status code: 200) | sibiti,cg
    Data loaded successfully | sibiti,cg
    *********************************
    Processing Record 18 of Set 8 | carauari,br
    Data found (status code: 200) | carauari,br
    Data loaded successfully | carauari,br
    *********************************
    Processing Record 19 of Set 8 | aitape,pg
    Data found (status code: 200) | aitape,pg
    Data loaded successfully | aitape,pg
    *********************************
    Processing Record 20 of Set 8 | rantepao,id
    Data found (status code: 200) | rantepao,id
    Data loaded successfully | rantepao,id
    *********************************
    Processing Record 21 of Set 8 | manggar,id
    Data found (status code: 200) | manggar,id
    Data loaded successfully | manggar,id
    *********************************
    Processing Record 22 of Set 8 | salinas,ec
    Data found (status code: 200) | salinas,ec
    Data loaded successfully | salinas,ec
    *********************************
    Processing Record 23 of Set 8 | kieta,pg
    Data found (status code: 200) | kieta,pg
    Data loaded successfully | kieta,pg
    *********************************
    Processing Record 24 of Set 8 | nioki,cd
    Data found (status code: 200) | nioki,cd
    Data loaded successfully | nioki,cd
    *********************************
    Processing Record 25 of Set 8 | hambantota,lk
    Data found (status code: 200) | hambantota,lk
    Data loaded successfully | hambantota,lk
    *********************************
    Processing Record 26 of Set 8 | itacoatiara,br
    Data found (status code: 200) | itacoatiara,br
    Data loaded successfully | itacoatiara,br
    *********************************
    Processing Record 27 of Set 8 | el alto,pe
    ERROR: Data retrieval unsuccessful for el alto,pe. Code: 404                       MSG: city not found
    Processing Record 28 of Set 8 | voi,ke
    Data found (status code: 200) | voi,ke
    Data loaded successfully | voi,ke
    *********************************
    Processing Record 29 of Set 8 | nabire,id
    Data found (status code: 200) | nabire,id
    Data loaded successfully | nabire,id
    *********************************
    Processing Record 30 of Set 8 | biharamulo,tz
    Data found (status code: 200) | biharamulo,tz
    Data loaded successfully | biharamulo,tz
    *********************************
    Processing Record 31 of Set 8 | omboue,ga
    Data found (status code: 200) | omboue,ga
    Data loaded successfully | omboue,ga
    *********************************
    Processing Record 32 of Set 8 | mentok,id
    ERROR: Data retrieval unsuccessful for mentok,id. Code: 404                       MSG: city not found
    Processing Record 33 of Set 8 | tabiauea,ki
    ERROR: Data retrieval unsuccessful for tabiauea,ki. Code: 404                       MSG: city not found
    Processing Record 34 of Set 8 | tabou,ci
    Data found (status code: 200) | tabou,ci
    Data loaded successfully | tabou,ci
    *********************************
    Processing Record 35 of Set 8 | alausi,ec
    Data found (status code: 200) | alausi,ec
    Data loaded successfully | alausi,ec
    *********************************
    Processing Record 36 of Set 8 | barreirinhas,br
    Data found (status code: 200) | barreirinhas,br
    Data loaded successfully | barreirinhas,br
    *********************************
    Processing Record 37 of Set 8 | aquiraz,br
    Data found (status code: 200) | aquiraz,br
    Data loaded successfully | aquiraz,br
    *********************************
    Processing Record 38 of Set 8 | kismayo,so
    ERROR: Data retrieval unsuccessful for kismayo,so. Code: 404                       MSG: city not found
    Processing Record 39 of Set 8 | biak,id
    Data found (status code: 200) | biak,id
    Data loaded successfully | biak,id
    *********************************
    Processing Record 40 of Set 8 | coahuayana,mx
    Data found (status code: 200) | coahuayana,mx
    Data loaded successfully | coahuayana,mx
    *********************************
    Processing Record 41 of Set 8 | lorengau,pg
    Data found (status code: 200) | lorengau,pg
    Data loaded successfully | lorengau,pg
    *********************************
    Processing Record 42 of Set 8 | luis correia,br
    Data found (status code: 200) | luis correia,br
    Data loaded successfully | luis correia,br
    *********************************
    Processing Record 43 of Set 8 | itarema,br
    Data found (status code: 200) | itarema,br
    Data loaded successfully | itarema,br
    *********************************
    Processing Record 44 of Set 8 | fonte boa,br
    Data found (status code: 200) | fonte boa,br
    Data loaded successfully | fonte boa,br
    *********************************
    Processing Record 45 of Set 8 | barcelos,br
    Data found (status code: 200) | barcelos,br
    Data loaded successfully | barcelos,br
    *********************************
    Processing Record 46 of Set 8 | rungata,ki
    ERROR: Data retrieval unsuccessful for rungata,ki. Code: 404                       MSG: city not found
    Processing Record 47 of Set 8 | puerto leguizamo,co
    Data found (status code: 200) | puerto leguizamo,co
    Data loaded successfully | puerto leguizamo,co
    *********************************
    Processing Record 48 of Set 8 | matara,lk
    Data found (status code: 200) | matara,lk
    Data loaded successfully | matara,lk
    *********************************
    Processing Record 49 of Set 8 | kindu,cd
    Data found (status code: 200) | kindu,cd
    Data loaded successfully | kindu,cd
    *********************************
    Processing Record 50 of Set 8 | luwuk,id
    Data found (status code: 200) | luwuk,id
    Data loaded successfully | luwuk,id
    *********************************
    Processing Record 51 of Set 8 | bontang,id
    Data found (status code: 200) | bontang,id
    Data loaded successfully | bontang,id
    *********************************
    Processing Record 52 of Set 8 | poso,id
    Data found (status code: 200) | poso,id
    Data loaded successfully | poso,id
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 9 | bushenyi,ug
    Data found (status code: 200) | bushenyi,ug
    Data loaded successfully | bushenyi,ug
    *********************************
    Processing Record 1 of Set 9 | kitui,ke
    Data found (status code: 200) | kitui,ke
    Data loaded successfully | kitui,ke
    *********************************
    Processing Record 2 of Set 9 | acarau,br
    ERROR: Data retrieval unsuccessful for acarau,br. Code: 404                       MSG: city not found
    Processing Record 3 of Set 9 | pontianak,id
    Data found (status code: 200) | pontianak,id
    Data loaded successfully | pontianak,id
    *********************************
    Processing Record 4 of Set 9 | tidore,id
    ERROR: Data retrieval unsuccessful for tidore,id. Code: 404                       MSG: city not found
    Processing Record 5 of Set 9 | barawe,so
    ERROR: Data retrieval unsuccessful for barawe,so. Code: 404                       MSG: city not found
    Processing Record 6 of Set 9 | mbandaka,cd
    Data found (status code: 200) | mbandaka,cd
    Data loaded successfully | mbandaka,cd
    *********************************
    Processing Record 7 of Set 9 | santa isabel do rio negro,br
    Data found (status code: 200) | santa isabel do rio negro,br
    Data loaded successfully | santa isabel do rio negro,br
    *********************************
    Processing Record 8 of Set 9 | cururupu,br
    Data found (status code: 200) | cururupu,br
    Data loaded successfully | cururupu,br
    *********************************
    Processing Record 9 of Set 9 | viligili,mv
    ERROR: Data retrieval unsuccessful for viligili,mv. Code: 404                       MSG: city not found
    Processing Record 10 of Set 9 | namatanai,pg
    Data found (status code: 200) | namatanai,pg
    Data loaded successfully | namatanai,pg
    *********************************
    Processing Record 11 of Set 9 | thinadhoo,mv
    Data found (status code: 200) | thinadhoo,mv
    Data loaded successfully | thinadhoo,mv
    *********************************
    Processing Record 12 of Set 9 | balaipungut,id
    Data found (status code: 200) | balaipungut,id
    Data loaded successfully | balaipungut,id
    *********************************
    Processing Record 13 of Set 9 | san gabriel,ec
    Data found (status code: 200) | san gabriel,ec
    Data loaded successfully | san gabriel,ec
    *********************************
    Processing Record 14 of Set 9 | kisangani,cd
    Data found (status code: 200) | kisangani,cd
    Data loaded successfully | kisangani,cd
    *********************************
    Processing Record 15 of Set 9 | san patricio,mx
    Data found (status code: 200) | san patricio,mx
    Data loaded successfully | san patricio,mx
    *********************************
    Processing Record 16 of Set 9 | ouesso,cg
    Data found (status code: 200) | ouesso,cg
    Data loaded successfully | ouesso,cg
    *********************************
    Processing Record 17 of Set 9 | prainha,br
    Data found (status code: 200) | prainha,br
    Data loaded successfully | prainha,br
    *********************************
    Processing Record 18 of Set 9 | jamame,so
    Data found (status code: 200) | jamame,so
    Data loaded successfully | jamame,so
    *********************************
    Processing Record 19 of Set 9 | muisne,ec
    Data found (status code: 200) | muisne,ec
    Data loaded successfully | muisne,ec
    *********************************
    Processing Record 20 of Set 9 | touros,br
    Data found (status code: 200) | touros,br
    Data loaded successfully | touros,br
    *********************************
    Processing Record 21 of Set 9 | basoko,cd
    Data found (status code: 200) | basoko,cd
    Data loaded successfully | basoko,cd
    *********************************
    Processing Record 22 of Set 9 | ixtapa,mx
    Data found (status code: 200) | ixtapa,mx
    Data loaded successfully | ixtapa,mx
    *********************************
    Processing Record 23 of Set 9 | kapit,my
    Data found (status code: 200) | kapit,my
    Data loaded successfully | kapit,my
    *********************************
    Processing Record 24 of Set 9 | aconibe,gq
    Data found (status code: 200) | aconibe,gq
    Data loaded successfully | aconibe,gq
    *********************************
    Processing Record 25 of Set 9 | singkawang,id
    ERROR: Data retrieval unsuccessful for singkawang,id. Code: 404                       MSG: city not found
    Processing Record 26 of Set 9 | sembe,cg
    Data found (status code: 200) | sembe,cg
    Data loaded successfully | sembe,cg
    *********************************
    Processing Record 27 of Set 9 | makokou,ga
    Data found (status code: 200) | makokou,ga
    Data loaded successfully | makokou,ga
    *********************************
    Processing Record 28 of Set 9 | hobyo,so
    Data found (status code: 200) | hobyo,so
    Data loaded successfully | hobyo,so
    *********************************
    Processing Record 29 of Set 9 | buariki,ki
    ERROR: Data retrieval unsuccessful for buariki,ki. Code: 404                       MSG: city not found
    Processing Record 30 of Set 9 | sao gabriel da cachoeira,br
    Data found (status code: 200) | sao gabriel da cachoeira,br
    Data loaded successfully | sao gabriel da cachoeira,br
    *********************************
    Processing Record 31 of Set 9 | gorontalo,id
    Data found (status code: 200) | gorontalo,id
    Data loaded successfully | gorontalo,id
    *********************************
    Processing Record 32 of Set 9 | butaritari,ki
    Data found (status code: 200) | butaritari,ki
    Data loaded successfully | butaritari,ki
    *********************************
    Processing Record 33 of Set 9 | kudahuvadhoo,mv
    Data found (status code: 200) | kudahuvadhoo,mv
    Data loaded successfully | kudahuvadhoo,mv
    *********************************
    Processing Record 34 of Set 9 | bengkalis,id
    ERROR: Data retrieval unsuccessful for bengkalis,id. Code: 404                       MSG: city not found
    Processing Record 35 of Set 9 | kuching,my
    Data found (status code: 200) | kuching,my
    Data loaded successfully | kuching,my
    *********************************
    Processing Record 36 of Set 9 | luba,gq
    Data found (status code: 200) | luba,gq
    Data loaded successfully | luba,gq
    *********************************
    Processing Record 37 of Set 9 | rawannawi,ki
    ERROR: Data retrieval unsuccessful for rawannawi,ki. Code: 404                       MSG: city not found
    Processing Record 38 of Set 9 | carutapera,br
    Data found (status code: 200) | carutapera,br
    Data loaded successfully | carutapera,br
    *********************************
    Processing Record 39 of Set 9 | wamba,cd
    Data found (status code: 200) | wamba,cd
    Data loaded successfully | wamba,cd
    *********************************
    Processing Record 40 of Set 9 | ebebiyin,gq
    Data found (status code: 200) | ebebiyin,gq
    Data loaded successfully | ebebiyin,gq
    *********************************
    Processing Record 41 of Set 9 | ternate,id
    Data found (status code: 200) | ternate,id
    Data loaded successfully | ternate,id
    *********************************
    Processing Record 42 of Set 9 | banda aceh,id
    Data found (status code: 200) | banda aceh,id
    Data loaded successfully | banda aceh,id
    *********************************
    Processing Record 43 of Set 9 | salinopolis,br
    Data found (status code: 200) | salinopolis,br
    Data loaded successfully | salinopolis,br
    *********************************
    Processing Record 44 of Set 9 | manokwari,id
    Data found (status code: 200) | manokwari,id
    Data loaded successfully | manokwari,id
    *********************************
    Processing Record 45 of Set 9 | watsa,cd
    Data found (status code: 200) | watsa,cd
    Data loaded successfully | watsa,cd
    *********************************
    Processing Record 46 of Set 9 | impfondo,cg
    Data found (status code: 200) | impfondo,cg
    Data loaded successfully | impfondo,cg
    *********************************
    Processing Record 47 of Set 9 | amapa,br
    Data found (status code: 200) | amapa,br
    Data loaded successfully | amapa,br
    *********************************
    Processing Record 48 of Set 9 | sibu,my
    Data found (status code: 200) | sibu,my
    Data loaded successfully | sibu,my
    *********************************
    Processing Record 49 of Set 9 | tutoia,br
    Data found (status code: 200) | tutoia,br
    Data loaded successfully | tutoia,br
    *********************************
    Processing Record 50 of Set 9 | kavieng,pg
    Data found (status code: 200) | kavieng,pg
    Data loaded successfully | kavieng,pg
    *********************************
    Processing Record 51 of Set 9 | grand-santi,gf
    Data found (status code: 200) | grand-santi,gf
    Data loaded successfully | grand-santi,gf
    *********************************
    Processing Record 52 of Set 9 | buta,cd
    Data found (status code: 200) | buta,cd
    Data loaded successfully | buta,cd
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 10 | kabanjahe,id
    Data found (status code: 200) | kabanjahe,id
    Data loaded successfully | kabanjahe,id
    *********************************
    Processing Record 1 of Set 10 | mattru,sl
    Data found (status code: 200) | mattru,sl
    Data loaded successfully | mattru,sl
    *********************************
    Processing Record 2 of Set 10 | gemena,cd
    Data found (status code: 200) | gemena,cd
    Data loaded successfully | gemena,cd
    *********************************
    Processing Record 3 of Set 10 | bumba,cd
    Data found (status code: 200) | bumba,cd
    Data loaded successfully | bumba,cd
    *********************************
    Processing Record 4 of Set 10 | argelia,co
    Data found (status code: 200) | argelia,co
    Data loaded successfully | argelia,co
    *********************************
    Processing Record 5 of Set 10 | brokopondo,sr
    Data found (status code: 200) | brokopondo,sr
    Data loaded successfully | brokopondo,sr
    *********************************
    Processing Record 6 of Set 10 | lira,ug
    Data found (status code: 200) | lira,ug
    Data loaded successfully | lira,ug
    *********************************
    Processing Record 7 of Set 10 | hilo,us
    Data found (status code: 200) | hilo,us
    Data loaded successfully | hilo,us
    *********************************
    Processing Record 8 of Set 10 | sorong,id
    Data found (status code: 200) | sorong,id
    Data loaded successfully | sorong,id
    *********************************
    Processing Record 9 of Set 10 | isiro,cd
    Data found (status code: 200) | isiro,cd
    Data loaded successfully | isiro,cd
    *********************************
    Processing Record 10 of Set 10 | lodwar,ke
    Data found (status code: 200) | lodwar,ke
    Data loaded successfully | lodwar,ke
    *********************************
    Processing Record 11 of Set 10 | yumbe,ug
    Data found (status code: 200) | yumbe,ug
    Data loaded successfully | yumbe,ug
    *********************************
    Processing Record 12 of Set 10 | akonolinga,cm
    Data found (status code: 200) | akonolinga,cm
    Data loaded successfully | akonolinga,cm
    *********************************
    Processing Record 13 of Set 10 | inirida,co
    Data found (status code: 200) | inirida,co
    Data loaded successfully | inirida,co
    *********************************
    Processing Record 14 of Set 10 | mandera,ke
    Data found (status code: 200) | mandera,ke
    Data loaded successfully | mandera,ke
    *********************************
    Processing Record 15 of Set 10 | takoradi,gh
    Data found (status code: 200) | takoradi,gh
    Data loaded successfully | takoradi,gh
    *********************************
    Processing Record 16 of Set 10 | puerto baquerizo moreno,ec
    Data found (status code: 200) | puerto baquerizo moreno,ec
    Data loaded successfully | puerto baquerizo moreno,ec
    *********************************
    Processing Record 17 of Set 10 | acapulco,mx
    Data found (status code: 200) | acapulco,mx
    Data loaded successfully | acapulco,mx
    *********************************
    Processing Record 18 of Set 10 | burica,pa
    ERROR: Data retrieval unsuccessful for burica,pa. Code: 404                       MSG: city not found
    Processing Record 19 of Set 10 | timbiqui,co
    Data found (status code: 200) | timbiqui,co
    Data loaded successfully | timbiqui,co
    *********************************
    Processing Record 20 of Set 10 | abonnema,ng
    Data found (status code: 200) | abonnema,ng
    Data loaded successfully | abonnema,ng
    *********************************
    Processing Record 21 of Set 10 | kuantan,my
    Data found (status code: 200) | kuantan,my
    Data loaded successfully | kuantan,my
    *********************************
    Processing Record 22 of Set 10 | mentakab,my
    Data found (status code: 200) | mentakab,my
    Data loaded successfully | mentakab,my
    *********************************
    Processing Record 23 of Set 10 | boa vista,br
    Data found (status code: 200) | boa vista,br
    Data loaded successfully | boa vista,br
    *********************************
    Processing Record 24 of Set 10 | sao filipe,cv
    Data found (status code: 200) | sao filipe,cv
    Data loaded successfully | sao filipe,cv
    *********************************
    Processing Record 25 of Set 10 | boda,cf
    Data found (status code: 200) | boda,cf
    Data loaded successfully | boda,cf
    *********************************
    Processing Record 26 of Set 10 | mahibadhoo,mv
    Data found (status code: 200) | mahibadhoo,mv
    Data loaded successfully | mahibadhoo,mv
    *********************************
    Processing Record 27 of Set 10 | manuk mangkaw,ph
    Data found (status code: 200) | manuk mangkaw,ph
    Data loaded successfully | manuk mangkaw,ph
    *********************************
    Processing Record 28 of Set 10 | pundaguitan,ph
    Data found (status code: 200) | pundaguitan,ph
    Data loaded successfully | pundaguitan,ph
    *********************************
    Processing Record 29 of Set 10 | wageningen,sr
    Data found (status code: 200) | wageningen,sr
    Data loaded successfully | wageningen,sr
    *********************************
    Processing Record 30 of Set 10 | bubaque,gw
    Data found (status code: 200) | bubaque,gw
    Data loaded successfully | bubaque,gw
    *********************************
    Processing Record 31 of Set 10 | xuddur,so
    Data found (status code: 200) | xuddur,so
    Data loaded successfully | xuddur,so
    *********************************
    Processing Record 32 of Set 10 | bintulu,my
    Data found (status code: 200) | bintulu,my
    Data loaded successfully | bintulu,my
    *********************************
    Processing Record 33 of Set 10 | sigli,id
    Data found (status code: 200) | sigli,id
    Data loaded successfully | sigli,id
    *********************************
    Processing Record 34 of Set 10 | ugoofaaru,mv
    Data found (status code: 200) | ugoofaaru,mv
    Data loaded successfully | ugoofaaru,mv
    *********************************
    Processing Record 35 of Set 10 | puerto escondido,mx
    Data found (status code: 200) | puerto escondido,mx
    Data loaded successfully | puerto escondido,mx
    *********************************
    Processing Record 36 of Set 10 | kloulklubed,pw
    Data found (status code: 200) | kloulklubed,pw
    Data loaded successfully | kloulklubed,pw
    *********************************
    Processing Record 37 of Set 10 | bonthe,sl
    Data found (status code: 200) | bonthe,sl
    Data loaded successfully | bonthe,sl
    *********************************
    Processing Record 38 of Set 10 | marang,my
    Data found (status code: 200) | marang,my
    Data loaded successfully | marang,my
    *********************************
    Processing Record 39 of Set 10 | galle,lk
    Data found (status code: 200) | galle,lk
    Data loaded successfully | galle,lk
    *********************************
    Processing Record 40 of Set 10 | ciudad bolivar,ve
    Data found (status code: 200) | ciudad bolivar,ve
    Data loaded successfully | ciudad bolivar,ve
    *********************************
    Processing Record 41 of Set 10 | la palma,pa
    Data found (status code: 200) | la palma,pa
    Data loaded successfully | la palma,pa
    *********************************
    Processing Record 42 of Set 10 | ouidah,bj
    Data found (status code: 200) | ouidah,bj
    Data loaded successfully | ouidah,bj
    *********************************
    Processing Record 43 of Set 10 | marienburg,sr
    Data found (status code: 200) | marienburg,sr
    Data loaded successfully | marienburg,sr
    *********************************
    Processing Record 44 of Set 10 | tuburan,ph
    Data found (status code: 200) | tuburan,ph
    Data loaded successfully | tuburan,ph
    *********************************
    Processing Record 45 of Set 10 | labuan,my
    Data found (status code: 200) | labuan,my
    Data loaded successfully | labuan,my
    *********************************
    Processing Record 46 of Set 10 | kalmunai,lk
    Data found (status code: 200) | kalmunai,lk
    Data loaded successfully | kalmunai,lk
    *********************************
    Processing Record 47 of Set 10 | cayenne,gf
    Data found (status code: 200) | cayenne,gf
    Data loaded successfully | cayenne,gf
    *********************************
    Processing Record 48 of Set 10 | juba,sd
    ERROR: Data retrieval unsuccessful for juba,sd. Code: 404                       MSG: city not found
    Processing Record 49 of Set 10 | bouar,cf
    Data found (status code: 200) | bouar,cf
    Data loaded successfully | bouar,cf
    *********************************
    Processing Record 50 of Set 10 | bandarbeyla,so
    Data found (status code: 200) | bandarbeyla,so
    Data loaded successfully | bandarbeyla,so
    *********************************
    Processing Record 51 of Set 10 | betong,th
    Data found (status code: 200) | betong,th
    Data loaded successfully | betong,th
    *********************************
    Processing Record 52 of Set 10 | epe,ng
    Data found (status code: 200) | epe,ng
    Data loaded successfully | epe,ng
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 11 | mizan teferi,et
    Data found (status code: 200) | mizan teferi,et
    Data loaded successfully | mizan teferi,et
    *********************************
    Processing Record 1 of Set 11 | airai,pw
    ERROR: Data retrieval unsuccessful for airai,pw. Code: 404                       MSG: city not found
    Processing Record 2 of Set 11 | iracoubo,gf
    Data found (status code: 200) | iracoubo,gf
    Data loaded successfully | iracoubo,gf
    *********************************
    Processing Record 3 of Set 11 | kuah,my
    Data found (status code: 200) | kuah,my
    Data loaded successfully | kuah,my
    *********************************
    Processing Record 4 of Set 11 | saint-georges,gf
    ERROR: Data retrieval unsuccessful for saint-georges,gf. Code: 404                       MSG: city not found
    Processing Record 5 of Set 11 | cabo san lucas,mx
    Data found (status code: 200) | cabo san lucas,mx
    Data loaded successfully | cabo san lucas,mx
    *********************************
    Processing Record 6 of Set 11 | makakilo city,us
    Data found (status code: 200) | makakilo city,us
    Data loaded successfully | makakilo city,us
    *********************************
    Processing Record 7 of Set 11 | san jose,gt
    Data found (status code: 200) | san jose,gt
    Data loaded successfully | san jose,gt
    *********************************
    Processing Record 8 of Set 11 | phuket,th
    Data found (status code: 200) | phuket,th
    Data loaded successfully | phuket,th
    *********************************
    Processing Record 9 of Set 11 | bozoum,cf
    Data found (status code: 200) | bozoum,cf
    Data loaded successfully | bozoum,cf
    *********************************
    Processing Record 10 of Set 11 | yirol,sd
    ERROR: Data retrieval unsuccessful for yirol,sd. Code: 404                       MSG: city not found
    Processing Record 11 of Set 11 | raga,sd
    ERROR: Data retrieval unsuccessful for raga,sd. Code: 404                       MSG: city not found
    Processing Record 12 of Set 11 | waw,sd
    ERROR: Data retrieval unsuccessful for waw,sd. Code: 404                       MSG: city not found
    Processing Record 13 of Set 11 | sinnamary,gf
    Data found (status code: 200) | sinnamary,gf
    Data loaded successfully | sinnamary,gf
    *********************************
    Processing Record 14 of Set 11 | mana,gf
    Data found (status code: 200) | mana,gf
    Data loaded successfully | mana,gf
    *********************************
    Processing Record 15 of Set 11 | santa cruz,cr
    Data found (status code: 200) | santa cruz,cr
    Data loaded successfully | santa cruz,cr
    *********************************
    Processing Record 16 of Set 11 | bocaranga,cf
    ERROR: Data retrieval unsuccessful for bocaranga,cf. Code: 404                       MSG: city not found
    Processing Record 17 of Set 11 | miri,my
    Data found (status code: 200) | miri,my
    Data loaded successfully | miri,my
    *********************************
    Processing Record 18 of Set 11 | dhidhdhoo,mv
    Data found (status code: 200) | dhidhdhoo,mv
    Data loaded successfully | dhidhdhoo,mv
    *********************************
    Processing Record 19 of Set 11 | bac lieu,vn
    ERROR: Data retrieval unsuccessful for bac lieu,vn. Code: 404                       MSG: city not found
    Processing Record 20 of Set 11 | champerico,gt
    Data found (status code: 200) | champerico,gt
    Data loaded successfully | champerico,gt
    *********************************
    Processing Record 21 of Set 11 | balabac,ph
    Data found (status code: 200) | balabac,ph
    Data loaded successfully | balabac,ph
    *********************************
    Processing Record 22 of Set 11 | beoumi,ci
    Data found (status code: 200) | beoumi,ci
    Data loaded successfully | beoumi,ci
    *********************************
    Processing Record 23 of Set 11 | ca mau,vn
    Data found (status code: 200) | ca mau,vn
    Data loaded successfully | ca mau,vn
    *********************************
    Processing Record 24 of Set 11 | tonj,sd
    ERROR: Data retrieval unsuccessful for tonj,sd. Code: 404                       MSG: city not found
    Processing Record 25 of Set 11 | zaraza,ve
    Data found (status code: 200) | zaraza,ve
    Data loaded successfully | zaraza,ve
    *********************************
    Processing Record 26 of Set 11 | calabozo,ve
    Data found (status code: 200) | calabozo,ve
    Data loaded successfully | calabozo,ve
    *********************************
    Processing Record 27 of Set 11 | thiruvananthapuram,in
    Data found (status code: 200) | thiruvananthapuram,in
    Data loaded successfully | thiruvananthapuram,in
    *********************************
    Processing Record 28 of Set 11 | cantagallo,co
    Data found (status code: 200) | cantagallo,co
    Data loaded successfully | cantagallo,co
    *********************************
    Processing Record 29 of Set 11 | sawkta,sl
    Data found (status code: 200) | sawkta,sl
    Data loaded successfully | sawkta,sl
    *********************************
    Processing Record 30 of Set 11 | kavaratti,in
    Data found (status code: 200) | kavaratti,in
    Data loaded successfully | kavaratti,in
    *********************************
    Processing Record 31 of Set 11 | odweyne,so
    ERROR: Data retrieval unsuccessful for odweyne,so. Code: 404                       MSG: city not found
    Processing Record 32 of Set 11 | bentiu,sd
    ERROR: Data retrieval unsuccessful for bentiu,sd. Code: 404                       MSG: city not found
    Processing Record 33 of Set 11 | bedele,et
    Data found (status code: 200) | bedele,et
    Data loaded successfully | bedele,et
    *********************************
    Processing Record 34 of Set 11 | gambela,et
    Data found (status code: 200) | gambela,et
    Data loaded successfully | gambela,et
    *********************************
    Processing Record 35 of Set 11 | jalingo,ng
    Data found (status code: 200) | jalingo,ng
    Data loaded successfully | jalingo,ng
    *********************************
    Processing Record 36 of Set 11 | asosa,et
    Data found (status code: 200) | asosa,et
    Data loaded successfully | asosa,et
    *********************************
    Processing Record 37 of Set 11 | nicoya,cr
    Data found (status code: 200) | nicoya,cr
    Data loaded successfully | nicoya,cr
    *********************************
    Processing Record 38 of Set 11 | siocon,ph
    Data found (status code: 200) | siocon,ph
    Data loaded successfully | siocon,ph
    *********************************
    Processing Record 39 of Set 11 | hatillo de loba,co
    Data found (status code: 200) | hatillo de loba,co
    Data loaded successfully | hatillo de loba,co
    *********************************
    Processing Record 40 of Set 11 | puerto escondido,co
    Data found (status code: 200) | puerto escondido,co
    Data loaded successfully | puerto escondido,co
    *********************************
    Processing Record 41 of Set 11 | bida,ng
    Data found (status code: 200) | bida,ng
    Data loaded successfully | bida,ng
    *********************************
    Processing Record 42 of Set 11 | yaring,th
    Data found (status code: 200) | yaring,th
    Data loaded successfully | yaring,th
    *********************************
    Processing Record 43 of Set 11 | altagracia de orituco,ve
    Data found (status code: 200) | altagracia de orituco,ve
    Data loaded successfully | altagracia de orituco,ve
    *********************************
    Processing Record 44 of Set 11 | yendi,gh
    Data found (status code: 200) | yendi,gh
    Data loaded successfully | yendi,gh
    *********************************
    Processing Record 45 of Set 11 | djougou,bj
    Data found (status code: 200) | djougou,bj
    Data loaded successfully | djougou,bj
    *********************************
    Processing Record 46 of Set 11 | gimbi,et
    Data found (status code: 200) | gimbi,et
    Data loaded successfully | gimbi,et
    *********************************
    Processing Record 47 of Set 11 | georgetown,gy
    Data found (status code: 200) | georgetown,gy
    Data loaded successfully | georgetown,gy
    *********************************
    Processing Record 48 of Set 11 | malakal,sd
    ERROR: Data retrieval unsuccessful for malakal,sd. Code: 404                       MSG: city not found
    Processing Record 49 of Set 11 | trincomalee,lk
    Data found (status code: 200) | trincomalee,lk
    Data loaded successfully | trincomalee,lk
    *********************************
    Processing Record 50 of Set 11 | mantalongon,ph
    Data found (status code: 200) | mantalongon,ph
    Data loaded successfully | mantalongon,ph
    *********************************
    Processing Record 51 of Set 11 | kissidougou,gn
    Data found (status code: 200) | kissidougou,gn
    Data loaded successfully | kissidougou,gn
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 12 | batticaloa,lk
    Data found (status code: 200) | batticaloa,lk
    Data loaded successfully | batticaloa,lk
    *********************************
    Processing Record 1 of Set 12 | bacolod,ph
    Data found (status code: 200) | bacolod,ph
    Data loaded successfully | bacolod,ph
    *********************************
    Processing Record 2 of Set 12 | kyabe,td
    Data found (status code: 200) | kyabe,td
    Data loaded successfully | kyabe,td
    *********************************
    Processing Record 3 of Set 12 | praia,cv
    Data found (status code: 200) | praia,cv
    Data loaded successfully | praia,cv
    *********************************
    Processing Record 4 of Set 12 | kilinochchi,lk
    Data found (status code: 200) | kilinochchi,lk
    Data loaded successfully | kilinochchi,lk
    *********************************
    Processing Record 5 of Set 12 | ko samui,th
    Data found (status code: 200) | ko samui,th
    Data loaded successfully | ko samui,th
    *********************************
    Processing Record 6 of Set 12 | jaffna,lk
    Data found (status code: 200) | jaffna,lk
    Data loaded successfully | jaffna,lk
    *********************************
    Processing Record 7 of Set 12 | phan rang,vn
    ERROR: Data retrieval unsuccessful for phan rang,vn. Code: 404                       MSG: city not found
    Processing Record 8 of Set 12 | kapaa,us
    Data found (status code: 200) | kapaa,us
    Data loaded successfully | kapaa,us
    *********************************
    Processing Record 9 of Set 12 | salalah,om
    Data found (status code: 200) | salalah,om
    Data loaded successfully | salalah,om
    *********************************
    Processing Record 10 of Set 12 | mango,tg
    ERROR: Data retrieval unsuccessful for mango,tg. Code: 404                       MSG: city not found
    Processing Record 11 of Set 12 | bosaso,so
    Data found (status code: 200) | bosaso,so
    Data loaded successfully | bosaso,so
    *********************************
    Processing Record 12 of Set 12 | ponnani,in
    Data found (status code: 200) | ponnani,in
    Data loaded successfully | ponnani,in
    *********************************
    Processing Record 13 of Set 12 | port blair,in
    Data found (status code: 200) | port blair,in
    Data loaded successfully | port blair,in
    *********************************
    Processing Record 14 of Set 12 | birao,cf
    Data found (status code: 200) | birao,cf
    Data loaded successfully | birao,cf
    *********************************
    Processing Record 15 of Set 12 | bateria,ph
    Data found (status code: 200) | bateria,ph
    Data loaded successfully | bateria,ph
    *********************************
    Processing Record 16 of Set 12 | kontagora,ng
    Data found (status code: 200) | kontagora,ng
    Data loaded successfully | kontagora,ng
    *********************************
    Processing Record 17 of Set 12 | mullaitivu,lk
    ERROR: Data retrieval unsuccessful for mullaitivu,lk. Code: 404                       MSG: city not found
    Processing Record 18 of Set 12 | bargal,so
    ERROR: Data retrieval unsuccessful for bargal,so. Code: 404                       MSG: city not found
    Processing Record 19 of Set 12 | portobelo,pa
    Data found (status code: 200) | portobelo,pa
    Data loaded successfully | portobelo,pa
    *********************************
    Processing Record 20 of Set 12 | acajutla,sv
    Data found (status code: 200) | acajutla,sv
    Data loaded successfully | acajutla,sv
    *********************************
    Processing Record 21 of Set 12 | babanusah,sd
    ERROR: Data retrieval unsuccessful for babanusah,sd. Code: 404                       MSG: city not found
    Processing Record 22 of Set 12 | calampisawan,ph
    ERROR: Data retrieval unsuccessful for calampisawan,ph. Code: 404                       MSG: city not found
    Processing Record 23 of Set 12 | siguiri,gn
    Data found (status code: 200) | siguiri,gn
    Data loaded successfully | siguiri,gn
    *********************************
    Processing Record 24 of Set 12 | roxana,cr
    Data found (status code: 200) | roxana,cr
    Data loaded successfully | roxana,cr
    *********************************
    Processing Record 25 of Set 12 | prachuap khiri khan,th
    Data found (status code: 200) | prachuap khiri khan,th
    Data loaded successfully | prachuap khiri khan,th
    *********************************
    Processing Record 26 of Set 12 | la asuncion,ve
    Data found (status code: 200) | la asuncion,ve
    Data loaded successfully | la asuncion,ve
    *********************************
    Processing Record 27 of Set 12 | bluefields,ni
    Data found (status code: 200) | bluefields,ni
    Data loaded successfully | bluefields,ni
    *********************************
    Processing Record 28 of Set 12 | bang saphan,th
    Data found (status code: 200) | bang saphan,th
    Data loaded successfully | bang saphan,th
    *********************************
    Processing Record 29 of Set 12 | sulangan,ph
    Data found (status code: 200) | sulangan,ph
    Data loaded successfully | sulangan,ph
    *********************************
    Processing Record 30 of Set 12 | meyungs,pw
    ERROR: Data retrieval unsuccessful for meyungs,pw. Code: 404                       MSG: city not found
    Processing Record 31 of Set 12 | san miguelito,ni
    Data found (status code: 200) | san miguelito,ni
    Data loaded successfully | san miguelito,ni
    *********************************
    Processing Record 32 of Set 12 | bama,ng
    Data found (status code: 200) | bama,ng
    Data loaded successfully | bama,ng
    *********************************
    Processing Record 33 of Set 12 | san andres,co
    Data found (status code: 200) | san andres,co
    Data loaded successfully | san andres,co
    *********************************
    Processing Record 34 of Set 12 | maiduguri,ng
    Data found (status code: 200) | maiduguri,ng
    Data loaded successfully | maiduguri,ng
    *********************************
    Processing Record 35 of Set 12 | oistins,bb
    Data found (status code: 200) | oistins,bb
    Data loaded successfully | oistins,bb
    *********************************
    Processing Record 36 of Set 12 | ouargaye,bf
    Data found (status code: 200) | ouargaye,bf
    Data loaded successfully | ouargaye,bf
    *********************************
    Processing Record 37 of Set 12 | diapaga,bf
    Data found (status code: 200) | diapaga,bf
    Data loaded successfully | diapaga,bf
    *********************************
    Processing Record 38 of Set 12 | tinogboc,ph
    Data found (status code: 200) | tinogboc,ph
    Data loaded successfully | tinogboc,ph
    *********************************
    Processing Record 39 of Set 12 | kampong cham,kh
    Data found (status code: 200) | kampong cham,kh
    Data loaded successfully | kampong cham,kh
    *********************************
    Processing Record 40 of Set 12 | panlaitan,ph
    Data found (status code: 200) | panlaitan,ph
    Data loaded successfully | panlaitan,ph
    *********************************
    Processing Record 41 of Set 12 | tigbao,ph
    Data found (status code: 200) | tigbao,ph
    Data loaded successfully | tigbao,ph
    *********************************
    Processing Record 42 of Set 12 | tecoanapa,mx
    Data found (status code: 200) | tecoanapa,mx
    Data loaded successfully | tecoanapa,mx
    *********************************
    Processing Record 43 of Set 12 | constitucion,mx
    Data found (status code: 200) | constitucion,mx
    Data loaded successfully | constitucion,mx
    *********************************
    Processing Record 44 of Set 12 | gunjur,gm
    Data found (status code: 200) | gunjur,gm
    Data loaded successfully | gunjur,gm
    *********************************
    Processing Record 45 of Set 12 | solenzo,bf
    Data found (status code: 200) | solenzo,bf
    Data loaded successfully | solenzo,bf
    *********************************
    Processing Record 46 of Set 12 | karumbakkam,in
    Data found (status code: 200) | karumbakkam,in
    Data loaded successfully | karumbakkam,in
    *********************************
    Processing Record 47 of Set 12 | say,ne
    Data found (status code: 200) | say,ne
    Data loaded successfully | say,ne
    *********************************
    Processing Record 48 of Set 12 | bereda,so
    ERROR: Data retrieval unsuccessful for bereda,so. Code: 404                       MSG: city not found
    Processing Record 49 of Set 12 | pochutla,mx
    Data found (status code: 200) | pochutla,mx
    Data loaded successfully | pochutla,mx
    *********************************
    Processing Record 50 of Set 12 | quezon,ph
    Data found (status code: 200) | quezon,ph
    Data loaded successfully | quezon,ph
    *********************************
    Processing Record 51 of Set 12 | sokoto,ng
    Data found (status code: 200) | sokoto,ng
    Data loaded successfully | sokoto,ng
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 13 | yanam,in
    Data found (status code: 200) | yanam,in
    Data loaded successfully | yanam,in
    *********************************
    Processing Record 1 of Set 13 | anito,ph
    Data found (status code: 200) | anito,ph
    Data loaded successfully | anito,ph
    *********************************
    Processing Record 2 of Set 13 | kousseri,cm
    Data found (status code: 200) | kousseri,cm
    Data loaded successfully | kousseri,cm
    *********************************
    Processing Record 3 of Set 13 | dabat,et
    Data found (status code: 200) | dabat,et
    Data loaded successfully | dabat,et
    *********************************
    Processing Record 4 of Set 13 | cartagena,co
    Data found (status code: 200) | cartagena,co
    Data loaded successfully | cartagena,co
    *********************************
    Processing Record 5 of Set 13 | san rafael,ph
    Data found (status code: 200) | san rafael,ph
    Data loaded successfully | san rafael,ph
    *********************************
    Processing Record 6 of Set 13 | guerrero negro,mx
    Data found (status code: 200) | guerrero negro,mx
    Data loaded successfully | guerrero negro,mx
    *********************************
    Processing Record 7 of Set 13 | cabra,ph
    Data found (status code: 200) | cabra,ph
    Data loaded successfully | cabra,ph
    *********************************
    Processing Record 8 of Set 13 | bathsheba,bb
    Data found (status code: 200) | bathsheba,bb
    Data loaded successfully | bathsheba,bb
    *********************************
    Processing Record 9 of Set 13 | thanyaburi,th
    Data found (status code: 200) | thanyaburi,th
    Data loaded successfully | thanyaburi,th
    *********************************
    Processing Record 10 of Set 13 | lahij,ye
    Data found (status code: 200) | lahij,ye
    Data loaded successfully | lahij,ye
    *********************************
    Processing Record 11 of Set 13 | dawei,mm
    Data found (status code: 200) | dawei,mm
    Data loaded successfully | dawei,mm
    *********************************
    Processing Record 12 of Set 13 | goure,ne
    Data found (status code: 200) | goure,ne
    Data loaded successfully | goure,ne
    *********************************
    Processing Record 13 of Set 13 | amalapuram,in
    Data found (status code: 200) | amalapuram,in
    Data loaded successfully | amalapuram,in
    *********************************
    Processing Record 14 of Set 13 | gashua,ng
    Data found (status code: 200) | gashua,ng
    Data loaded successfully | gashua,ng
    *********************************
    Processing Record 15 of Set 13 | westpunt,an
    ERROR: Data retrieval unsuccessful for westpunt,an. Code: 404                       MSG: city not found
    Processing Record 16 of Set 13 | arroyo,us
    Data found (status code: 200) | arroyo,us
    Data loaded successfully | arroyo,us
    *********************************
    Processing Record 17 of Set 13 | canaries,lc
    ERROR: Data retrieval unsuccessful for canaries,lc. Code: 404                       MSG: city not found
    Processing Record 18 of Set 13 | surin,th
    Data found (status code: 200) | surin,th
    Data loaded successfully | surin,th
    *********************************
    Processing Record 19 of Set 13 | bara,sd
    ERROR: Data retrieval unsuccessful for bara,sd. Code: 404                       MSG: city not found
    Processing Record 20 of Set 13 | kayes,ml
    Data found (status code: 200) | kayes,ml
    Data loaded successfully | kayes,ml
    *********************************
    Processing Record 21 of Set 13 | abeche,td
    Data found (status code: 200) | abeche,td
    Data loaded successfully | abeche,td
    *********************************
    Processing Record 22 of Set 13 | porto novo,cv
    Data found (status code: 200) | porto novo,cv
    Data loaded successfully | porto novo,cv
    *********************************
    Processing Record 23 of Set 13 | djenne,ml
    Data found (status code: 200) | djenne,ml
    Data loaded successfully | djenne,ml
    *********************************
    Processing Record 24 of Set 13 | manaure,co
    Data found (status code: 200) | manaure,co
    Data loaded successfully | manaure,co
    *********************************
    Processing Record 25 of Set 13 | kassala,sd
    Data found (status code: 200) | kassala,sd
    Data loaded successfully | kassala,sd
    *********************************
    Processing Record 26 of Set 13 | kon tum,vn
    Data found (status code: 200) | kon tum,vn
    Data loaded successfully | kon tum,vn
    *********************************
    Processing Record 27 of Set 13 | ginda,er
    ERROR: Data retrieval unsuccessful for ginda,er. Code: 404                       MSG: city not found
    Processing Record 28 of Set 13 | san jose de rio tinto,hn
    Data found (status code: 200) | san jose de rio tinto,hn
    Data loaded successfully | san jose de rio tinto,hn
    *********************************
    Processing Record 29 of Set 13 | nanakuli,us
    Data found (status code: 200) | nanakuli,us
    Data loaded successfully | nanakuli,us
    *********************************
    Processing Record 30 of Set 13 | filingue,ne
    Data found (status code: 200) | filingue,ne
    Data loaded successfully | filingue,ne
    *********************************
    Processing Record 31 of Set 13 | bakel,sn
    Data found (status code: 200) | bakel,sn
    Data loaded successfully | bakel,sn
    *********************************
    Processing Record 32 of Set 13 | bajil,ye
    Data found (status code: 200) | bajil,ye
    Data loaded successfully | bajil,ye
    *********************************
    Processing Record 33 of Set 13 | petoa,hn
    Data found (status code: 200) | petoa,hn
    Data loaded successfully | petoa,hn
    *********************************
    Processing Record 34 of Set 13 | sainte-marie,mq
    Data found (status code: 200) | sainte-marie,mq
    Data loaded successfully | sainte-marie,mq
    *********************************
    Processing Record 35 of Set 13 | sur,om
    Data found (status code: 200) | sur,om
    Data loaded successfully | sur,om
    *********************************
    Processing Record 36 of Set 13 | addi ugri,er
    ERROR: Data retrieval unsuccessful for addi ugri,er. Code: 404                       MSG: city not found
    Processing Record 37 of Set 13 | tanout,ne
    Data found (status code: 200) | tanout,ne
    Data loaded successfully | tanout,ne
    *********************************
    Processing Record 38 of Set 13 | iralaya,hn
    Data found (status code: 200) | iralaya,hn
    Data loaded successfully | iralaya,hn
    *********************************
    Processing Record 39 of Set 13 | aloleng,ph
    Data found (status code: 200) | aloleng,ph
    Data loaded successfully | aloleng,ph
    *********************************
    Processing Record 40 of Set 13 | pinotepa nacional,mx
    ERROR: Data retrieval unsuccessful for pinotepa nacional,mx. Code: 404                       MSG: city not found
    Processing Record 41 of Set 13 | ati,td
    Data found (status code: 200) | ati,td
    Data loaded successfully | ati,td
    *********************************
    Processing Record 42 of Set 13 | gigmoto,ph
    Data found (status code: 200) | gigmoto,ph
    Data loaded successfully | gigmoto,ph
    *********************************
    Processing Record 43 of Set 13 | payo,ph
    ERROR: Data retrieval unsuccessful for payo,ph. Code: 404                       MSG: city not found
    Processing Record 44 of Set 13 | ayorou,ne
    Data found (status code: 200) | ayorou,ne
    Data loaded successfully | ayorou,ne
    *********************************
    Processing Record 45 of Set 13 | huazolotitlan,mx
    ERROR: Data retrieval unsuccessful for huazolotitlan,mx. Code: 404                       MSG: city not found
    Processing Record 46 of Set 13 | kanigiri,in
    Data found (status code: 200) | kanigiri,in
    Data loaded successfully | kanigiri,in
    *********************************
    Processing Record 47 of Set 13 | bacsay,ph
    Data found (status code: 200) | bacsay,ph
    Data loaded successfully | bacsay,ph
    *********************************
    Processing Record 48 of Set 13 | bani,do
    Data found (status code: 200) | bani,do
    Data loaded successfully | bani,do
    *********************************
    Processing Record 49 of Set 13 | les cayes,ht
    Data found (status code: 200) | les cayes,ht
    Data loaded successfully | les cayes,ht
    *********************************
    Processing Record 50 of Set 13 | morant bay,jm
    Data found (status code: 200) | morant bay,jm
    Data loaded successfully | morant bay,jm
    *********************************
    Processing Record 51 of Set 13 | ratnagiri,in
    Data found (status code: 200) | ratnagiri,in
    Data loaded successfully | ratnagiri,in
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 14 | mae sot,th
    Data found (status code: 200) | mae sot,th
    Data loaded successfully | mae sot,th
    *********************************
    Processing Record 1 of Set 14 | benemerito de las americas,mx
    Data found (status code: 200) | benemerito de las americas,mx
    Data loaded successfully | benemerito de las americas,mx
    *********************************
    Processing Record 2 of Set 14 | bull savanna,jm
    Data found (status code: 200) | bull savanna,jm
    Data loaded successfully | bull savanna,jm
    *********************************
    Processing Record 3 of Set 14 | cacahuatepec,mx
    Data found (status code: 200) | cacahuatepec,mx
    Data loaded successfully | cacahuatepec,mx
    *********************************
    Processing Record 4 of Set 14 | wagar,sd
    Data found (status code: 200) | wagar,sd
    Data loaded successfully | wagar,sd
    *********************************
    Processing Record 5 of Set 14 | pho chai,th
    Data found (status code: 200) | pho chai,th
    Data loaded successfully | pho chai,th
    *********************************
    Processing Record 6 of Set 14 | ponta do sol,cv
    Data found (status code: 200) | ponta do sol,cv
    Data loaded successfully | ponta do sol,cv
    *********************************
    Processing Record 7 of Set 14 | ewa beach,us
    Data found (status code: 200) | ewa beach,us
    Data loaded successfully | ewa beach,us
    *********************************
    Processing Record 8 of Set 14 | old road,ag
    ERROR: Data retrieval unsuccessful for old road,ag. Code: 404                       MSG: city not found
    Processing Record 9 of Set 14 | nioro,ml
    ERROR: Data retrieval unsuccessful for nioro,ml. Code: 404                       MSG: city not found
    Processing Record 10 of Set 14 | santa maria,cv
    Data found (status code: 200) | santa maria,cv
    Data loaded successfully | santa maria,cv
    *********************************
    Processing Record 11 of Set 14 | guanica,us
    ERROR: Data retrieval unsuccessful for guanica,us. Code: 404                       MSG: city not found
    Processing Record 12 of Set 14 | puri,in
    Data found (status code: 200) | puri,in
    Data loaded successfully | puri,in
    *********************************
    Processing Record 13 of Set 14 | niltepec,mx
    Data found (status code: 200) | niltepec,mx
    Data loaded successfully | niltepec,mx
    *********************************
    Processing Record 14 of Set 14 | gao,ml
    Data found (status code: 200) | gao,ml
    Data loaded successfully | gao,ml
    *********************************
    Processing Record 15 of Set 14 | nouakchott,mr
    Data found (status code: 200) | nouakchott,mr
    Data loaded successfully | nouakchott,mr
    *********************************
    Processing Record 16 of Set 14 | lom sak,th
    Data found (status code: 200) | lom sak,th
    Data loaded successfully | lom sak,th
    *********************************
    Processing Record 17 of Set 14 | guhagar,in
    Data found (status code: 200) | guhagar,in
    Data loaded successfully | guhagar,in
    *********************************
    Processing Record 18 of Set 14 | sabya,sa
    Data found (status code: 200) | sabya,sa
    Data loaded successfully | sabya,sa
    *********************************
    Processing Record 19 of Set 14 | najran,sa
    Data found (status code: 200) | najran,sa
    Data loaded successfully | najran,sa
    *********************************
    Processing Record 20 of Set 14 | mae ramat,th
    Data found (status code: 200) | mae ramat,th
    Data loaded successfully | mae ramat,th
    *********************************
    Processing Record 21 of Set 14 | veraval,in
    Data found (status code: 200) | veraval,in
    Data loaded successfully | veraval,in
    *********************************
    Processing Record 22 of Set 14 | nara,ml
    Data found (status code: 200) | nara,ml
    Data loaded successfully | nara,ml
    *********************************
    Processing Record 23 of Set 14 | saint-francois,gp
    Data found (status code: 200) | saint-francois,gp
    Data loaded successfully | saint-francois,gp
    *********************************
    Processing Record 24 of Set 14 | tombouctou,ml
    Data found (status code: 200) | tombouctou,ml
    Data loaded successfully | tombouctou,ml
    *********************************
    Processing Record 25 of Set 14 | benito soliven,ph
    Data found (status code: 200) | benito soliven,ph
    Data loaded successfully | benito soliven,ph
    *********************************
    Processing Record 26 of Set 14 | pathein,mm
    Data found (status code: 200) | pathein,mm
    Data loaded successfully | pathein,mm
    *********************************
    Processing Record 27 of Set 14 | bilma,ne
    Data found (status code: 200) | bilma,ne
    Data loaded successfully | bilma,ne
    *********************************
    Processing Record 28 of Set 14 | bodden town,ky
    Data found (status code: 200) | bodden town,ky
    Data loaded successfully | bodden town,ky
    *********************************
    Processing Record 29 of Set 14 | lompoc,us
    Data found (status code: 200) | lompoc,us
    Data loaded successfully | lompoc,us
    *********************************
    Processing Record 30 of Set 14 | kidal,ml
    Data found (status code: 200) | kidal,ml
    Data loaded successfully | kidal,ml
    *********************************
    Processing Record 31 of Set 14 | pyay,mm
    Data found (status code: 200) | pyay,mm
    Data loaded successfully | pyay,mm
    *********************************
    Processing Record 32 of Set 14 | alugan,ph
    Data found (status code: 200) | alugan,ph
    Data loaded successfully | alugan,ph
    *********************************
    Processing Record 33 of Set 14 | sholapur,in
    ERROR: Data retrieval unsuccessful for sholapur,in. Code: 404                       MSG: city not found
    Processing Record 34 of Set 14 | zhuhai,cn
    Data found (status code: 200) | zhuhai,cn
    Data loaded successfully | zhuhai,cn
    *********************************
    Processing Record 35 of Set 14 | goundam,ml
    Data found (status code: 200) | goundam,ml
    Data loaded successfully | goundam,ml
    *********************************
    Processing Record 36 of Set 14 | george town,ky
    Data found (status code: 200) | george town,ky
    Data loaded successfully | george town,ky
    *********************************
    Processing Record 37 of Set 14 | faya,td
    ERROR: Data retrieval unsuccessful for faya,td. Code: 404                       MSG: city not found
    Processing Record 38 of Set 14 | san ignacio,bz
    Data found (status code: 200) | san ignacio,bz
    Data loaded successfully | san ignacio,bz
    *********************************
    Processing Record 39 of Set 14 | porbandar,in
    Data found (status code: 200) | porbandar,in
    Data loaded successfully | porbandar,in
    *********************************
    Processing Record 40 of Set 14 | nishihara,jp
    Data found (status code: 200) | nishihara,jp
    Data loaded successfully | nishihara,jp
    *********************************
    Processing Record 41 of Set 14 | dwarka,in
    Data found (status code: 200) | dwarka,in
    Data loaded successfully | dwarka,in
    *********************************
    Processing Record 42 of Set 14 | san quintin,mx
    ERROR: Data retrieval unsuccessful for san quintin,mx. Code: 404                       MSG: city not found
    Processing Record 43 of Set 14 | khed,in
    Data found (status code: 200) | khed,in
    Data loaded successfully | khed,in
    *********************************
    Processing Record 44 of Set 14 | loei,th
    Data found (status code: 200) | loei,th
    Data loaded successfully | loei,th
    *********************************
    Processing Record 45 of Set 14 | akyab,mm
    ERROR: Data retrieval unsuccessful for akyab,mm. Code: 404                       MSG: city not found
    Processing Record 46 of Set 14 | lucea,jm
    Data found (status code: 200) | lucea,jm
    Data loaded successfully | lucea,jm
    *********************************
    Processing Record 47 of Set 14 | abha,sa
    Data found (status code: 200) | abha,sa
    Data loaded successfully | abha,sa
    *********************************
    Processing Record 48 of Set 14 | guadalupe victoria,mx
    Data found (status code: 200) | guadalupe victoria,mx
    Data loaded successfully | guadalupe victoria,mx
    *********************************
    Processing Record 49 of Set 14 | itoman,jp
    Data found (status code: 200) | itoman,jp
    Data loaded successfully | itoman,jp
    *********************************
    Processing Record 50 of Set 14 | vinh,vn
    Data found (status code: 200) | vinh,vn
    Data loaded successfully | vinh,vn
    *********************************
    Processing Record 51 of Set 14 | el seibo,do
    Data found (status code: 200) | el seibo,do
    Data loaded successfully | el seibo,do
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 15 | paradwip,in
    ERROR: Data retrieval unsuccessful for paradwip,in. Code: 404                       MSG: city not found
    Processing Record 1 of Set 15 | kutum,sd
    Data found (status code: 200) | kutum,sd
    Data loaded successfully | kutum,sd
    *********************************
    Processing Record 2 of Set 15 | frontera,mx
    Data found (status code: 200) | frontera,mx
    Data loaded successfully | frontera,mx
    *********************************
    Processing Record 3 of Set 15 | atar,mr
    Data found (status code: 200) | atar,mr
    Data loaded successfully | atar,mr
    *********************************
    Processing Record 4 of Set 15 | katsuura,jp
    Data found (status code: 200) | katsuura,jp
    Data loaded successfully | katsuura,jp
    *********************************
    Processing Record 5 of Set 15 | guisa,cu
    Data found (status code: 200) | guisa,cu
    Data loaded successfully | guisa,cu
    *********************************
    Processing Record 6 of Set 15 | tawkar,sd
    ERROR: Data retrieval unsuccessful for tawkar,sd. Code: 404                       MSG: city not found
    Processing Record 7 of Set 15 | araouane,ml
    Data found (status code: 200) | araouane,ml
    Data loaded successfully | araouane,ml
    *********************************
    Processing Record 8 of Set 15 | carmen,mx
    ERROR: Data retrieval unsuccessful for carmen,mx. Code: 404                       MSG: city not found
    Processing Record 9 of Set 15 | jieshi,cn
    Data found (status code: 200) | jieshi,cn
    Data loaded successfully | jieshi,cn
    *********************************
    Processing Record 10 of Set 15 | palma soriano,cu
    Data found (status code: 200) | palma soriano,cu
    Data loaded successfully | palma soriano,cu
    *********************************
    Processing Record 11 of Set 15 | chinchani,in
    Data found (status code: 200) | chinchani,in
    Data loaded successfully | chinchani,in
    *********************************
    Processing Record 12 of Set 15 | adilabad,in
    Data found (status code: 200) | adilabad,in
    Data loaded successfully | adilabad,in
    *********************************
    Processing Record 13 of Set 15 | la huerta,mx
    Data found (status code: 200) | la huerta,mx
    Data loaded successfully | la huerta,mx
    *********************************
    Processing Record 14 of Set 15 | marawi,sd
    Data found (status code: 200) | marawi,sd
    Data loaded successfully | marawi,sd
    *********************************
    Processing Record 15 of Set 15 | sawakin,sd
    Data found (status code: 200) | sawakin,sd
    Data loaded successfully | sawakin,sd
    *********************************
    Processing Record 16 of Set 15 | sangamner,in
    Data found (status code: 200) | sangamner,in
    Data loaded successfully | sangamner,in
    *********************************
    Processing Record 17 of Set 15 | thanh hoa,vn
    Data found (status code: 200) | thanh hoa,vn
    Data loaded successfully | thanh hoa,vn
    *********************************
    Processing Record 18 of Set 15 | naze,jp
    Data found (status code: 200) | naze,jp
    Data loaded successfully | naze,jp
    *********************************
    Processing Record 19 of Set 15 | nizwa,om
    Data found (status code: 200) | nizwa,om
    Data loaded successfully | nizwa,om
    *********************************
    Processing Record 20 of Set 15 | basco,ph
    Data found (status code: 200) | basco,ph
    Data loaded successfully | basco,ph
    *********************************
    Processing Record 21 of Set 15 | celestun,mx
    Data found (status code: 200) | celestun,mx
    Data loaded successfully | celestun,mx
    *********************************
    Processing Record 22 of Set 15 | campeche,mx
    Data found (status code: 200) | campeche,mx
    Data loaded successfully | campeche,mx
    *********************************
    Processing Record 23 of Set 15 | honolulu,us
    Data found (status code: 200) | honolulu,us
    Data loaded successfully | honolulu,us
    *********************************
    Processing Record 24 of Set 15 | trimbak,in
    Data found (status code: 200) | trimbak,in
    Data loaded successfully | trimbak,in
    *********************************
    Processing Record 25 of Set 15 | road town,vg
    Data found (status code: 200) | road town,vg
    Data loaded successfully | road town,vg
    *********************************
    Processing Record 26 of Set 15 | diu,in
    Data found (status code: 200) | diu,in
    Data loaded successfully | diu,in
    *********************************
    Processing Record 27 of Set 15 | jiddah,sa
    ERROR: Data retrieval unsuccessful for jiddah,sa. Code: 404                       MSG: city not found
    Processing Record 28 of Set 15 | dien bien,vn
    ERROR: Data retrieval unsuccessful for dien bien,vn. Code: 404                       MSG: city not found
    Processing Record 29 of Set 15 | emilio carranza,mx
    Data found (status code: 200) | emilio carranza,mx
    Data loaded successfully | emilio carranza,mx
    *********************************
    Processing Record 30 of Set 15 | guantanamo,cu
    Data found (status code: 200) | guantanamo,cu
    Data loaded successfully | guantanamo,cu
    *********************************
    Processing Record 31 of Set 15 | codrington,ag
    ERROR: Data retrieval unsuccessful for codrington,ag. Code: 404                       MSG: city not found
    Processing Record 32 of Set 15 | sotuta,mx
    Data found (status code: 200) | sotuta,mx
    Data loaded successfully | sotuta,mx
    *********************************
    Processing Record 33 of Set 15 | las varas,mx
    Data found (status code: 200) | las varas,mx
    Data loaded successfully | las varas,mx
    *********************************
    Processing Record 34 of Set 15 | udala,in
    ERROR: Data retrieval unsuccessful for udala,in. Code: 404                       MSG: city not found
    Processing Record 35 of Set 15 | isla mujeres,mx
    Data found (status code: 200) | isla mujeres,mx
    Data loaded successfully | isla mujeres,mx
    *********************************
    Processing Record 36 of Set 15 | mecca,sa
    Data found (status code: 200) | mecca,sa
    Data loaded successfully | mecca,sa
    *********************************
    Processing Record 37 of Set 15 | calnali,mx
    Data found (status code: 200) | calnali,mx
    Data loaded successfully | calnali,mx
    *********************************
    Processing Record 38 of Set 15 | abu dhabi,ae
    Data found (status code: 200) | abu dhabi,ae
    Data loaded successfully | abu dhabi,ae
    *********************************
    Processing Record 39 of Set 15 | mohpa,in
    Data found (status code: 200) | mohpa,in
    Data loaded successfully | mohpa,in
    *********************************
    Processing Record 40 of Set 15 | tessalit,ml
    Data found (status code: 200) | tessalit,ml
    Data loaded successfully | tessalit,ml
    *********************************
    Processing Record 41 of Set 15 | nouadhibou,mr
    Data found (status code: 200) | nouadhibou,mr
    Data loaded successfully | nouadhibou,mr
    *********************************
    Processing Record 42 of Set 15 | san juan de los lagos,mx
    Data found (status code: 200) | san juan de los lagos,mx
    Data loaded successfully | san juan de los lagos,mx
    *********************************
    Processing Record 43 of Set 15 | abu samrah,qa
    ERROR: Data retrieval unsuccessful for abu samrah,qa. Code: 404                       MSG: city not found
    Processing Record 44 of Set 15 | la ceiba,mx
    Data found (status code: 200) | la ceiba,mx
    Data loaded successfully | la ceiba,mx
    *********************************
    Processing Record 45 of Set 15 | tierranueva,mx
    Data found (status code: 200) | tierranueva,mx
    Data loaded successfully | tierranueva,mx
    *********************************
    Processing Record 46 of Set 15 | saiha,in
    Data found (status code: 200) | saiha,in
    Data loaded successfully | saiha,in
    *********************************
    Processing Record 47 of Set 15 | gat,ly
    ERROR: Data retrieval unsuccessful for gat,ly. Code: 404                       MSG: city not found
    Processing Record 48 of Set 15 | maymyo,mm
    Data found (status code: 200) | maymyo,mm
    Data loaded successfully | maymyo,mm
    *********************************
    Processing Record 49 of Set 15 | ishigaki,jp
    Data found (status code: 200) | ishigaki,jp
    Data loaded successfully | ishigaki,jp
    *********************************
    Processing Record 50 of Set 15 | cockburn town,tc
    Data found (status code: 200) | cockburn town,tc
    Data loaded successfully | cockburn town,tc
    *********************************
    Processing Record 51 of Set 15 | santa fe,cu
    Data found (status code: 200) | santa fe,cu
    Data loaded successfully | santa fe,cu
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 16 | taoudenni,ml
    Data found (status code: 200) | taoudenni,ml
    Data loaded successfully | taoudenni,ml
    *********************************
    Processing Record 1 of Set 16 | digha,in
    Data found (status code: 200) | digha,in
    Data loaded successfully | digha,in
    *********************************
    Processing Record 2 of Set 16 | sundargarh,in
    Data found (status code: 200) | sundargarh,in
    Data loaded successfully | sundargarh,in
    *********************************
    Processing Record 3 of Set 16 | moron,cu
    Data found (status code: 200) | moron,cu
    Data loaded successfully | moron,cu
    *********************************
    Processing Record 4 of Set 16 | todos santos,mx
    Data found (status code: 200) | todos santos,mx
    Data loaded successfully | todos santos,mx
    *********************************
    Processing Record 5 of Set 16 | falam,mm
    Data found (status code: 200) | falam,mm
    Data loaded successfully | falam,mm
    *********************************
    Processing Record 6 of Set 16 | the valley,ai
    Data found (status code: 200) | the valley,ai
    Data loaded successfully | the valley,ai
    *********************************
    Processing Record 7 of Set 16 | shimoda,jp
    Data found (status code: 200) | shimoda,jp
    Data loaded successfully | shimoda,jp
    *********************************
    Processing Record 8 of Set 16 | ducheng,cn
    Data found (status code: 200) | ducheng,cn
    Data loaded successfully | ducheng,cn
    *********************************
    Processing Record 9 of Set 16 | huejuquilla el alto,mx
    Data found (status code: 200) | huejuquilla el alto,mx
    Data loaded successfully | huejuquilla el alto,mx
    *********************************
    Processing Record 10 of Set 16 | lao cai,vn
    Data found (status code: 200) | lao cai,vn
    Data loaded successfully | lao cai,vn
    *********************************
    Processing Record 11 of Set 16 | ha giang,vn
    Data found (status code: 200) | ha giang,vn
    Data loaded successfully | ha giang,vn
    *********************************
    Processing Record 12 of Set 16 | mogok,mm
    Data found (status code: 200) | mogok,mm
    Data loaded successfully | mogok,mm
    *********************************
    Processing Record 13 of Set 16 | buraydah,sa
    Data found (status code: 200) | buraydah,sa
    Data loaded successfully | buraydah,sa
    *********************************
    Processing Record 14 of Set 16 | nuevitas,cu
    Data found (status code: 200) | nuevitas,cu
    Data loaded successfully | nuevitas,cu
    *********************************
    Processing Record 15 of Set 16 | basirhat,in
    Data found (status code: 200) | basirhat,in
    Data loaded successfully | basirhat,in
    *********************************
    Processing Record 16 of Set 16 | hasaki,jp
    Data found (status code: 200) | hasaki,jp
    Data loaded successfully | hasaki,jp
    *********************************
    Processing Record 17 of Set 16 | clarence town,bs
    Data found (status code: 200) | clarence town,bs
    Data loaded successfully | clarence town,bs
    *********************************
    Processing Record 18 of Set 16 | san juan,us
    Data found (status code: 200) | san juan,us
    Data loaded successfully | san juan,us
    *********************************
    Processing Record 19 of Set 16 | jaumave,mx
    Data found (status code: 200) | jaumave,mx
    Data loaded successfully | jaumave,mx
    *********************************
    Processing Record 20 of Set 16 | cockburn town,bs
    Data found (status code: 200) | cockburn town,bs
    Data loaded successfully | cockburn town,bs
    *********************************
    Processing Record 21 of Set 16 | panaba,mx
    Data found (status code: 200) | panaba,mx
    Data loaded successfully | panaba,mx
    *********************************
    Processing Record 22 of Set 16 | kotma,in
    Data found (status code: 200) | kotma,in
    Data loaded successfully | kotma,in
    *********************************
    Processing Record 23 of Set 16 | nuevo progreso,mx
    Data found (status code: 200) | nuevo progreso,mx
    Data loaded successfully | nuevo progreso,mx
    *********************************
    Processing Record 24 of Set 16 | kaiyuan,cn
    Data found (status code: 200) | kaiyuan,cn
    Data loaded successfully | kaiyuan,cn
    *********************************
    Processing Record 25 of Set 16 | bangaon,in
    Data found (status code: 200) | bangaon,in
    Data loaded successfully | bangaon,in
    *********************************
    Processing Record 26 of Set 16 | riyadh,sa
    Data found (status code: 200) | riyadh,sa
    Data loaded successfully | riyadh,sa
    *********************************
    Processing Record 27 of Set 16 | mantua,cu
    Data found (status code: 200) | mantua,cu
    Data loaded successfully | mantua,cu
    *********************************
    Processing Record 28 of Set 16 | tula,mx
    Data found (status code: 200) | tula,mx
    Data loaded successfully | tula,mx
    *********************************
    Processing Record 29 of Set 16 | aswan,eg
    Data found (status code: 200) | aswan,eg
    Data loaded successfully | aswan,eg
    *********************************
    Processing Record 30 of Set 16 | marzuq,ly
    ERROR: Data retrieval unsuccessful for marzuq,ly. Code: 404                       MSG: city not found
    Processing Record 31 of Set 16 | rock sound,bs
    Data found (status code: 200) | rock sound,bs
    Data loaded successfully | rock sound,bs
    *********************************
    Processing Record 32 of Set 16 | bahia honda,cu
    Data found (status code: 200) | bahia honda,cu
    Data loaded successfully | bahia honda,cu
    *********************************
    Processing Record 33 of Set 16 | jiwani,pk
    Data found (status code: 200) | jiwani,pk
    Data loaded successfully | jiwani,pk
    *********************************
    Processing Record 34 of Set 16 | govindgarh,in
    Data found (status code: 200) | govindgarh,in
    Data loaded successfully | govindgarh,in
    *********************************
    Processing Record 35 of Set 16 | suao,tw
    ERROR: Data retrieval unsuccessful for suao,tw. Code: 404                       MSG: city not found
    Processing Record 36 of Set 16 | chaozhou,cn
    Data found (status code: 200) | chaozhou,cn
    Data loaded successfully | chaozhou,cn
    *********************************
    Processing Record 37 of Set 16 | matamoros,mx
    Data found (status code: 200) | matamoros,mx
    Data loaded successfully | matamoros,mx
    *********************************
    Processing Record 38 of Set 16 | lotung,tw
    ERROR: Data retrieval unsuccessful for lotung,tw. Code: 404                       MSG: city not found
    Processing Record 39 of Set 16 | jhudo,pk
    Data found (status code: 200) | jhudo,pk
    Data loaded successfully | jhudo,pk
    *********************************
    Processing Record 40 of Set 16 | puerto del rosario,es
    Data found (status code: 200) | puerto del rosario,es
    Data loaded successfully | puerto del rosario,es
    *********************************
    Processing Record 41 of Set 16 | los llanos de aridane,es
    Data found (status code: 200) | los llanos de aridane,es
    Data loaded successfully | los llanos de aridane,es
    *********************************
    Processing Record 42 of Set 16 | imphal,in
    Data found (status code: 200) | imphal,in
    Data loaded successfully | imphal,in
    *********************************
    Processing Record 43 of Set 16 | santa catarina de tepehuanes,mx
    Data found (status code: 200) | santa catarina de tepehuanes,mx
    Data loaded successfully | santa catarina de tepehuanes,mx
    *********************************
    Processing Record 44 of Set 16 | guamuchil,mx
    Data found (status code: 200) | guamuchil,mx
    Data loaded successfully | guamuchil,mx
    *********************************
    Processing Record 45 of Set 16 | sharjah,ae
    Data found (status code: 200) | sharjah,ae
    Data loaded successfully | sharjah,ae
    *********************************
    Processing Record 46 of Set 16 | tiznit,ma
    Data found (status code: 200) | tiznit,ma
    Data loaded successfully | tiznit,ma
    *********************************
    Processing Record 47 of Set 16 | ormara,pk
    Data found (status code: 200) | ormara,pk
    Data loaded successfully | ormara,pk
    *********************************
    Processing Record 48 of Set 16 | purnia,in
    Data found (status code: 200) | purnia,in
    Data loaded successfully | purnia,in
    *********************************
    Processing Record 49 of Set 16 | nabisar,pk
    Data found (status code: 200) | nabisar,pk
    Data loaded successfully | nabisar,pk
    *********************************
    Processing Record 50 of Set 16 | safaga,eg
    ERROR: Data retrieval unsuccessful for safaga,eg. Code: 404                       MSG: city not found
    Processing Record 51 of Set 16 | palera,in
    Data found (status code: 200) | palera,in
    Data loaded successfully | palera,in
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 17 | aguimes,es
    Data found (status code: 200) | aguimes,es
    Data loaded successfully | aguimes,es
    *********************************
    Processing Record 1 of Set 17 | pasni,pk
    Data found (status code: 200) | pasni,pk
    Data loaded successfully | pasni,pk
    *********************************
    Processing Record 2 of Set 17 | kahului,us
    Data found (status code: 200) | kahului,us
    Data loaded successfully | kahului,us
    *********************************
    Processing Record 3 of Set 17 | khor,qa
    ERROR: Data retrieval unsuccessful for khor,qa. Code: 404                       MSG: city not found
    Processing Record 4 of Set 17 | manama,bh
    Data found (status code: 200) | manama,bh
    Data loaded successfully | manama,bh
    *********************************
    Processing Record 5 of Set 17 | loreto,mx
    Data found (status code: 200) | loreto,mx
    Data loaded successfully | loreto,mx
    *********************************
    Processing Record 6 of Set 17 | arona,es
    Data found (status code: 200) | arona,es
    Data loaded successfully | arona,es
    *********************************
    Processing Record 7 of Set 17 | estelle,us
    Data found (status code: 200) | estelle,us
    Data loaded successfully | estelle,us
    *********************************
    Processing Record 8 of Set 17 | kailua,us
    Data found (status code: 200) | kailua,us
    Data loaded successfully | kailua,us
    *********************************
    Processing Record 9 of Set 17 | buqayq,sa
    ERROR: Data retrieval unsuccessful for buqayq,sa. Code: 404                       MSG: city not found
    Processing Record 10 of Set 17 | gushikawa,jp
    Data found (status code: 200) | gushikawa,jp
    Data loaded successfully | gushikawa,jp
    *********************************
    Processing Record 11 of Set 17 | umm lajj,sa
    Data found (status code: 200) | umm lajj,sa
    Data loaded successfully | umm lajj,sa
    *********************************
    Processing Record 12 of Set 17 | adrar,dz
    Data found (status code: 200) | adrar,dz
    Data loaded successfully | adrar,dz
    *********************************
    Processing Record 13 of Set 17 | fuzhou,cn
    Data found (status code: 200) | fuzhou,cn
    Data loaded successfully | fuzhou,cn
    *********************************
    Processing Record 14 of Set 17 | santa lucia,es
    Data found (status code: 200) | santa lucia,es
    Data loaded successfully | santa lucia,es
    *********************************
    Processing Record 15 of Set 17 | bela,pk
    Data found (status code: 200) | bela,pk
    Data loaded successfully | bela,pk
    *********************************
    Processing Record 16 of Set 17 | jalu,ly
    Data found (status code: 200) | jalu,ly
    Data loaded successfully | jalu,ly
    *********************************
    Processing Record 17 of Set 17 | san benito,us
    Data found (status code: 200) | san benito,us
    Data loaded successfully | san benito,us
    *********************************
    Processing Record 18 of Set 17 | qeshm,ir
    Data found (status code: 200) | qeshm,ir
    Data loaded successfully | qeshm,ir
    *********************************
    Processing Record 19 of Set 17 | sabha,ly
    Data found (status code: 200) | sabha,ly
    Data loaded successfully | sabha,ly
    *********************************
    Processing Record 20 of Set 17 | ribeira grande,pt
    Data found (status code: 200) | ribeira grande,pt
    Data loaded successfully | ribeira grande,pt
    *********************************
    Processing Record 21 of Set 17 | marsh harbour,bs
    Data found (status code: 200) | marsh harbour,bs
    Data loaded successfully | marsh harbour,bs
    *********************************
    Processing Record 22 of Set 17 | tomigusuku,jp
    Data found (status code: 200) | tomigusuku,jp
    Data loaded successfully | tomigusuku,jp
    *********************************
    Processing Record 23 of Set 17 | sentyabrskiy,ru
    ERROR: Data retrieval unsuccessful for sentyabrskiy,ru. Code: 404                       MSG: city not found
    Processing Record 24 of Set 17 | juifang,tw
    ERROR: Data retrieval unsuccessful for juifang,tw. Code: 404                       MSG: city not found
    Processing Record 25 of Set 17 | jarwal,in
    Data found (status code: 200) | jarwal,in
    Data loaded successfully | jarwal,in
    *********************************
    Processing Record 26 of Set 17 | minab,ir
    Data found (status code: 200) | minab,ir
    Data loaded successfully | minab,ir
    *********************************
    Processing Record 27 of Set 17 | turbat,pk
    Data found (status code: 200) | turbat,pk
    Data loaded successfully | turbat,pk
    *********************************
    Processing Record 28 of Set 17 | aransas pass,us
    Data found (status code: 200) | aransas pass,us
    Data loaded successfully | aransas pass,us
    *********************************
    Processing Record 29 of Set 17 | pacific grove,us
    Data found (status code: 200) | pacific grove,us
    Data loaded successfully | pacific grove,us
    *********************************
    Processing Record 30 of Set 17 | tabuk,sa
    Data found (status code: 200) | tabuk,sa
    Data loaded successfully | tabuk,sa
    *********************************
    Processing Record 31 of Set 17 | saint george,bm
    Data found (status code: 200) | saint george,bm
    Data loaded successfully | saint george,bm
    *********************************
    Processing Record 32 of Set 17 | anahuac,mx
    Data found (status code: 200) | anahuac,mx
    Data loaded successfully | anahuac,mx
    *********************************
    Processing Record 33 of Set 17 | valentin gomez farias,mx
    Data found (status code: 200) | valentin gomez farias,mx
    Data loaded successfully | valentin gomez farias,mx
    *********************************
    Processing Record 34 of Set 17 | springfield,us
    Data found (status code: 200) | springfield,us
    Data loaded successfully | springfield,us
    *********************************
    Processing Record 35 of Set 17 | hongjiang,cn
    Data found (status code: 200) | hongjiang,cn
    Data loaded successfully | hongjiang,cn
    *********************************
    Processing Record 36 of Set 17 | lachhmangarh,in
    Data found (status code: 200) | lachhmangarh,in
    Data loaded successfully | lachhmangarh,in
    *********************************
    Processing Record 37 of Set 17 | sakakah,sa
    ERROR: Data retrieval unsuccessful for sakakah,sa. Code: 404                       MSG: city not found
    Processing Record 38 of Set 17 | safwah,sa
    ERROR: Data retrieval unsuccessful for safwah,sa. Code: 404                       MSG: city not found
    Processing Record 39 of Set 17 | iglas,in
    Data found (status code: 200) | iglas,in
    Data loaded successfully | iglas,in
    *********************************
    Processing Record 40 of Set 17 | zhicheng,cn
    Data found (status code: 200) | zhicheng,cn
    Data loaded successfully | zhicheng,cn
    *********************************
    Processing Record 41 of Set 17 | ruian,cn
    ERROR: Data retrieval unsuccessful for ruian,cn. Code: 404                       MSG: city not found
    Processing Record 42 of Set 17 | matay,eg
    Data found (status code: 200) | matay,eg
    Data loaded successfully | matay,eg
    *********************************
    Processing Record 43 of Set 17 | freeport,bs
    Data found (status code: 200) | freeport,bs
    Data loaded successfully | freeport,bs
    *********************************
    Processing Record 44 of Set 17 | waddan,ly
    Data found (status code: 200) | waddan,ly
    Data loaded successfully | waddan,ly
    *********************************
    Processing Record 45 of Set 17 | shingu,jp
    Data found (status code: 200) | shingu,jp
    Data loaded successfully | shingu,jp
    *********************************
    Processing Record 46 of Set 17 | banepa,np
    Data found (status code: 200) | banepa,np
    Data loaded successfully | banepa,np
    *********************************
    Processing Record 47 of Set 17 | khandbari,np
    Data found (status code: 200) | khandbari,np
    Data loaded successfully | khandbari,np
    *********************************
    Processing Record 48 of Set 17 | santa barbara,mx
    Data found (status code: 200) | santa barbara,mx
    Data loaded successfully | santa barbara,mx
    *********************************
    Processing Record 49 of Set 17 | wenling,cn
    Data found (status code: 200) | wenling,cn
    Data loaded successfully | wenling,cn
    *********************************
    Processing Record 50 of Set 17 | hamilton,bm
    Data found (status code: 200) | hamilton,bm
    Data loaded successfully | hamilton,bm
    *********************************
    Processing Record 51 of Set 17 | khash,ir
    Data found (status code: 200) | khash,ir
    Data loaded successfully | khash,ir
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 18 | tarudant,ma
    ERROR: Data retrieval unsuccessful for tarudant,ma. Code: 404                       MSG: city not found
    Processing Record 1 of Set 18 | awbari,ly
    Data found (status code: 200) | awbari,ly
    Data loaded successfully | awbari,ly
    *********************************
    Processing Record 2 of Set 18 | bardiyah,ly
    Data found (status code: 200) | bardiyah,ly
    Data loaded successfully | bardiyah,ly
    *********************************
    Processing Record 3 of Set 18 | warqla,dz
    ERROR: Data retrieval unsuccessful for warqla,dz. Code: 404                       MSG: city not found
    Processing Record 4 of Set 18 | xichang,cn
    Data found (status code: 200) | xichang,cn
    Data loaded successfully | xichang,cn
    *********************************
    Processing Record 5 of Set 18 | jingdezhen,cn
    Data found (status code: 200) | jingdezhen,cn
    Data loaded successfully | jingdezhen,cn
    *********************************
    Processing Record 6 of Set 18 | chalmette,us
    Data found (status code: 200) | chalmette,us
    Data loaded successfully | chalmette,us
    *********************************
    Processing Record 7 of Set 18 | dayong,cn
    Data found (status code: 200) | dayong,cn
    Data loaded successfully | dayong,cn
    *********************************
    Processing Record 8 of Set 18 | beni suef,eg
    Data found (status code: 200) | beni suef,eg
    Data loaded successfully | beni suef,eg
    *********************************
    Processing Record 9 of Set 18 | marsa matruh,eg
    Data found (status code: 200) | marsa matruh,eg
    Data loaded successfully | marsa matruh,eg
    *********************************
    Processing Record 10 of Set 18 | arrecife,es
    Data found (status code: 200) | arrecife,es
    Data loaded successfully | arrecife,es
    *********************************
    Processing Record 11 of Set 18 | suez,eg
    Data found (status code: 200) | suez,eg
    Data loaded successfully | suez,eg
    *********************************
    Processing Record 12 of Set 18 | ures,mx
    Data found (status code: 200) | ures,mx
    Data loaded successfully | ures,mx
    *********************************
    Processing Record 13 of Set 18 | hun,ly
    Data found (status code: 200) | hun,ly
    Data loaded successfully | hun,ly
    *********************************
    Processing Record 14 of Set 18 | longhua,cn
    Data found (status code: 200) | longhua,cn
    Data loaded successfully | longhua,cn
    *********************************
    Processing Record 15 of Set 18 | doha,kw
    ERROR: Data retrieval unsuccessful for doha,kw. Code: 404                       MSG: city not found
    Processing Record 16 of Set 18 | ramnagar,in
    Data found (status code: 200) | ramnagar,in
    Data loaded successfully | ramnagar,in
    *********************************
    Processing Record 17 of Set 18 | daan,cn
    Data found (status code: 200) | daan,cn
    Data loaded successfully | daan,cn
    *********************************
    Processing Record 18 of Set 18 | jijiang,cn
    Data found (status code: 200) | jijiang,cn
    Data loaded successfully | jijiang,cn
    *********************************
    Processing Record 19 of Set 18 | lasa,cn
    ERROR: Data retrieval unsuccessful for lasa,cn. Code: 404                       MSG: city not found
    Processing Record 20 of Set 18 | half moon bay,us
    Data found (status code: 200) | half moon bay,us
    Data loaded successfully | half moon bay,us
    *********************************
    Processing Record 21 of Set 18 | shenjiamen,cn
    Data found (status code: 200) | shenjiamen,cn
    Data loaded successfully | shenjiamen,cn
    *********************************
    Processing Record 22 of Set 18 | ponta delgada,pt
    Data found (status code: 200) | ponta delgada,pt
    Data loaded successfully | ponta delgada,pt
    *********************************
    Processing Record 23 of Set 18 | teguise,es
    Data found (status code: 200) | teguise,es
    Data loaded successfully | teguise,es
    *********************************
    Processing Record 24 of Set 18 | lockhart,us
    Data found (status code: 200) | lockhart,us
    Data loaded successfully | lockhart,us
    *********************************
    Processing Record 25 of Set 18 | rudbar,af
    Data found (status code: 200) | rudbar,af
    Data loaded successfully | rudbar,af
    *********************************
    Processing Record 26 of Set 18 | tias,es
    Data found (status code: 200) | tias,es
    Data loaded successfully | tias,es
    *********************************
    Processing Record 27 of Set 18 | nalut,ly
    Data found (status code: 200) | nalut,ly
    Data loaded successfully | nalut,ly
    *********************************
    Processing Record 28 of Set 18 | mastung,pk
    Data found (status code: 200) | mastung,pk
    Data loaded successfully | mastung,pk
    *********************************
    Processing Record 29 of Set 18 | orange park,us
    Data found (status code: 200) | orange park,us
    Data loaded successfully | orange park,us
    *********************************
    Processing Record 30 of Set 18 | ojinaga,mx
    Data found (status code: 200) | ojinaga,mx
    Data loaded successfully | ojinaga,mx
    *********************************
    Processing Record 31 of Set 18 | kushima,jp
    Data found (status code: 200) | kushima,jp
    Data loaded successfully | kushima,jp
    *********************************
    Processing Record 32 of Set 18 | lichuan,cn
    Data found (status code: 200) | lichuan,cn
    Data loaded successfully | lichuan,cn
    *********************************
    Processing Record 33 of Set 18 | along,in
    Data found (status code: 200) | along,in
    Data loaded successfully | along,in
    *********************************
    Processing Record 34 of Set 18 | tacoronte,es
    Data found (status code: 200) | tacoronte,es
    Data loaded successfully | tacoronte,es
    *********************************
    Processing Record 35 of Set 18 | oyama,jp
    Data found (status code: 200) | oyama,jp
    Data loaded successfully | oyama,jp
    *********************************
    Processing Record 36 of Set 18 | bardsir,ir
    Data found (status code: 200) | bardsir,ir
    Data loaded successfully | bardsir,ir
    *********************************
    Processing Record 37 of Set 18 | fukue,jp
    Data found (status code: 200) | fukue,jp
    Data loaded successfully | fukue,jp
    *********************************
    Processing Record 38 of Set 18 | shadegan,ir
    Data found (status code: 200) | shadegan,ir
    Data loaded successfully | shadegan,ir
    *********************************
    Processing Record 39 of Set 18 | college station,us
    Data found (status code: 200) | college station,us
    Data loaded successfully | college station,us
    *********************************
    Processing Record 40 of Set 18 | severo-kurilsk,ru
    Data found (status code: 200) | severo-kurilsk,ru
    Data loaded successfully | severo-kurilsk,ru
    *********************************
    Processing Record 41 of Set 18 | daphne,us
    Data found (status code: 200) | daphne,us
    Data loaded successfully | daphne,us
    *********************************
    Processing Record 42 of Set 18 | jumla,np
    Data found (status code: 200) | jumla,np
    Data loaded successfully | jumla,np
    *********************************
    Processing Record 43 of Set 18 | agadir,ma
    Data found (status code: 200) | agadir,ma
    Data loaded successfully | agadir,ma
    *********************************
    Processing Record 44 of Set 18 | ravar,ir
    Data found (status code: 200) | ravar,ir
    Data loaded successfully | ravar,ir
    *********************************
    Processing Record 45 of Set 18 | netivot,il
    Data found (status code: 200) | netivot,il
    Data loaded successfully | netivot,il
    *********************************
    Processing Record 46 of Set 18 | marv dasht,ir
    ERROR: Data retrieval unsuccessful for marv dasht,ir. Code: 404                       MSG: city not found
    Processing Record 47 of Set 18 | nikolskoye,ru
    Data found (status code: 200) | nikolskoye,ru
    Data loaded successfully | nikolskoye,ru
    *********************************
    Processing Record 48 of Set 18 | ajdabiya,ly
    Data found (status code: 200) | ajdabiya,ly
    Data loaded successfully | ajdabiya,ly
    *********************************
    Processing Record 49 of Set 18 | pecos,us
    Data found (status code: 200) | pecos,us
    Data loaded successfully | pecos,us
    *********************************
    Processing Record 50 of Set 18 | fortuna,us
    Data found (status code: 200) | fortuna,us
    Data loaded successfully | fortuna,us
    *********************************
    Processing Record 51 of Set 18 | chaman,pk
    Data found (status code: 200) | chaman,pk
    Data loaded successfully | chaman,pk
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 19 | mehriz,ir
    Data found (status code: 200) | mehriz,ir
    Data loaded successfully | mehriz,ir
    *********************************
    Processing Record 1 of Set 19 | yaan,cn
    ERROR: Data retrieval unsuccessful for yaan,cn. Code: 404                       MSG: city not found
    Processing Record 2 of Set 19 | puerto palomas,mx
    Data found (status code: 200) | puerto palomas,mx
    Data loaded successfully | puerto palomas,mx
    *********************************
    Processing Record 3 of Set 19 | zabol,ir
    Data found (status code: 200) | zabol,ir
    Data loaded successfully | zabol,ir
    *********************************
    Processing Record 4 of Set 19 | ascension,mx
    ERROR: Data retrieval unsuccessful for ascension,mx. Code: 404                       MSG: city not found
    Processing Record 5 of Set 19 | hit,iq
    Data found (status code: 200) | hit,iq
    Data loaded successfully | hit,iq
    *********************************
    Processing Record 6 of Set 19 | sabha,jo
    Data found (status code: 200) | sabha,jo
    Data loaded successfully | sabha,jo
    *********************************
    Processing Record 7 of Set 19 | rosetta,eg
    Data found (status code: 200) | rosetta,eg
    Data loaded successfully | rosetta,eg
    *********************************
    Processing Record 8 of Set 19 | asfi,ma
    ERROR: Data retrieval unsuccessful for asfi,ma. Code: 404                       MSG: city not found
    Processing Record 9 of Set 19 | sundarnagar,in
    Data found (status code: 200) | sundarnagar,in
    Data loaded successfully | sundarnagar,in
    *********************************
    Processing Record 10 of Set 19 | torbay,ca
    Data found (status code: 200) | torbay,ca
    Data loaded successfully | torbay,ca
    *********************************
    Processing Record 11 of Set 19 | douglas,us
    Data found (status code: 200) | douglas,us
    Data loaded successfully | douglas,us
    *********************************
    Processing Record 12 of Set 19 | machico,pt
    Data found (status code: 200) | machico,pt
    Data loaded successfully | machico,pt
    *********************************
    Processing Record 13 of Set 19 | big spring,us
    Data found (status code: 200) | big spring,us
    Data loaded successfully | big spring,us
    *********************************
    Processing Record 14 of Set 19 | danjiangkou,cn
    Data found (status code: 200) | danjiangkou,cn
    Data loaded successfully | danjiangkou,cn
    *********************************
    Processing Record 15 of Set 19 | borujan,ir
    ERROR: Data retrieval unsuccessful for borujan,ir. Code: 404                       MSG: city not found
    Processing Record 16 of Set 19 | huilong,cn
    Data found (status code: 200) | huilong,cn
    Data loaded successfully | huilong,cn
    *********************************
    Processing Record 17 of Set 19 | progreso,mx
    Data found (status code: 200) | progreso,mx
    Data loaded successfully | progreso,mx
    *********************************
    Processing Record 18 of Set 19 | gatesville,us
    Data found (status code: 200) | gatesville,us
    Data loaded successfully | gatesville,us
    *********************************
    Processing Record 19 of Set 19 | dublin,us
    Data found (status code: 200) | dublin,us
    Data loaded successfully | dublin,us
    *********************************
    Processing Record 20 of Set 19 | havelock,us
    Data found (status code: 200) | havelock,us
    Data loaded successfully | havelock,us
    *********************************
    Processing Record 21 of Set 19 | joshimath,in
    Data found (status code: 200) | joshimath,in
    Data loaded successfully | joshimath,in
    *********************************
    Processing Record 22 of Set 19 | liliani,pk
    Data found (status code: 200) | liliani,pk
    Data loaded successfully | liliani,pk
    *********************************
    Processing Record 23 of Set 19 | tezu,in
    Data found (status code: 200) | tezu,in
    Data loaded successfully | tezu,in
    *********************************
    Processing Record 24 of Set 19 | deir hanna,il
    Data found (status code: 200) | deir hanna,il
    Data loaded successfully | deir hanna,il
    *********************************
    Processing Record 25 of Set 19 | rancho palos verdes,us
    Data found (status code: 200) | rancho palos verdes,us
    Data loaded successfully | rancho palos verdes,us
    *********************************
    Processing Record 26 of Set 19 | shahreza,ir
    Data found (status code: 200) | shahreza,ir
    Data loaded successfully | shahreza,ir
    *********************************
    Processing Record 27 of Set 19 | pasighat,in
    Data found (status code: 200) | pasighat,in
    Data loaded successfully | pasighat,in
    *********************************
    Processing Record 28 of Set 19 | dehloran,ir
    Data found (status code: 200) | dehloran,ir
    Data loaded successfully | dehloran,ir
    *********************************
    Processing Record 29 of Set 19 | brownfield,us
    Data found (status code: 200) | brownfield,us
    Data loaded successfully | brownfield,us
    *********************************
    Processing Record 30 of Set 19 | north myrtle beach,us
    Data found (status code: 200) | north myrtle beach,us
    Data loaded successfully | north myrtle beach,us
    *********************************
    Processing Record 31 of Set 19 | vila franca do campo,pt
    Data found (status code: 200) | vila franca do campo,pt
    Data loaded successfully | vila franca do campo,pt
    *********************************
    Processing Record 32 of Set 19 | beppu,jp
    Data found (status code: 200) | beppu,jp
    Data loaded successfully | beppu,jp
    *********************************
    Processing Record 33 of Set 19 | columbus,us
    Data found (status code: 200) | columbus,us
    Data loaded successfully | columbus,us
    *********************************
    Processing Record 34 of Set 19 | artesia,us
    Data found (status code: 200) | artesia,us
    Data loaded successfully | artesia,us
    *********************************
    Processing Record 35 of Set 19 | georgetown,us
    Data found (status code: 200) | georgetown,us
    Data loaded successfully | georgetown,us
    *********************************
    Processing Record 36 of Set 19 | centerville,us
    Data found (status code: 200) | centerville,us
    Data loaded successfully | centerville,us
    *********************************
    Processing Record 37 of Set 19 | xining,cn
    Data found (status code: 200) | xining,cn
    Data loaded successfully | xining,cn
    *********************************
    Processing Record 38 of Set 19 | orangeburg,us
    Data found (status code: 200) | orangeburg,us
    Data loaded successfully | orangeburg,us
    *********************************
    Processing Record 39 of Set 19 | roswell,us
    Data found (status code: 200) | roswell,us
    Data loaded successfully | roswell,us
    *********************************
    Processing Record 40 of Set 19 | hope,us
    Data found (status code: 200) | hope,us
    Data loaded successfully | hope,us
    *********************************
    Processing Record 41 of Set 19 | camacha,pt
    Data found (status code: 200) | camacha,pt
    Data loaded successfully | camacha,pt
    *********************************
    Processing Record 42 of Set 19 | ponta do sol,pt
    Data found (status code: 200) | ponta do sol,pt
    Data loaded successfully | ponta do sol,pt
    *********************************
    Processing Record 43 of Set 19 | masallatah,ly
    Data found (status code: 200) | masallatah,ly
    Data loaded successfully | masallatah,ly
    *********************************
    Processing Record 44 of Set 19 | koutsouras,gr
    Data found (status code: 200) | koutsouras,gr
    Data loaded successfully | koutsouras,gr
    *********************************
    Processing Record 45 of Set 19 | abu kamal,sy
    Data found (status code: 200) | abu kamal,sy
    Data loaded successfully | abu kamal,sy
    *********************************
    Processing Record 46 of Set 19 | ruidoso,us
    Data found (status code: 200) | ruidoso,us
    Data loaded successfully | ruidoso,us
    *********************************
    Processing Record 47 of Set 19 | taywarah,af
    Data found (status code: 200) | taywarah,af
    Data loaded successfully | taywarah,af
    *********************************
    Processing Record 48 of Set 19 | arak,ir
    Data found (status code: 200) | arak,ir
    Data loaded successfully | arak,ir
    *********************************
    Processing Record 49 of Set 19 | toba,jp
    Data found (status code: 200) | toba,jp
    Data loaded successfully | toba,jp
    *********************************
    Processing Record 50 of Set 19 | linxia,cn
    Data found (status code: 200) | linxia,cn
    Data loaded successfully | linxia,cn
    *********************************
    Processing Record 51 of Set 19 | xuchang,cn
    Data found (status code: 200) | xuchang,cn
    Data loaded successfully | xuchang,cn
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 20 | jishui,cn
    Data found (status code: 200) | jishui,cn
    Data loaded successfully | jishui,cn
    *********************************
    Processing Record 1 of Set 20 | socorro,us
    Data found (status code: 200) | socorro,us
    Data loaded successfully | socorro,us
    *********************************
    Processing Record 2 of Set 20 | gobo,jp
    Data found (status code: 200) | gobo,jp
    Data loaded successfully | gobo,jp
    *********************************
    Processing Record 3 of Set 20 | naftah,tn
    ERROR: Data retrieval unsuccessful for naftah,tn. Code: 404                       MSG: city not found
    Processing Record 4 of Set 20 | jasper,us
    Data found (status code: 200) | jasper,us
    Data loaded successfully | jasper,us
    *********************************
    Processing Record 5 of Set 20 | west point,us
    Data found (status code: 200) | west point,us
    Data loaded successfully | west point,us
    *********************************
    Processing Record 6 of Set 20 | hirado,jp
    Data found (status code: 200) | hirado,jp
    Data loaded successfully | hirado,jp
    *********************************
    Processing Record 7 of Set 20 | dancheng,cn
    Data found (status code: 200) | dancheng,cn
    Data loaded successfully | dancheng,cn
    *********************************
    Processing Record 8 of Set 20 | izumi,jp
    Data found (status code: 200) | izumi,jp
    Data loaded successfully | izumi,jp
    *********************************
    Processing Record 9 of Set 20 | yumen,cn
    Data found (status code: 200) | yumen,cn
    Data loaded successfully | yumen,cn
    *********************************
    Processing Record 10 of Set 20 | tucumcari,us
    Data found (status code: 200) | tucumcari,us
    Data loaded successfully | tucumcari,us
    *********************************
    Processing Record 11 of Set 20 | dingli,mt
    Data found (status code: 200) | dingli,mt
    Data loaded successfully | dingli,mt
    *********************************
    Processing Record 12 of Set 20 | yatou,cn
    Data found (status code: 200) | yatou,cn
    Data loaded successfully | yatou,cn
    *********************************
    Processing Record 13 of Set 20 | stuttgart,us
    Data found (status code: 200) | stuttgart,us
    Data loaded successfully | stuttgart,us
    *********************************
    Processing Record 14 of Set 20 | rizhao,cn
    Data found (status code: 200) | rizhao,cn
    Data loaded successfully | rizhao,cn
    *********************************
    Processing Record 15 of Set 20 | futtsu,jp
    Data found (status code: 200) | futtsu,jp
    Data loaded successfully | futtsu,jp
    *********************************
    Processing Record 16 of Set 20 | canyon,us
    Data found (status code: 200) | canyon,us
    Data loaded successfully | canyon,us
    *********************************
    Processing Record 17 of Set 20 | zurrieq,mt
    Data found (status code: 200) | zurrieq,mt
    Data loaded successfully | zurrieq,mt
    *********************************
    Processing Record 18 of Set 20 | herat,af
    Data found (status code: 200) | herat,af
    Data loaded successfully | herat,af
    *********************************
    Processing Record 19 of Set 20 | apple valley,us
    Data found (status code: 200) | apple valley,us
    Data loaded successfully | apple valley,us
    *********************************
    Processing Record 20 of Set 20 | dumas,us
    Data found (status code: 200) | dumas,us
    Data loaded successfully | dumas,us
    *********************************
    Processing Record 21 of Set 20 | sinjar,iq
    Data found (status code: 200) | sinjar,iq
    Data loaded successfully | sinjar,iq
    *********************************
    Processing Record 22 of Set 20 | martil,ma
    Data found (status code: 200) | martil,ma
    Data loaded successfully | martil,ma
    *********************************
    Processing Record 23 of Set 20 | obama,jp
    Data found (status code: 200) | obama,jp
    Data loaded successfully | obama,jp
    *********************************
    Processing Record 24 of Set 20 | leh,in
    Data found (status code: 200) | leh,in
    Data loaded successfully | leh,in
    *********************************
    Processing Record 25 of Set 20 | kenitra,ma
    Data found (status code: 200) | kenitra,ma
    Data loaded successfully | kenitra,ma
    *********************************
    Processing Record 26 of Set 20 | karpathos,gr
    Data found (status code: 200) | karpathos,gr
    Data loaded successfully | karpathos,gr
    *********************************
    Processing Record 27 of Set 20 | garmsar,ir
    Data found (status code: 200) | garmsar,ir
    Data loaded successfully | garmsar,ir
    *********************************
    Processing Record 28 of Set 20 | morro bay,us
    Data found (status code: 200) | morro bay,us
    Data loaded successfully | morro bay,us
    *********************************
    Processing Record 29 of Set 20 | sarakhs,ir
    Data found (status code: 200) | sarakhs,ir
    Data loaded successfully | sarakhs,ir
    *********************************
    Processing Record 30 of Set 20 | gardan diwal,af
    ERROR: Data retrieval unsuccessful for gardan diwal,af. Code: 404                       MSG: city not found
    Processing Record 31 of Set 20 | kumluca,tr
    Data found (status code: 200) | kumluca,tr
    Data loaded successfully | kumluca,tr
    *********************************
    Processing Record 32 of Set 20 | fayetteville,us
    Data found (status code: 200) | fayetteville,us
    Data loaded successfully | fayetteville,us
    *********************************
    Processing Record 33 of Set 20 | tazmalt,dz
    Data found (status code: 200) | tazmalt,dz
    Data loaded successfully | tazmalt,dz
    *********************************
    Processing Record 34 of Set 20 | ukiah,us
    Data found (status code: 200) | ukiah,us
    Data loaded successfully | ukiah,us
    *********************************
    Processing Record 35 of Set 20 | hunza,pk
    ERROR: Data retrieval unsuccessful for hunza,pk. Code: 404                       MSG: city not found
    Processing Record 36 of Set 20 | manzil salim,tn
    Data found (status code: 200) | manzil salim,tn
    Data loaded successfully | manzil salim,tn
    *********************************
    Processing Record 37 of Set 20 | mooresville,us
    Data found (status code: 200) | mooresville,us
    Data loaded successfully | mooresville,us
    *********************************
    Processing Record 38 of Set 20 | asmar,af
    Data found (status code: 200) | asmar,af
    Data loaded successfully | asmar,af
    *********************************
    Processing Record 39 of Set 20 | kargil,in
    Data found (status code: 200) | kargil,in
    Data loaded successfully | kargil,in
    *********************************
    Processing Record 40 of Set 20 | puyang,cn
    Data found (status code: 200) | puyang,cn
    Data loaded successfully | puyang,cn
    *********************************
    Processing Record 41 of Set 20 | wuan,cn
    Data found (status code: 200) | wuan,cn
    Data loaded successfully | wuan,cn
    *********************************
    Processing Record 42 of Set 20 | kamaishi,jp
    Data found (status code: 200) | kamaishi,jp
    Data loaded successfully | kamaishi,jp
    *********************************
    Processing Record 43 of Set 20 | tsubata,jp
    Data found (status code: 200) | tsubata,jp
    Data loaded successfully | tsubata,jp
    *********************************
    Processing Record 44 of Set 20 | blytheville,us
    Data found (status code: 200) | blytheville,us
    Data loaded successfully | blytheville,us
    *********************************
    Processing Record 45 of Set 20 | manbij,sy
    Data found (status code: 200) | manbij,sy
    Data loaded successfully | manbij,sy
    *********************************
    Processing Record 46 of Set 20 | monte gordo,pt
    Data found (status code: 200) | monte gordo,pt
    Data loaded successfully | monte gordo,pt
    *********************************
    Processing Record 47 of Set 20 | handan,cn
    Data found (status code: 200) | handan,cn
    Data loaded successfully | handan,cn
    *********************************
    Processing Record 48 of Set 20 | viransehir,tr
    Data found (status code: 200) | viransehir,tr
    Data loaded successfully | viransehir,tr
    *********************************
    Processing Record 49 of Set 20 | kingsport,us
    Data found (status code: 200) | kingsport,us
    Data loaded successfully | kingsport,us
    *********************************
    Processing Record 50 of Set 20 | anamur,tr
    Data found (status code: 200) | anamur,tr
    Data loaded successfully | anamur,tr
    *********************************
    Processing Record 51 of Set 20 | berea,us
    Data found (status code: 200) | berea,us
    Data loaded successfully | berea,us
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 21 | bozova,tr
    Data found (status code: 200) | bozova,tr
    Data loaded successfully | bozova,tr
    *********************************
    Processing Record 1 of Set 21 | pacifica,us
    Data found (status code: 200) | pacifica,us
    Data loaded successfully | pacifica,us
    *********************************
    Processing Record 2 of Set 21 | khalkhal,ir
    Data found (status code: 200) | khalkhal,ir
    Data loaded successfully | khalkhal,ir
    *********************************
    Processing Record 3 of Set 21 | jinan,cn
    Data found (status code: 200) | jinan,cn
    Data loaded successfully | jinan,cn
    *********************************
    Processing Record 4 of Set 21 | roquetas de mar,es
    Data found (status code: 200) | roquetas de mar,es
    Data loaded successfully | roquetas de mar,es
    *********************************
    Processing Record 5 of Set 21 | bukan,ir
    Data found (status code: 200) | bukan,ir
    Data loaded successfully | bukan,ir
    *********************************
    Processing Record 6 of Set 21 | berja,es
    Data found (status code: 200) | berja,es
    Data loaded successfully | berja,es
    *********************************
    Processing Record 7 of Set 21 | guymon,us
    Data found (status code: 200) | guymon,us
    Data loaded successfully | guymon,us
    *********************************
    Processing Record 8 of Set 21 | baiyin,cn
    Data found (status code: 200) | baiyin,cn
    Data loaded successfully | baiyin,cn
    *********************************
    Processing Record 9 of Set 21 | namie,jp
    Data found (status code: 200) | namie,jp
    Data loaded successfully | namie,jp
    *********************************
    Processing Record 10 of Set 21 | dodge city,us
    Data found (status code: 200) | dodge city,us
    Data loaded successfully | dodge city,us
    *********************************
    Processing Record 11 of Set 21 | salisbury,us
    Data found (status code: 200) | salisbury,us
    Data loaded successfully | salisbury,us
    *********************************
    Processing Record 12 of Set 21 | constantine,dz
    Data found (status code: 200) | constantine,dz
    Data loaded successfully | constantine,dz
    *********************************
    Processing Record 13 of Set 21 | gorgan,ir
    Data found (status code: 200) | gorgan,ir
    Data loaded successfully | gorgan,ir
    *********************************
    Processing Record 14 of Set 21 | korla,cn
    Data found (status code: 200) | korla,cn
    Data loaded successfully | korla,cn
    *********************************
    Processing Record 15 of Set 21 | poros,gr
    Data found (status code: 200) | poros,gr
    Data loaded successfully | poros,gr
    *********************************
    Processing Record 16 of Set 21 | oshnaviyeh,ir
    Data found (status code: 200) | oshnaviyeh,ir
    Data loaded successfully | oshnaviyeh,ir
    *********************************
    Processing Record 17 of Set 21 | yanan,cn
    ERROR: Data retrieval unsuccessful for yanan,cn. Code: 404                       MSG: city not found
    Processing Record 18 of Set 21 | zhangye,cn
    Data found (status code: 200) | zhangye,cn
    Data loaded successfully | zhangye,cn
    *********************************
    Processing Record 19 of Set 21 | hutchinson,us
    Data found (status code: 200) | hutchinson,us
    Data loaded successfully | hutchinson,us
    *********************************
    Processing Record 20 of Set 21 | la union,es
    Data found (status code: 200) | la union,es
    Data loaded successfully | la union,es
    *********************************
    Processing Record 21 of Set 21 | montrose,us
    Data found (status code: 200) | montrose,us
    Data loaded successfully | montrose,us
    *********************************
    Processing Record 22 of Set 21 | gonbad-e qabus,ir
    Data found (status code: 200) | gonbad-e qabus,ir
    Data loaded successfully | gonbad-e qabus,ir
    *********************************
    Processing Record 23 of Set 21 | bandar-e torkaman,ir
    ERROR: Data retrieval unsuccessful for bandar-e torkaman,ir. Code: 404                       MSG: city not found
    Processing Record 24 of Set 21 | kardamas,gr
    Data found (status code: 200) | kardamas,gr
    Data loaded successfully | kardamas,gr
    *********************************
    Processing Record 25 of Set 21 | nantucket,us
    Data found (status code: 200) | nantucket,us
    Data loaded successfully | nantucket,us
    *********************************
    Processing Record 26 of Set 21 | hollins,us
    Data found (status code: 200) | hollins,us
    Data loaded successfully | hollins,us
    *********************************
    Processing Record 27 of Set 21 | andros,gr
    Data found (status code: 200) | andros,gr
    Data loaded successfully | andros,gr
    *********************************
    Processing Record 28 of Set 21 | fallon,us
    Data found (status code: 200) | fallon,us
    Data loaded successfully | fallon,us
    *********************************
    Processing Record 29 of Set 21 | astara,ir
    Data found (status code: 200) | astara,ir
    Data loaded successfully | astara,ir
    *********************************
    Processing Record 30 of Set 21 | genc,tr
    Data found (status code: 200) | genc,tr
    Data loaded successfully | genc,tr
    *********************************
    Processing Record 31 of Set 21 | gazanjyk,tm
    Data found (status code: 200) | gazanjyk,tm
    Data loaded successfully | gazanjyk,tm
    *********************************
    Processing Record 32 of Set 21 | mastic beach,us
    Data found (status code: 200) | mastic beach,us
    Data loaded successfully | mastic beach,us
    *********************************
    Processing Record 33 of Set 21 | wajima,jp
    Data found (status code: 200) | wajima,jp
    Data loaded successfully | wajima,jp
    *********************************
    Processing Record 34 of Set 21 | lamar,us
    Data found (status code: 200) | lamar,us
    Data loaded successfully | lamar,us
    *********************************
    Processing Record 35 of Set 21 | ahar,ir
    Data found (status code: 200) | ahar,ir
    Data loaded successfully | ahar,ir
    *********************************
    Processing Record 36 of Set 21 | topeka,us
    Data found (status code: 200) | topeka,us
    Data loaded successfully | topeka,us
    *********************************
    Processing Record 37 of Set 21 | silvan,tr
    Data found (status code: 200) | silvan,tr
    Data loaded successfully | silvan,tr
    *********************************
    Processing Record 38 of Set 21 | benidorm,es
    Data found (status code: 200) | benidorm,es
    Data loaded successfully | benidorm,es
    *********************************
    Processing Record 39 of Set 21 | kodiak,us
    Data found (status code: 200) | kodiak,us
    Data loaded successfully | kodiak,us
    *********************************
    Processing Record 40 of Set 21 | garden city,us
    Data found (status code: 200) | garden city,us
    Data loaded successfully | garden city,us
    *********************************
    Processing Record 41 of Set 21 | athens,us
    Data found (status code: 200) | athens,us
    Data loaded successfully | athens,us
    *********************************
    Processing Record 42 of Set 21 | emirdag,tr
    Data found (status code: 200) | emirdag,tr
    Data loaded successfully | emirdag,tr
    *********************************
    Processing Record 43 of Set 21 | marand,ir
    Data found (status code: 200) | marand,ir
    Data loaded successfully | marand,ir
    *********************************
    Processing Record 44 of Set 21 | seoul,kr
    Data found (status code: 200) | seoul,kr
    Data loaded successfully | seoul,kr
    *********************************
    Processing Record 45 of Set 21 | sakaiminato,jp
    Data found (status code: 200) | sakaiminato,jp
    Data loaded successfully | sakaiminato,jp
    *********************************
    Processing Record 46 of Set 21 | dongsheng,cn
    Data found (status code: 200) | dongsheng,cn
    Data loaded successfully | dongsheng,cn
    *********************************
    Processing Record 47 of Set 21 | glenwood springs,us
    Data found (status code: 200) | glenwood springs,us
    Data loaded successfully | glenwood springs,us
    *********************************
    Processing Record 48 of Set 21 | colares,pt
    Data found (status code: 200) | colares,pt
    Data loaded successfully | colares,pt
    *********************************
    Processing Record 49 of Set 21 | estremoz,pt
    Data found (status code: 200) | estremoz,pt
    Data loaded successfully | estremoz,pt
    *********************************
    Processing Record 50 of Set 21 | fernley,us
    Data found (status code: 200) | fernley,us
    Data loaded successfully | fernley,us
    *********************************
    Processing Record 51 of Set 21 | rabo de peixe,pt
    Data found (status code: 200) | rabo de peixe,pt
    Data loaded successfully | rabo de peixe,pt
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 22 | rossano,it
    Data found (status code: 200) | rossano,it
    Data loaded successfully | rossano,it
    *********************************
    Processing Record 1 of Set 22 | front royal,us
    Data found (status code: 200) | front royal,us
    Data loaded successfully | front royal,us
    *********************************
    Processing Record 2 of Set 22 | vienna,us
    Data found (status code: 200) | vienna,us
    Data loaded successfully | vienna,us
    *********************************
    Processing Record 3 of Set 22 | brigantine,us
    Data found (status code: 200) | brigantine,us
    Data loaded successfully | brigantine,us
    *********************************
    Processing Record 4 of Set 22 | seydi,tm
    Data found (status code: 200) | seydi,tm
    Data loaded successfully | seydi,tm
    *********************************
    Processing Record 5 of Set 22 | saint-pierre,pm
    Data found (status code: 200) | saint-pierre,pm
    Data loaded successfully | saint-pierre,pm
    *********************************
    Processing Record 6 of Set 22 | reno,us
    Data found (status code: 200) | reno,us
    Data loaded successfully | reno,us
    *********************************
    Processing Record 7 of Set 22 | columbia,us
    Data found (status code: 200) | columbia,us
    Data loaded successfully | columbia,us
    *********************************
    Processing Record 8 of Set 22 | oliva,es
    Data found (status code: 200) | oliva,es
    Data loaded successfully | oliva,es
    *********************************
    Processing Record 9 of Set 22 | khasan,ru
    Data found (status code: 200) | khasan,ru
    Data loaded successfully | khasan,ru
    *********************************
    Processing Record 10 of Set 22 | shache,cn
    Data found (status code: 200) | shache,cn
    Data loaded successfully | shache,cn
    *********************************
    Processing Record 11 of Set 22 | black forest,us
    Data found (status code: 200) | black forest,us
    Data loaded successfully | black forest,us
    *********************************
    Processing Record 12 of Set 22 | dasoguz,tm
    Data found (status code: 200) | dasoguz,tm
    Data loaded successfully | dasoguz,tm
    *********************************
    Processing Record 13 of Set 22 | halifax,ca
    Data found (status code: 200) | halifax,ca
    Data loaded successfully | halifax,ca
    *********************************
    Processing Record 14 of Set 22 | burhaniye,tr
    Data found (status code: 200) | burhaniye,tr
    Data loaded successfully | burhaniye,tr
    *********************************
    Processing Record 15 of Set 22 | sabirabad,az
    Data found (status code: 200) | sabirabad,az
    Data loaded successfully | sabirabad,az
    *********************************
    Processing Record 16 of Set 22 | chkalovsk,tj
    ERROR: Data retrieval unsuccessful for chkalovsk,tj. Code: 404                       MSG: city not found
    Processing Record 17 of Set 22 | noramarg,am
    Data found (status code: 200) | noramarg,am
    Data loaded successfully | noramarg,am
    *********************************
    Processing Record 18 of Set 22 | antissa,gr
    ERROR: Data retrieval unsuccessful for antissa,gr. Code: 404                       MSG: city not found
    Processing Record 19 of Set 22 | shitanjing,cn
    Data found (status code: 200) | shitanjing,cn
    Data loaded successfully | shitanjing,cn
    *********************************
    Processing Record 20 of Set 22 | oga,jp
    ERROR: Data retrieval unsuccessful for oga,jp. Code: 404                       MSG: city not found
    Processing Record 21 of Set 22 | nemuro,jp
    Data found (status code: 200) | nemuro,jp
    Data loaded successfully | nemuro,jp
    *********************************
    Processing Record 22 of Set 22 | miyako,jp
    Data found (status code: 200) | miyako,jp
    Data loaded successfully | miyako,jp
    *********************************
    Processing Record 23 of Set 22 | gazojak,tm
    Data found (status code: 200) | gazojak,tm
    Data loaded successfully | gazojak,tm
    *********************************
    Processing Record 24 of Set 22 | jiayuguan,cn
    Data found (status code: 200) | jiayuguan,cn
    Data loaded successfully | jiayuguan,cn
    *********************************
    Processing Record 25 of Set 22 | siniscola,it
    Data found (status code: 200) | siniscola,it
    Data loaded successfully | siniscola,it
    *********************************
    Processing Record 26 of Set 22 | lagoa,pt
    Data found (status code: 200) | lagoa,pt
    Data loaded successfully | lagoa,pt
    *********************************
    Processing Record 27 of Set 22 | shelburne,ca
    Data found (status code: 200) | shelburne,ca
    Data loaded successfully | shelburne,ca
    *********************************
    Processing Record 28 of Set 22 | kuche,cn
    ERROR: Data retrieval unsuccessful for kuche,cn. Code: 404                       MSG: city not found
    Processing Record 29 of Set 22 | soure,pt
    Data found (status code: 200) | soure,pt
    Data loaded successfully | soure,pt
    *********************************
    Processing Record 30 of Set 22 | salou,es
    Data found (status code: 200) | salou,es
    Data loaded successfully | salou,es
    *********************************
    Processing Record 31 of Set 22 | xingcheng,cn
    Data found (status code: 200) | xingcheng,cn
    Data loaded successfully | xingcheng,cn
    *********************************
    Processing Record 32 of Set 22 | louisbourg,ca
    ERROR: Data retrieval unsuccessful for louisbourg,ca. Code: 404                       MSG: city not found
    Processing Record 33 of Set 22 | nardaran,az
    Data found (status code: 200) | nardaran,az
    Data loaded successfully | nardaran,az
    *********************************
    Processing Record 34 of Set 22 | winnemucca,us
    Data found (status code: 200) | winnemucca,us
    Data loaded successfully | winnemucca,us
    *********************************
    Processing Record 35 of Set 22 | almazar,uz
    Data found (status code: 200) | almazar,uz
    Data loaded successfully | almazar,uz
    *********************************
    Processing Record 36 of Set 22 | hami,cn
    Data found (status code: 200) | hami,cn
    Data loaded successfully | hami,cn
    *********************************
    Processing Record 37 of Set 22 | streator,us
    Data found (status code: 200) | streator,us
    Data loaded successfully | streator,us
    *********************************
    Processing Record 38 of Set 22 | yarmouth,ca
    Data found (status code: 200) | yarmouth,ca
    Data loaded successfully | yarmouth,ca
    *********************************
    Processing Record 39 of Set 22 | state college,us
    Data found (status code: 200) | state college,us
    Data loaded successfully | state college,us
    *********************************
    Processing Record 40 of Set 22 | andijon,uz
    Data found (status code: 200) | andijon,uz
    Data loaded successfully | andijon,uz
    *********************************
    Processing Record 41 of Set 22 | akcaabat,tr
    Data found (status code: 200) | akcaabat,tr
    Data loaded successfully | akcaabat,tr
    *********************************
    Processing Record 42 of Set 22 | huntington,us
    Data found (status code: 200) | huntington,us
    Data loaded successfully | huntington,us
    *********************************
    Processing Record 43 of Set 22 | gilazi,az
    ERROR: Data retrieval unsuccessful for gilazi,az. Code: 404                       MSG: city not found
    Processing Record 44 of Set 22 | bethel,us
    Data found (status code: 200) | bethel,us
    Data loaded successfully | bethel,us
    *********************************
    Processing Record 45 of Set 22 | grand island,us
    Data found (status code: 200) | grand island,us
    Data loaded successfully | grand island,us
    *********************************
    Processing Record 46 of Set 22 | north platte,us
    Data found (status code: 200) | north platte,us
    Data loaded successfully | north platte,us
    *********************************
    Processing Record 47 of Set 22 | port hawkesbury,ca
    Data found (status code: 200) | port hawkesbury,ca
    Data loaded successfully | port hawkesbury,ca
    *********************************
    Processing Record 48 of Set 22 | klos,al
    Data found (status code: 200) | klos,al
    Data loaded successfully | klos,al
    *********************************
    Processing Record 49 of Set 22 | akdepe,tm
    Data found (status code: 200) | akdepe,tm
    Data loaded successfully | akdepe,tm
    *********************************
    Processing Record 50 of Set 22 | chardara,kz
    ERROR: Data retrieval unsuccessful for chardara,kz. Code: 404                       MSG: city not found
    Processing Record 51 of Set 22 | bari,it
    Data found (status code: 200) | bari,it
    Data loaded successfully | bari,it
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 23 | besikduzu,tr
    Data found (status code: 200) | besikduzu,tr
    Data loaded successfully | besikduzu,tr
    *********************************
    Processing Record 1 of Set 23 | peniche,pt
    Data found (status code: 200) | peniche,pt
    Data loaded successfully | peniche,pt
    *********************************
    Processing Record 2 of Set 23 | provideniya,ru
    Data found (status code: 200) | provideniya,ru
    Data loaded successfully | provideniya,ru
    *********************************
    Processing Record 3 of Set 23 | eureka,us
    Data found (status code: 200) | eureka,us
    Data loaded successfully | eureka,us
    *********************************
    Processing Record 4 of Set 23 | triggiano,it
    Data found (status code: 200) | triggiano,it
    Data loaded successfully | triggiano,it
    *********************************
    Processing Record 5 of Set 23 | muros,es
    Data found (status code: 200) | muros,es
    Data loaded successfully | muros,es
    *********************************
    Processing Record 6 of Set 23 | safranbolu,tr
    Data found (status code: 200) | safranbolu,tr
    Data loaded successfully | safranbolu,tr
    *********************************
    Processing Record 7 of Set 23 | jining,cn
    Data found (status code: 200) | jining,cn
    Data loaded successfully | jining,cn
    *********************************
    Processing Record 8 of Set 23 | songjianghe,cn
    Data found (status code: 200) | songjianghe,cn
    Data loaded successfully | songjianghe,cn
    *********************************
    Processing Record 9 of Set 23 | beruni,uz
    ERROR: Data retrieval unsuccessful for beruni,uz. Code: 404                       MSG: city not found
    Processing Record 10 of Set 23 | porto-vecchio,fr
    Data found (status code: 200) | porto-vecchio,fr
    Data loaded successfully | porto-vecchio,fr
    *********************************
    Processing Record 11 of Set 23 | saint marys,us
    Data found (status code: 200) | saint marys,us
    Data loaded successfully | saint marys,us
    *********************************
    Processing Record 12 of Set 23 | kingston,us
    Data found (status code: 200) | kingston,us
    Data loaded successfully | kingston,us
    *********************************
    Processing Record 13 of Set 23 | port hardy,ca
    Data found (status code: 200) | port hardy,ca
    Data loaded successfully | port hardy,ca
    *********************************
    Processing Record 14 of Set 23 | tarazona,es
    Data found (status code: 200) | tarazona,es
    Data loaded successfully | tarazona,es
    *********************************
    Processing Record 15 of Set 23 | twin falls,us
    Data found (status code: 200) | twin falls,us
    Data loaded successfully | twin falls,us
    *********************************
    Processing Record 16 of Set 23 | hyeres,fr
    Data found (status code: 200) | hyeres,fr
    Data loaded successfully | hyeres,fr
    *********************************
    Processing Record 17 of Set 23 | evanston,us
    Data found (status code: 200) | evanston,us
    Data loaded successfully | evanston,us
    *********************************
    Processing Record 18 of Set 23 | mizur,ru
    Data found (status code: 200) | mizur,ru
    Data loaded successfully | mizur,ru
    *********************************
    Processing Record 19 of Set 23 | nurota,uz
    Data found (status code: 200) | nurota,uz
    Data loaded successfully | nurota,uz
    *********************************
    Processing Record 20 of Set 23 | hovd,mn
    Data found (status code: 200) | hovd,mn
    Data loaded successfully | hovd,mn
    *********************************
    Processing Record 21 of Set 23 | coos bay,us
    Data found (status code: 200) | coos bay,us
    Data loaded successfully | coos bay,us
    *********************************
    Processing Record 22 of Set 23 | mangit,uz
    Data found (status code: 200) | mangit,uz
    Data loaded successfully | mangit,uz
    *********************************
    Processing Record 23 of Set 23 | vrangel,ru
    Data found (status code: 200) | vrangel,ru
    Data loaded successfully | vrangel,ru
    *********************************
    Processing Record 24 of Set 23 | kuryk,kz
    Data found (status code: 200) | kuryk,kz
    Data loaded successfully | kuryk,kz
    *********************************
    Processing Record 25 of Set 23 | lourdes,fr
    Data found (status code: 200) | lourdes,fr
    Data loaded successfully | lourdes,fr
    *********************************
    Processing Record 26 of Set 23 | babusnica,rs
    Data found (status code: 200) | babusnica,rs
    Data loaded successfully | babusnica,rs
    *********************************
    Processing Record 27 of Set 23 | burgas,bg
    Data found (status code: 200) | burgas,bg
    Data loaded successfully | burgas,bg
    *********************************
    Processing Record 28 of Set 23 | olot,es
    Data found (status code: 200) | olot,es
    Data loaded successfully | olot,es
    *********************************
    Processing Record 29 of Set 23 | shieli,kz
    Data found (status code: 200) | shieli,kz
    Data loaded successfully | shieli,kz
    *********************************
    Processing Record 30 of Set 23 | waverly,us
    Data found (status code: 200) | waverly,us
    Data loaded successfully | waverly,us
    *********************************
    Processing Record 31 of Set 23 | mountain home,us
    Data found (status code: 200) | mountain home,us
    Data loaded successfully | mountain home,us
    *********************************
    Processing Record 32 of Set 23 | meihekou,cn
    Data found (status code: 200) | meihekou,cn
    Data loaded successfully | meihekou,cn
    *********************************
    Processing Record 33 of Set 23 | north bend,us
    Data found (status code: 200) | north bend,us
    Data loaded successfully | north bend,us
    *********************************
    Processing Record 34 of Set 23 | utica,us
    Data found (status code: 200) | utica,us
    Data loaded successfully | utica,us
    *********************************
    Processing Record 35 of Set 23 | monsummano terme,it
    Data found (status code: 200) | monsummano terme,it
    Data loaded successfully | monsummano terme,it
    *********************************
    Processing Record 36 of Set 23 | belogradcik,bg
    ERROR: Data retrieval unsuccessful for belogradcik,bg. Code: 404                       MSG: city not found
    Processing Record 37 of Set 23 | baker city,us
    Data found (status code: 200) | baker city,us
    Data loaded successfully | baker city,us
    *********************************
    Processing Record 38 of Set 23 | kushiro,jp
    Data found (status code: 200) | kushiro,jp
    Data loaded successfully | kushiro,jp
    *********************************
    Processing Record 39 of Set 23 | mount pleasant,us
    Data found (status code: 200) | mount pleasant,us
    Data loaded successfully | mount pleasant,us
    *********************************
    Processing Record 40 of Set 23 | lugo,es
    Data found (status code: 200) | lugo,es
    Data loaded successfully | lugo,es
    *********************************
    Processing Record 41 of Set 23 | riverton,us
    Data found (status code: 200) | riverton,us
    Data loaded successfully | riverton,us
    *********************************
    Processing Record 42 of Set 23 | kentau,kz
    Data found (status code: 200) | kentau,kz
    Data loaded successfully | kentau,kz
    *********************************
    Processing Record 43 of Set 23 | dzhusaly,kz
    ERROR: Data retrieval unsuccessful for dzhusaly,kz. Code: 404                       MSG: city not found
    Processing Record 44 of Set 23 | fort-shevchenko,kz
    Data found (status code: 200) | fort-shevchenko,kz
    Data loaded successfully | fort-shevchenko,kz
    *********************************
    Processing Record 45 of Set 23 | shalazhi,ru
    Data found (status code: 200) | shalazhi,ru
    Data loaded successfully | shalazhi,ru
    *********************************
    Processing Record 46 of Set 23 | todi,it
    Data found (status code: 200) | todi,it
    Data loaded successfully | todi,it
    *********************************
    Processing Record 47 of Set 23 | chifeng,cn
    Data found (status code: 200) | chifeng,cn
    Data loaded successfully | chifeng,cn
    *********************************
    Processing Record 48 of Set 23 | georgiyevka,kz
    Data found (status code: 200) | georgiyevka,kz
    Data loaded successfully | georgiyevka,kz
    *********************************
    Processing Record 49 of Set 23 | terney,ru
    Data found (status code: 200) | terney,ru
    Data loaded successfully | terney,ru
    *********************************
    Processing Record 50 of Set 23 | haverhill,us
    Data found (status code: 200) | haverhill,us
    Data loaded successfully | haverhill,us
    *********************************
    Processing Record 51 of Set 23 | zadar,hr
    Data found (status code: 200) | zadar,hr
    Data loaded successfully | zadar,hr
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 24 | mujiayingzi,cn
    Data found (status code: 200) | mujiayingzi,cn
    Data loaded successfully | mujiayingzi,cn
    *********************************
    Processing Record 1 of Set 24 | praia da vitoria,pt
    Data found (status code: 200) | praia da vitoria,pt
    Data loaded successfully | praia da vitoria,pt
    *********************************
    Processing Record 2 of Set 24 | rapid valley,us
    Data found (status code: 200) | rapid valley,us
    Data loaded successfully | rapid valley,us
    *********************************
    Processing Record 3 of Set 24 | darhan,mn
    Data found (status code: 200) | darhan,mn
    Data loaded successfully | darhan,mn
    *********************************
    Processing Record 4 of Set 24 | otradnoye,ru
    Data found (status code: 200) | otradnoye,ru
    Data loaded successfully | otradnoye,ru
    *********************************
    Processing Record 5 of Set 24 | taggia,it
    Data found (status code: 200) | taggia,it
    Data loaded successfully | taggia,it
    *********************************
    Processing Record 6 of Set 24 | rimini,it
    Data found (status code: 200) | rimini,it
    Data loaded successfully | rimini,it
    *********************************
    Processing Record 7 of Set 24 | spearfish,us
    Data found (status code: 200) | spearfish,us
    Data loaded successfully | spearfish,us
    *********************************
    Processing Record 8 of Set 24 | north mankato,us
    Data found (status code: 200) | north mankato,us
    Data loaded successfully | north mankato,us
    *********************************
    Processing Record 9 of Set 24 | huron,us
    Data found (status code: 200) | huron,us
    Data loaded successfully | huron,us
    *********************************
    Processing Record 10 of Set 24 | cavaillon,fr
    Data found (status code: 200) | cavaillon,fr
    Data loaded successfully | cavaillon,fr
    *********************************
    Processing Record 11 of Set 24 | llanes,es
    Data found (status code: 200) | llanes,es
    Data loaded successfully | llanes,es
    *********************************
    Processing Record 12 of Set 24 | saryozek,kz
    Data found (status code: 200) | saryozek,kz
    Data loaded successfully | saryozek,kz
    *********************************
    Processing Record 13 of Set 24 | casselman,ca
    Data found (status code: 200) | casselman,ca
    Data loaded successfully | casselman,ca
    *********************************
    Processing Record 14 of Set 24 | marystown,ca
    Data found (status code: 200) | marystown,ca
    Data loaded successfully | marystown,ca
    *********************************
    Processing Record 15 of Set 24 | sturgis,us
    Data found (status code: 200) | sturgis,us
    Data loaded successfully | sturgis,us
    *********************************
    Processing Record 16 of Set 24 | redmond,us
    Data found (status code: 200) | redmond,us
    Data loaded successfully | redmond,us
    *********************************
    Processing Record 17 of Set 24 | saryshagan,kz
    ERROR: Data retrieval unsuccessful for saryshagan,kz. Code: 404                       MSG: city not found
    Processing Record 18 of Set 24 | beyneu,kz
    Data found (status code: 200) | beyneu,kz
    Data loaded successfully | beyneu,kz
    *********************************
    Processing Record 19 of Set 24 | cecava,ba
    Data found (status code: 200) | cecava,ba
    Data loaded successfully | cecava,ba
    *********************************
    Processing Record 20 of Set 24 | emmett,us
    Data found (status code: 200) | emmett,us
    Data loaded successfully | emmett,us
    *********************************
    Processing Record 21 of Set 24 | suifenhe,cn
    Data found (status code: 200) | suifenhe,cn
    Data loaded successfully | suifenhe,cn
    *********************************
    Processing Record 22 of Set 24 | mandalgovi,mn
    Data found (status code: 200) | mandalgovi,mn
    Data loaded successfully | mandalgovi,mn
    *********************************
    Processing Record 23 of Set 24 | carballo,es
    Data found (status code: 200) | carballo,es
    Data loaded successfully | carballo,es
    *********************************
    Processing Record 24 of Set 24 | newberg,us
    Data found (status code: 200) | newberg,us
    Data loaded successfully | newberg,us
    *********************************
    Processing Record 25 of Set 24 | montreal,ca
    Data found (status code: 200) | montreal,ca
    Data loaded successfully | montreal,ca
    *********************************
    Processing Record 26 of Set 24 | arzgir,ru
    Data found (status code: 200) | arzgir,ru
    Data loaded successfully | arzgir,ru
    *********************************
    Processing Record 27 of Set 24 | karaton,kz
    ERROR: Data retrieval unsuccessful for karaton,kz. Code: 404                       MSG: city not found
    Processing Record 28 of Set 24 | sarkand,kz
    Data found (status code: 200) | sarkand,kz
    Data loaded successfully | sarkand,kz
    *********************************
    Processing Record 29 of Set 24 | tasbuget,kz
    ERROR: Data retrieval unsuccessful for tasbuget,kz. Code: 404                       MSG: city not found
    Processing Record 30 of Set 24 | willmar,us
    Data found (status code: 200) | willmar,us
    Data loaded successfully | willmar,us
    *********************************
    Processing Record 31 of Set 24 | la grande,us
    Data found (status code: 200) | la grande,us
    Data loaded successfully | la grande,us
    *********************************
    Processing Record 32 of Set 24 | iki-burul,ru
    Data found (status code: 200) | iki-burul,ru
    Data loaded successfully | iki-burul,ru
    *********************************
    Processing Record 33 of Set 24 | ucluelet,ca
    Data found (status code: 200) | ucluelet,ca
    Data loaded successfully | ucluelet,ca
    *********************************
    Processing Record 34 of Set 24 | balkhash,kz
    Data found (status code: 200) | balkhash,kz
    Data loaded successfully | balkhash,kz
    *********************************
    Processing Record 35 of Set 24 | baruun-urt,mn
    Data found (status code: 200) | baruun-urt,mn
    Data loaded successfully | baruun-urt,mn
    *********************************
    Processing Record 36 of Set 24 | miles city,us
    Data found (status code: 200) | miles city,us
    Data loaded successfully | miles city,us
    *********************************
    Processing Record 37 of Set 24 | marquette,us
    Data found (status code: 200) | marquette,us
    Data loaded successfully | marquette,us
    *********************************
    Processing Record 38 of Set 24 | kazalinsk,kz
    ERROR: Data retrieval unsuccessful for kazalinsk,kz. Code: 404                       MSG: city not found
    Processing Record 39 of Set 24 | prince rupert,ca
    Data found (status code: 200) | prince rupert,ca
    Data loaded successfully | prince rupert,ca
    *********************************
    Processing Record 40 of Set 24 | anaconda,us
    Data found (status code: 200) | anaconda,us
    Data loaded successfully | anaconda,us
    *********************************
    Processing Record 41 of Set 24 | naron,es
    Data found (status code: 200) | naron,es
    Data loaded successfully | naron,es
    *********************************
    Processing Record 42 of Set 24 | erzin,ru
    Data found (status code: 200) | erzin,ru
    Data loaded successfully | erzin,ru
    *********************************
    Processing Record 43 of Set 24 | sheridan,us
    Data found (status code: 200) | sheridan,us
    Data loaded successfully | sheridan,us
    *********************************
    Processing Record 44 of Set 24 | altay,cn
    Data found (status code: 200) | altay,cn
    Data loaded successfully | altay,cn
    *********************************
    Processing Record 45 of Set 24 | rochefort,fr
    Data found (status code: 200) | rochefort,fr
    Data loaded successfully | rochefort,fr
    *********************************
    Processing Record 46 of Set 24 | pergine valsugana,it
    Data found (status code: 200) | pergine valsugana,it
    Data loaded successfully | pergine valsugana,it
    *********************************
    Processing Record 47 of Set 24 | urdzhar,kz
    ERROR: Data retrieval unsuccessful for urdzhar,kz. Code: 404                       MSG: city not found
    Processing Record 48 of Set 24 | chateauroux,fr
    Data found (status code: 200) | chateauroux,fr
    Data loaded successfully | chateauroux,fr
    *********************************
    Processing Record 49 of Set 24 | sofronea,ro
    Data found (status code: 200) | sofronea,ro
    Data loaded successfully | sofronea,ro
    *********************************
    Processing Record 50 of Set 24 | svetlaya,ru
    Data found (status code: 200) | svetlaya,ru
    Data loaded successfully | svetlaya,ru
    *********************************
    Processing Record 51 of Set 24 | livingston,us
    Data found (status code: 200) | livingston,us
    Data loaded successfully | livingston,us
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 25 | inverness,ca
    Data found (status code: 200) | inverness,ca
    Data loaded successfully | inverness,ca
    *********************************
    Processing Record 1 of Set 25 | ivesti,ro
    Data found (status code: 200) | ivesti,ro
    Data loaded successfully | ivesti,ro
    *********************************
    Processing Record 2 of Set 25 | pembroke,ca
    Data found (status code: 200) | pembroke,ca
    Data loaded successfully | pembroke,ca
    *********************************
    Processing Record 3 of Set 25 | butte,us
    Data found (status code: 200) | butte,us
    Data loaded successfully | butte,us
    *********************************
    Processing Record 4 of Set 25 | channel-port aux basques,ca
    Data found (status code: 200) | channel-port aux basques,ca
    Data loaded successfully | channel-port aux basques,ca
    *********************************
    Processing Record 5 of Set 25 | sitka,us
    Data found (status code: 200) | sitka,us
    Data loaded successfully | sitka,us
    *********************************
    Processing Record 6 of Set 25 | glendive,us
    Data found (status code: 200) | glendive,us
    Data loaded successfully | glendive,us
    *********************************
    Processing Record 7 of Set 25 | grand bank,ca
    Data found (status code: 200) | grand bank,ca
    Data loaded successfully | grand bank,ca
    *********************************
    Processing Record 8 of Set 25 | ulaanbaatar,mn
    Data found (status code: 200) | ulaanbaatar,mn
    Data loaded successfully | ulaanbaatar,mn
    *********************************
    Processing Record 9 of Set 25 | quetigny,fr
    Data found (status code: 200) | quetigny,fr
    Data loaded successfully | quetigny,fr
    *********************************
    Processing Record 10 of Set 25 | grandview,us
    Data found (status code: 200) | grandview,us
    Data loaded successfully | grandview,us
    *********************************
    Processing Record 11 of Set 25 | wabana,ca
    Data found (status code: 200) | wabana,ca
    Data loaded successfully | wabana,ca
    *********************************
    Processing Record 12 of Set 25 | zaysan,kz
    Data found (status code: 200) | zaysan,kz
    Data loaded successfully | zaysan,kz
    *********************************
    Processing Record 13 of Set 25 | emba,kz
    ERROR: Data retrieval unsuccessful for emba,kz. Code: 404                       MSG: city not found
    Processing Record 14 of Set 25 | dingle,ie
    Data found (status code: 200) | dingle,ie
    Data loaded successfully | dingle,ie
    *********************************
    Processing Record 15 of Set 25 | qingan,cn
    ERROR: Data retrieval unsuccessful for qingan,cn. Code: 404                       MSG: city not found
    Processing Record 16 of Set 25 | senneterre,ca
    Data found (status code: 200) | senneterre,ca
    Data loaded successfully | senneterre,ca
    *********************************
    Processing Record 17 of Set 25 | szentgotthard,hu
    Data found (status code: 200) | szentgotthard,hu
    Data loaded successfully | szentgotthard,hu
    *********************************
    Processing Record 18 of Set 25 | nancha,cn
    Data found (status code: 200) | nancha,cn
    Data loaded successfully | nancha,cn
    *********************************
    Processing Record 19 of Set 25 | longfeng,cn
    Data found (status code: 200) | longfeng,cn
    Data loaded successfully | longfeng,cn
    *********************************
    Processing Record 20 of Set 25 | montmagny,ca
    Data found (status code: 200) | montmagny,ca
    Data loaded successfully | montmagny,ca
    *********************************
    Processing Record 21 of Set 25 | vzmorye,ru
    ERROR: Data retrieval unsuccessful for vzmorye,ru. Code: 404                       MSG: city not found
    Processing Record 22 of Set 25 | pinkafeld,at
    Data found (status code: 200) | pinkafeld,at
    Data loaded successfully | pinkafeld,at
    *********************************
    Processing Record 23 of Set 25 | wulanhaote,cn
    ERROR: Data retrieval unsuccessful for wulanhaote,cn. Code: 404                       MSG: city not found
    Processing Record 24 of Set 25 | deleni,ro
    Data found (status code: 200) | deleni,ro
    Data loaded successfully | deleni,ro
    *********************************
    Processing Record 25 of Set 25 | moses lake,us
    Data found (status code: 200) | moses lake,us
    Data loaded successfully | moses lake,us
    *********************************
    Processing Record 26 of Set 25 | dolinsk,ru
    Data found (status code: 200) | dolinsk,ru
    Data loaded successfully | dolinsk,ru
    *********************************
    Processing Record 27 of Set 25 | lewistown,us
    Data found (status code: 200) | lewistown,us
    Data loaded successfully | lewistown,us
    *********************************
    Processing Record 28 of Set 25 | terrace bay,ca
    Data found (status code: 200) | terrace bay,ca
    Data loaded successfully | terrace bay,ca
    *********************************
    Processing Record 29 of Set 25 | hare bay,ca
    Data found (status code: 200) | hare bay,ca
    Data loaded successfully | hare bay,ca
    *********************************
    Processing Record 30 of Set 25 | aichach,de
    Data found (status code: 200) | aichach,de
    Data loaded successfully | aichach,de
    *********************************
    Processing Record 31 of Set 25 | shubarshi,kz
    Data found (status code: 200) | shubarshi,kz
    Data loaded successfully | shubarshi,kz
    *********************************
    Processing Record 32 of Set 25 | englehart,ca
    Data found (status code: 200) | englehart,ca
    Data loaded successfully | englehart,ca
    *********************************
    Processing Record 33 of Set 25 | amurzet,ru
    Data found (status code: 200) | amurzet,ru
    Data loaded successfully | amurzet,ru
    *********************************
    Processing Record 34 of Set 25 | atasu,kz
    Data found (status code: 200) | atasu,kz
    Data loaded successfully | atasu,kz
    *********************************
    Processing Record 35 of Set 25 | skibbereen,ie
    Data found (status code: 200) | skibbereen,ie
    Data loaded successfully | skibbereen,ie
    *********************************
    Processing Record 36 of Set 25 | grand forks,us
    Data found (status code: 200) | grand forks,us
    Data loaded successfully | grand forks,us
    *********************************
    Processing Record 37 of Set 25 | nogent-le-rotrou,fr
    Data found (status code: 200) | nogent-le-rotrou,fr
    Data loaded successfully | nogent-le-rotrou,fr
    *********************************
    Processing Record 38 of Set 25 | oxbow,ca
    Data found (status code: 200) | oxbow,ca
    Data loaded successfully | oxbow,ca
    *********************************
    Processing Record 39 of Set 25 | manzhouli,cn
    Data found (status code: 200) | manzhouli,cn
    Data loaded successfully | manzhouli,cn
    *********************************
    Processing Record 40 of Set 25 | ayagoz,kz
    Data found (status code: 200) | ayagoz,kz
    Data loaded successfully | ayagoz,kz
    *********************************
    Processing Record 41 of Set 25 | roberval,ca
    Data found (status code: 200) | roberval,ca
    Data loaded successfully | roberval,ca
    *********************************
    Processing Record 42 of Set 25 | stephenville,ca
    Data found (status code: 200) | stephenville,ca
    Data loaded successfully | stephenville,ca
    *********************************
    Processing Record 43 of Set 25 | chapleau,ca
    Data found (status code: 200) | chapleau,ca
    Data loaded successfully | chapleau,ca
    *********************************
    Processing Record 44 of Set 25 | thunder bay,ca
    Data found (status code: 200) | thunder bay,ca
    Data loaded successfully | thunder bay,ca
    *********************************
    Processing Record 45 of Set 25 | victoria,ca
    Data found (status code: 200) | victoria,ca
    Data loaded successfully | victoria,ca
    *********************************
    Processing Record 46 of Set 25 | zhezkazgan,kz
    Data found (status code: 200) | zhezkazgan,kz
    Data loaded successfully | zhezkazgan,kz
    *********************************
    Processing Record 47 of Set 25 | bereslavka,ru
    Data found (status code: 200) | bereslavka,ru
    Data loaded successfully | bereslavka,ru
    *********************************
    Processing Record 48 of Set 25 | landerneau,fr
    Data found (status code: 200) | landerneau,fr
    Data loaded successfully | landerneau,fr
    *********************************
    Processing Record 49 of Set 25 | freiburg,de
    Data found (status code: 200) | freiburg,de
    Data loaded successfully | freiburg,de
    *********************************
    Processing Record 50 of Set 25 | penzance,gb
    Data found (status code: 200) | penzance,gb
    Data loaded successfully | penzance,gb
    *********************************
    Processing Record 51 of Set 25 | cap-aux-meules,ca
    Data found (status code: 200) | cap-aux-meules,ca
    Data loaded successfully | cap-aux-meules,ca
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 26 | nehe,cn
    Data found (status code: 200) | nehe,cn
    Data loaded successfully | nehe,cn
    *********************************
    Processing Record 1 of Set 26 | bemidji,us
    Data found (status code: 200) | bemidji,us
    Data loaded successfully | bemidji,us
    *********************************
    Processing Record 2 of Set 26 | gurskoye,ru
    ERROR: Data retrieval unsuccessful for gurskoye,ru. Code: 404                       MSG: city not found
    Processing Record 3 of Set 26 | monastyryshche,ua
    Data found (status code: 200) | monastyryshche,ua
    Data loaded successfully | monastyryshche,ua
    *********************************
    Processing Record 4 of Set 26 | cap-chat,ca
    Data found (status code: 200) | cap-chat,ca
    Data loaded successfully | cap-chat,ca
    *********************************
    Processing Record 5 of Set 26 | senica,sk
    Data found (status code: 200) | senica,sk
    Data loaded successfully | senica,sk
    *********************************
    Processing Record 6 of Set 26 | octeville,fr
    Data found (status code: 200) | octeville,fr
    Data loaded successfully | octeville,fr
    *********************************
    Processing Record 7 of Set 26 | sedan,fr
    Data found (status code: 200) | sedan,fr
    Data loaded successfully | sedan,fr
    *********************************
    Processing Record 8 of Set 26 | kenora,ca
    Data found (status code: 200) | kenora,ca
    Data loaded successfully | kenora,ca
    *********************************
    Processing Record 9 of Set 26 | sainte-savine,fr
    Data found (status code: 200) | sainte-savine,fr
    Data loaded successfully | sainte-savine,fr
    *********************************
    Processing Record 10 of Set 26 | rimavska sobota,sk
    Data found (status code: 200) | rimavska sobota,sk
    Data loaded successfully | rimavska sobota,sk
    *********************************
    Processing Record 11 of Set 26 | stephenville crossing,ca
    Data found (status code: 200) | stephenville crossing,ca
    Data loaded successfully | stephenville crossing,ca
    *********************************
    Processing Record 12 of Set 26 | virden,ca
    Data found (status code: 200) | virden,ca
    Data loaded successfully | virden,ca
    *********************************
    Processing Record 13 of Set 26 | derzhavinsk,kz
    Data found (status code: 200) | derzhavinsk,kz
    Data loaded successfully | derzhavinsk,kz
    *********************************
    Processing Record 14 of Set 26 | petropavlovsk-kamchatskiy,ru
    Data found (status code: 200) | petropavlovsk-kamchatskiy,ru
    Data loaded successfully | petropavlovsk-kamchatskiy,ru
    *********************************
    Processing Record 15 of Set 26 | sol-iletsk,ru
    Data found (status code: 200) | sol-iletsk,ru
    Data loaded successfully | sol-iletsk,ru
    *********************************
    Processing Record 16 of Set 26 | suhbaatar,mn
    Data found (status code: 200) | suhbaatar,mn
    Data loaded successfully | suhbaatar,mn
    *********************************
    Processing Record 17 of Set 26 | oskemen,kz
    Data found (status code: 200) | oskemen,kz
    Data loaded successfully | oskemen,kz
    *********************************
    Processing Record 18 of Set 26 | ketchikan,us
    Data found (status code: 200) | ketchikan,us
    Data loaded successfully | ketchikan,us
    *********************************
    Processing Record 19 of Set 26 | kazanskaya,ru
    Data found (status code: 200) | kazanskaya,ru
    Data loaded successfully | kazanskaya,ru
    *********************************
    Processing Record 20 of Set 26 | ulaangom,mn
    Data found (status code: 200) | ulaangom,mn
    Data loaded successfully | ulaangom,mn
    *********************************
    Processing Record 21 of Set 26 | nenjiang,cn
    Data found (status code: 200) | nenjiang,cn
    Data loaded successfully | nenjiang,cn
    *********************************
    Processing Record 22 of Set 26 | zakamensk,ru
    Data found (status code: 200) | zakamensk,ru
    Data loaded successfully | zakamensk,ru
    *********************************
    Processing Record 23 of Set 26 | tlucna,cz
    Data found (status code: 200) | tlucna,cz
    Data loaded successfully | tlucna,cz
    *********************************
    Processing Record 24 of Set 26 | saint-augustin,ca
    Data found (status code: 200) | saint-augustin,ca
    Data loaded successfully | saint-augustin,ca
    *********************************
    Processing Record 25 of Set 26 | bonavista,ca
    Data found (status code: 200) | bonavista,ca
    Data loaded successfully | bonavista,ca
    *********************************
    Processing Record 26 of Set 26 | dombarovskiy,ru
    Data found (status code: 200) | dombarovskiy,ru
    Data loaded successfully | dombarovskiy,ru
    *********************************
    Processing Record 27 of Set 26 | kholtoson,ru
    Data found (status code: 200) | kholtoson,ru
    Data loaded successfully | kholtoson,ru
    *********************************
    Processing Record 28 of Set 26 | matagami,ca
    Data found (status code: 200) | matagami,ca
    Data loaded successfully | matagami,ca
    *********************************
    Processing Record 29 of Set 26 | sioux lookout,ca
    Data found (status code: 200) | sioux lookout,ca
    Data loaded successfully | sioux lookout,ca
    *********************************
    Processing Record 30 of Set 26 | hauterive,ca
    Data found (status code: 200) | hauterive,ca
    Data loaded successfully | hauterive,ca
    *********************************
    Processing Record 31 of Set 26 | semey,kz
    Data found (status code: 200) | semey,kz
    Data loaded successfully | semey,kz
    *********************************
    Processing Record 32 of Set 26 | ust-koksa,ru
    Data found (status code: 200) | ust-koksa,ru
    Data loaded successfully | ust-koksa,ru
    *********************************
    Processing Record 33 of Set 26 | dovbysh,ua
    Data found (status code: 200) | dovbysh,ua
    Data loaded successfully | dovbysh,ua
    *********************************
    Processing Record 34 of Set 26 | nizhniy tsasuchey,ru
    Data found (status code: 200) | nizhniy tsasuchey,ru
    Data loaded successfully | nizhniy tsasuchey,ru
    *********************************
    Processing Record 35 of Set 26 | kluczbork,pl
    Data found (status code: 200) | kluczbork,pl
    Data loaded successfully | kluczbork,pl
    *********************************
    Processing Record 36 of Set 26 | sherlovaya gora,ru
    Data found (status code: 200) | sherlovaya gora,ru
    Data loaded successfully | sherlovaya gora,ru
    *********************************
    Processing Record 37 of Set 26 | moosomin,ca
    Data found (status code: 200) | moosomin,ca
    Data loaded successfully | moosomin,ca
    *********************************
    Processing Record 38 of Set 26 | huty,ua
    Data found (status code: 200) | huty,ua
    Data loaded successfully | huty,ua
    *********************************
    Processing Record 39 of Set 26 | black diamond,ca
    Data found (status code: 200) | black diamond,ca
    Data loaded successfully | black diamond,ca
    *********************************
    Processing Record 40 of Set 26 | ozernovskiy,ru
    Data found (status code: 200) | ozernovskiy,ru
    Data loaded successfully | ozernovskiy,ru
    *********************************
    Processing Record 41 of Set 26 | nanortalik,gl
    Data found (status code: 200) | nanortalik,gl
    Data loaded successfully | nanortalik,gl
    *********************************
    Processing Record 42 of Set 26 | zachagansk,kz
    ERROR: Data retrieval unsuccessful for zachagansk,kz. Code: 404                       MSG: city not found
    Processing Record 43 of Set 26 | kulynychi,ua
    Data found (status code: 200) | kulynychi,ua
    Data loaded successfully | kulynychi,ua
    *********************************
    Processing Record 44 of Set 26 | hrebinka,ua
    Data found (status code: 200) | hrebinka,ua
    Data loaded successfully | hrebinka,ua
    *********************************
    Processing Record 45 of Set 26 | nakusp,ca
    Data found (status code: 200) | nakusp,ca
    Data loaded successfully | nakusp,ca
    *********************************
    Processing Record 46 of Set 26 | alihe,cn
    Data found (status code: 200) | alihe,cn
    Data loaded successfully | alihe,cn
    *********************************
    Processing Record 47 of Set 26 | dolbeau,ca
    ERROR: Data retrieval unsuccessful for dolbeau,ca. Code: 404                       MSG: city not found
    Processing Record 48 of Set 26 | sandomierz,pl
    Data found (status code: 200) | sandomierz,pl
    Data loaded successfully | sandomierz,pl
    *********************************
    Processing Record 49 of Set 26 | uglovskoye,ru
    Data found (status code: 200) | uglovskoye,ru
    Data loaded successfully | uglovskoye,ru
    *********************************
    Processing Record 50 of Set 26 | rivers,ca
    Data found (status code: 200) | rivers,ca
    Data loaded successfully | rivers,ca
    *********************************
    Processing Record 51 of Set 26 | novopavlovka,ru
    Data found (status code: 200) | novopavlovka,ru
    Data loaded successfully | novopavlovka,ru
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 27 | sofiysk,ru
    ERROR: Data retrieval unsuccessful for sofiysk,ru. Code: 404                       MSG: city not found
    Processing Record 1 of Set 27 | bayangol,ru
    Data found (status code: 200) | bayangol,ru
    Data loaded successfully | bayangol,ru
    *********************************
    Processing Record 2 of Set 27 | pemberton,ca
    Data found (status code: 200) | pemberton,ca
    Data loaded successfully | pemberton,ca
    *********************************
    Processing Record 3 of Set 27 | shebalino,ru
    Data found (status code: 200) | shebalino,ru
    Data loaded successfully | shebalino,ru
    *********************************
    Processing Record 4 of Set 27 | dauphin,ca
    Data found (status code: 200) | dauphin,ca
    Data loaded successfully | dauphin,ca
    *********************************
    Processing Record 5 of Set 27 | sinzig,de
    Data found (status code: 200) | sinzig,de
    Data loaded successfully | sinzig,de
    *********************************
    Processing Record 6 of Set 27 | havre-saint-pierre,ca
    Data found (status code: 200) | havre-saint-pierre,ca
    Data loaded successfully | havre-saint-pierre,ca
    *********************************
    Processing Record 7 of Set 27 | decin,cz
    Data found (status code: 200) | decin,cz
    Data loaded successfully | decin,cz
    *********************************
    Processing Record 8 of Set 27 | wanze,be
    Data found (status code: 200) | wanze,be
    Data loaded successfully | wanze,be
    *********************************
    Processing Record 9 of Set 27 | teeli,ru
    Data found (status code: 200) | teeli,ru
    Data loaded successfully | teeli,ru
    *********************************
    Processing Record 10 of Set 27 | mugur-aksy,ru
    Data found (status code: 200) | mugur-aksy,ru
    Data loaded successfully | mugur-aksy,ru
    *********************************
    Processing Record 11 of Set 27 | irricana,ca
    Data found (status code: 200) | irricana,ca
    Data loaded successfully | irricana,ca
    *********************************
    Processing Record 12 of Set 27 | aksay,kz
    Data found (status code: 200) | aksay,kz
    Data loaded successfully | aksay,kz
    *********************************
    Processing Record 13 of Set 27 | berezovyy,ru
    Data found (status code: 200) | berezovyy,ru
    Data loaded successfully | berezovyy,ru
    *********************************
    Processing Record 14 of Set 27 | gimli,ca
    Data found (status code: 200) | gimli,ca
    Data loaded successfully | gimli,ca
    *********************************
    Processing Record 15 of Set 27 | engels,ru
    Data found (status code: 200) | engels,ru
    Data loaded successfully | engels,ru
    *********************************
    Processing Record 16 of Set 27 | lisakovsk,kz
    Data found (status code: 200) | lisakovsk,kz
    Data loaded successfully | lisakovsk,kz
    *********************************
    Processing Record 17 of Set 27 | altayskoye,ru
    Data found (status code: 200) | altayskoye,ru
    Data loaded successfully | altayskoye,ru
    *********************************
    Processing Record 18 of Set 27 | tymovskoye,ru
    Data found (status code: 200) | tymovskoye,ru
    Data loaded successfully | tymovskoye,ru
    *********************************
    Processing Record 19 of Set 27 | barry,gb
    Data found (status code: 200) | barry,gb
    Data loaded successfully | barry,gb
    *********************************
    Processing Record 20 of Set 27 | geraldton,ca
    Data found (status code: 200) | geraldton,ca
    Data loaded successfully | geraldton,ca
    *********************************
    Processing Record 21 of Set 27 | makinsk,kz
    Data found (status code: 200) | makinsk,kz
    Data loaded successfully | makinsk,kz
    *********************************
    Processing Record 22 of Set 27 | skorodnoye,ru
    Data found (status code: 200) | skorodnoye,ru
    Data loaded successfully | skorodnoye,ru
    *********************************
    Processing Record 23 of Set 27 | kushmurun,kz
    ERROR: Data retrieval unsuccessful for kushmurun,kz. Code: 404                       MSG: city not found
    Processing Record 24 of Set 27 | verkhnyaya tishanka,ru
    Data found (status code: 200) | verkhnyaya tishanka,ru
    Data loaded successfully | verkhnyaya tishanka,ru
    *********************************
    Processing Record 25 of Set 27 | sept-iles,ca
    Data found (status code: 200) | sept-iles,ca
    Data loaded successfully | sept-iles,ca
    *********************************
    Processing Record 26 of Set 27 | shostka,ua
    Data found (status code: 200) | shostka,ua
    Data loaded successfully | shostka,ua
    *********************************
    Processing Record 27 of Set 27 | khlevnoye,ru
    Data found (status code: 200) | khlevnoye,ru
    Data loaded successfully | khlevnoye,ru
    *********************************
    Processing Record 28 of Set 27 | one hundred mile house,ca
    ERROR: Data retrieval unsuccessful for one hundred mile house,ca. Code: 404                       MSG: city not found
    Processing Record 29 of Set 27 | ipswich,gb
    Data found (status code: 200) | ipswich,gb
    Data loaded successfully | ipswich,gb
    *********************************
    Processing Record 30 of Set 27 | attawapiskat,ca
    ERROR: Data retrieval unsuccessful for attawapiskat,ca. Code: 404                       MSG: city not found
    Processing Record 31 of Set 27 | pavlodar,kz
    Data found (status code: 200) | pavlodar,kz
    Data loaded successfully | pavlodar,kz
    *********************************
    Processing Record 32 of Set 27 | svobodnyy,ru
    Data found (status code: 200) | svobodnyy,ru
    Data loaded successfully | svobodnyy,ru
    *********************************
    Processing Record 33 of Set 27 | dubrovytsya,ua
    Data found (status code: 200) | dubrovytsya,ua
    Data loaded successfully | dubrovytsya,ua
    *********************************
    Processing Record 34 of Set 27 | campbell river,ca
    Data found (status code: 200) | campbell river,ca
    Data loaded successfully | campbell river,ca
    *********************************
    Processing Record 35 of Set 27 | hanna,ca
    Data found (status code: 200) | hanna,ca
    Data loaded successfully | hanna,ca
    *********************************
    Processing Record 36 of Set 27 | novosergiyevka,ru
    Data found (status code: 200) | novosergiyevka,ru
    Data loaded successfully | novosergiyevka,ru
    *********************************
    Processing Record 37 of Set 27 | saint anthony,ca
    ERROR: Data retrieval unsuccessful for saint anthony,ca. Code: 404                       MSG: city not found
    Processing Record 38 of Set 27 | tasiilaq,gl
    Data found (status code: 200) | tasiilaq,gl
    Data loaded successfully | tasiilaq,gl
    *********************************
    Processing Record 39 of Set 27 | fevralsk,ru
    ERROR: Data retrieval unsuccessful for fevralsk,ru. Code: 404                       MSG: city not found
    Processing Record 40 of Set 27 | rocky mountain house,ca
    Data found (status code: 200) | rocky mountain house,ca
    Data loaded successfully | rocky mountain house,ca
    *********************************
    Processing Record 41 of Set 27 | kachiry,kz
    Data found (status code: 200) | kachiry,kz
    Data loaded successfully | kachiry,kz
    *********************************
    Processing Record 42 of Set 27 | plonsk,pl
    Data found (status code: 200) | plonsk,pl
    Data loaded successfully | plonsk,pl
    *********************************
    Processing Record 43 of Set 27 | tahe,cn
    Data found (status code: 200) | tahe,cn
    Data loaded successfully | tahe,cn
    *********************************
    Processing Record 44 of Set 27 | athlone,ie
    Data found (status code: 200) | athlone,ie
    Data loaded successfully | athlone,ie
    *********************************
    Processing Record 45 of Set 27 | sierpc,pl
    Data found (status code: 200) | sierpc,pl
    Data loaded successfully | sierpc,pl
    *********************************
    Processing Record 46 of Set 27 | stepnyak,kz
    Data found (status code: 200) | stepnyak,kz
    Data loaded successfully | stepnyak,kz
    *********************************
    Processing Record 47 of Set 27 | askiz,ru
    Data found (status code: 200) | askiz,ru
    Data loaded successfully | askiz,ru
    *********************************
    Processing Record 48 of Set 27 | kaa-khem,ru
    Data found (status code: 200) | kaa-khem,ru
    Data loaded successfully | kaa-khem,ru
    *********************************
    Processing Record 49 of Set 27 | wittingen,de
    Data found (status code: 200) | wittingen,de
    Data loaded successfully | wittingen,de
    *********************************
    Processing Record 50 of Set 27 | burns lake,ca
    Data found (status code: 200) | burns lake,ca
    Data loaded successfully | burns lake,ca
    *********************************
    Processing Record 51 of Set 27 | heerenveen,nl
    Data found (status code: 200) | heerenveen,nl
    Data loaded successfully | heerenveen,nl
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 28 | wainwright,ca
    Data found (status code: 200) | wainwright,ca
    Data loaded successfully | wainwright,ca
    *********************************
    Processing Record 1 of Set 28 | chapaevsk,ru
    Data found (status code: 200) | chapaevsk,ru
    Data loaded successfully | chapaevsk,ru
    *********************************
    Processing Record 2 of Set 28 | orlik,ru
    Data found (status code: 200) | orlik,ru
    Data loaded successfully | orlik,ru
    *********************************
    Processing Record 3 of Set 28 | rypin,pl
    Data found (status code: 200) | rypin,pl
    Data loaded successfully | rypin,pl
    *********************************
    Processing Record 4 of Set 28 | imeni poliny osipenko,ru
    Data found (status code: 200) | imeni poliny osipenko,ru
    Data loaded successfully | imeni poliny osipenko,ru
    *********************************
    Processing Record 5 of Set 28 | barrhead,ca
    Data found (status code: 200) | barrhead,ca
    Data loaded successfully | barrhead,ca
    *********************************
    Processing Record 6 of Set 28 | sobolevo,ru
    Data found (status code: 200) | sobolevo,ru
    Data loaded successfully | sobolevo,ru
    *********************************
    Processing Record 7 of Set 28 | askarovo,ru
    Data found (status code: 200) | askarovo,ru
    Data loaded successfully | askarovo,ru
    *********************************
    Processing Record 8 of Set 28 | abdulino,ru
    Data found (status code: 200) | abdulino,ru
    Data loaded successfully | abdulino,ru
    *********************************
    Processing Record 9 of Set 28 | beringovskiy,ru
    Data found (status code: 200) | beringovskiy,ru
    Data loaded successfully | beringovskiy,ru
    *********************************
    Processing Record 10 of Set 28 | grajewo,pl
    Data found (status code: 200) | grajewo,pl
    Data loaded successfully | grajewo,pl
    *********************************
    Processing Record 11 of Set 28 | gomel,by
    ERROR: Data retrieval unsuccessful for gomel,by. Code: 404                       MSG: city not found
    Processing Record 12 of Set 28 | nikolayevsk-na-amure,ru
    Data found (status code: 200) | nikolayevsk-na-amure,ru
    Data loaded successfully | nikolayevsk-na-amure,ru
    *********************************
    Processing Record 13 of Set 28 | mezhdurechensk,ru
    Data found (status code: 200) | mezhdurechensk,ru
    Data loaded successfully | mezhdurechensk,ru
    *********************************
    Processing Record 14 of Set 28 | mago,ru
    Data found (status code: 200) | mago,ru
    Data loaded successfully | mago,ru
    *********************************
    Processing Record 15 of Set 28 | schwedt,de
    ERROR: Data retrieval unsuccessful for schwedt,de. Code: 404                       MSG: city not found
    Processing Record 16 of Set 28 | mnogovershinnyy,ru
    Data found (status code: 200) | mnogovershinnyy,ru
    Data loaded successfully | mnogovershinnyy,ru
    *********************************
    Processing Record 17 of Set 28 | vestmannaeyjar,is
    Data found (status code: 200) | vestmannaeyjar,is
    Data loaded successfully | vestmannaeyjar,is
    *********************************
    Processing Record 18 of Set 28 | prince george,ca
    Data found (status code: 200) | prince george,ca
    Data loaded successfully | prince george,ca
    *********************************
    Processing Record 19 of Set 28 | zlobin,by
    ERROR: Data retrieval unsuccessful for zlobin,by. Code: 404                       MSG: city not found
    Processing Record 20 of Set 28 | okha,ru
    Data found (status code: 200) | okha,ru
    Data loaded successfully | okha,ru
    *********************************
    Processing Record 21 of Set 28 | boyle,ie
    Data found (status code: 200) | boyle,ie
    Data loaded successfully | boyle,ie
    *********************************
    Processing Record 22 of Set 28 | shellbrook,ca
    Data found (status code: 200) | shellbrook,ca
    Data loaded successfully | shellbrook,ca
    *********************************
    Processing Record 23 of Set 28 | great yarmouth,gb
    Data found (status code: 200) | great yarmouth,gb
    Data loaded successfully | great yarmouth,gb
    *********************************
    Processing Record 24 of Set 28 | grindavik,is
    Data found (status code: 200) | grindavik,is
    Data loaded successfully | grindavik,is
    *********************************
    Processing Record 25 of Set 28 | ust-bolsheretsk,ru
    ERROR: Data retrieval unsuccessful for ust-bolsheretsk,ru. Code: 404                       MSG: city not found
    Processing Record 26 of Set 28 | toora-khem,ru
    Data found (status code: 200) | toora-khem,ru
    Data loaded successfully | toora-khem,ru
    *********************************
    Processing Record 27 of Set 28 | qostanay,kz
    Data found (status code: 200) | qostanay,kz
    Data loaded successfully | qostanay,kz
    *********************************
    Processing Record 28 of Set 28 | yelkhovka,ru
    Data found (status code: 200) | yelkhovka,ru
    Data loaded successfully | yelkhovka,ru
    *********************************
    Processing Record 29 of Set 28 | bon accord,ca
    Data found (status code: 200) | bon accord,ca
    Data loaded successfully | bon accord,ca
    *********************************
    Processing Record 30 of Set 28 | hudson bay,ca
    Data found (status code: 200) | hudson bay,ca
    Data loaded successfully | hudson bay,ca
    *********************************
    Processing Record 31 of Set 28 | koshki,ru
    Data found (status code: 200) | koshki,ru
    Data loaded successfully | koshki,ru
    *********************************
    Processing Record 32 of Set 28 | newry,gb
    Data found (status code: 200) | newry,gb
    Data loaded successfully | newry,gb
    *********************************
    Processing Record 33 of Set 28 | hinton,ca
    Data found (status code: 200) | hinton,ca
    Data loaded successfully | hinton,ca
    *********************************
    Processing Record 34 of Set 28 | thompson,ca
    Data found (status code: 200) | thompson,ca
    Data loaded successfully | thompson,ca
    *********************************
    Processing Record 35 of Set 28 | ferzikovo,ru
    Data found (status code: 200) | ferzikovo,ru
    Data loaded successfully | ferzikovo,ru
    *********************************
    Processing Record 36 of Set 28 | milkovo,ru
    ERROR: Data retrieval unsuccessful for milkovo,ru. Code: 404                       MSG: city not found
    Processing Record 37 of Set 28 | novoulyanovsk,ru
    Data found (status code: 200) | novoulyanovsk,ru
    Data loaded successfully | novoulyanovsk,ru
    *********************************
    Processing Record 38 of Set 28 | leninogorsk,ru
    Data found (status code: 200) | leninogorsk,ru
    Data loaded successfully | leninogorsk,ru
    *********************************
    Processing Record 39 of Set 28 | scarborough,gb
    Data found (status code: 200) | scarborough,gb
    Data loaded successfully | scarborough,gb
    *********************************
    Processing Record 40 of Set 28 | kuytun,ru
    Data found (status code: 200) | kuytun,ru
    Data loaded successfully | kuytun,ru
    *********************************
    Processing Record 41 of Set 28 | irbeyskoye,ru
    Data found (status code: 200) | irbeyskoye,ru
    Data loaded successfully | irbeyskoye,ru
    *********************************
    Processing Record 42 of Set 28 | slave lake,ca
    Data found (status code: 200) | slave lake,ca
    Data loaded successfully | slave lake,ca
    *********************************
    Processing Record 43 of Set 28 | chekhov,ru
    Data found (status code: 200) | chekhov,ru
    Data loaded successfully | chekhov,ru
    *********************************
    Processing Record 44 of Set 28 | wembley,ca
    Data found (status code: 200) | wembley,ca
    Data loaded successfully | wembley,ca
    *********************************
    Processing Record 45 of Set 28 | vilkija,lt
    Data found (status code: 200) | vilkija,lt
    Data loaded successfully | vilkija,lt
    *********************************
    Processing Record 46 of Set 28 | houston,ca
    Data found (status code: 200) | houston,ca
    Data loaded successfully | houston,ca
    *********************************
    Processing Record 47 of Set 28 | ballina,ie
    Data found (status code: 200) | ballina,ie
    Data loaded successfully | ballina,ie
    *********************************
    Processing Record 48 of Set 28 | undory,ru
    Data found (status code: 200) | undory,ru
    Data loaded successfully | undory,ru
    *********************************
    Processing Record 49 of Set 28 | moose factory,ca
    Data found (status code: 200) | moose factory,ca
    Data loaded successfully | moose factory,ca
    *********************************
    Processing Record 50 of Set 28 | dawson creek,ca
    Data found (status code: 200) | dawson creek,ca
    Data loaded successfully | dawson creek,ca
    *********************************
    Processing Record 51 of Set 28 | urusha,ru
    Data found (status code: 200) | urusha,ru
    Data loaded successfully | urusha,ru
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 29 | batyrevo,ru
    Data found (status code: 200) | batyrevo,ru
    Data loaded successfully | batyrevo,ru
    *********************************
    Processing Record 1 of Set 29 | pitelino,ru
    Data found (status code: 200) | pitelino,ru
    Data loaded successfully | pitelino,ru
    *********************************
    Processing Record 2 of Set 29 | ystad,se
    Data found (status code: 200) | ystad,se
    Data loaded successfully | ystad,se
    *********************************
    Processing Record 3 of Set 29 | bridlington,gb
    Data found (status code: 200) | bridlington,gb
    Data loaded successfully | bridlington,gb
    *********************************
    Processing Record 4 of Set 29 | mackenzie,ca
    Data found (status code: 200) | mackenzie,ca
    Data loaded successfully | mackenzie,ca
    *********************************
    Processing Record 5 of Set 29 | zeya,ru
    Data found (status code: 200) | zeya,ru
    Data loaded successfully | zeya,ru
    *********************************
    Processing Record 6 of Set 29 | coleraine,gb
    Data found (status code: 200) | coleraine,gb
    Data loaded successfully | coleraine,gb
    *********************************
    Processing Record 7 of Set 29 | den helder,nl
    Data found (status code: 200) | den helder,nl
    Data loaded successfully | den helder,nl
    *********************************
    Processing Record 8 of Set 29 | olafsvik,is
    ERROR: Data retrieval unsuccessful for olafsvik,is. Code: 404                       MSG: city not found
    Processing Record 9 of Set 29 | kazachinskoye,ru
    Data found (status code: 200) | kazachinskoye,ru
    Data loaded successfully | kazachinskoye,ru
    *********************************
    Processing Record 10 of Set 29 | kichera,ru
    Data found (status code: 200) | kichera,ru
    Data loaded successfully | kichera,ru
    *********************************
    Processing Record 11 of Set 29 | high prairie,ca
    Data found (status code: 200) | high prairie,ca
    Data loaded successfully | high prairie,ca
    *********************************
    Processing Record 12 of Set 29 | wladyslawowo,pl
    Data found (status code: 200) | wladyslawowo,pl
    Data loaded successfully | wladyslawowo,pl
    *********************************
    Processing Record 13 of Set 29 | yazykovo,ru
    Data found (status code: 200) | yazykovo,ru
    Data loaded successfully | yazykovo,ru
    *********************************
    Processing Record 14 of Set 29 | chapais,ca
    Data found (status code: 200) | chapais,ca
    Data loaded successfully | chapais,ca
    *********************************
    Processing Record 15 of Set 29 | flin flon,ca
    Data found (status code: 200) | flin flon,ca
    Data loaded successfully | flin flon,ca
    *********************************
    Processing Record 16 of Set 29 | esso,ru
    Data found (status code: 200) | esso,ru
    Data loaded successfully | esso,ru
    *********************************
    Processing Record 17 of Set 29 | yantal,ru
    Data found (status code: 200) | yantal,ru
    Data loaded successfully | yantal,ru
    *********************************
    Processing Record 18 of Set 29 | qaqortoq,gl
    Data found (status code: 200) | qaqortoq,gl
    Data loaded successfully | qaqortoq,gl
    *********************************
    Processing Record 19 of Set 29 | abatskoye,ru
    Data found (status code: 200) | abatskoye,ru
    Data loaded successfully | abatskoye,ru
    *********************************
    Processing Record 20 of Set 29 | ust-kamchatsk,ru
    ERROR: Data retrieval unsuccessful for ust-kamchatsk,ru. Code: 404                       MSG: city not found
    Processing Record 21 of Set 29 | magistralnyy,ru
    Data found (status code: 200) | magistralnyy,ru
    Data loaded successfully | magistralnyy,ru
    *********************************
    Processing Record 22 of Set 29 | westport,ie
    Data found (status code: 200) | westport,ie
    Data loaded successfully | westport,ie
    *********************************
    Processing Record 23 of Set 29 | itatskiy,ru
    Data found (status code: 200) | itatskiy,ru
    Data loaded successfully | itatskiy,ru
    *********************************
    Processing Record 24 of Set 29 | kunashak,ru
    Data found (status code: 200) | kunashak,ru
    Data loaded successfully | kunashak,ru
    *********************************
    Processing Record 25 of Set 29 | chagda,ru
    ERROR: Data retrieval unsuccessful for chagda,ru. Code: 404                       MSG: city not found
    Processing Record 26 of Set 29 | svinninge,dk
    Data found (status code: 200) | svinninge,dk
    Data loaded successfully | svinninge,dk
    *********************************
    Processing Record 27 of Set 29 | palmer,us
    Data found (status code: 200) | palmer,us
    Data loaded successfully | palmer,us
    *********************************
    Processing Record 28 of Set 29 | chaykovskiy,ru
    Data found (status code: 200) | chaykovskiy,ru
    Data loaded successfully | chaykovskiy,ru
    *********************************
    Processing Record 29 of Set 29 | torzhok,ru
    Data found (status code: 200) | torzhok,ru
    Data loaded successfully | torzhok,ru
    *********************************
    Processing Record 30 of Set 29 | kolosovka,ru
    Data found (status code: 200) | kolosovka,ru
    Data loaded successfully | kolosovka,ru
    *********************************
    Processing Record 31 of Set 29 | neryungri,ru
    Data found (status code: 200) | neryungri,ru
    Data loaded successfully | neryungri,ru
    *********************************
    Processing Record 32 of Set 29 | yurino,ru
    Data found (status code: 200) | yurino,ru
    Data loaded successfully | yurino,ru
    *********************************
    Processing Record 33 of Set 29 | jaunjelgava,lv
    Data found (status code: 200) | jaunjelgava,lv
    Data loaded successfully | jaunjelgava,lv
    *********************************
    Processing Record 34 of Set 29 | harboore,dk
    Data found (status code: 200) | harboore,dk
    Data loaded successfully | harboore,dk
    *********************************
    Processing Record 35 of Set 29 | kiknur,ru
    Data found (status code: 200) | kiknur,ru
    Data loaded successfully | kiknur,ru
    *********************************
    Processing Record 36 of Set 29 | likhoslavl,ru
    Data found (status code: 200) | likhoslavl,ru
    Data loaded successfully | likhoslavl,ru
    *********************************
    Processing Record 37 of Set 29 | arman,ru
    Data found (status code: 200) | arman,ru
    Data loaded successfully | arman,ru
    *********************************
    Processing Record 38 of Set 29 | chernushka,ru
    Data found (status code: 200) | chernushka,ru
    Data loaded successfully | chernushka,ru
    *********************************
    Processing Record 39 of Set 29 | uinskoye,ru
    Data found (status code: 200) | uinskoye,ru
    Data loaded successfully | uinskoye,ru
    *********************************
    Processing Record 40 of Set 29 | ola,ru
    Data found (status code: 200) | ola,ru
    Data loaded successfully | ola,ru
    *********************************
    Processing Record 41 of Set 29 | oban,gb
    Data found (status code: 200) | oban,gb
    Data loaded successfully | oban,gb
    *********************************
    Processing Record 42 of Set 29 | saulkrasti,lv
    Data found (status code: 200) | saulkrasti,lv
    Data loaded successfully | saulkrasti,lv
    *********************************
    Processing Record 43 of Set 29 | paamiut,gl
    Data found (status code: 200) | paamiut,gl
    Data loaded successfully | paamiut,gl
    *********************************
    Processing Record 44 of Set 29 | grand centre,ca
    ERROR: Data retrieval unsuccessful for grand centre,ca. Code: 404                       MSG: city not found
    Processing Record 45 of Set 29 | voskresenskoye,ru
    Data found (status code: 200) | voskresenskoye,ru
    Data loaded successfully | voskresenskoye,ru
    *********************************
    Processing Record 46 of Set 29 | aban,ru
    Data found (status code: 200) | aban,ru
    Data loaded successfully | aban,ru
    *********************************
    Processing Record 47 of Set 29 | sogdiondon,ru
    ERROR: Data retrieval unsuccessful for sogdiondon,ru. Code: 404                       MSG: city not found
    Processing Record 48 of Set 29 | bakchar,ru
    Data found (status code: 200) | bakchar,ru
    Data loaded successfully | bakchar,ru
    *********************************
    Processing Record 49 of Set 29 | ust-ilimsk,ru
    Data found (status code: 200) | ust-ilimsk,ru
    Data loaded successfully | ust-ilimsk,ru
    *********************************
    Processing Record 50 of Set 29 | iqaluit,ca
    Data found (status code: 200) | iqaluit,ca
    Data loaded successfully | iqaluit,ca
    *********************************
    Processing Record 51 of Set 29 | kyshtovka,ru
    Data found (status code: 200) | kyshtovka,ru
    Data loaded successfully | kyshtovka,ru
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 30 | velikooktyabrskiy,ru
    Data found (status code: 200) | velikooktyabrskiy,ru
    Data loaded successfully | velikooktyabrskiy,ru
    *********************************
    Processing Record 1 of Set 30 | sedelnikovo,ru
    ERROR: Data retrieval unsuccessful for sedelnikovo,ru. Code: 404                       MSG: city not found
    Processing Record 2 of Set 30 | tilichiki,ru
    Data found (status code: 200) | tilichiki,ru
    Data loaded successfully | tilichiki,ru
    *********************************
    Processing Record 3 of Set 30 | sudislavl,ru
    Data found (status code: 200) | sudislavl,ru
    Data loaded successfully | sudislavl,ru
    *********************************
    Processing Record 4 of Set 30 | golspie,gb
    Data found (status code: 200) | golspie,gb
    Data loaded successfully | golspie,gb
    *********************************
    Processing Record 5 of Set 30 | portree,gb
    Data found (status code: 200) | portree,gb
    Data loaded successfully | portree,gb
    *********************************
    Processing Record 6 of Set 30 | fort nelson,ca
    Data found (status code: 200) | fort nelson,ca
    Data loaded successfully | fort nelson,ca
    *********************************
    Processing Record 7 of Set 30 | ventspils,lv
    Data found (status code: 200) | ventspils,lv
    Data loaded successfully | ventspils,lv
    *********************************
    Processing Record 8 of Set 30 | khani,ru
    ERROR: Data retrieval unsuccessful for khani,ru. Code: 404                       MSG: city not found
    Processing Record 9 of Set 30 | ayan,ru
    ERROR: Data retrieval unsuccessful for ayan,ru. Code: 404                       MSG: city not found
    Processing Record 10 of Set 30 | privolzhsk,ru
    Data found (status code: 200) | privolzhsk,ru
    Data loaded successfully | privolzhsk,ru
    *********************************
    Processing Record 11 of Set 30 | lebedinyy,ru
    Data found (status code: 200) | lebedinyy,ru
    Data loaded successfully | lebedinyy,ru
    *********************************
    Processing Record 12 of Set 30 | okhotsk,ru
    Data found (status code: 200) | okhotsk,ru
    Data loaded successfully | okhotsk,ru
    *********************************
    Processing Record 13 of Set 30 | visnes,no
    Data found (status code: 200) | visnes,no
    Data loaded successfully | visnes,no
    *********************************
    Processing Record 14 of Set 30 | kargasok,ru
    Data found (status code: 200) | kargasok,ru
    Data loaded successfully | kargasok,ru
    *********************************
    Processing Record 15 of Set 30 | tommot,ru
    Data found (status code: 200) | tommot,ru
    Data loaded successfully | tommot,ru
    *********************************
    Processing Record 16 of Set 30 | peterhead,gb
    Data found (status code: 200) | peterhead,gb
    Data loaded successfully | peterhead,gb
    *********************************
    Processing Record 17 of Set 30 | ullapool,gb
    Data found (status code: 200) | ullapool,gb
    Data loaded successfully | ullapool,gb
    *********************************
    Processing Record 18 of Set 30 | kodinsk,ru
    Data found (status code: 200) | kodinsk,ru
    Data loaded successfully | kodinsk,ru
    *********************************
    Processing Record 19 of Set 30 | varhaug,no
    Data found (status code: 200) | varhaug,no
    Data loaded successfully | varhaug,no
    *********************************
    Processing Record 20 of Set 30 | strezhevoy,ru
    Data found (status code: 200) | strezhevoy,ru
    Data loaded successfully | strezhevoy,ru
    *********************************
    Processing Record 21 of Set 30 | high level,ca
    Data found (status code: 200) | high level,ca
    Data loaded successfully | high level,ca
    *********************************
    Processing Record 22 of Set 30 | la ronge,ca
    Data found (status code: 200) | la ronge,ca
    Data loaded successfully | la ronge,ca
    *********************************
    Processing Record 23 of Set 30 | vesyegonsk,ru
    Data found (status code: 200) | vesyegonsk,ru
    Data loaded successfully | vesyegonsk,ru
    *********************************
    Processing Record 24 of Set 30 | anchorage,us
    Data found (status code: 200) | anchorage,us
    Data loaded successfully | anchorage,us
    *********************************
    Processing Record 25 of Set 30 | stornoway,gb
    Data found (status code: 200) | stornoway,gb
    Data loaded successfully | stornoway,gb
    *********************************
    Processing Record 26 of Set 30 | solnechnyy,ru
    Data found (status code: 200) | solnechnyy,ru
    Data loaded successfully | solnechnyy,ru
    *********************************
    Processing Record 27 of Set 30 | yeniseysk,ru
    Data found (status code: 200) | yeniseysk,ru
    Data loaded successfully | yeniseysk,ru
    *********************************
    Processing Record 28 of Set 30 | hay river,ca
    Data found (status code: 200) | hay river,ca
    Data loaded successfully | hay river,ca
    *********************************
    Processing Record 29 of Set 30 | yellowknife,ca
    Data found (status code: 200) | yellowknife,ca
    Data loaded successfully | yellowknife,ca
    *********************************
    Processing Record 30 of Set 30 | yuzhno-yeniseyskiy,ru
    ERROR: Data retrieval unsuccessful for yuzhno-yeniseyskiy,ru. Code: 404                       MSG: city not found
    Processing Record 31 of Set 30 | karla,ee
    Data found (status code: 200) | karla,ee
    Data loaded successfully | karla,ee
    *********************************
    Processing Record 32 of Set 30 | kondinskoye,ru
    Data found (status code: 200) | kondinskoye,ru
    Data loaded successfully | kondinskoye,ru
    *********************************
    Processing Record 33 of Set 30 | tabory,ru
    Data found (status code: 200) | tabory,ru
    Data loaded successfully | tabory,ru
    *********************************
    Processing Record 34 of Set 30 | haines junction,ca
    Data found (status code: 200) | haines junction,ca
    Data loaded successfully | haines junction,ca
    *********************************
    Processing Record 35 of Set 30 | talaya,ru
    Data found (status code: 200) | talaya,ru
    Data loaded successfully | talaya,ru
    *********************************
    Processing Record 36 of Set 30 | vitim,ru
    Data found (status code: 200) | vitim,ru
    Data loaded successfully | vitim,ru
    *********************************
    Processing Record 37 of Set 30 | novobirilyussy,ru
    Data found (status code: 200) | novobirilyussy,ru
    Data loaded successfully | novobirilyussy,ru
    *********************************
    Processing Record 38 of Set 30 | juneau,us
    Data found (status code: 200) | juneau,us
    Data loaded successfully | juneau,us
    *********************************
    Processing Record 39 of Set 30 | chara,ru
    Data found (status code: 200) | chara,ru
    Data loaded successfully | chara,ru
    *********************************
    Processing Record 40 of Set 30 | belyy yar,ru
    Data found (status code: 200) | belyy yar,ru
    Data loaded successfully | belyy yar,ru
    *********************************
    Processing Record 41 of Set 30 | salym,ru
    Data found (status code: 200) | salym,ru
    Data loaded successfully | salym,ru
    *********************************
    Processing Record 42 of Set 30 | sandwick,gb
    Data found (status code: 200) | sandwick,gb
    Data loaded successfully | sandwick,gb
    *********************************
    Processing Record 43 of Set 30 | oktyabrskiy,ru
    Data found (status code: 200) | oktyabrskiy,ru
    Data loaded successfully | oktyabrskiy,ru
    *********************************
    Processing Record 44 of Set 30 | nuuk,gl
    Data found (status code: 200) | nuuk,gl
    Data loaded successfully | nuuk,gl
    *********************************
    Processing Record 45 of Set 30 | sorvag,fo
    ERROR: Data retrieval unsuccessful for sorvag,fo. Code: 404                       MSG: city not found
    Processing Record 46 of Set 30 | nome,us
    Data found (status code: 200) | nome,us
    Data loaded successfully | nome,us
    *********************************
    Processing Record 47 of Set 30 | homer,us
    Data found (status code: 200) | homer,us
    Data loaded successfully | homer,us
    *********************************
    Processing Record 48 of Set 30 | vanavara,ru
    Data found (status code: 200) | vanavara,ru
    Data loaded successfully | vanavara,ru
    *********************************
    Processing Record 49 of Set 30 | tigil,ru
    Data found (status code: 200) | tigil,ru
    Data loaded successfully | tigil,ru
    *********************************
    Processing Record 50 of Set 30 | hofn,is
    Data found (status code: 200) | hofn,is
    Data loaded successfully | hofn,is
    *********************************
    Processing Record 51 of Set 30 | naantali,fi
    Data found (status code: 200) | naantali,fi
    Data loaded successfully | naantali,fi
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 31 | whitehorse,ca
    Data found (status code: 200) | whitehorse,ca
    Data loaded successfully | whitehorse,ca
    *********************************
    Processing Record 1 of Set 31 | uusikaupunki,fi
    Data found (status code: 200) | uusikaupunki,fi
    Data loaded successfully | uusikaupunki,fi
    *********************************
    Processing Record 2 of Set 31 | lensk,ru
    Data found (status code: 200) | lensk,ru
    Data loaded successfully | lensk,ru
    *********************************
    Processing Record 3 of Set 31 | teya,ru
    Data found (status code: 200) | teya,ru
    Data loaded successfully | teya,ru
    *********************************
    Processing Record 4 of Set 31 | severo-yeniseyskiy,ru
    Data found (status code: 200) | severo-yeniseyskiy,ru
    Data loaded successfully | severo-yeniseyskiy,ru
    *********************************
    Processing Record 5 of Set 31 | teguldet,ru
    Data found (status code: 200) | teguldet,ru
    Data loaded successfully | teguldet,ru
    *********************************
    Processing Record 6 of Set 31 | kroderen,no
    ERROR: Data retrieval unsuccessful for kroderen,no. Code: 404                       MSG: city not found
    Processing Record 7 of Set 31 | baykit,ru
    Data found (status code: 200) | baykit,ru
    Data loaded successfully | baykit,ru
    *********************************
    Processing Record 8 of Set 31 | peleduy,ru
    Data found (status code: 200) | peleduy,ru
    Data loaded successfully | peleduy,ru
    *********************************
    Processing Record 9 of Set 31 | atka,ru
    ERROR: Data retrieval unsuccessful for atka,ru. Code: 404                       MSG: city not found
    Processing Record 10 of Set 31 | loyga,ru
    Data found (status code: 200) | loyga,ru
    Data loaded successfully | loyga,ru
    *********************************
    Processing Record 11 of Set 31 | uray,ru
    Data found (status code: 200) | uray,ru
    Data loaded successfully | uray,ru
    *********************************
    Processing Record 12 of Set 31 | cheuskiny,ru
    ERROR: Data retrieval unsuccessful for cheuskiny,ru. Code: 404                       MSG: city not found
    Processing Record 13 of Set 31 | lavrentiya,ru
    Data found (status code: 200) | lavrentiya,ru
    Data loaded successfully | lavrentiya,ru
    *********************************
    Processing Record 14 of Set 31 | sovetskiy,ru
    Data found (status code: 200) | sovetskiy,ru
    Data loaded successfully | sovetskiy,ru
    *********************************
    Processing Record 15 of Set 31 | berdigestyakh,ru
    Data found (status code: 200) | berdigestyakh,ru
    Data loaded successfully | berdigestyakh,ru
    *********************************
    Processing Record 16 of Set 31 | amga,ru
    Data found (status code: 200) | amga,ru
    Data loaded successfully | amga,ru
    *********************************
    Processing Record 17 of Set 31 | kizema,ru
    Data found (status code: 200) | kizema,ru
    Data loaded successfully | kizema,ru
    *********************************
    Processing Record 18 of Set 31 | eldikan,ru
    ERROR: Data retrieval unsuccessful for eldikan,ru. Code: 404                       MSG: city not found
    Processing Record 19 of Set 31 | pangnirtung,ca
    Data found (status code: 200) | pangnirtung,ca
    Data loaded successfully | pangnirtung,ca
    *********************************
    Processing Record 20 of Set 31 | fairbanks,us
    Data found (status code: 200) | fairbanks,us
    Data loaded successfully | fairbanks,us
    *********************************
    Processing Record 21 of Set 31 | suntar,ru
    Data found (status code: 200) | suntar,ru
    Data loaded successfully | suntar,ru
    *********************************
    Processing Record 22 of Set 31 | anadyr,ru
    Data found (status code: 200) | anadyr,ru
    Data loaded successfully | anadyr,ru
    *********************************
    Processing Record 23 of Set 31 | valer,no
    Data found (status code: 200) | valer,no
    Data loaded successfully | valer,no
    *********************************
    Processing Record 24 of Set 31 | yerbogachen,ru
    Data found (status code: 200) | yerbogachen,ru
    Data loaded successfully | yerbogachen,ru
    *********************************
    Processing Record 25 of Set 31 | kenai,us
    Data found (status code: 200) | kenai,us
    Data loaded successfully | kenai,us
    *********************************
    Processing Record 26 of Set 31 | kuloy,ru
    Data found (status code: 200) | kuloy,ru
    Data loaded successfully | kuloy,ru
    *********************************
    Processing Record 27 of Set 31 | harnosand,se
    Data found (status code: 200) | harnosand,se
    Data loaded successfully | harnosand,se
    *********************************
    Processing Record 28 of Set 31 | almaznyy,ru
    Data found (status code: 200) | almaznyy,ru
    Data loaded successfully | almaznyy,ru
    *********************************
    Processing Record 29 of Set 31 | kamenskoye,ru
    ERROR: Data retrieval unsuccessful for kamenskoye,ru. Code: 404                       MSG: city not found
    Processing Record 30 of Set 31 | mayo,ca
    Data found (status code: 200) | mayo,ca
    Data loaded successfully | mayo,ca
    *********************************
    Processing Record 31 of Set 31 | turukhansk,ru
    Data found (status code: 200) | turukhansk,ru
    Data loaded successfully | turukhansk,ru
    *********************************
    Processing Record 32 of Set 31 | ossora,ru
    Data found (status code: 200) | ossora,ru
    Data loaded successfully | ossora,ru
    *********************************
    Processing Record 33 of Set 31 | ornskoldsvik,se
    Data found (status code: 200) | ornskoldsvik,se
    Data loaded successfully | ornskoldsvik,se
    *********************************
    Processing Record 34 of Set 31 | chernyshevskiy,ru
    Data found (status code: 200) | chernyshevskiy,ru
    Data loaded successfully | chernyshevskiy,ru
    *********************************
    Processing Record 35 of Set 31 | roros,no
    Data found (status code: 200) | roros,no
    Data loaded successfully | roros,no
    *********************************
    Processing Record 36 of Set 31 | egvekinot,ru
    Data found (status code: 200) | egvekinot,ru
    Data loaded successfully | egvekinot,ru
    *********************************
    Processing Record 37 of Set 31 | artyk,ru
    ERROR: Data retrieval unsuccessful for artyk,ru. Code: 404                       MSG: city not found
    Processing Record 38 of Set 31 | khandyga,ru
    Data found (status code: 200) | khandyga,ru
    Data loaded successfully | khandyga,ru
    *********************************
    Processing Record 39 of Set 31 | voyvozh,ru
    Data found (status code: 200) | voyvozh,ru
    Data loaded successfully | voyvozh,ru
    *********************************
    Processing Record 40 of Set 31 | kholodnyy,ru
    Data found (status code: 200) | kholodnyy,ru
    Data loaded successfully | kholodnyy,ru
    *********************************
    Processing Record 41 of Set 31 | wasilla,us
    Data found (status code: 200) | wasilla,us
    Data loaded successfully | wasilla,us
    *********************************
    Processing Record 42 of Set 31 | krasnoselkup,ru
    ERROR: Data retrieval unsuccessful for krasnoselkup,ru. Code: 404                       MSG: city not found
    Processing Record 43 of Set 31 | pudozh,ru
    Data found (status code: 200) | pudozh,ru
    Data loaded successfully | pudozh,ru
    *********************************
    Processing Record 44 of Set 31 | brae,gb
    Data found (status code: 200) | brae,gb
    Data loaded successfully | brae,gb
    *********************************
    Processing Record 45 of Set 31 | raudeberg,no
    Data found (status code: 200) | raudeberg,no
    Data loaded successfully | raudeberg,no
    *********************************
    Processing Record 46 of Set 31 | agirish,ru
    Data found (status code: 200) | agirish,ru
    Data loaded successfully | agirish,ru
    *********************************
    Processing Record 47 of Set 31 | dukat,ru
    Data found (status code: 200) | dukat,ru
    Data loaded successfully | dukat,ru
    *********************************
    Processing Record 48 of Set 31 | muyezerskiy,ru
    Data found (status code: 200) | muyezerskiy,ru
    Data loaded successfully | muyezerskiy,ru
    *********************************
    Processing Record 49 of Set 31 | evensk,ru
    Data found (status code: 200) | evensk,ru
    Data loaded successfully | evensk,ru
    *********************************
    Processing Record 50 of Set 31 | omsukchan,ru
    Data found (status code: 200) | omsukchan,ru
    Data loaded successfully | omsukchan,ru
    *********************************
    Processing Record 51 of Set 31 | bereznik,ru
    Data found (status code: 200) | bereznik,ru
    Data loaded successfully | bereznik,ru
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 32 | umea,se
    Data found (status code: 200) | umea,se
    Data loaded successfully | umea,se
    *********************************
    Processing Record 1 of Set 32 | troitsko-pechorsk,ru
    Data found (status code: 200) | troitsko-pechorsk,ru
    Data loaded successfully | troitsko-pechorsk,ru
    *********************************
    Processing Record 2 of Set 32 | ust-nera,ru
    Data found (status code: 200) | ust-nera,ru
    Data loaded successfully | ust-nera,ru
    *********************************
    Processing Record 3 of Set 32 | klaksvik,fo
    Data found (status code: 200) | klaksvik,fo
    Data loaded successfully | klaksvik,fo
    *********************************
    Processing Record 4 of Set 32 | mudyuga,ru
    ERROR: Data retrieval unsuccessful for mudyuga,ru. Code: 404                       MSG: city not found
    Processing Record 5 of Set 32 | ostersund,se
    Data found (status code: 200) | ostersund,se
    Data loaded successfully | ostersund,se
    *********************************
    Processing Record 6 of Set 32 | siilinjarvi,fi
    Data found (status code: 200) | siilinjarvi,fi
    Data loaded successfully | siilinjarvi,fi
    *********************************
    Processing Record 7 of Set 32 | eyrarbakki,is
    Data found (status code: 200) | eyrarbakki,is
    Data loaded successfully | eyrarbakki,is
    *********************************
    Processing Record 8 of Set 32 | tura,ru
    Data found (status code: 200) | tura,ru
    Data loaded successfully | tura,ru
    *********************************
    Processing Record 9 of Set 32 | kajaani,fi
    Data found (status code: 200) | kajaani,fi
    Data loaded successfully | kajaani,fi
    *********************************
    Processing Record 10 of Set 32 | college,us
    Data found (status code: 200) | college,us
    Data loaded successfully | college,us
    *********************************
    Processing Record 11 of Set 32 | synya,ru
    Data found (status code: 200) | synya,ru
    Data loaded successfully | synya,ru
    *********************************
    Processing Record 12 of Set 32 | tarko-sale,ru
    Data found (status code: 200) | tarko-sale,ru
    Data loaded successfully | tarko-sale,ru
    *********************************
    Processing Record 13 of Set 32 | leshukonskoye,ru
    Data found (status code: 200) | leshukonskoye,ru
    Data loaded successfully | leshukonskoye,ru
    *********************************
    Processing Record 14 of Set 32 | norman wells,ca
    Data found (status code: 200) | norman wells,ca
    Data loaded successfully | norman wells,ca
    *********************************
    Processing Record 15 of Set 32 | verkhnevilyuysk,ru
    Data found (status code: 200) | verkhnevilyuysk,ru
    Data loaded successfully | verkhnevilyuysk,ru
    *********************************
    Processing Record 16 of Set 32 | sisimiut,gl
    Data found (status code: 200) | sisimiut,gl
    Data loaded successfully | sisimiut,gl
    *********************************
    Processing Record 17 of Set 32 | kemi,fi
    Data found (status code: 200) | kemi,fi
    Data loaded successfully | kemi,fi
    *********************************
    Processing Record 18 of Set 32 | skelleftea,se
    Data found (status code: 200) | skelleftea,se
    Data loaded successfully | skelleftea,se
    *********************************
    Processing Record 19 of Set 32 | kysyl-syr,ru
    Data found (status code: 200) | kysyl-syr,ru
    Data loaded successfully | kysyl-syr,ru
    *********************************
    Processing Record 20 of Set 32 | bilibino,ru
    Data found (status code: 200) | bilibino,ru
    Data loaded successfully | bilibino,ru
    *********************************
    Processing Record 21 of Set 32 | barrow,us
    Data found (status code: 200) | barrow,us
    Data loaded successfully | barrow,us
    *********************************
    Processing Record 22 of Set 32 | aykhal,ru
    Data found (status code: 200) | aykhal,ru
    Data loaded successfully | aykhal,ru
    *********************************
    Processing Record 23 of Set 32 | nyurba,ru
    Data found (status code: 200) | nyurba,ru
    Data loaded successfully | nyurba,ru
    *********************************
    Processing Record 24 of Set 32 | rorvik,no
    Data found (status code: 200) | rorvik,no
    Data loaded successfully | rorvik,no
    *********************************
    Processing Record 25 of Set 32 | salekhard,ru
    Data found (status code: 200) | salekhard,ru
    Data loaded successfully | salekhard,ru
    *********************************
    Processing Record 26 of Set 32 | udachnyy,ru
    Data found (status code: 200) | udachnyy,ru
    Data loaded successfully | udachnyy,ru
    *********************************
    Processing Record 27 of Set 32 | rognan,no
    Data found (status code: 200) | rognan,no
    Data loaded successfully | rognan,no
    *********************************
    Processing Record 28 of Set 32 | srednekolymsk,ru
    Data found (status code: 200) | srednekolymsk,ru
    Data loaded successfully | srednekolymsk,ru
    *********************************
    Processing Record 29 of Set 32 | inuvik,ca
    Data found (status code: 200) | inuvik,ca
    Data loaded successfully | inuvik,ca
    *********************************
    Processing Record 30 of Set 32 | maniitsoq,gl
    Data found (status code: 200) | maniitsoq,gl
    Data loaded successfully | maniitsoq,gl
    *********************************
    Processing Record 31 of Set 32 | husavik,is
    Data found (status code: 200) | husavik,is
    Data loaded successfully | husavik,is
    *********************************
    Processing Record 32 of Set 32 | umba,ru
    Data found (status code: 200) | umba,ru
    Data loaded successfully | umba,ru
    *********************************
    Processing Record 33 of Set 32 | haukipudas,fi
    Data found (status code: 200) | haukipudas,fi
    Data loaded successfully | haukipudas,fi
    *********************************
    Processing Record 34 of Set 32 | kuusamo,fi
    Data found (status code: 200) | kuusamo,fi
    Data loaded successfully | kuusamo,fi
    *********************************
    Processing Record 35 of Set 32 | bolungarvik,is
    ERROR: Data retrieval unsuccessful for bolungarvik,is. Code: 404                       MSG: city not found
    Processing Record 36 of Set 32 | koslan,ru
    Data found (status code: 200) | koslan,ru
    Data loaded successfully | koslan,ru
    *********************************
    Processing Record 37 of Set 32 | pyaozerskiy,ru
    Data found (status code: 200) | pyaozerskiy,ru
    Data loaded successfully | pyaozerskiy,ru
    *********************************
    Processing Record 38 of Set 32 | zyryanka,ru
    Data found (status code: 200) | zyryanka,ru
    Data loaded successfully | zyryanka,ru
    *********************************
    Processing Record 39 of Set 32 | muzhi,ru
    Data found (status code: 200) | muzhi,ru
    Data loaded successfully | muzhi,ru
    *********************************
    Processing Record 40 of Set 32 | khonuu,ru
    ERROR: Data retrieval unsuccessful for khonuu,ru. Code: 404                       MSG: city not found
    Processing Record 41 of Set 32 | svetlogorsk,ru
    Data found (status code: 200) | svetlogorsk,ru
    Data loaded successfully | svetlogorsk,ru
    *********************************
    Processing Record 42 of Set 32 | rovaniemi,fi
    Data found (status code: 200) | rovaniemi,fi
    Data loaded successfully | rovaniemi,fi
    *********************************
    Processing Record 43 of Set 32 | roald,no
    Data found (status code: 200) | roald,no
    Data loaded successfully | roald,no
    *********************************
    Processing Record 44 of Set 32 | skagastrond,is
    ERROR: Data retrieval unsuccessful for skagastrond,is. Code: 404                       MSG: city not found
    Processing Record 45 of Set 32 | vestmanna,fo
    Data found (status code: 200) | vestmanna,fo
    Data loaded successfully | vestmanna,fo
    *********************************
    Processing Record 46 of Set 32 | qasigiannguit,gl
    Data found (status code: 200) | qasigiannguit,gl
    Data loaded successfully | qasigiannguit,gl
    *********************************
    Processing Record 47 of Set 32 | pevek,ru
    Data found (status code: 200) | pevek,ru
    Data loaded successfully | pevek,ru
    *********************************
    Processing Record 48 of Set 32 | bud,no
    Data found (status code: 200) | bud,no
    Data loaded successfully | bud,no
    *********************************
    Processing Record 49 of Set 32 | zhigansk,ru
    Data found (status code: 200) | zhigansk,ru
    Data loaded successfully | zhigansk,ru
    *********************************
    Processing Record 50 of Set 32 | aklavik,ca
    Data found (status code: 200) | aklavik,ca
    Data loaded successfully | aklavik,ca
    *********************************
    Processing Record 51 of Set 32 | kiruna,se
    Data found (status code: 200) | kiruna,se
    Data loaded successfully | kiruna,se
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    Processing Record 0 of Set 33 | belaya gora,ru
    Data found (status code: 200) | belaya gora,ru
    Data loaded successfully | belaya gora,ru
    *********************************
    Processing Record 1 of Set 33 | kamenka,ru
    Data found (status code: 200) | kamenka,ru
    Data loaded successfully | kamenka,ru
    *********************************
    Processing Record 2 of Set 33 | clyde river,ca
    Data found (status code: 200) | clyde river,ca
    Data loaded successfully | clyde river,ca
    *********************************
    Processing Record 3 of Set 33 | tazovskiy,ru
    Data found (status code: 200) | tazovskiy,ru
    Data loaded successfully | tazovskiy,ru
    *********************************
    Processing Record 4 of Set 33 | batagay-alyta,ru
    Data found (status code: 200) | batagay-alyta,ru
    Data loaded successfully | batagay-alyta,ru
    *********************************
    Processing Record 5 of Set 33 | cherskiy,ru
    Data found (status code: 200) | cherskiy,ru
    Data loaded successfully | cherskiy,ru
    *********************************
    Processing Record 6 of Set 33 | tuktoyaktuk,ca
    Data found (status code: 200) | tuktoyaktuk,ca
    Data loaded successfully | tuktoyaktuk,ca
    *********************************
    Processing Record 7 of Set 33 | staryy nadym,ru
    Data found (status code: 200) | staryy nadym,ru
    Data loaded successfully | staryy nadym,ru
    *********************************
    Processing Record 8 of Set 33 | kangaatsiaq,gl
    Data found (status code: 200) | kangaatsiaq,gl
    Data loaded successfully | kangaatsiaq,gl
    *********************************
    Processing Record 9 of Set 33 | sorland,no
    Data found (status code: 200) | sorland,no
    Data loaded successfully | sorland,no
    *********************************
    Processing Record 10 of Set 33 | deputatskiy,ru
    Data found (status code: 200) | deputatskiy,ru
    Data loaded successfully | deputatskiy,ru
    *********************************
    Processing Record 11 of Set 33 | batagay,ru
    Data found (status code: 200) | batagay,ru
    Data loaded successfully | batagay,ru
    *********************************
    Processing Record 12 of Set 33 | yar-sale,ru
    Data found (status code: 200) | yar-sale,ru
    Data loaded successfully | yar-sale,ru
    *********************************
    Processing Record 13 of Set 33 | khatanga,ru
    Data found (status code: 200) | khatanga,ru
    Data loaded successfully | khatanga,ru
    *********************************
    Processing Record 14 of Set 33 | aksarka,ru
    Data found (status code: 200) | aksarka,ru
    Data loaded successfully | aksarka,ru
    *********************************
    Processing Record 15 of Set 33 | ilulissat,gl
    Data found (status code: 200) | ilulissat,gl
    Data loaded successfully | ilulissat,gl
    *********************************
    Processing Record 16 of Set 33 | tumannyy,ru
    ERROR: Data retrieval unsuccessful for tumannyy,ru. Code: 404                       MSG: city not found
    Processing Record 17 of Set 33 | bjornevatn,no
    Data found (status code: 200) | bjornevatn,no
    Data loaded successfully | bjornevatn,no
    *********************************
    Processing Record 18 of Set 33 | mys shmidta,ru
    ERROR: Data retrieval unsuccessful for mys shmidta,ru. Code: 404                       MSG: city not found
    Processing Record 19 of Set 33 | ostrovnoy,ru
    Data found (status code: 200) | ostrovnoy,ru
    Data loaded successfully | ostrovnoy,ru
    *********************************
    Processing Record 20 of Set 33 | komsomolskiy,ru
    Data found (status code: 200) | komsomolskiy,ru
    Data loaded successfully | komsomolskiy,ru
    *********************************
    Processing Record 21 of Set 33 | leningradskiy,ru
    Data found (status code: 200) | leningradskiy,ru
    Data loaded successfully | leningradskiy,ru
    *********************************
    Processing Record 22 of Set 33 | talnakh,ru
    Data found (status code: 200) | talnakh,ru
    Data loaded successfully | talnakh,ru
    *********************************
    Processing Record 23 of Set 33 | tiksi,ru
    Data found (status code: 200) | tiksi,ru
    Data loaded successfully | tiksi,ru
    *********************************
    Processing Record 24 of Set 33 | upernavik,gl
    Data found (status code: 200) | upernavik,gl
    Data loaded successfully | upernavik,gl
    *********************************
    Processing Record 25 of Set 33 | karaul,ru
    ERROR: Data retrieval unsuccessful for karaul,ru. Code: 404                       MSG: city not found
    Processing Record 26 of Set 33 | illoqqortoormiut,gl
    ERROR: Data retrieval unsuccessful for illoqqortoormiut,gl. Code: 404                       MSG: city not found
    Processing Record 27 of Set 33 | skalistyy,ru
    ERROR: Data retrieval unsuccessful for skalistyy,ru. Code: 404                       MSG: city not found
    Processing Record 28 of Set 33 | sistranda,no
    Data found (status code: 200) | sistranda,no
    Data loaded successfully | sistranda,no
    *********************************
    Processing Record 29 of Set 33 | amderma,ru
    ERROR: Data retrieval unsuccessful for amderma,ru. Code: 404                       MSG: city not found
    Processing Record 30 of Set 33 | kayerkan,ru
    Data found (status code: 200) | kayerkan,ru
    Data loaded successfully | kayerkan,ru
    *********************************
    Processing Record 31 of Set 33 | andenes,no
    Data found (status code: 200) | andenes,no
    Data loaded successfully | andenes,no
    *********************************
    Processing Record 32 of Set 33 | stokmarknes,no
    Data found (status code: 200) | stokmarknes,no
    Data loaded successfully | stokmarknes,no
    *********************************
    Processing Record 33 of Set 33 | belushya guba,ru
    ERROR: Data retrieval unsuccessful for belushya guba,ru. Code: 404                       MSG: city not found
    Processing Record 34 of Set 33 | saskylakh,ru
    Data found (status code: 200) | saskylakh,ru
    Data loaded successfully | saskylakh,ru
    *********************************
    Processing Record 35 of Set 33 | chokurdakh,ru
    Data found (status code: 200) | chokurdakh,ru
    Data loaded successfully | chokurdakh,ru
    *********************************
    Processing Record 36 of Set 33 | finnsnes,no
    Data found (status code: 200) | finnsnes,no
    Data loaded successfully | finnsnes,no
    *********************************
    Processing Record 37 of Set 33 | tromso,no
    Data found (status code: 200) | tromso,no
    Data loaded successfully | tromso,no
    *********************************
    Processing Record 38 of Set 33 | nizhneyansk,ru
    ERROR: Data retrieval unsuccessful for nizhneyansk,ru. Code: 404                       MSG: city not found
    Processing Record 39 of Set 33 | dudinka,ru
    Data found (status code: 200) | dudinka,ru
    Data loaded successfully | dudinka,ru
    *********************************
    Processing Record 40 of Set 33 | dikson,ru
    Data found (status code: 200) | dikson,ru
    Data loaded successfully | dikson,ru
    *********************************
    Processing Record 41 of Set 33 | hammerfest,no
    Data found (status code: 200) | hammerfest,no
    Data loaded successfully | hammerfest,no
    *********************************
    Processing Record 42 of Set 33 | qaanaaq,gl
    Data found (status code: 200) | qaanaaq,gl
    Data loaded successfully | qaanaaq,gl
    *********************************
    Processing Record 43 of Set 33 | kjollefjord,no
    Data found (status code: 200) | kjollefjord,no
    Data loaded successfully | kjollefjord,no
    *********************************
    Processing Record 44 of Set 33 | barentsburg,sj
    ERROR: Data retrieval unsuccessful for barentsburg,sj. Code: 404                       MSG: city not found
    Processing Record 45 of Set 33 | oksfjord,no
    Data found (status code: 200) | oksfjord,no
    Data loaded successfully | oksfjord,no
    *********************************
    Processing Record 46 of Set 33 | narsaq,gl
    Data found (status code: 200) | narsaq,gl
    Data loaded successfully | narsaq,gl
    *********************************
    Processing Record 47 of Set 33 | longyearbyen,sj
    Data found (status code: 200) | longyearbyen,sj
    Data loaded successfully | longyearbyen,sj
    *********************************
    Processing Record 48 of Set 33 | mehamn,no
    Data found (status code: 200) | mehamn,no
    Data loaded successfully | mehamn,no
    *********************************
    Processing Record 49 of Set 33 | havoysund,no
    Data found (status code: 200) | havoysund,no
    Data loaded successfully | havoysund,no
    *********************************
    Processing Record 50 of Set 33 | berlevag,no
    Data found (status code: 200) | berlevag,no
    Data loaded successfully | berlevag,no
    *********************************
    Processing Record 51 of Set 33 | vardo,no
    Data found (status code: 200) | vardo,no
    Data loaded successfully | vardo,no
    *********************************
    Sleeping 60 seconds before next round to avoid usage limits
    *********************************
    * DATA COLLECTION COMPLETE..... 



```python
# create dataframe from dictionary
city_data_df = pd.DataFrame(city_data_dict)

# calculate record date for plots
record_date = datetime.datetime.fromtimestamp(int(city_data_df['record_dt'][0])).strftime('%m/%d/%y')

#Reorganize the DataFrame for saving as a csv and easy viewing
city_data_display = city_data_df[['city_name', 'city_country_code', 'city_id', 'city_lat', 'city_lng',
                                 'city_max_temp', 'city_humidity', 'city_cloud_cover', 'city_wind_speed',
                                 'record_dt']].set_index('city_name')
#Print a sample of the dataframe
city_data_display.head()
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
      <th>city_country_code</th>
      <th>city_id</th>
      <th>city_lat</th>
      <th>city_lng</th>
      <th>city_max_temp</th>
      <th>city_humidity</th>
      <th>city_cloud_cover</th>
      <th>city_wind_speed</th>
      <th>record_dt</th>
    </tr>
    <tr>
      <th>city_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Hobart</th>
      <td>AU</td>
      <td>2163355</td>
      <td>-42.88</td>
      <td>147.33</td>
      <td>53.60</td>
      <td>93</td>
      <td>0</td>
      <td>2.28</td>
      <td>1524315600</td>
    </tr>
    <tr>
      <th>Port Alfred</th>
      <td>ZA</td>
      <td>964432</td>
      <td>-33.59</td>
      <td>26.89</td>
      <td>74.45</td>
      <td>77</td>
      <td>80</td>
      <td>8.61</td>
      <td>1524316437</td>
    </tr>
    <tr>
      <th>Rikitea</th>
      <td>PF</td>
      <td>4030556</td>
      <td>-23.12</td>
      <td>-134.97</td>
      <td>77.06</td>
      <td>100</td>
      <td>80</td>
      <td>10.29</td>
      <td>1524316437</td>
    </tr>
    <tr>
      <th>Hermanus</th>
      <td>ZA</td>
      <td>3366880</td>
      <td>-34.42</td>
      <td>19.24</td>
      <td>73.28</td>
      <td>42</td>
      <td>0</td>
      <td>4.81</td>
      <td>1524316437</td>
    </tr>
    <tr>
      <th>Ushuaia</th>
      <td>AR</td>
      <td>3833367</td>
      <td>-54.81</td>
      <td>-68.31</td>
      <td>42.80</td>
      <td>60</td>
      <td>40</td>
      <td>4.70</td>
      <td>1524312000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# create a comma-delimited file
city_data_display.to_csv("output/weatherpy.csv")
```

## Latitude (Degrees) vs Max Temperature (F)


```python
# Build a scatter plot for each data type
plt.scatter(city_data_df["city_lat"],
            city_data_df["city_max_temp"],
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="City")

# Incorporate the other graph properties
chart_title = f'City Latitude vs Max Temperature ({record_date})'
plt.title(chart_title)
plt.ylabel("Temperature (Fahrenheit)")
plt.xlabel("Latitude (Degrees)")
plt.grid(True)
plt.xlim([-90, 90])
plt.ylim([-20, 110])
plt.savefig("output/lat_vs_max_temp.png")
plt.show()
```


![png](output_8_0.png)


## Latitude (Degrees) vs Humidity (%)


```python
# Build a scatter plot for each data type
plt.scatter(city_data_df["city_lat"],
            city_data_df["city_humidity"],
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="City")

# Incorporate the other graph properties
chart_title = f'City Latitude vs Max Humidity ({record_date})'
plt.title(chart_title)
plt.ylabel("Humidity (%)")
plt.xlabel("Latitude (Degrees)")
plt.grid(True)
plt.xlim([-90, 90])
plt.ylim([-10, 110])
plt.savefig("output/lat_vs_max_humidity.png")
plt.show()
```


![png](output_10_0.png)


## Latitude (Degrees) vs Cloudiness (%)


```python
# Build a scatter plot for each data type
plt.scatter(city_data_df["city_lat"],
            city_data_df["city_cloud_cover"],
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="City")

# Incorporate the other graph properties
chart_title = f'City Latitude vs Cloud Cover ({record_date})'
plt.title(chart_title)
plt.ylabel("Cloud Cover (%)")
plt.xlabel("Latitude (Degrees)")
plt.grid(True)
plt.xlim([-90, 90])
plt.ylim([-10, 110])
plt.savefig("output/lat_vs_cloud_cover.png")
plt.show()
```


![png](output_12_0.png)


## Latitude (Degrees) vs Windspeed (MPH)


```python
# Build a scatter plot for each data type
plt.scatter(city_data_df["city_lat"],
            city_data_df["city_wind_speed"],
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="City")

# Incorporate the other graph properties
chart_title = f'City Latitude vs Wind Speed ({record_date})'
plt.title(chart_title)
plt.ylabel("Wind Speed (MPH)")
plt.xlabel("Latitude (Degrees)")
plt.grid(True)
plt.xlim([-90, 90])
plt.ylim([0, 30])
plt.savefig("output/lat_vs_wind_speed.png")
plt.show()
```


![png](output_14_0.png)

