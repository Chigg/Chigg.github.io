---
layout: post
title: Twitter - Social Network Analysis 1
featured-img: sleek
---
# Why Twitter?

Twitter is interesting for social network analysis because it allows for bilateral and unilateral connections. Unlike on social media sites like Facebook and Linkedin, users can follow others without inherent reciprocity. This provides deeper context about the connections generated within the service.

### Table of Contents

##### Theory
1. [What is Social Network Analysis?](#chapter1)

##### Technical
2. [Connecting to the API](#chapter2)
3. [Determining the Objective](#chapter3)
4. [Get Followers](#chapter4)
5. [Check Connectivity](#chapter5)

<a name="chapter1"></a>
# What is Social Network Analysis? 

A "social network," while these days synonymous for a *social media* site such as Twitter or Facebook, was originally a sociological concept. A social network analysis (SNA) investigates the layout of a social system's relationships / ties. 

Imagine a nuclear family's structure. You know your father, your father knows your mother, your mother knows you and so on and so forth. This would likely be a rather organized social network with evenly spaced ties. Although not very interesting on it's own, placing that network within a more complex social system would allow us to generate more interesting insights about that family and the individual nodes / members of that family. Social network analysis has been used for targeted advertising, law enforcement, and to study disease transmission.

Since this is only intended to be a primer for the purposes of this project, I'll only discuss a few relevant topics of importance in SNA before moving on to the code.

### Centrality

Centrality refers to the total number of ties an individual (node) has with other nodes. For the purposes of this project, since we are studying an individual's Twitter connections, the **most** central node is always going to be the user since the scope of our study is limited to their followers. The most central node is called the root. Nodes with higher centrality are positioned closer to the root, and will have the most mutual connections with the root user.

### Cliques

Cliques are sections of the social network with a high number of mutual connections. An example of a clique would be the nuclear family mentioned above. Cliques are important when determining groups. A social network without cliques isn't very insightful. 

### Bridging ties

Bridging ties are a difficult concept to succinctly describe. In essence, theoretically, they are ties that are not homophilous to the studied node. Mark Granovetter is a prominent SNA theorist within sociology, and in his book [Getting a Job: A Study of Contacts and Careers](https://www.amazon.com/Getting-Job-Study-Contacts-Careers/dp/0226305813) he positions bridging ties as sources of novel information. Whereas the people you are generally close to are similar to you and thereby reinforce your current social status, bridging ties are generally people in different social circles. Therefore, they're sources of new (novel) information. Novel information can be anything from new opportunities to means of elevating one's social status. While important in the overall science of social networks, we'll see that on Twitter, sometimes they're difficult to locate. 

# Connecting to the API <a name="chapter2"></a>

In order to access the data from a particular social media site, we need to communicate with that service. To do this, we use an API, which has site specific documentation. Python has a really nice library for Twitter's API called Tweepy that makes this conversation a breeze. First, you'll need to register your twitter account as a developer in order to get the keys necessary to access the API. You can use these keys for any number of interesting applications, and the Twitter application management platform is fairly intuitive.

Below is the initial set up for the program

```python
#import Tweepy
#documentation at http://www.tweepy.org/
import tweepy

#we'll use these later
import csv
import time

#since we're posting this code online, we don't want to give out our keys
#they're how we access our application
keys = []

#so I made a simple text file called keys and pasted each of the 4 keys in new lines
#and placed it on the Desktop. Find a better filepath.
inFile = open("/home/colin/Desktop/keys", "r")
for line in inFile:
    line = line.strip()
    keys.append(line)

consumer_key = str(keys[0])
consumer_secret = str(keys[1])
access_key = str(keys[2])
access_secret = str(keys[3])

#this block authenticates us with Twitter
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_key, access_secret)
api = tweepy.API(auth)
```

To play around with SNA, this set up is ideal because it allows us to call our keys from multiple applications using the same lines of code. This is intuitive as long as the keys.txt file we created remains in the same place. Since we're really only using this locally, registering multiple applications doesn't really make sense. If you were using this on a website for a live feed, you'd probably want to register a new application for the sake of tracking its use.

# Determining the Objective <a name="chapter3"></a>

To make a SNA, we need to determine the number of dimensions we want to study. There are a certain level of degrees that we can consider when looking for followers. For the sake of this project, we're looking at 2nd degree connections. This means for every follower, we'll need to check if any other follower has a mutual connection in order to determine interconnectivity. It is important to consider resource cost when determining the number of degrees we're looking through. If we were looking at the 7th degree connections, we might find Kevin Bacon but would also have to sift through millions of nodes to do so. Knowing this objective is important when outlining our code's structure. So firstly, we need to find out where we are starting. For this project, I want to look at bots to determine whether their networks appear structurally different than an organically formed user's network. To that end, [I found a list of users who auto-retweet](https://github.com/Plazmaz/Twitter-Bots-List). Since we're using a list, we'll need a for loop that iterates through each entry in that list.

```python
nameFile = open("/home/higgins_colin/tweepy_followers/usernames", "r")
        
        for username in nameFile:
                centrality_list = []

                #we'll talk about this line below
                data = api.rate_limit_status()
                x = data['resources']['followers']['/followers/ids']
                rate_limit_left = x['remaining']

                #each username gets stripped so we insure it's a self contain value
                username = username.strip()
```

Twitter's rate limit also becomes a significant factor when determining our project's complexity. For the free developer access, there's a 15 call per 15 minute limit for collecting a user's followers. Which means whenever we query the API 15 times, we have to wait 900 seconds before we make another call. There are multiple cooldowns for different types of queries, and the number of queries you can make depends on what type of query it is. So to be most efficient, we'd alternate between queries while the other is sleeping. The ratelimit is atrocious and severely limits our investigative ability but we'll have to make lemonaid with our sour sour lemons. In `data = api.rate_limit_status()` we're making a variable called data and applying the api object to it. from there, `x = data['resources']['followers']['/followers/ids']` narrows that object to the followers query, while the rate_limit_left variable then takes the key 'remaining' to return the value pair of remaining queries.

```python
if(rate_limit_left == 1):
        #since the sleep function is going to take awhile, we want to post a current time for debugging purposes
        print("Follower ID check rate-limited at {0}: Waiting 15 mins".format(strftime("%H:%M:%S"), gmtime()))
        time.sleep(900)
        downtime += 900

            print(username)

            try:
                    ids = get_followers(username)

                    #if the user has less than 150 followers, continue
                    if len(ids) <= 150:
                        centrality_list, downtime = check_connectivity(ids, original_user)
                    else:
                        continue
            #if account privacy set to private
            #skip account and print this message
            except:
        print("THIS USERS ACCOUNT IS HIDDEN.")
```
Since we now keep track of the remaining queries we have, we can force our program to sleep whenever we approach the rate limit. If you hit the rate limit too many times, your API access can be restricted or banned. Hitting the rate limit isn't the only restriction we want to apply. This block of code insures that we also do not hit an errors. The API is only capable of accessing accounts' information whose privacy is set to public. The try / except clause makes sure that if we encounter a hidden or private account, our program skips it and continues as normal. In order to finish in a timely manner (since we are going through a rather large list of accounts) we only want to access accounts with 150 followers.

As you can tell by the above block, there are two functions. get_followers() and check_connectivity(). This corresponds to the two main actions we outlined in the first paragraph.

# Get_followers <a name="chapter4"></a>

The get_followers() function will take a supplied username and return a list of followers by twitter user ID. It's a fairly small block of code with it's main purpose is to query the API. Since we limited our scope to accounts with less than 150 followers, we only need to query it once to determine whether or not it falls within our scope. Luckily, Twitter gives us 500 IDs per query.

```python
def get_followers(username):

        ids = []
        i = 0

        #finds follower ids
        for page in tweepy.Cursor(api.followers_ids, screen_name = username).pages():

                ids.extend(page)
                i += 1
                print("getting followers: %s" %(len(ids)))

                #only collect up to 1000
                if len(ids) >= 500:
                                break

                #3 queries then 5 min break, 500 users per query
                if i == 3:
                        print("Avoiding rate_limit. Wait 5 mins.")
                        time.sleep(300)
                        i = 0

        #prints total gathered followers
        print("Follower count gathered: ", len(ids))

return(ids)
```

If we wanted to increase the scope of our project, we would only need to increase `if len(ids) >= 500:` in chunks of 500 that correspond to the number of followers we're interested in. As you can see, in this block I have the time.sleep() timer broken up into chunks of 5 minutes. You can similarly implement it like this in all of the other instances. 

This is also an artificial means of adhering to the API rate limit, but it's technically less resource intensive since we don't have to query the API every time we want to make a different query. Although Twitter doesn't rate_limit the API rate_limit_remaining checks (confusing wording, I know) it's just another way of doing it that I wanted to illustrate. This is dangerous because if Twitter changes the rate limit, we don't automatically check with the service. I would only advise doing this if you're doing a smaller, quicker, study.

# Check_connectivity <a name="chapter5"></a>

The check_connectivity() function is a little more complex. In essence, we want to write to a new .csv for every new user's network we're studying.

```python
#since we created an original_user variable in the original for loop
#we can use that here to create our .csv
print ("writing to {0}_followers_tweets.csv".format(original_user))
        with open("{0}_followers.csv".format(original_user) , 'w') as file:
            writer = csv.writer(file, delimiter= '|')

            #start with original user's followers' ids
            out_list = ids
            #this adds original_user at position 0 of the out_list of ids
            out_list.insert(0, original_user)
            
            #write this row to the csv
            writer.writerow(out_list)
            #and clear list to prepare for new rows
            out_list = []

            #now that we wrote the original user row, we can remove original_user
            #from the check since we already know the ids are friends
            ids.remove(original_user)
```
Now we have to compare every node in that list of ids to every other node in that list of ids. So we have to create a double for loop. Here we also establish a counter called num_mutual which we will explain further later.

```python
for root in ids:
        out_list = []
        out_list.insert(0, root)
        print("follower ", i+1, " of ", len(ids))
        num_mutual = 0

    for node in ids:
        rate_count += 1

        if root == node:
                continue
```

We also have a `while True` loop that sustains the loop until we collect all of the information we need. Once the `try:` succeeds, we `break` the `while True` loop. When we hit the exception, we `continue` until the `try` succeeds. The try exception clause notifies us when we hit an error by generating a `tweepy.TweepError as e`. When we print e.reason, it gives us the error the API returned. With this code, e.reason would eventually print "rate limit exceeded" as an error because we don't adhere to the rate limit using the rate_limit dictionary for friendships. Try fixing this for yourself.

*Hint: The above solution in the Determining the Objective section of this tutorial would work if altered for the api.show_friendship query.*
Alternatively, if you just want solution, refer to the github repository at the end of this tutorial for the completed project.

```python
#this loop is within the double for loop
while True:
        try:
            #check if user is friends
            out = (api.show_friendship(source_id = root, target_id = node))

            #rate_count = num of queries
            #this if statement attempts to prevent hitting rate limit
            if rate_count == 150:
                    print("Avoiding rate Limit at {0}: Waiting 15 mins".format(strftime("%H:%M:%S"), gmtime()))
                    time.sleep(900)
                    downtime += 900
                    rate_count = 0

        except tweepy.TweepError as e:
                print("you hit an error!:")
                print(e.api_code)
                print(e.reason)
                print("Waiting 15 mins".format(strftime("%H:%M:%S"), gmtime()))
                time.sleep(900)
                downtime += 900
                continue
        break
```
api.show_friendship() takes the two nodes we are comparing and supplies a bunch of information about their relationship (for example, whether they've blocked one another). Pertinent to this project, it shows in what order they are following each other. If node is following root, `out[0].following` has to be true. Thus, this block would go within the double `for` loop we established above, since we have to check for every node.

```python
#if user is following = true
if(out[0].following):
        num_mutual += 1
        print("Bing")
        out_list.append(node)
```

We only append nodes that are following root, because when we generate our final .csv file, each filled row's cell will correspond with a line between the filled cell's ID and the ID value at column 0. It sounds confusing now, but it'll be a lot simpler when you see it.

Finally, num_mutual is the index for which we will understand connectivity. Nodes with a higher index of connectivity will have a higher centrality than those with less. If a user has no other mutual connections, their centrality value is 0.0. Each user has a variable called num_mutual which recieves += 1 whenever they share a mutual connection. To establish a unit of centrality, I divided num_mutual by the total possible number of mutual connections, len(ids). Thereby, the highest possible unit of centrality is 1.

To end the function, I return both centrality_list (a list of every user's unit of centrality) and downtime. Downtime is a value I created to analyize the projects idle time. Whenever we call a time.sleep() we also add the number of seconds the project is asleep to the downtime. This will allow us to tweak the code in order to reduce downtime, and also analyze how much time we have the server running without it being in use. 

The primary function of check_connectivity() is to write a row of each root and the corresponding mutual connections in each column. This is done by `writer.writerow(out_list)` after the `for node in ids:` loop is finished.

This ends the code tutorial for Twitter SNA. 