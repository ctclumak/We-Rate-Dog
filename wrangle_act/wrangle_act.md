
## We Rate Dogs 
### Introduction

This project is part of Udaciy Data Wrangling and Visulization project. I use Python and its libraries to gather data from a variety of sources to analyze and visulize the tweet archive of Twitter user @dog_rates, also known as WeRateDogs, which is a Twitter account that rates people's dogs with a humorous comment about the dog. The data comes in a variety of formats, I assess its quality and tidiness, then clean it to analyze and visulize the key findings.

### The Data Wrangling Process
**Gather**.Data is gathered from three sources

- Enhanced Twitter Archive: This is a csv file provided by Udacity. The archive contains each tweet's text, which Udacity used to enhance by extracting rating, dog name, and dog 'stage' (doggo, floofer, pupper, and puppo).
- Image Predictions File: a table full of image predictions (the top three only) alongside each tweet ID, image URL, and the image number that corresponded to the most confident prediction (numbered 1 to 4 since tweets can have up to four images). The Requests library is used to downloaded the file programmatically from Udcity's servers.
- Additional Data via the Twitter API,including twitter ID, retweet count and favorite count. I use the requests library to download the json.txt file provided by Udacity its servers.

**Assess**. Data is inspected for quality (content) issues and tidiness (structural)issues. 
- Low quality (‘dirty’) data has content issues such as missing, invalid, inaccurate, and inconsistent data. I assessed removing unnecessary columns, converting data types, making dog names title case, removing entries that were not really dogs.
- Untidy (‘messy’) data has structural issues. I assessed gathering dog stages from multiple columns into one, and combining the three datasets into one.

**Clean**. I first addressed missing data, then structural issues, then quality issues. 

For this project, we only want original ratings (no retweets) that have images.The requirements of this project are to assess and clean at least 8 quality and 2 tidiness issues in this dataset.Cleaning includes removing missing value, creating new columns, merging individual pieces of data.






```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import requests
import json
import tweepy
import time
import datetime as dt
%matplotlib inline
```

**Gather**

a. Read in csv file as pandas dataframe and quick check to view structure


```python
twitter_archive = pd.read_csv("twitter-archive-enhanced.csv")
twitter_archive.head()
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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421...</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



b. Use Requests library to programmatically download tsv file from a website and view structure


```python

# import tweet image   
import requests
url1 = "https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv"

response = requests.get(url1, allow_redirects=True)
open('tweet_json.txt', 'wb').write(response.content)

#the data is store in the computer response library, we can access the data with .content
#save html to file
with open("image_predictions.tsv", 'wb') as file:
    for chunk in response.iter_content(chunk_size=128):
        file.write(response.content)
#open the file        
image_predictions = pd.read_csv("image_predictions.tsv", sep= "\t")

# view structure
image_predictions.head()
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
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>666020888022790149</td>
      <td>https://pbs.twimg.com/media/CT4udn0WwAA0aMy.jpg</td>
      <td>1</td>
      <td>Welsh_springer_spaniel</td>
      <td>0.465074</td>
      <td>True</td>
      <td>collie</td>
      <td>0.156665</td>
      <td>True</td>
      <td>Shetland_sheepdog</td>
      <td>0.0614285</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>666029285002620928</td>
      <td>https://pbs.twimg.com/media/CT42GRgUYAA5iDo.jpg</td>
      <td>1</td>
      <td>redbone</td>
      <td>0.506826</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.07419169999999999</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.07201</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666033412701032449</td>
      <td>https://pbs.twimg.com/media/CT4521TWwAEvMyu.jpg</td>
      <td>1</td>
      <td>German_shepherd</td>
      <td>0.596461</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.13858399999999998</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.11619700000000001</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>666044226329800704</td>
      <td>https://pbs.twimg.com/media/CT5Dr8HUEAA-lEu.jpg</td>
      <td>1</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.408143</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.360687</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.222752</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>666049248165822465</td>
      <td>https://pbs.twimg.com/media/CT5IQmsXIAAKY4A.jpg</td>
      <td>1</td>
      <td>miniature_pinscher</td>
      <td>0.560311</td>
      <td>True</td>
      <td>Rottweiler</td>
      <td>0.243682</td>
      <td>True</td>
      <td>Doberman</td>
      <td>0.154629</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



c. Use Requests library to programmatically download json file from the services


```python
url1 = 'https://s3.amazonaws.com/video.udacity-data.com/topher/2018/November/5be5fb7d_tweet-json/tweet-json.txt'

r = requests.get(url1, allow_redirects=True)
open('tweet-json.txt', 'wb').write(r.content)

tweets_data = []
tweets_file = open('tweet-json.txt', "r")
for line in tweets_file:
    try:
        tweet= json.loads(line)
        tweets_data.append(tweet)
    except:
        continue
```

d. Create dataframe, extract "tweet_id", "retweet_count", "favorite_count", populate the data, and view structure.


```python
tweet_json = pd.DataFrame()
tweet_json['tweet_id'] = list(map(lambda tweet: tweet['id'], tweets_data))
tweet_json['retweet_count'] = list(map(lambda tweet: tweet['retweet_count'], tweets_data))
tweet_json['favorite_count'] = list(map(lambda tweet: tweet['favorite_count'], tweets_data))

tweet_json.head()

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
      <th>tweet_id</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>8853</td>
      <td>39467</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>6514</td>
      <td>33819</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>4328</td>
      <td>25461</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>8964</td>
      <td>42908</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>9774</td>
      <td>41048</td>
    </tr>
  </tbody>
</table>
</div>



**Access**

Dataframe Summary:
- twitter_achieve
- image_predictions
- tweet_json

e. Find out the size of each data set


```python
print('  archive tweet count = ' + str(len(twitter_archive)))
print('   images tweet count = ' + str(len(image_predictions)))
print('tweet_json tweet count = ' + str(len(tweet_json)))
```

      archive tweet count = 2356
       images tweet count = 5434967
    tweet_json tweet count = 2354


f. Access the structure of each data set


```python
twitter_archive.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2356 entries, 0 to 2355
    Data columns (total 17 columns):
    tweet_id                      2356 non-null int64
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2356 non-null object
    source                        2356 non-null object
    text                          2356 non-null object
    retweeted_status_id           181 non-null float64
    retweeted_status_user_id      181 non-null float64
    retweeted_status_timestamp    181 non-null object
    expanded_urls                 2297 non-null object
    rating_numerator              2356 non-null int64
    rating_denominator            2356 non-null int64
    name                          2356 non-null object
    doggo                         2356 non-null object
    floofer                       2356 non-null object
    pupper                        2356 non-null object
    puppo                         2356 non-null object
    dtypes: float64(4), int64(3), object(10)
    memory usage: 313.0+ KB



```python
image_predictions.describe()
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
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
      <td>5434967</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>2076</td>
      <td>2010</td>
      <td>5</td>
      <td>379</td>
      <td>2007</td>
      <td>3</td>
      <td>406</td>
      <td>2005</td>
      <td>3</td>
      <td>409</td>
      <td>2007</td>
      <td>3</td>
    </tr>
    <tr>
      <th>top</th>
      <td>679462823135686656</td>
      <td>https://pbs.twimg.com/ext_tw_video_thumb/67535...</td>
      <td>1</td>
      <td>golden_retriever</td>
      <td>0.806757</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.0693617</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.0461082</td>
      <td>True</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>2618</td>
      <td>5236</td>
      <td>4660040</td>
      <td>392700</td>
      <td>5236</td>
      <td>4010776</td>
      <td>272272</td>
      <td>7854</td>
      <td>4065754</td>
      <td>206822</td>
      <td>5236</td>
      <td>3924382</td>
    </tr>
  </tbody>
</table>
</div>




```python
tweet_json.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2354 entries, 0 to 2353
    Data columns (total 3 columns):
    tweet_id          2354 non-null int64
    retweet_count     2354 non-null int64
    favorite_count    2354 non-null int64
    dtypes: int64(3)
    memory usage: 55.2 KB


***twitter achieve***


```python
twitter_archive.name
```




    0        Phineas
    1          Tilly
    2         Archie
    3          Darla
    4       Franklin
    5           None
    6            Jax
    7           None
    8           Zoey
    9         Cassie
    10          Koda
    11         Bruno
    12          None
    13           Ted
    14        Stuart
    15        Oliver
    16           Jim
    17          Zeke
    18       Ralphus
    19        Canela
    20        Gerald
    21       Jeffrey
    22          such
    23        Canela
    24          None
    25          None
    26          Maya
    27        Mingus
    28         Derek
    29        Roscoe
              ...   
    2326       quite
    2327           a
    2328        None
    2329        None
    2330        None
    2331        None
    2332        None
    2333          an
    2334           a
    2335          an
    2336        None
    2337        None
    2338        None
    2339        None
    2340        None
    2341        None
    2342        None
    2343        None
    2344        None
    2345         the
    2346         the
    2347           a
    2348           a
    2349          an
    2350           a
    2351        None
    2352           a
    2353           a
    2354           a
    2355        None
    Name: name, Length: 2356, dtype: object




```python
twitter_archive.rating_denominator.value_counts()
```




    10     2333
    11        3
    50        3
    80        2
    20        2
    2         1
    16        1
    40        1
    70        1
    15        1
    90        1
    110       1
    120       1
    130       1
    150       1
    170       1
    7         1
    0         1
    Name: rating_denominator, dtype: int64




```python
twitter_archive[twitter_archive["expanded_urls"].duplicated()].count()
```




    tweet_id                      137
    in_reply_to_status_id          54
    in_reply_to_user_id            54
    timestamp                     137
    source                        137
    text                          137
    retweeted_status_id             1
    retweeted_status_user_id        1
    retweeted_status_timestamp      1
    expanded_urls                  79
    rating_numerator              137
    rating_denominator            137
    name                          137
    doggo                         137
    floofer                       137
    pupper                        137
    puppo                         137
    dtype: int64




```python
twitter_archive.rating_numerator.describe()
```




    count    2356.000000
    mean       13.126486
    std        45.876648
    min         0.000000
    25%        10.000000
    50%        11.000000
    75%        12.000000
    max      1776.000000
    Name: rating_numerator, dtype: float64




```python
twitter_archive.rating_denominator.describe()
```




    count    2356.000000
    mean       10.455433
    std         6.745237
    min         0.000000
    25%        10.000000
    50%        10.000000
    75%        10.000000
    max       170.000000
    Name: rating_denominator, dtype: float64



**Key Findings**

tweet_json has 2354 entries, which is the smallest among the three datasets.Image_predictions has over 50k entries. 
- The following columns of tweet_achieve have missing data: in_reply_to_status_id,in_reply_to_user_id,retweeted_status_id,retweeted_status_user_id and retweeted_status_timestamp.
- Some columns have wrong data type, such as timestamp,rating_numerator, id. The rating numerator is in an integer in the dataset, it should be a float, also some of the text shows the ratting are decimals.
- The text and url columns have cut off, need to adjust the column width to fit the text.
- The dog stages are in different columns, should be groupped into one column
- There are missing value in name columns of twitter_achieve file. Not all entries appear to be the correct names nor are they in title case, will clean after merging the datasets
- The data source columns of image_prediction file shows the url link and link text, which is hard to read and need to tidy the data in a readable way.
- The rating denominator should have number of 10, but there are some outliers since the max is 170
- The interquartile range of the numerator rating is between 10 and 12. Since the max is 1776 there are likely outliers.
- 137 twitter id have duplicated expanded urls

***Image Predictions***


```python
list(image_predictions[image_predictions.jpg_url.duplicated()].tweet_id)
```




    ['752309394570878976',
     '754874841593970688',
     '757729163776290825',
     '759159934323924993',
     '759566828574212096',
     '761371037149827077',
     '761750502866649088',
     '766078092750233600',
     '770093767776997377',
     '771171053431250945',
     '772615324260794368',
     '775898661951791106',
     '776819012571455488',
     '777641927919427584',
     '778396591732486144',
     '780496263422808064',
     '782021823840026624',
     '783347506784731136',
     '786036967502913536',
     '788070120937619456',
     '790723298204217344',
     '791026214425268224',
     '793614319594401792',
     '794355576146903043',
     '794983741416415232',
     '796177847564038144',
     '798340744599797760',
     '798628517273620480',
     '798644042770751489',
     '798665375516884993',
     '798673117451325440',
     '798694562394996736',
     '798697898615730177',
     '799774291445383169',
     '800443802682937345',
     '802247111496568832',
     '802624713319034886',
     '803692223237865472',
     '804413760345620481',
     '805958939288408065',
     '806242860592926720',
     '807059379405148160',
     '808134635716833280',
     '809808892968534016',
     '813944609378369540',
     '816014286006976512',
     '816829038950027264',
     '817181837579653120',
     '818588835076603904',
     '819015331746349057',
     '819015337530290176',
     '820446719150292993',
     '821813639212650496',
     '822647212903690241',
     '823269594223824897',
     '824796380199809024',
     '829878982036299777',
     '832040443403784192',
     '832215726631055365',
     '841833993020538882',
     '842892208864923648',
     '851953902622658560',
     '861769973181624320',
     '873697596434513921',
     '885311592912609280',
     '888202515573088257',
     '666020888022790149',
     '666029285002620928',
     '666033412701032449',
     '666044226329800704',
     '666049248165822465',
     '666050758794694657',
     '666051853826850816',
     '666055525042405380',
     '666057090499244032',
     '666058600524156928',
     '666063827256086533',
     '666071193221509120',
     '666073100786774016',
     '666082916733198337',
     '666094000022159362',
     '666099513787052032',
     '666102155909144576',
     '666104133288665088',
     '666268910803644416',
     '666273097616637952',
     '666287406224695296',
     '666293911632134144',
     '666337882303524864',
     '666345417576210432',
     '666353288456101888',
     '666362758909284353',
     '666373753744588802',
     '666396247373291520',
     '666407126856765440',
     '666411507551481857',
     '666418789513326592',
     '666421158376562688',
     '666428276349472768',
     '666430724426358785',
     '666435652385423360',
     '666437273139982337',
     '666447344410484738',
     '666454714377183233',
     '666644823164719104',
     '666649482315059201',
     '666691418707132416',
     '666701168228331520',
     '666739327293083650',
     '666776908487630848',
     '666781792255496192',
     '666786068205871104',
     '666804364988780544',
     '666817836334096384',
     '666826780179869698',
     '666835007768551424',
     '666837028449972224',
     '666983947667116034',
     '666996132027977728',
     '667012601033924608',
     '667044094246576128',
     '667062181243039745',
     '667065535570550784',
     '667073648344346624',
     '667090893657276420',
     '667119796878725120',
     '667138269671505920',
     '667152164079423490',
     '667160273090932737',
     '667165590075940865',
     '667171260800061440',
     '667174963120574464',
     '667176164155375616',
     '667177989038297088',
     '667182792070062081',
     '667188689915760640',
     '667192066997374976',
     '667200525029539841',
     '667211855547486208',
     '667369227918143488',
     '667393430834667520',
     '667405339315146752',
     '667435689202614272',
     '667437278097252352',
     '667443425659232256',
     '667453023279554560',
     '667455448082227200',
     '667470559035432960',
     '667491009379606528',
     '667495797102141441',
     '667502640335572993',
     '667509364010450944',
     '667517642048163840',
     '667524857454854144',
     '667530908589760512',
     '667534815156183040',
     '667538891197542400',
     '667544320556335104',
     '667546741521195010',
     '667549055577362432',
     '667550882905632768',
     '667550904950915073',
     '667724302356258817',
     '667728196545200128',
     '667766675769573376',
     '667773195014021121',
     '667782464991965184',
     '667793409583771648',
     '667801013445750784',
     '667806454573760512',
     '667832474953625600',
     '667861340749471744',
     '667866724293877760',
     '667873844930215936',
     '667878741721415682',
     '667885044254572545',
     '667886921285246976',
     '667902449697558528',
     '667911425562669056',
     '667915453470232577',
     '667924896115245057',
     '667937095915278337',
     '668113020489474048',
     '668142349051129856',
     '668154635664932864',
     '668171859951755264',
     '668190681446379520',
     '668204964695683073',
     '668221241640230912',
     '668226093875376128',
     '668237644992782336',
     '668248472370458624',
     '668256321989451776',
     '668268907921326080',
     '668274247790391296',
     '668286279830867968',
     '668291999406125056',
     '668297328638447616',
     '668466899341221888',
     '668480044826800133',
     '668484198282485761',
     '668496999348633600',
     '668507509523615744',
     '668528771708952576',
     '668537837512433665',
     '668542336805281792',
     '668544745690562560',
     '668567822092664832',
     '668614819948453888',
     '668620235289837568',
     '668623201287675904',
     '668625577880875008',
     '668627278264475648',
     '668631377374486528',
     '668633411083464705',
     '668636665813057536',
     '668641109086707712',
     '668643542311546881',
     '668645506898350081',
     '668655139528511488',
     '668779399630725120',
     '668815180734689280',
     '668826086256599040',
     '668852170888998912',
     '668872652652679168',
     '668892474547511297',
     '668902994700836864',
     '668932921458302977',
     '668955713004314625',
     '668960084974809088',
     '668975677807423489',
     '668979806671884288',
     '668981893510119424',
     '668986018524233728',
     '668988183816871936',
     '668989615043424256',
     '668992363537309700',
     '668994913074286592',
     '669000397445533696',
     '669006782128353280',
     '669015743032369152',
     '669037058363662336',
     '669203728096960512',
     '669214165781868544',
     '669216679721873412',
     '669324657376567296',
     '669327207240699904',
     '669328503091937280',
     '669351434509529089',
     '669353438988365824',
     '669354382627049472',
     '669359674819481600',
     '669363888236994561',
     '669367896104181761',
     '669371483794317312',
     '669375718304980992',
     '669393256313184256',
     '669564461267722241',
     '669567591774625800',
     '669571471778410496',
     '669573570759163904',
     '669583744538451968',
     '669597912108789760',
     '669603084620980224',
     '669625907762618368',
     '669661792646373376',
     '669680153564442624',
     '669682095984410625',
     '669683899023405056',
     '669749430875258880',
     '669753178989142016',
     '669923323644657664',
     '669926384437997569',
     '669942763794931712',
     '669970042633789440',
     '669972011175813120',
     '669993076832759809',
     '670003130994700288',
     '670037189829525505',
     '670040295598354432',
     '670046952931721218',
     '670055038660800512',
     '670061506722140161',
     '670069087419133954',
     '670073503555706880',
     '670079681849372674',
     '670086499208155136',
     '670093938074779648',
     '670290420111441920',
     '670303360680108032',
     '670319130621435904',
     '670338931251150849',
     '670361874861563904',
     '670374371102445568',
     '670385711116361728',
     '670403879788544000',
     '670408998013820928',
     '670411370698022913',
     '670417414769758208',
     '670420569653809152',
     '670421925039075328',
     '670427002554466305',
     '670428280563085312',
     '670433248821026816',
     '670434127938719744',
     '670435821946826752',
     '670442337873600512',
     '670444955656130560',
     '670449342516494336',
     '670452855871037440',
     '670465786746662913',
     '670468609693655041',
     '670474236058800128',
     '670668383499735048',
     '670676092097810432',
     '670679630144274432',
     '670691627984359425',
     '670704688707301377',
     '670717338665226240',
     '670727704916926465',
     '670733412878163972',
     '670755717859713024',
     '670764103623966721',
     '670778058496974848',
     '670780561024270336',
     '670782429121134593',
     '670783437142401025',
     '670786190031921152',
     '670789397210615808',
     '670792680469889025',
     '670797304698376195',
     '670803562457407488',
     '670804601705242624',
     '670807719151067136',
     '670811965569282048',
     '670815497391357952',
     '670822709593571328',
     '670823764196741120',
     '670826280409919488',
     '670832455012716544',
     '670833812859932673',
     '670838202509447168',
     '670840546554966016',
     '670842764863651840',
     '670995969505435648',
     '671109016219725825',
     '671115716440031232',
     '671122204919246848',
     '671134062904504320',
     '671138694582165504',
     '671141549288370177',
     '671147085991960577',
     '671151324042559489',
     '671154572044468225',
     '671159727754231808',
     '671163268581498880',
     '671166507850801152',
     '671182547775299584',
     '671186162933985280',
     '671347597085433856',
     '671355857343524864',
     '671357843010908160',
     '671362598324076544',
     '671390180817915904',
     '671485057807351808',
     '671486386088865792',
     '671488513339211776',
     '671497587707535361',
     '671504605491109889',
     '671511350426865664',
     '671518598289059840',
     '671520732782923777',
     '671528761649688577',
     '671533943490011136',
     '671536543010570240',
     '671538301157904385',
     '671542985629241344',
     '671544874165002241',
     '671547767500775424',
     '671561002136281088',
     '671729906628341761',
     '671735591348891648',
     '671743150407421952',
     '671744970634719232',
     '671763349865160704',
     '671768281401958400',
     '671789708968640512',
     '671855973984772097',
     '671866342182637568',
     '671874878652489728',
     '671879137494245376',
     '671882082306625538',
     '671891728106971137',
     '671896809300709376',
     '672068090318987265',
     '672082170312290304',
     '672095186491711488',
     '672125275208069120',
     '672139350159835138',
     '672160042234327040',
     '672169685991993344',
     '672205392827572224',
     '672222792075620352',
     '672231046314901505',
     '672239279297454080',
     '672245253877968896',
     '672248013293752320',
     '672254177670729728',
     '672256522047614977',
     '672264251789176834',
     '672267570918129665',
     '672272411274932228',
     '672466075045466113',
     '672475084225949696',
     '672481316919734272',
     '672482722825261057',
     '672488522314567680',
     '672523490734551040',
     '672538107540070400',
     '672591271085670400',
     '672591762242805761',
     '672594978741354496',
     '672604026190569472',
     '672609152938721280',
     '672614745925664768',
     '672622327801233409',
     '672640509974827008',
     '672828477930868736',
     '672834301050937345',
     '672877615439593473',
     '672884426393653248',
     '672898206762672129',
     '672902681409806336',
     '672964561327235073',
     '672968025906282496',
     '672970152493887488',
     '672975131468300288',
     '672980819271634944',
     '672984142909456390',
     '672988786805112832',
     '672995267319328768',
     '672997845381865473',
     '673148804208660480',
     '673213039743795200',
     '673240798075449344',
     '673270968295534593',
     '673295268553605120',
     '673317986296586240',
     '673320132811366400',
     '673342308415348736',
     '673343217010679808',
     '673345638550134785',
     '673350198937153538',
     '673352124999274496',
     '673355879178194945',
     '673359818736984064',
     '673363615379013632',
     '673576835670777856',
     '673580926094458881',
     '673583129559498752',
     '673612854080196609',
     '673636718965334016',
     '673656262056419329',
     '673662677122719744',
     '673680198160809984',
     '673686845050527744',
     '673688752737402881',
     '673689733134946305',
     '673697980713705472',
     '673700254269775872',
     '673705679337693185',
     '673707060090052608',
     '673708611235921920',
     '673709992831262724',
     '673711475735838725',
     '673715861853720576',
     '673887867907739649',
     '673906403526995968',
     '673919437611909120',
     '673956914389192708',
     '674008982932058114',
     '674014384960745472',
     '674019345211760640',
     '674024893172875264',
     '674036086168010753',
     '674038233588723717',
     '674042553264685056',
     '674045139690631169',
     '674051556661161984',
     '674053186244734976',
     '674063288070742018',
     '674075285688614912',
     '674082852460433408',
     '674255168825880576',
     '674262580978937856',
     '674265582246694913',
     '674269164442398721',
     '674271431610523648',
     '674291837063053312',
     '674318007229923329',
     '674372068062928900',
     '674394782723014656',
     '674410619106390016',
     '674416750885273600',
     '674422304705744896',
     '674436901579923456',
     '674447403907457024',
     '674468880899788800',
     '674632714662858753',
     '674638615994089473',
     '674644256330530816',
     '674646392044941312',
     '674664755118911488',
     '674670581682434048',
     '674690135443775488',
     '674737130913071104',
     '674739953134403584',
     '674743008475090944',
     '674752233200820224',
     '674754018082705410',
     '674764817387900928',
     '674767892831932416',
     '674774481756377088',
     '674781762103414784',
     '674788554665512960',
     '674790488185167872',
     '674793399141146624',
     '674800520222154752',
     '674805413498527744',
     '674999807681908736',
     '675003128568291329',
     '675006312288268288',
     '675015141583413248',
     '675047298674663426',
     '675109292475830276',
     '675111688094527488',
     '675113801096802304',
     '675135153782571009',
     '675145476954566656',
     '675146535592706048',
     '675147105808306176',
     '675149409102012420',
     '675153376133427200',
     '675166823650848770',
     '675334060156301312',
     '675349384339542016',
     '675354435921575936',
     '675362609739206656',
     '675372240448454658',
     '675432746517426176',
     '675483430902214656',
     '675489971617296384',
     '675497103322386432',
     '675501075957489664',
     '675517828909424640',
     '675522403582218240',
     '675531475945709568',
     '675534494439489536',
     '675706639471788032',
     '675707330206547968',
     '675710890956750848',
     '675740360753160193',
     '675781562965868544',
     '675798442703122432',
     '675820929667219457',
     '675822767435051008',
     '675845657354215424',
     '675853064436391936',
     '675870721063669760',
     '675878199931371520',
     '675888385639251968',
     '675891555769696257',
     '675898130735476737',
     '676089483918516224',
     '676098748976615425',
     '676101918813499392',
     '676146341966438401',
     '676191832485810177',
     '676215927814406144',
     '676219687039057920',
     '676237365392908289',
     '676263575653122048',
     '676430933382295552',
     '676440007570247681',
     '676470639084101634',
     '676496375194980353',
     '676533798876651520',
     '676575501977128964',
     '676582956622721024',
     '676588346097852417',
     '676603393314578432',
     '676606785097199616',
     '676613908052996102',
     '676617503762681856',
     '676776431406465024',
     '676811746707918848',
     '676819651066732545',
     '676821958043033607',
     '676864501615042560',
     '676897532954456065',
     '676936541936185344',
     '676942428000112642',
     '676946864479084545',
     '676948236477857792',
     '676949632774234114',
     '676957860086095872',
     '676975532580409345',
     '677187300187611136',
     '677228873407442944',
     '677269281705472000',
     '677301033169788928',
     '677314812125323265',
     '677328882937298944',
     '677331501395156992',
     '677334615166730240',
     '677530072887205888',
     '677547928504967168',
     '677557565589463040',
     '677565715327688705',
     '677573743309385728',
     '677644091929329666',
     '677662372920729601',
     '677673981332312066',
     '677687604918272002',
     '677698403548192770',
     '677700003327029250',
     '677716515794329600',
     '677895101218201600',
     '677918531514703872',
     '678021115718029313',
     '678255464182861824',
     '678278586130948096',
     '678334497360859136',
     '678341075375947776',
     '678380236862578688',
     '678389028614488064',
     '678396796259975168',
     '678399652199309312',
     '678410210315247616',
     '678424312106393600',
     '678446151570427904',
     '678643457146150913',
     '678675843183484930',
     '678740035362037760',
     '678755239630127104',
     '678764513869611008',
     '678767140346941444',
     '678774928607469569',
     '678798276842360832',
     '678800283649069056',
     '678969228704284672',
     '678991772295516161',
     '679047485189439488',
     '679062614270468097',
     '679111216690831360',
     '679132435750195208',
     '679148763231985668',
     '679158373988876288',
     '679462823135686656',
     '679475951516934144',
     '679503373272485890',
     '679511351870550016',
     '679527802031484928',
     '679530280114372609',
     '679722016581222400',
     '679729593985699840',
     '679736210798047232',
     '679777920601223168',
     '679828447187857408',
     '679844490799091713',
     '679854723806179328',
     '679862121895714818',
     '679877062409191424',
     '680055455951884288',
     '680070545539371008',
     '680085611152338944',
     '680100725817409536',
     '680115823365742593',
     '680130881361686529',
     '680145970311643136',
     '680161097740095489',
     '680176173301628928',
     '680191257256136705',
     '680206703334408192',
     '680221482581123072',
     '680440374763077632',
     '680473011644985345',
     '680494726643068929',
     '680497766108381184',
     '680583894916304897',
     '680609293079592961',
     '680798457301471234',
     '680801747103793152',
     '680836378243002368',
     '680889648562991104',
     '680913438424612864',
     '680934982542561280',
     '680940246314430465',
     '680959110691590145',
     '680970795137544192',
     '681193455364796417',
     '681231109724700672',
     '681242418453299201',
     '681261549936340994',
     '681281657291280384',
     '681297372102656000',
     '681302363064414209',
     '681320187870711809',
     '681339448655802368',
     '681523177663676416',
     '681579835668455424',
     '681610798867845120',
     '681654059175129088',
     '681679526984871937',
     '681694085539872773',
     '681891461017812993',
     '681981167097122816',
     '682003177596559360',
     '682032003584274432',
     '682047327939461121',
     '682059653698686977',
     '682242692827447297',
     '682259524040966145',
     '682303737705140231',
     '682389078323662849',
     '682393905736888321',
     '682406705142087680',
     '682429480204398592',
     '682638830361513985',
     '682662431982772225',
     '682697186228989953',
     '682750546109968385',
     '682788441537560576',
     '682962037429899265',
     '683030066213818368',
     '683078886620553216',
     '683098815881154561',
     '683111407806746624',
     '683142553609318400',
     '683357973142474752',
     '683391852557561860',
     '683449695444799489',
     '683462770029932544',
     '683481228088049664',
     '683498322573824003',
     '683742671509258241',
     '683773439333797890',
     '683828599284170753',
     '683834909291606017',
     '683849932751646720',
     '683852578183077888',
     '683857920510050305',
     '684097758874210310',
     '684122891630342144',
     '684177701129875456',
     '684188786104872960',
     '684195085588783105',
     '684200372118904832',
     '684222868335505415',
     '684225744407494656',
     '684241637099323392',
     '684460069371654144',
     '684481074559381504',
     '684538444857667585',
     '684567543613382656',
     '684594889858887680',
     '684800227459624960',
     '684880619965411328',
     '684902183876321280',
     '684914660081053696',
     '684926975086034944',
     '684940049151070208',
     '684959798585110529',
     '685169283572338688',
     '685198997565345792',
     '685268753634967552',
     '685307451701334016',
     '685315239903100929',
     '685321586178670592',
     '685325112850124800',
     '685532292383666176',
     '685547936038666240',
     '685641971164143616',
     '685663452032069632',
     '685667379192414208',
     '685906723014619143',
     '685943807276412928',
     '685973236358713344',
     '686003207160610816',
     '686007916130873345',
     '686034024800862208',
     '686050296934563840',
     '686358356425093120',
     '686377065986265092',
     '686386521809772549',
     '686606069955735556',
     '686618349602762752',
     '686683045143953408',
     '686730991906516992',
     '686749460672679938',
     '686947101016735744',
     '687096057537363968',
     '687102708889812993',
     '687109925361856513',
     '687124485711986689',
     '687127927494963200',
     '687312378585812992',
     '687317306314240000',
     '687460506001633280',
     '687476254459715584',
     '687480748861947905',
     '687494652870668288',
     '687664829264453632',
     '687704180304273409',
     '687807801670897665',
     '687818504314159109',
     '687826841265172480',
     '688064179421470721',
     '688116655151435777',
     '688179443353796608',
     '688211956440801280',
     '688385280030670848',
     '688519176466644993',
     '688547210804498433',
     '688789766343622656',
     '688804835492233216',
     '688828561667567616',
     '688894073864884227',
     '688898160958271489',
     '688908934925697024',
     '688916208532455424',
     '689143371370250240',
     '689154315265683456',
     '689275259254616065',
     '689280876073582592',
     '689283819090870273',
     '689289219123089408',
     '689517482558820352',
     '689557536375177216',
     '689599056876867584',
     '689623661272240129',
     '689659372465688576',
     '689661964914655233',
     '689835978131935233',
     '689877686181715968',
     '689905486972461056',
     '689977555533848577',
     '689999384604450816',
     '690005060500217858',
     '690015576308211712',
     '690021994562220032',
     '690248561355657216',
     '690360449368465409',
     '690374419777196032',
     '690400367696297985',
     '690597161306841088',
     '690649993829576704',
     '690690673629138944',
     '690728923253055490',
     '690735892932222976',
     '690932576555528194',
     '690938899477221376',
     '690959652130045952',
     '691090071332753408',
     '691096613310316544',
     '691321916024623104',
     '691416866452082688',
     '691444869282295808',
     '691459709405118465',
     '691483041324204033',
     '691675652215414786',
     '691756958957883396',
     '691820333922455552',
     '692017291282812928',
     '692142790915014657',
     '692158366030913536',
     '692187005137076224',
     '692417313023332352',
     '692530551048294401',
     '692535307825213440',
     '692568918515392513',
     '692752401762250755',
     '692828166163931137',
     '692894228850999298',
     '692901601640583168',
     '692905862751522816',
     '692919143163629568',
     '693095443459342336',
     '693109034023534592',
     '693155686491000832',
     '693231807727280129',
     '693262851218264065',
     '693280720173801472',
     '693486665285931008',
     '693590843962331137',
     '693622659251335168',
     '693629975228977152',
     '693642232151285760',
     '693647888581312512',
     '693942351086120961',
     '694001791655137281',
     '694183373896572928',
     '694206574471057408',
     '694329668942569472',
     '694352839993344000',
     '694356675654983680',
     '694669722378485760',
     '694905863685980160',
     '695051054296211456',
     '695064344191721472',
     '695074328191332352',
     '695095422348574720',
     '695314793360662529',
     '695409464418041856',
     '695446424020918272',
     '695629776980148225',
     '695767669421768709',
     '695794761660297217',
     '695816827381944320',
     '696405997980676096',
     '696488710901260288',
     '696713835009417216',
     '696754882863349760',
     '696877980375769088',
     '696886256886657024',
     '696894894812565505',
     '696900204696625153',
     '697242256848379904',
     '697255105972801536',
     '697259378236399616',
     '697270446429966336',
     '697463031882764288',
     '697482927769255936',
     '697575480820686848',
     '697596423848730625',
     '697616773278015490',
     '697881462549430272',
     '697943111201378304',
     '697990423684476929',
     '697995514407682048',
     '698178924120031232',
     '698195409219559425',
     '698262614669991936',
     '698342080612007937',
     '698355670425473025',
     '698549713696649216',
     '698635131305795584',
     '698703483621523456',
     '698710712454139905',
     '698907974262222848',
     '698953797952008193',
     '698989035503689728',
     '699036661657767936',
     '699072405256409088',
     '699079609774645248',
     '699088579889332224',
     '699323444782047232',
     '699370870310113280',
     '699413908797464576',
     '699423671849451520',
     '699434518667751424',
     '699446877801091073',
     '699691744225525762',
     '699775878809702401',
     '699779630832685056',
     '699788877217865730',
     '699801817392291840',
     '700002074055016451',
     '700029284593901568',
     '700062718104104960',
     '700143752053182464',
     '700151421916807169',
     '700167517596164096',
     '700462010979500032',
     '700505138482569216',
     '700518061187723268',
     '700747788515020802',
     '700796979434098688',
     '700847567345688576',
     '700864154249383937',
     '700890391244103680',
     '701214700881756160',
     '701545186879471618',
     '701570477911896070',
     '701601587219795968',
     '701889187134500865',
     '701952816642965504',
     '701981390485725185',
     '702217446468493312',
     '702276748847800320',
     '702321140488925184',
     '702539513671897089',
     '702598099714314240',
     '702671118226825216',
     '702684942141153280',
     '702932127499816960',
     '703041949650034688',
     '703079050210877440',
     '703268521220972544',
     '703356393781329922',
     ...]




```python
image_predictions[image_predictions.jpg_url.duplicated()].count()
```




    tweet_id    5432957
    jpg_url     5432957
    img_num     5432957
    p1          5432957
    p1_conf     5432957
    p1_dog      5432957
    p2          5432957
    p2_conf     5432957
    p2_dog      5432957
    p3          5432957
    p3_conf     5432957
    p3_dog      5432957
    dtype: int64




```python
len(image_predictions) - len(image_predictions[(image_predictions['p1_dog']== True) | (image_predictions['p2_dog'] == True) | (image_predictions['p3_dog'] == True)])
```




    5434967




```python
str(image_predictions.p1.nunique()),str(image_predictions.p2.nunique()), str(image_predictions.p3.nunique())
```




    ('379', '406', '409')




```python
image_predictions.jpg_url.nunique(), image_predictions.tweet_id.nunique()
```




    (2010, 2076)



**Key Findings**
- The image prediction have 5432957 duplicated value
- p1 has 379 unique number,p2 has 406 unique number and p3 has 409 unique number
- There are 2076 unique id, but only has 2010 unique image url, which means some tweet id has the same image

** Assessment Observations**

Low quality, also known as dirty, data has content issues such as missing, invalid, inaccurate, and inconsistent
data. Untidy, also known as messy, data has structural issues: each variable should form a column, each
observation should form a row, and each observational unit a table. Assessment observations are not action
items; actions items will be defined when cleaning.

Quality
- There are some unnecessary columns
- The retweets rows need to be removed
- Need to convert timestamp to datetime object
- Need to convert ratings to float and fix rating issue that are not extracted properly, some should be decimals
- Need to update source columns from url to text format
- Make ratings_denominator "10" for consistency and remove outliers
- convert non-dog names to "None" and set p1,p2,p3 to title case
- Convert tweet_id datatype to string object

Tidiness
- Combined dog stages to one column call "dog stage" and remove individual stage columns
- Join three dataset to one master dataset on "twitter_id"

**Clean**
- A copy of original dataset will be created for data analysis
- The 3 copied dataset will be merged into one master table for visualization
- During the data cleaning process, I first remove the missing and duplicated data, next is to address quality and tidiness issue


```python
# Create copies of original dataframes
twitter_clean = twitter_archive.copy()
images_clean = image_predictions.copy()
tweet_clean = tweet_json.copy()
```


```python
#check if the dataset copy have the same number of entries
len(tweet_clean), len(twitter_clean), len(images_clean)
```




    (2354, 2356, 5434967)




```python
#excluding retweeted status because they are not needed
twitter_clean = twitter_clean[pd.isnull(twitter_clean.retweeted_status_id)]
twitter_clean = twitter_clean.drop_duplicates()
#Removinge rows where there are no images (expanded_urls).
twitter_clean = twitter_clean.dropna(subset=['expanded_urls'])
```


```python
pd.set_option('display.max_colwidth', -1)

```


```python
#create 'dog_stage"
twitter_clean["dog_stage"] = twitter_clean.text.str.extract('(puppo|pupper|floofer|doggo)', expand=True)
```


```python
columns = ['doggo', 'floofer', 'pupper', 'puppo']
twitter_clean.drop(columns, axis=1,inplace = True)
```


```python
twitter_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2117 entries, 0 to 2355
    Data columns (total 13 columns):
    tweet_id                      2117 non-null int64
    in_reply_to_status_id         23 non-null float64
    in_reply_to_user_id           23 non-null float64
    timestamp                     2117 non-null object
    source                        2117 non-null object
    text                          2117 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 2117 non-null object
    rating_numerator              2117 non-null int64
    rating_denominator            2117 non-null int64
    name                          2117 non-null object
    dtypes: float64(4), int64(3), object(6)
    memory usage: 231.5+ KB



```python
twitter_clean.drop(columns = ["in_reply_to_status_id", 'in_reply_to_user_id', 'retweeted_status_id', 'retweeted_status_user_id', 'retweeted_status_timestamp'],inplace = True)


```


```python
twitter_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2117 entries, 0 to 2355
    Data columns (total 8 columns):
    tweet_id              2117 non-null int64
    timestamp             2117 non-null object
    source                2117 non-null object
    text                  2117 non-null object
    expanded_urls         2117 non-null object
    rating_numerator      2117 non-null int64
    rating_denominator    2117 non-null int64
    name                  2117 non-null object
    dtypes: int64(3), object(5)
    memory usage: 148.9+ KB



```python
# delete duplicated jpg_url
images_clean = images_clean.drop_duplicates(subset = ["jpg_url"], keep = "last")
```


```python
images_clean['p1'] = images_clean['p1'].str.title()
images_clean['p2'] = images_clean['p2'].str.title()
images_clean['p3'] = images_clean['p3'].str.title()
```


```python
not_named_to_replace = twitter_clean.loc[(twitter_clean['name'].str.islower())]
```


```python
not_named_to_replace_list = not_named_to_replace['text'].tolist()
```


```python
for entry in not_named_to_replace_list:
    mask = twitter_clean.text == entry
    name_column = 'name'
    twitter_clean.loc[mask, name_column] = "None"
```


```python

twitter_clean.loc[(twitter_clean['name'].str.islower())]
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
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
#change data type to the correct type
twitter_clean["timestamp"] = pd.to_datetime(twitter_clean['timestamp'])
```


```python
twitter_clean["tweet_id"] = twitter_clean["tweet_id"].astype('str')
```


```python
images_clean["tweet_id"] = images_clean["tweet_id"].astype('str')
```


```python
tweet_clean["tweet_id"] = tweet_clean["tweet_id"].astype('str')
```


```python
tweet_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2354 entries, 0 to 2353
    Data columns (total 3 columns):
    tweet_id          2354 non-null object
    retweet_count     2354 non-null int64
    favorite_count    2354 non-null int64
    dtypes: int64(2), object(1)
    memory usage: 55.2+ KB



```python
twitter_clean = pd.merge(left=twitter_clean, right=tweet_clean, left_on='tweet_id', right_on='tweet_id', how='inner')
twitter_clean = twitter_clean.merge(images_clean, on='tweet_id', how='inner')
```


```python
twitter_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1928 entries, 0 to 1927
    Data columns (total 21 columns):
    tweet_id              1928 non-null object
    timestamp             1928 non-null datetime64[ns]
    source                1928 non-null object
    text                  1928 non-null object
    expanded_urls         1928 non-null object
    rating_numerator      1928 non-null int64
    rating_denominator    1928 non-null int64
    name                  1928 non-null object
    retweet_count         1928 non-null int64
    favorite_count        1928 non-null int64
    jpg_url               1928 non-null object
    img_num               1928 non-null object
    p1                    1928 non-null object
    p1_conf               1928 non-null object
    p1_dog                1928 non-null object
    p2                    1928 non-null object
    p2_conf               1928 non-null object
    p2_dog                1928 non-null object
    p3                    1928 non-null object
    p3_conf               1928 non-null object
    p3_dog                1928 non-null object
    dtypes: datetime64[ns](1), int64(4), object(16)
    memory usage: 331.4+ KB



```python
twitter_clean['name'] = twitter_clean['name'].replace('None', np.NaN)
```


```python
twitter_clean[twitter_clean.text.str.contains(r"(\d+\.\d*\/\d+)")]
```

    /anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:1: UserWarning: This pattern has match groups. To actually get the groups, use str.extract.
      """Entry point for launching an IPython kernel.





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
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>...</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>38</th>
      <td>883482846933004288</td>
      <td>2017-07-08 00:28:19</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Bella. She hopes her smile made you smile. If not, she is also offering you her favorite monkey. 13.5/10 https://t.co/qjrljjt948</td>
      <td>https://twitter.com/dog_rates/status/883482846933004288/photo/1,https://twitter.com/dog_rates/status/883482846933004288/photo/1</td>
      <td>5</td>
      <td>10</td>
      <td>Bella</td>
      <td>10407</td>
      <td>46860</td>
      <td>...</td>
      <td>1</td>
      <td>Golden_Retriever</td>
      <td>0.943082</td>
      <td>True</td>
      <td>Labrador_Retriever</td>
      <td>0.032409</td>
      <td>True</td>
      <td>Kuvasz</td>
      <td>0.00550072</td>
      <td>True</td>
    </tr>
    <tr>
      <th>478</th>
      <td>786709082849828864</td>
      <td>2016-10-13 23:23:56</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Logan, the Chow who lived. He solemnly swears he's up to lots of good. H*ckin magical af 9.75/10 https://t.co/yBO5wuqaPS</td>
      <td>https://twitter.com/dog_rates/status/786709082849828864/photo/1</td>
      <td>75</td>
      <td>10</td>
      <td>Logan</td>
      <td>7069</td>
      <td>20296</td>
      <td>...</td>
      <td>1</td>
      <td>Pomeranian</td>
      <td>0.467321</td>
      <td>True</td>
      <td>Persian_Cat</td>
      <td>0.122978</td>
      <td>False</td>
      <td>Chow</td>
      <td>0.102654</td>
      <td>True</td>
    </tr>
    <tr>
      <th>521</th>
      <td>778027034220126208</td>
      <td>2016-09-20 00:24:34</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Sophie. She's a Jubilant Bush Pupper. Super h*ckin rare. Appears at random just to smile at the locals. 11.27/10 would smile back https://t.co/QFaUiIHxHq</td>
      <td>https://twitter.com/dog_rates/status/778027034220126208/photo/1</td>
      <td>27</td>
      <td>10</td>
      <td>Sophie</td>
      <td>1885</td>
      <td>7320</td>
      <td>...</td>
      <td>1</td>
      <td>Clumber</td>
      <td>0.9467180000000001</td>
      <td>True</td>
      <td>Cocker_Spaniel</td>
      <td>0.0159499</td>
      <td>True</td>
      <td>Lhasa</td>
      <td>0.00651911</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1319</th>
      <td>680494726643068929</td>
      <td>2015-12-25 21:06:00</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have uncovered an entire battalion of holiday puppers. Average of 11.26/10 https://t.co/eNm2S6p9BD</td>
      <td>https://twitter.com/dog_rates/status/680494726643068929/photo/1</td>
      <td>26</td>
      <td>10</td>
      <td>NaN</td>
      <td>542</td>
      <td>1879</td>
      <td>...</td>
      <td>1</td>
      <td>Kuvasz</td>
      <td>0.43862700000000004</td>
      <td>True</td>
      <td>Samoyed</td>
      <td>0.111622</td>
      <td>True</td>
      <td>Great_Pyrenees</td>
      <td>0.0640608</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>4 rows × 21 columns</p>
</div>




```python
# Remove url from sources
twitter_clean['source'] = twitter_clean['source'].str.replace('<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'Twitter for iPhone')
twitter_clean['source'] = twitter_clean['source'].str.replace('<a href="http://vine.co" rel="nofollow">Vine - Make a Scene</a>', 'Vine')
twitter_clean['source'] = twitter_clean['source'].str.replace('<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>', 'Twitter Web Client')
twitter_clean['source'] = twitter_clean['source'].str.replace('<a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>', 'TweetDeck')
```


```python

# Change datatype to category
twitter_clean['source'] = twitter_clean['source'].astype('category')
```


```python
twitter_clean.source.value_counts()
```




    Twitter for iPhone    1891
    Twitter Web Client    26  
    TweetDeck             11  
    Name: source, dtype: int64




```python
twitter_clean['dog_stage'] = twitter_clean['dog_stage'].astype('category')
```


```python

twitter_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1928 entries, 0 to 1927
    Data columns (total 22 columns):
    tweet_id              1928 non-null object
    timestamp             1928 non-null datetime64[ns]
    source                1928 non-null category
    text                  1928 non-null object
    expanded_urls         1928 non-null object
    rating_numerator      1928 non-null int64
    rating_denominator    1928 non-null int64
    name                  1303 non-null object
    retweet_count         1928 non-null int64
    favorite_count        1928 non-null int64
    jpg_url               1928 non-null object
    img_num               1928 non-null object
    p1                    1928 non-null object
    p1_conf               1928 non-null object
    p1_dog                1928 non-null object
    p2                    1928 non-null object
    p2_conf               1928 non-null object
    p2_dog                1928 non-null object
    p3                    1928 non-null object
    p3_conf               1928 non-null object
    p3_dog                1928 non-null object
    dog_stage             317 non-null category
    dtypes: category(2), datetime64[ns](1), int64(4), object(15)
    memory usage: 320.4+ KB



```python
twitter_clean.to_csv('twitter_archive_master.csv')
```


### Analysis & Visualizations



a. How's the distribution of the rating_numerator? 


```python
len(twitter_archive[twitter_archive.rating_numerator > 15])
```




    26




```python
plot = plt.hist(twitter_archive.rating_numerator.sort_values()[0:2330]);
plot;
```


![png](output_66_0.png)


**Key Findings:**
There are 26 ratings greater than 15. Majority of the ratting are range between 10 to 13, with 13 received the most rattings.


```python
plt.savefig('rating_numerator_plot.png')
```


    <matplotlib.figure.Figure at 0x119ffaa20>


b. How's the distribution of the favories count and retreet count? 


```python
df = twitter_clean[['retweet_count', 'favorite_count', 'dog_stage']]
```


```python
df_puppers = df[df['dog_stage'] == 'pupper']
df_puppo = df[df['dog_stage'] == 'puppo']
df_doggo = df[df['dog_stage'] == 'doggo']
df_floofer = df[df['dog_stage'] == 'floofer']
df_none = df[df['dog_stage'] == 'None']
```


```python
# Plot all data to see general shape
bx = df_puppers.plot(kind = 'scatter', x='favorite_count', y='retweet_count', color='Orange', alpha=0.3)
df_doggo.plot(kind = 'scatter', x='favorite_count', y='retweet_count', color='Blue', ax=bx, alpha=0.3)
df_puppo.plot(kind = 'scatter', x='favorite_count', y='retweet_count', color='Green', ax=bx, alpha=0.3)
df_floofer.plot(kind = 'scatter', x='favorite_count', y='retweet_count', color='Red', ax=bx, alpha=0.3);
```


![png](output_72_0.png)


** Key Findings**
The chart shows there are direct relationship between retweet_count and favorite_count. The more favorite the dog received, the more retweet will happen. Most of the images have less than 20000 favories.

c. How's the retweet and favorite changed over time?


```python
time_df = twitter_clean[['timestamp', 'retweet_count', 'favorite_count', 'rating_numerator', 'rating_denominator']].copy()
```


```python

# Set the index to be the timestamp so time is displayed properly in plots
time_df.set_index('timestamp', inplace=True)
```


```python
time_df['retweet_count'].plot()
plt.xlabel('Tweet timestamp')
plt.ylabel('Count')
plt.title('Retweets over time');
```


![png](output_77_0.png)



```python
time_df['favorite_count'].plot()
plt.xlabel('Favorite timestamp')
plt.ylabel('Count')
plt.title('Favorities over time');
```


![png](output_78_0.png)



```python
%matplotlib inline
# Creating column of a rolling 30-day average for favorites and retweets
twitter_clean['rolling_favorite'] = twitter_clean.favorite_count.rolling(window = 30).mean()
twitter_clean['rolling_retweet'] = twitter_clean.retweet_count.rolling(window = 30).mean()

# Plotting the 30-day average on favorites and retweets.
plt.plot(twitter_clean.timestamp, twitter_clean.rolling_favorite, label = 'Favorites')
plt.plot(twitter_clean.timestamp, twitter_clean.rolling_retweet, label = 'Retweets')
plt.title('30-day Rolling Average Tweet Performance')
plt.xlabel('Dates')
plt.ylabel('30-day Mean')
plt.legend(loc='best')
plt.xticks(rotation='vertical')
plt.show();
```


![png](output_79_0.png)


**Key Findings**
Both Favorities and Retweets saw increase over time. There are a spike in both favories and retweet in June 2016. The favories count increase rate is bigger than retweets.

d. How are the different dog stages?


```python
twitter_clean['dog_stage'].value_counts().plot(kind='bar');
                                     
```


![png](output_82_0.png)


**Key Findings**
Pupper is the most popular dog, followed by doggo, with floofer being the least

**References:**
1.https://static1.squarespace.com/static/55bfa8e4e4b007976149574e/t/5b870d818a922de25747cbdc/1535577475794/wrangle_act.pdf
2. http://www.karenbevis.com/weratedogs/
3. https://ryanwingate.com/projects/3/




```python

```
