---
title: "Developing Data Products Week 2 Project 1 - Mapping US National Parks in R with Leaflet"
linktitle: "Developing Data Products Week 2 Project 1 - Mapping US National Parks in R with Leaflet"
author: "Jed Raynes"
date: 2021-04-01
type:
- post 
- posts
weight: 10
# series:
# - Hugo 101
aliases:
- /posts/Mapping-US-National-Parks-in-R-with-Leaflet
---

# Mapping US National Parks

### # Overview and Sources

---

**Created:** April 1, 2021

This webpage is my project submission for the Developing Data Products Week 2 Project.
The source of the data is found [here](https://download.geonames.org/export/dump/) and the data dictionary can be found [here](https://download.geonames.org/export/dump/readme.txt).The listing of US National Park Service National Parks can be found [here](https://www.nps.gov/aboutus/national-park-system.htm). I used the NPS listing to check that all national parks (63 in total) were included in my listing.

### # Data Wrangling

---

To start, let's load our packages. For this, we'll use leaflet to plot the parks, dplyr to manipulate our data,  data.table to copy data frames for internal version control, and htmlwidgets and htmltools to save our map for the website.


```r
# load libraries
suppressPackageStartupMessages(pacman::p_load(leaflet, 
                                              dplyr, 
                                              data.table, 
                                              htmlwidgets, 
                                              htmltools))
```

Next, we'll define the working directory to access our files and load in the data.



```r
# set cwd
setwd('C:\\Users\\Jed\\iCloudDrive\\Documents\\Learn\\R\\Johns Hopkins Data Science Specialization\\9 Developing Data Products\\Week 2')

# define column names
col_names <- c('geonameid', 
               'name', 
               'asciiname', 
               'alternatenames', 
               'latitude', 
               'longitude', 
               'feature_class', 
               'feature_code', 
               'country_code', 
               'cc2', 
               'admin1_code', 
               'admin2_code',
               'admin3_code', 
               'admin4_code', 
               'population', 
               'elevation', 
               'dem', 
               'timezone', 
               'modification_date')

# load data
df_all <- read.csv('US.txt', sep = '\t', col.names = col_names)
```


Then, we clean the loaded data to prepare a dataframe that is structured for our leaflet plot. I first copied the necessary columns from the entire data. I then filtered for class 'L' which represents parks and areas per the data dictionary, filtered where the name contained 'National Park', and arranged it alphabetically so I could easily compare it to the NPS site which is listed A-Z.

After doing so, I compiled a list of names in my listing that weren't national parks and removed these items from my dataframe. After doing so, I grouped my dataframe by the names and took the average latitude and longitude to group away parks that were listed more than once.

Finally, I compared the dataframe of 59 parks to the NPS listing of 63 parks and determined which parks were missing. It turns out I was missing four national parks: Gateway Arch National Park, Indiana Dunes National Park, National Park of American Samoa, New River Gorge National Park, and White Sands National Park. I grabbed the latitude and longitude from this [site](https://www.gps-coordinates.net/) and created a dataframe to append to my master list. After appending, I checked my listing and confirmed the 63 parks I had in my dataset matched the NPS listing per their site.


```r
# copy df and only select the necessary columns
df <- copy(df_all)[, c(2, 5, 6, 7)]

# subset the df to only national parks
df <- df %>%
  filter(feature_class == 'L') %>%
  filter(grepl('National Park', name)) %>%
  arrange(name)

# define items in the subset that aren't NPS national parks and remove them
not_parks <- c('Capital Reef National Park', 
               'Congaree National Park Auxillary Parking Lot', 
               'Congaree National Park Wilderness',
               'Denali National Park and Preserve Headquarters', 
               'Fiske Hill National Park', 
               'Fort Hunt National Park', 
               'Fort Nonsense Historical National Park', 
               'Great Sand Dunes National Park and Preserve', 
               'Great Smoky Mountains National Park (North Carolina secion)', 
               'Horseshoe Canyon Unit- Canyonlands National Park', 
               'National Park', 
               'National Park Service Headquarters', 
               'Olympic National Park Headquarters Historic District', 
               'Prairie National Park', 
               'Rio Cacheu National Park', 
               'Rocky Mountain National Park Administration Building', 
               'Rocky Mountain National Park Wilderness', 
               'Saguaro National Park East Unit (historical)', 
               'Sequoia and Kings Canyon National Parks', 
               'Southeast Archeological Center National Park Service', 
               'Theodore Roosevelt National Park - Nor`th Unit', 
               'Washington Headquarters National Park', 
               'Wolf Trap National Park for the Performing Arts')
df <- df[!(df$name %in% not_parks), ]

# to remove the duplicates, we'll group by name and take the average latitude and longitude
df <- df %>% 
  group_by(name) %>%
  summarize(latitude = mean(latitude), 
            longitude = mean(longitude))
  
# define four missing national parks
parks_to_add <- data.frame(name = c('Gateway Arch National Park',  
                                    'Indiana Dunes National Park', 
                                    'National Park of American Samoa', 
                                    'New River Gorge National Park', 
                                    'White Sands National Park'), 
                           latitude = c(38.6246071, 
                                        41.663543701171875, 
                                        -14.2619444, 
                                        37.84026336669922, 
                                        32.783233642578125), 
                           longitude = c(-90.1849717, 
                                         -87.03569793701172, 
                                         -170.6880556, 
                                         -80.98970794677734, 
                                         -106.33186340332031))

# layer in missing parks
df <- df %>%
  rbind(parks_to_add) %>%
  arrange(name)
```

After organizing my data, I created an icon using the NPS logo to make my map have a bit more flair. Finally, I plotted my map using leaflet and used the default clustering to visualize where most parks were located (western US). The most interesting one, in my opinion, is the National Park of American Samoa, that's definitely going on my travel list.

<iframe seamless src="/static/leaflet/np_map.html" width="100%" height="500"></iframe>
