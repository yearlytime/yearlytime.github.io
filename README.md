Aidan Levyne

William Healy

GitPages site: https://yearlytime.github.io

## Writeup

For our final project, we will use data analysis to to determine whether or not there exists a correlation between certain attributes and price sold on discogs.
We are considering using our own dataset collected from Discogs.com and the Last.fm Million Song Dataset. Discogs has long been the premiere website for music release data. Since 2000, the site has accumulated nearly 15 million release submissions, cataloguing artwork, genre (albeit very general), music style - subgenre, label, different pressings, users who have a release, users who want a release, and much more information. The user data is mostly reliable, as the features on the site for both owners and potential buyers act as incentives; those who collect the item are able to neatly categorize and search their collection by any attribute or keyword, and those who want to buy the album are notified when a release comes for sale. We will use user feedback as input to determine the supply and demand for items. We’re hoping to find how accurately we can use objective community submitted data to predict the sale price for an item on average.

https://www.discogs.com/search/?ev=em_rs&type=master 
To achieve this, we will create our own dataset combining Discogs.com webpages, the last.fm million song dataset, and the first editions for the top 1000 masters on Discogs. Whereas we could attempt to use an archive of the site data completely, it will take more time and effort than will be necessary to our end goal. 

The Last.fm Million Song Dataset contains data including community-submitted genre tags, artist images, and number of listens and listeners. This data set is already collected and is stored in json files separated by three levels of single letter folders. Because we recently agreed to use this dataset, we are still developing code to utilize the information it provides.
http://millionsongdataset.com/ 
A specific example, using Nirvana, we can see that Smells Like Teen Spirit is the most popular song with over two million listens, and the most popular genre tag is Grunge. In addition, Nirvana has important artist information to those unfamiliar with the group. However, because Nirvana is a popular name among several music acts and artists, there are multiple with the same name. For this issue, the genre tags may not be accurate for an obscure group with a general name, but the artist information lists multiple artists so as not to assume which one is looking for. Artist information is listed numerically in no particular order in this dataset. If our algorithm were to come across the fourth Nirvana in this list, the Dutch pop group formed in 1985, we would need to figure out how to accurately show only information relavent to the pop group and none relating to the Grunge group. However, using a smaller dataset with the most popular releases will likely mitigate this issue.
https://www.last.fm/music/Nirvana

The First edition record data from Discogs dataset is a small dataset uploaded to Kaggle by Beth Baumann, that contains user and sale data from the initial releases of discogs' most popular albums. This dataset was uploaded a year ago, which is relatively current. However, we plan on reassembling this dataset with up to date information to attempt to find any notable changes and attempt to determine if we can correlate time with sale price.
https://www.kaggle.com/bethbaumann/first-edition-record-data-from-discogs


## Collaboration Plan

For our collaboration plan, we will continue to meet two to three times a week to discuss plans and progress with our seperate roles in research. One of us works with understanding the datasets to merge, while the other works on means to efficiently scrape our own data from Discogs site. We also set up a second GitHub private repo to exchange data and code seamlessly. Although we have assigned roles in the project, we often check in to ensure both of us are on the same page with each new step. We have met in the business school and Gibson to collaborate and exchange information. Both of us feel passionately about this project, and additional meetings happen spontaneously.

## Data


```python
!pip install pandas_read_xml
```

    Collecting pandas_read_xml
      Downloading pandas_read_xml-0.0.7-py3-none-any.whl (6.2 kB)
    Requirement already satisfied: requests<3,>=2.23.0 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (2.24.0)
    Requirement already satisfied: six<2,>=1.14.0 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (1.15.0)
    Requirement already satisfied: urllib3<2,>=1.25.9 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (1.25.10)
    Collecting xmltodict<1,>=0.12.0
      Downloading xmltodict-0.12.0-py2.py3-none-any.whl (9.2 kB)
    Requirement already satisfied: chardet<4,>=3.0.4 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (3.0.4)
    Requirement already satisfied: pandas<2,>=1.0.3 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (1.1.2)
    Requirement already satisfied: certifi<2021,>=2020.4.5.1 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (2020.6.20)
    Requirement already satisfied: numpy<2,>=1.18.4 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (1.19.1)
    Requirement already satisfied: pytz<2021,>=2020.1 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (2020.1)
    Requirement already satisfied: idna<3,>=2.9 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (2.10)
    Collecting pyarrow<1,>=0.17.0
      Downloading pyarrow-0.17.1-cp38-cp38-manylinux2014_x86_64.whl (63.8 MB)
    [K     |████████████████████████████████| 63.8 MB 37.5 MB/s eta 0:00:01   |█▌                              | 3.1 MB 1.7 MB/s eta 0:00:36     |██▌                             | 5.0 MB 1.7 MB/s eta 0:00:35     |████████████████████            | 39.9 MB 13.0 MB/s eta 0:00:02     |███████████████████████████▌    | 54.7 MB 13.0 MB/s eta 0:00:01
    [?25hRequirement already satisfied: python-dateutil<3,>=2.8.1 in /opt/conda/lib/python3.8/site-packages (from pandas_read_xml) (2.8.1)
    Installing collected packages: xmltodict, pyarrow, pandas-read-xml
    Successfully installed pandas-read-xml-0.0.7 pyarrow-0.17.1 xmltodict-0.12.0
    


```python
import pandas as pd #pandas.read_json
import pandas_read_xml as pdx
import numpy as np
import json
import os
import sys
import xml.etree.ElementTree as et 
import requests


```

    C:\Users\booke\anaconda3\lib\site-packages\requests\__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.7) or chardet (3.0.4) doesn't match a supported version!
      warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported "
    


```python
#load test data from discogs api
r = requests.get('https://api.discogs.com/artists/1/releases?page=1&per_page=250')

#convert data to JSON
artist_releases = json.loads(r.text)

#delete empty or redundant columns
del artist_releases['pagination']


#creating list of main releases and masters; delete redundant information ("id","status",);
masters=np.empty(0)
main_releases=np.empty(0)
for i in artist_releases['releases']:
    i["have"]=i["stats"]["community"]["in_collection"]
    i["want"]=i["stats"]["community"]["in_wantlist"]
    del i["stats"]
    del i["id"]
    i["trackinfo"]=0
    i["status"]=0
    del i["trackinfo"]
    del i["status"]
    del i["thumb"]
    if i["type"]=="master":
        masters=np.append(masters,i)
    if (i["type"]=="release"):
        if (i["role"]=="main"):
            main_releases=np.append(main_releases,i)
    if i["role"]!="main":
        del i
            

#convert JSON dict to dataframe
ardf_all= pd.DataFrame.from_dict(artist_releases['releases'],
                             orient='columns',
                            dtype=object)
#view data
ardf_all
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
      <th>type</th>
      <th>format</th>
      <th>label</th>
      <th>title</th>
      <th>resource_url</th>
      <th>role</th>
      <th>artist</th>
      <th>year</th>
      <th>have</th>
      <th>want</th>
      <th>main_release</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>release</td>
      <td>10"</td>
      <td>Svek</td>
      <td>Kaos</td>
      <td>https://api.discogs.com/releases/20209</td>
      <td>Main</td>
      <td>Stephan-G* &amp; The Persuader</td>
      <td>1997</td>
      <td>347</td>
      <td>1315</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>release</td>
      <td>12"</td>
      <td>Svek</td>
      <td>Snorkelmannen Ø Hans Vänner</td>
      <td>https://api.discogs.com/releases/62584</td>
      <td>Main</td>
      <td>Mr. Barth* &amp; The Persuader</td>
      <td>1997</td>
      <td>616</td>
      <td>732</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>master</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Stockholm By Night</td>
      <td>https://api.discogs.com/masters/321212</td>
      <td>Main</td>
      <td>The Persuader</td>
      <td>1997</td>
      <td>464</td>
      <td>671</td>
      <td>4664</td>
    </tr>
    <tr>
      <th>3</th>
      <td>release</td>
      <td>12"</td>
      <td>Svek</td>
      <td>A View From Slussen</td>
      <td>https://api.discogs.com/releases/400</td>
      <td>Main</td>
      <td>The Persuader</td>
      <td>1998</td>
      <td>520</td>
      <td>1132</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>master</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>City Of Islands</td>
      <td>https://api.discogs.com/masters/4242</td>
      <td>Main</td>
      <td>The Persuader</td>
      <td>1998</td>
      <td>495</td>
      <td>993</td>
      <td>79</td>
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
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>111</th>
      <td>release</td>
      <td>Cass, Mixed, Promo, C90</td>
      <td>Sole Unlimited</td>
      <td>Live @ The Traxx</td>
      <td>https://api.discogs.com/releases/12526186</td>
      <td>TrackAppearance</td>
      <td>Ian Pooley</td>
      <td>NaN</td>
      <td>4</td>
      <td>13</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>112</th>
      <td>master</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Every Woman Have To Drive A Car</td>
      <td>https://api.discogs.com/masters/1744570</td>
      <td>UnofficialRelease</td>
      <td>DJ Anton Kubikoff*</td>
      <td>1999</td>
      <td>12</td>
      <td>7</td>
      <td>1878188</td>
    </tr>
    <tr>
      <th>113</th>
      <td>master</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Profound Sounds Vol. 1</td>
      <td>https://api.discogs.com/masters/66526</td>
      <td>UnofficialRelease</td>
      <td>Josh Wink</td>
      <td>1999</td>
      <td>10</td>
      <td>15</td>
      <td>5780201</td>
    </tr>
    <tr>
      <th>114</th>
      <td>master</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>She's A Dancing Machine</td>
      <td>https://api.discogs.com/masters/599477</td>
      <td>UnofficialRelease</td>
      <td>Magda</td>
      <td>2006</td>
      <td>1</td>
      <td>5</td>
      <td>15023744</td>
    </tr>
    <tr>
      <th>115</th>
      <td>release</td>
      <td>CDr, CD-ROM, Comp, Unofficial, MP3</td>
      <td>MP3SERVICE</td>
      <td>MP3 Collection: Рак - Музыкальный Гороскоп (De...</td>
      <td>https://api.discogs.com/releases/9369505</td>
      <td>UnofficialRelease</td>
      <td>Various</td>
      <td>NaN</td>
      <td>1</td>
      <td>3</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>116 rows × 11 columns</p>
</div>




```python

```
