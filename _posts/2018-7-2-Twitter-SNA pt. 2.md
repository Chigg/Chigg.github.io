---
layout: post
title: Twitter - Social Network Analysis 2
featured-img: shane-rounce-205187
---

### Table of Contents

##### Technical
1. [The Data](#chapter1)
2. [Tools](#chapter2)
3. [The Hypothesis](#chapter3)
4. [The Results](#chapter4)

<a name="chapter1"></a>
# The Data

As of right now, the data from this project took around 13 days to gather. Much of that time was spent, however, in limbo. Unfortunately when hosting this project on an IaaS platform like Google's compute engine, the IP address from your queries would be indistinguishable from another platform user. 

Meaning if multiple people are querying Twitter's API from the same IP address at the same time, there's a high liklihood that those queries will get blocked. Unfortunately it was my only option, but there are a couple of ways you could solve this.

Perhaps the simplest and cheapest way, would be to invest in a [Raspberry Pi.](https://www.amazon.com/Raspberry-Pi-RASPBERRYPI3-MODB-1GB-Model-Motherboard/dp/B01CD5VC92) The programs execution isn't hardware intensive. For much of the project's runtime, the smallest possible VM's CPU (g1-small (1 vCPU, 1.7 GB memory)) was only executing at around .5% or less CPU with occasional blips when I would edit code. The only real limiting factor for running this project is a consistent network connection.
<a name = "chapter2"></a>

# Tools




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

As you can tell, since I looked at the documentation, I know the output is going to be formated as ['data'][i][*(value)*] so I'm pulling that and assigning it to a variable according to what it is. I'm storing the data in a list of lists, so temp is only for variables for this iteration, and product is the complete output that gets returned.

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
   height: inherit;
   width: inherit;
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

Well, considering the likes data is simulated, the next natural steps for this project would be to flesh it out as an app and get it approved. Furthermore, there are no controls for adhereing to Instagram's ratelimit. The API doesn't provide a way to check for rate limit, and it seems as if Instagram has undergone a lot of changes recently regarding how many queries they are actually allowing developers. There's a lot of misinformation currently about how many queries there are. 

One way to adhere to the ratelimit would be creating a counter based on whatever instagram's rate limit actually is. `out_data = requests.get(request_url).json()` and the equivalent code for each of the other types of queries would need to be gated according to a global counter. You'd run into the same problems as discussed in [Twitter - Social Network Analysis 1](https://chigg.github.io/Twitter-SNA/). Ultimately, you wouldn't be able to detect API ratelimit changes dynamically, which means if the ratelimit changed, you'd need to alter the counter manually.

Considering the scope of this project, however, I don't think it's really necessary. We don't really query that much at all during data collection while looking for locations, and we don't query at all when getting average likes.

One way to improve the location collection would be to define gps coordinates for map corners. Then you could subtract or add values to iterate through those corners as if you were hitting every intersection on a grid. You'd then stop to pull locations at every intersection.

It was a lot of fun to explore QGIS and Instagram's API.

As always, here's the git repo for this project: <https://github.com/Chigg/GEO_IG>