---
layout: post
title: Simulating 6th - Instagram API
featured-img: shane-rounce-205187
---

I ultimately decided to scale-up the userlist for the Twitter SNA, so data collection is taking a lot longer than expected. In the interim, I thought it would be fun to explore another API and show how we can use analytics to map data geospatially. Specifically, I provided the GPS coordinates for 6th Street and check local businesses's instagrams.

### Table of Contents

##### Technical
1. [What is Different?](#chapter1)
2. [Tools](#chapter2)
3. [Connecting to the API](#chapter3)
4. [Collecting Data](#chapter4)
5. [Analysis](#chapter5)

<a name="chapter1"></a>
# What is different? 

Although much of basics remain the same, using the Instagram API is a bit different than Twitter's. For starters, I didn't end up using a library for the Instagram API, so it's a lot more rudimentary. It'll make sense when you see it. There are librarys for Instagram, but the API has undergone a lot of changes since the start of 2018, so some of the library features aren't up-to-date. In this case, I prefer to just forgo the library.

Another major difference is Instagram's APP review period. In order to actually access data you have to have an app that is approved, and that serves customers in one or more usecases that've been outlined in the [developer policy.](https://www.instagram.com/developer/review/) Until you get reviewed and approved, your client exists in a "sandbox," which means you can't access public data. You're able to access your own data, plus up to 10 people you invite into your sandbox. But you're only allowed to look at the 20 most recent uploads. This still makes it useful if you're doing personal analysis, or have a small scope, but makes large scale analysis impossible. For instance, you could write a quick script to download 20 of your recent posts, but couldn't look at your entire history.

For this project I wanted to input a GPS coordinate and find nearby businesses to determine which had a higher average of likes.

Since this was a side project, and I'm not too serious about it, I decided to just simulate the data I needed as opposed to trying to get an app approved. This wasn't that difficult, but it definitely makes this data pretty much useless outside of a tutorial sense.

<a name = "chapter2"></a>

# Tools

For this project, I'm using Python again. I'm also trying out vim and [QGIS.](https://qgis.org/en/site/) I've wanted to set up a vim environment for awhile, and I think this is a small enough project to test it out on. I'm not covering it in this tutorial, but so far I definitely liked having something as lightweight as vim. It almost feels like a rite of passage. 

QGIS is another tool I've had my eye on for awhile that I've been meaning to try. It's a pretty standard open source geo-info system. Since I wanted to use satellite imagery of sixth street, I also downloaded GPSBabel. This will allow us to import our csv data for use within QGIS.

Installing these on a unix system is as simple as:
`sudo apt-get install qgis gpsbabel
`
 Within QGIS, I've found that OpenLayers is incredibly valuable. It allows us to apply basemaps from within the service. I prefer to use Google Earth satellite, which I ended up using. So I had to register to get an Google Earth API token from the Google Cloud Platform. The Google Earth service is pay-per-use, and has more than enough free monthly queries than we'll ever need for this project.


<a name = "chapter3"></a>

# Connecting to the API

Connecting to the API is a bit different, but still fairly straight forward. When you first register your app, you're provided with a client ID, client Secret, and a redirect URL that you initially provided during setup.

For this project, I used client-side (implicit) Authentication. The details are in the dev documentation, which you'll definitely need to make yourself familiar with just to use the end-points for data collection.

Basically, you direct yourself to the URL:
`https://api.instagram.com/oauth/authorize/?client_id=[CLIENT-ID]&redirect_uri=[REDIRECT-URI]&response_type=token
` 
where you change [CLIENT-ID] and [REDIRECT-URI] for whatever you recieved upon app creation. 

From there, you look at the URL you're directed to and copy the long string of characters that should be at the end of wherever you end up. `"access_token=[COPY THIS]"`

This string will be what you use to access the API. As always, it's best to keep it private, for example, make note of how I call in a .txt file when uploading my code to Github.

<a name = "chapter4"></a>

# Collecting Data

Collecting data is a matter of using the endpoint urls appropriately whilst defining parameters. For example, a query to find nearby locations looks like:

`base_url = 'https://api.instagram.com/v1/'`

```python
def get_loc_search(lat, lng, distance):

    request_url = (base_url + 'locations/search?lat=%s&lng=%s&distance=%s&access_token=%s') 
    	% (lat, lng, distance, access_token)

    print 'GET REQUEST URL : %s' % (request_url)

out_data = requests.get(request_url).json()
```

As you can see, a request url is specific to what data you want to gather. Formating that request url is based on the endpoints in the dev documentation. So for finding location/search data, we need latitude, longitude, distance (which is optional), and for every query we need our access token.

Python's requests library is pretty dope, it makes an http request at request_url, and returns a json.

You can see the format of the json that it returns by going to the URL, but it's also fairly easy to look at the endpoint documentation to find out product format.

```python
product = []
i = 0

#if meta code doesn't == 200, there's an error so we stop.
if out_data['meta']['code'] == 200:
        if len(out_data['data']):
                #here's where most of the magic happens
                try:
                    while i < (len(out_data['data'])+1):
                        i += 1
                        #collecting all the data
                        #product = an individual location

                        product_id = out_data['data'][i]['id'].encode("utf-8")         
                        product_lat = out_data['data'][i]['latitude']
                        product_long = out_data['data'][i]['longitude']                           
                        product_name = out_data['data'][i]['name'].encode("utf-8")

                        temp = []
                        temp.append(product_id)
                        temp.append(product_lat)
                        temp.append(product_long)
                        temp.append(product_name)
                    	product.append(temp)
```

As you can tell, since I looked at the documentation, I know the output is going to be formated as ['data'][i][*(value)*] so I'm pulling that and assigning it to a variable according to what it is. I'm storing the data in a list of lists, so temp is of variables for this iteration, and product is the complete output that gets returned.

In the sandbox, this returns 20 locations corresponding to the GPS coordinates, the name, and the ID that instagram associates with that location.

Every other query is pretty much formatted the same, and you can refer to the source code on github for the other examples. The link to the repo will be at the end.

<a name = "chapter5"></a>

# Analysis

Importing data into QGIS is pretty straightforward, and there are plenty of tutorials that will have more indepth explanations than I can provide. Since I'm not an expert, I'll refer you to <http://all-geo.org/volcan01010/2014/10/easily-plot-data-on-a-google-maps-background-with-the-qgis-openlayers-plugin/> if you want to learn more about the software. 

One thing that that tutorial doesn't mention is the CRS you'll want to give your data in order for it to line up with the Google Earth basemap. WGS84 - EPSG:4326 works correctly with the default CRS that the basemap has. That's pretty much standard, but for someone new to GIS software, it'll save you a lot of time to know that. Trust me.

<html>
 <head>
  <style>
  * {
   margin: 0;
   padding: 0;
  }
  .imgbox {
   display: grid;
   height: 50%;
   transition: transform .2s;
  }
  .center-fit {
   max-width: 100%;
   max-height: 100%;
   margin: auto;
  }
  .imgbox:hover{
    transform: scale(1.5);
  }
  </style>
 </head>
 <body>
  <div class="imgbox">
   <img class="center-fit" src='{{site.url}}/assets/img/posts/output_highest_stD_likes2.png'>
  </div>
 </body>
</html>

This is an example output from QGIS using GE Satellite imagery. As you can see, naturally, the Gates-Dell Complex is popping for instagram pics. This was really just coincidental, as the way it's simulated is more or less just rolling a dice. The red marked areas are locations with the highest average of likes. I filtered the data to ignore about 80% of the locations, so that we could only see the highest. Another interesting way to visualize this data might be to generate a heat map. There is a fairly popular heatmap plugin for QGIS which I've seen a lot of people use. However, I didn't think I had enough "data" to warrant the effort.

<a name = "chapter6"></a>

# What's next?

Well, considering the likes data is simulated, the next natural steps for this project would be to flesh it out as an app and get it approved. Furthermore, there are no controls for adhereing to Instagram's ratelimit. The API doesn't provide a way to check for rate limit, and it seems as if Instagram has undergone a lot of changes recently regarding how many queries they are actually allowing developers. 

One way to do that would be creating a counter based on whatever instagram's rate limit actually is. `out_data = requests.get(request_url).json()` and the equivalent code for each of the other types of queries would need to be gated according to a global counter. You'd run into the same problems as discussed in [Twitter - Social Network Analysis 1](https://chigg.github.io/Twitter-SNA/). Ultimately, you wouldn't be able to detect API ratelimit changes dynamically, which means if the ratelimit changed, you'd need to alter the counter manually.

Considering the scope of this project, however, I don't think it's really necessary. We don't really query that much at all during data collection while looking for locations, and we don't query at all when getting average likes.

One way to improve the location collection would be to define gps coordinates for map corners. Then you could subtract or add values to iterate through those corners as if you were hitting every intersection on a grid. You'd then stop to pull locations at every intersection.

It was a lot of fun to explore QGIS and Instagram's API.

As always, here's the git repo for this project: <https://github.com/Chigg/GEO_IG>