2
# OpenStreetMap Data Wrangling with MongoDB
## Data Wrangling Project
#### Data Analyst Nanodegree (Udacity)
Project submission by Edward Minnett (ed@methodic.io).

August 2nd 2016. (Revision 2)

----------

## Overview

This project seeks to apply data munging techniques to first analyse and clean Open Street Map data for a single city and then perform an exploratory analysis of the data after importing it into a MongoDB collection.

I chose to explore the Open Street Map data for Oxford England as for the last couple of years I have lived in a small village south of Oxford. Having visited the city on quite a few occasions, I am confident that there is a lot of interesting information to be discovered through this analysis.

The data for this project was acquired from Map Zen. The compressed Oxford, England OSM XML data set can be downloaded by following this [link](https://s3.amazonaws.com/metro-extracts.mapzen.com/oxford_england.osm.bz2). The OSM file is 5.0 MB compressed and 66.2 MB uncompressed.


```python
# The following is needed at the beginning of the notebook to make sure the cells execute ok.
import sys
sys.path.append('./case_study_files')

data_name = "oxford_england"
OSMFILE = "{}.osm".format(data_name)
```

## Problems encountered in the map

As you would expect from any crowd-sourced data, there is evidence of inconsistent naming conventions as well as evidence of human error when the data was entered. The two main areas of inconsistency that affect this project is in the names used to describe the type of record in the data and the strings used as values within the data. It isn't feasible to analyse all values in the data, but street names are one type of value where inconsistency in values has a significant impact on the quality of the data.

The following analysis attempts to find problems with tag key names along with suggested fixes for the problems. There is a similar analysis and set of recommendations for fixing the values used for street names.

### Tag key name problems


```python
# Find problems with tag names
import tags as tags_processor
tag_problems = tags_processor.process_map(OSMFILE)

# Additional key categories have been added in an attempt to catch additional key name patterns and minimise
# the number of keys that fall within the 'other' category.

# The problems are defined as follows:
#   "lower", for tags that contain only lowercase letters and are valid,
#   "upper", for tags that contain upercase letters,
#   "lower_colon", for otherwise valid tags with a colon in their names,
#   "uper_colon", for tags contain both uppercase strings delimited by a colon,
#   "multiple_colons", for tags contain more than two colon seperated strings,
#   "numbers", for tags that contain a digit, and
#   "problemchars", for tags with problematic characters, and
#   "other", for other tags that do not fall into the other three categories.
print "The number of keys in each of the 'problem' categories:"
print tag_problems['counts']
unique_key_names = tags_processor.unique_tag_keys(OSMFILE)
print "There are {} unique tag key names in the data set.".format(len(unique_key_names))
```

    The number of keys in each of the 'problem' categories:
    {'problemchars': 5, 'upper': 120, 'lower': 143057, 'upper_colon': 7638, 'numbers': 769, 'multiple_colons': 608, 'lower_colon': 47850, 'other': 1}
    There are 1011 unique tag key names in the data set.


It is the tags that fall under 'problemchars' that will be particularly problematic as these tag key names aren't valid key names in MongoDB. Let's see what these tags look like and how they could be fixed.


```python
tag_problems['keys']['problemchars']
```




    ['leaving for now',
     'fee:amount:box_van&minibus',
     'note:0.1',
     'note:0.2',
     'note 2']



These issues can be easily fixed by making the following replacements:

- Replace ' ' with '\_'
- Replace '&' with '\_and\_'
- Replace '.' with '\_'

The logic to do this has been added to the case_study_files/data.py so that the key values are tidied when the elements are being shaped.

We will also add logic to handle the keys that fall under 'upper' and 'upper_colon'. This can be fixed by ensuring that shape_element uses the lowercase version of the strings.

The key names that fall under 'multiple_colons' can be handled by adding additional nesting within the JSON data created by shape_element.

Tag key names that fall within the 'numbers' category are useful to know about, but numbers are valid characters in MongoDB strings so no additional processing is required.

It is also worth looking at the one remaining key that fall under 'other' to see why it doesn't fall within one of the other categories.


```python
set(tag_problems['keys']['other'])
```




    {'name:sr-Latn'}



Like the tag key names that contain numbers, the hyphen is also a valid character in a MongoDB key as this one one 'other' key name is fine as it is.

### Street name problems


```python
# Find problems with street names
import audit as street_name_auditor
street_name_auditor.audit(OSMFILE)
```




    defaultdict(set,
                {'1': {'Avenue 1'},
                 '2': {'Avenue 2'},
                 '3': {'Avenue 3'},
                 '4': {'Avenue 4'},
                 "Aldate's": {"St Aldate's"},
                 'Ave': {'Waverly Ave'},
                 'Barr': {'Upper Barr'},
                 'Bridge': {'Folly Bridge'},
                 'Broadway': {'The Broadway'},
                 'Buildings': {'Manor Buildings'},
                 'Castle': {'Oxford Castle'},
                 'Centre': {'Fairfax Centre'},
                 'Centremead': {'Centremead'},
                 'Chorefields': {'Chorefields'},
                 'Clements': {'St Clements'},
                 'Cloisters': {'Temple Cloisters'},
                 'Close': {'Acland Close',
                  'Acre Close',
                  'Amory Close',
                  'Ashcroft Close',
                  'Ashroft Close',
                  'Barn Close',
                  'Bartlemas Close',
                  'Benouville Close',
                  'Blackburn Close',
                  'Broad Close',
                  'Browns Close',
                  'Bullstake Close',
                  'Burgan Close',
                  'Burrows Close',
                  'Bushy Close',
                  'Cardinal Close',
                  'Carey Close',
                  'Cholsey Close',
                  'Clover Close',
                  'Compass Close',
                  'Complins Close',
                  'Coolidge Close',
                  'Cornwallis Close',
                  'Denton Close',
                  'Don Bosco Close',
                  'Dora Carr Close',
                  'Dover Close',
                  'East Field Close',
                  'Egrove Close',
                  'Eleanor Close',
                  'Evelyn Close',
                  'Galpin Close',
                  'Goodey Close',
                  'Halls Close',
                  'Hayday Close',
                  'Highbank Close',
                  'Hillsborough Close',
                  'Homestall Close',
                  'Hundred Acres Close',
                  'Hunter Close',
                  'Hutchcomb Farm Close',
                  'Ivy Close',
                  'John Parker Close',
                  'Kames Close',
                  'Kennedy Close',
                  'Lambton Close',
                  'Long Close',
                  'Marley Close',
                  'Meyseys Close',
                  'Nobles Close',
                  'Osborne Close',
                  'Owlington Close',
                  'Pottle Close',
                  'Pulker Close',
                  'Queens Close',
                  'Riely Close',
                  'Robinson Close',
                  'Roundham Close',
                  'Silkdale Close',
                  'Skene Close',
                  'Songers Close',
                  'South Close',
                  "St Ebba's Close",
                  "Stimpson's Close",
                  'Stone Close',
                  'Stubble Close',
                  'Troy Close',
                  'Venneit Close',
                  'Yeats Close'},
                 'Crescent': {'Brookfield Crescent',
                  'Canning Crescent',
                  'Corunna Crescent',
                  'Fox Crescent',
                  'Herschel Crescent',
                  'Hugh Allen Crescent',
                  'Kendall Crescent',
                  'Kersington Crescent',
                  'Lockheart Crescent',
                  'Normandy Crescent',
                  'Salisbury Crescent',
                  'Walton Crescent',
                  'Westbury Crescent',
                  'Wykeham Crescent'},
                 'Down': {'Colegrove Down'},
                 'Driftway': {'Brasenose Driftway', 'Horspath Driftway'},
                 'East': {'Woodstock Road East'},
                 'Entry': {'Friars Entry'},
                 'Furze': {'Demesne Furze', 'Inott Furze', 'Town Furze'},
                 'Gardens': {'Court Place Gardens',
                  'Croxford Gardens',
                  'Mileway Gardens',
                  'Norham Gardens'},
                 'Giles': {'St Giles'},
                 "Giles'": {"Saint Giles'", "St Giles'"},
                 'Glebe': {'The Glebe'},
                 'Glebelands': {'Glebelands'},
                 'Grates': {'The Grates'},
                 'Green': {'Gloucester Green'},
                 'Ground': {"Cox's Ground"},
                 'Hill': {'Cumnior Hill',
                  'Cumnor Hill',
                  'Heyford Hill',
                  'Rose Hill',
                  'The Cedars, Cumnor Hill',
                  'Tumbledown Hill'},
                 'Hillside': {'Hillside'},
                 'Hollow': {'Quarry Hollow'},
                 'House': {'Salesian House'},
                 'Hurdeswell': {'Hurdeswell'},
                 'Mead': {'Osney Mead'},
                 'Meadow': {'Stone Meadow', 'Upper Meadow'},
                 'Mews': {'Fairfax Mews', 'Temple Mews'},
                 'Moors': {'Peat Moors'},
                 'Parade': {'Elms Parade', 'South Parade', 'The Parade'},
                 'Park': {'Seacourt Tower Retail Park',
                  'Southfield Park',
                  'Templers Shopping Park',
                  'The Park'},
                 'Path': {'The Towing Path'},
                 'Phelps': {'The Phelps'},
                 'Plain': {'The Plain'},
                 'Quarter': {'Oxford Castle Quarter'},
                 'Quorum': {'The Quorum'},
                 'Rd': {'Abingdon Rd',
                  'Appleton Rd',
                  'Faringdon Rd',
                  'Fogwell Rd',
                  'Glebe Rd',
                  'Oxford Rd'},
                 'Ridgeway': {'The Ridgeway'},
                 'Rise': {'Third Acre Rise'},
                 'Roundabout': {'Wolvercote Roundabout'},
                 'Roundway': {'The Roundway'},
                 'Row': {'Hollybush Row'},
                 'Slade': {'The Slade'},
                 'St': {'High St'},
                 'Sunnyside': {'Sunnyside'},
                 'Terrace': {'Cranham Terrace', 'Weymann Terrace'},
                 'Town': {'Park Town'},
                 'Trees': {'Cherry Trees'},
                 'Turn': {'Iffley Turn'},
                 'Valley': {'Lye Valley'},
                 'View': {'Fair View'},
                 'Village': {'Wootton Village'},
                 'Walk': {'Church Walk'},
                 'Way': {'Alec Issigonis Way',
                  "Arnold's Way",
                  'Arnolds Way',
                  'Ashhurst Way',
                  'Bellenger Way',
                  'Church Way',
                  'Delamare Way',
                  'Dunnock Way',
                  'Elizabeth Jennings Way',
                  'Gordon Woodward Way',
                  'Headley Way',
                  'Hollow Way',
                  'Leander Way',
                  'Manzil Way',
                  'Mazil Way',
                  'Middle Way',
                  'Navigation Way',
                  'Oakwood Way',
                  'Orchard Way',
                  'Pinnocks Way',
                  'Reliance Way',
                  'Reliuance Way',
                  'Seven Sisters Way',
                  'Sheldon Way',
                  'Unit 1, Seacourt Towers, West Way',
                  'West Way',
                  'Wilsdon Way'},
                 'Way,': {'Roger Dudman Way,'},
                 'Way?': {'Reliance Way?'},
                 'West': {'Sandy Lane West'},
                 'Winnyards': {'The Winnyards'},
                 'Woodfield': {'Woodfield'},
                 'Yard': {'New Inn Yard'},
                 'road': {'Marston road'},
                 'way': {'Reliance way'}})



The large majority of these street names are valid, but there are a few problems. There are a few cases of abbreviated street names such as 'Ave', 'Rd' and 'St'. There are also several cases of punctuations mistakes such as 'Way?' and 'Way,'. There are cases of both 'road' and 'way' being lowercase. There is also a typo where 'Reliuance Way' should be 'Reliance Way'. The oddest problems are the cases of 'Avenue 1' through 'Avenue 4'. In Kennington just outside of Oxford the main street is called 'The Avenue'. I believe these street names may be mistakes or parsing issues. They don't match any obvious street names and as a result will be treated as anomalies and ignored.

Logic has been added to case_study_files/data.py to clean up these inconsistencies.

### Cuisine problems


```python
import cuisine as cuisine_auditor
food_nodes = cuisine_auditor.audit(OSMFILE)

food_nodes_with_cuisine_and_amenity = [n for n in food_nodes if 'cuisine' in n and 'amenity' in n]
food_nodes_without_cuisine = [n for n in food_nodes if 'cuisine' not in n]
food_nodes_without_amenity = [n for n in food_nodes if 'amenity' not in n]
print "Number of food nodes: {}".format(len(food_nodes))
print "Number of food nodes with a cuisine and amenity: {}".format(len(food_nodes_with_cuisine_and_amenity))
print "Number of food nodes without a cuisine: {}".format(len(food_nodes_without_cuisine))
print "Number of food nodes without an amenity: {}".format(len(food_nodes_without_amenity))
```

    Number of food nodes: 523
    Number of food nodes with a cuisine and amenity: 213
    Number of food nodes without a cuisine: 303
    Number of food nodes without an amenity: 7



```python
from collections import Counter

amenities = []
for node in food_nodes_without_cuisine:
    amenities.append(node['amenity'])

print "Amenity counts for food nodes without a cuisine:"
print Counter(amenities)
```

    Amenity counts for food nodes without a cuisine:
    Counter({'pub': 152, 'cafe': 73, 'restaurant': 31, 'fast_food': 26, 'bar': 21})



```python
def reasonable_anomoly(n):
    return 'name' in n and ('shop' in n or 'disused' in n)
[{'name': n['name'], 'cuisine': n['cuisine']} for n in food_nodes_without_amenity if reasonable_anomoly(n)]
```




    [{'cuisine': 'sandwich', 'name': "Bunny's"},
     {'cuisine': 'chinese', 'name': 'Chopsticks Chinese Restaurant'},
     {'cuisine': 'greek', 'name': 'Meli Deli'},
     {'cuisine': 'donut', 'name': 'Krispy Kreme'},
     {'cuisine': 'polish', 'name': 'Euro Foods'}]




```python
for amenity in cuisine_auditor.food_amenities:
    cuisines = []
    relevant_nodes = [n for n in food_nodes_with_cuisine_and_amenity if n['amenity'] == amenity]
    for node in relevant_nodes:
        cuisines.append(node['cuisine'])
    print "Cuisine counts for {} nodes:".format(amenity)
    print Counter(cuisines)
    print '\n'
```

    Cuisine counts for restaurant nodes:
    Counter({'indian': 17, 'chinese': 12, 'italian': 9, 'thai': 6, 'burger': 4, 'pizza': 4, 'french': 4, 'tapas': 4, 'asian': 3, 'nepalese': 3, 'sandwich': 3, 'lebanese': 3, 'sushi': 2, 'japanese': 2, 'turkish': 2, 'mexican': 1, 'fish': 1, 'japanese;sushi': 1, 'chinese;indian;thai;halal': 1, 'italian;pizza': 1, 'chicken': 1, 'indian;curry': 1, 'sausage': 1, 'international': 1, 'korean': 1, 'asian;noodle;soup': 1, 'english': 1, 'pizza;pasta': 1, 'crepe': 1, 'pizza,burger': 1, 'greek': 1, 'fish_and_chips': 1, 'Nepalese': 1, 'curry': 1, 'seafood': 1})


    Cuisine counts for cafe nodes:
    Counter({'coffee_shop': 15, 'sandwich': 10, 'ice_cream': 4, 'portuguese': 1, 'bubbletea': 1, 'pie': 1, 'bangladeshi': 1})


    Cuisine counts for pub nodes:
    Counter({'burger': 2, 'vegetarian': 1, 'pizza': 1, 'american': 1, 'Fish and Chips and other pub favourites': 1})


    Cuisine counts for bar nodes:
    Counter({'jamaican': 1, 'drinks': 1})


    Cuisine counts for fast_food nodes:
    Counter({'chinese': 17, 'sandwich': 12, 'fish_and_chips': 10, 'pizza': 6, 'burger': 5, 'kebab': 3, 'indian': 3, 'mexican': 2, 'kebab;middle_east': 2, 'asian': 2, 'chicken': 2, 'sushi': 1, 'japanese': 1, 'burgers;snacks;full_english': 1, 'cornish_pasty': 1, 'persian': 1, 'moroccan': 1, 'crepe': 1, 'burger;sausage': 1, 'italian': 1})


    Cuisine counts for delicatessen nodes:
    Counter({'brazillian;portuguese': 1})





```python
from difflib import SequenceMatcher

def similarity_by_name(a, b):
    if 'name' in a and 'name' in b:
        a = a['name'].replace('the', '').lower()
        b = b['name'].replace('the', '').lower()
        return SequenceMatcher(None, a, b).ratio()
    else:
        return 0

subject = food_nodes_without_cuisine[0]

processed_nodes = []
food_nodes_with_same_amenity = [n for n in food_nodes_with_cuisine_and_amenity if n['amenity'] == subject['amenity']]
for node in food_nodes_with_same_amenity:
    if 'name' in node:
        processed_nodes.append({'similarity': similarity_by_name(subject, node), 'node': node})

print "Subject name: {}   Subject amenity: {} \n".format(subject['name'], subject['amenity'])
sorted_results = sorted(processed_nodes, key=lambda k: k['similarity'], reverse=True)
for result in sorted_results[:5]:
    node = result['node']
    score = '%.3f' % result['similarity']
    print "Similarity score: {}   Name: {}   Cuisine: {}".format(score, node['name'], node['cuisine'])
```

    Subject name: The Tree   Subject amenity: pub

    Similarity score: 0.538   Name: The Gardeners Arms   Cuisine: vegetarian
    Similarity score: 0.526   Name: White Horse   Cuisine: Fish and Chips and other pub favourites
    Similarity score: 0.500   Name: The White Rabbit   Cuisine: pizza
    Similarity score: 0.500   Name: The Chequers   Cuisine: burger
    Similarity score: 0.462   Name: The Head of the River   Cuisine: burger


This analysis shows that there are definite problems with the cuisine classifications of food nodes within the data. Of the 523 nodes analysed, 303 have amenity types but no cuisine type and 7 have a cuisine type but no amenity type. The 7 nodes that have a cuisine but are missing an amenity can be explained as follows and don't need further analysis: four are shops and don't require an amenity and two are marked as disused one of which is missing. Only 'The Oriental Condor' looks like it should have an amenity type but does not.

The 303 nodes that have food related amenities but lack a cuisine are broken down by amenity as follows:

- pub: 152
- cafe: 73
- restaurant: 31
- fast_food: 26
- bar: 21

I believe that the disproportionate representation of pubs within this data can be explained by either the pubs not serving food and as a result not needing a cuisine type although I find it hard to believe that Oxford has 153 pubs none of which serve food. What may be more likely is that the creators of this data assumed that a node marked as a 'pub' serves 'pub food' and as a result does not need a 'cuisine'.

An analysis of the cuisines for each of the remaining 213 nodes broken down by amenity does not show a consistent pattern. A winner take all strategy (majority cuisine by amenity) to fix the 303 problem nodes would mean assigning 'indian' to all 'restaurants' and 'chinese' to all 'fast_food' nodes. This hardly seems appropriate.

A more sophisticated strategy would be to do a similarity analysis between node names and see if appropriate cuisines could be inferred from the node names. An initial attempt at fixing the 303 problem nodes this way also proved inappropriate. A sample application of the approach resulted in 'The Tree' pub being considered most similar to 'The Gardeners Arms' pub. This is a clever result in terms of string similarity, but 'The Gardeners Arms' servers vegetarian food and a little bit of research shows that the pub at The Tree Hotel serves standard pub fair (curry, steak etc.). This illustrates why trying to infer the cuisine simply by doing a string similarity analysis on the node names will likely result in badly assigned cuisine types. For this reason, I decided not to attempt to fix the cuisine problems deciding that adding bad information to the data is worse than accepting that some information is missing.

If fixing these problems was imperative, I would attempt to train a supervised learning algorithm to try and harness more information than just the names in attempt to more accurately label the missing cuisine types. If I were to apply this strategy, I would look for more data than just the 213 nodes from the Oxford data set as it would be unlikely that out of 516 nodes, 213 labeled data would be able to generalise over the remaining 303.

## Load the Data


```python
# This snippet uses the process_map function from the case study data.py script. This process is idempotent and will
# only load more data into MongoDB if any of the following happens:
#     - The number of records in the MongoDB collection matching the data_name variable has changed.
#     - The value of the data_name variable changes.
#     - The number of records in the OSM file matching data_name changes.

import data as data_processor
data = data_processor.process_map(OSMFILE, True)

from pymongo import MongoClient
client = MongoClient("mongodb://localhost:27017")
db = client.oxford_england_sample
collection = getattr(db, data_name)

collection_count = collection.count()

# If the collection size dowsn't match the data size, load/reload the data.
if collection_count != len(data):
    if collection_count > 0:
        collection.drop()
    collection.insert_many(data)

collection_count = collection.count()
sample_record = collection.find_one()
print "Number of records in the {} MongoDB collection: {}".format(data_name, collection_count)
print "A sample record from the {} MongoDB collection: {}".format(data_name, sample_record)
```

    Number of records in the oxford_england MongoDB collection: 321322
    A sample record from the oxford_england MongoDB collection: {u'created': {u'changeset': u'10706805', u'version': u'4', u'uid': u'27408', u'timestamp': u'2012-02-16T23:08:31Z', u'user': u'Andrew Chadwick'}, u'pos': [51.6994959, -1.2645627], u'visible': None, u'_id': ObjectId('57963746d0958625140ac407'), u'type': u'node', u'id': u'194502'}


## Overview of the data


```python
# Size of the OSM and JSON files.
import os
def get_size_in_mb_of_relative_file(file_name):
    wd = %pwd
    return os.stat(wd + '/' + file_name).st_size / 1000.0 / 1000.0

for file_name in [OSMFILE, OSMFILE + '.json']:
    file_size = get_size_in_mb_of_relative_file(file_name)
    print "{} is {} MB in size.".format(file_name, file_size)
```

    oxford_england.osm is 66.214418 MB in size.
    oxford_england.osm.json is 98.967713 MB in size.



```python
# Number of records:
print "Number of records in the {} MongoDB collection: {}".format(data_name, collection_count)
```

    Number of records in the oxford_england MongoDB collection: 321322



```python
# Number of nodes:
num_nodes = collection.find({"type":"node"}).count()
print "Number of 'node' records in the {} MongoDB collection: {}".format(data_name, num_nodes)
```

    Number of 'node' records in the oxford_england MongoDB collection: 274664



```python
# Number of ways:
num_ways = collection.find({"type":"way"}).count()
print "Number of 'way' records in the {} MongoDB collection: {}".format(data_name, num_ways)
```

    Number of 'way' records in the oxford_england MongoDB collection: 46600



```python
# Number of unique users:
num_unique_users = len(collection.distinct("created.user"))
print "Number of unique 'users' in the {} MongoDB collection: {}".format(data_name, num_unique_users)
```

    Number of unique 'users' in the oxford_england MongoDB collection: 567



```python
# Number of unique amenity types:
num_unique_amenities = len(collection.distinct("amenity"))
print "Number of unique 'amenity' values in the {} MongoDB collection: {}".format(data_name, num_unique_amenities)
```

    Number of unique 'amenity' values in the oxford_england MongoDB collection: 127



```python
# Number of unique cuisine types:
num_unique_cuisines = len(collection.distinct("cuisine"))
print "Number of unique 'cuisine' values in the {} MongoDB collection: {}".format(data_name, num_unique_cuisines)
```

    Number of unique 'cuisine' values in the oxford_england MongoDB collection: 57



```python
# Number of universities and colleges:
num_colleges = collection.find({"amenity": {"$in": ["university", "college"]}}).count()
print "Number of 'university' and 'college' records in the {} MongoDB collection: {}".format(data_name, num_colleges)
```

    Number of 'university' and 'college' records in the oxford_england MongoDB collection: 119


## Other ideas about the datasets

Oxford is a very interesting city not simply because of the history and prestige of Oxford University, but also because of its density and diversity of amenities woven within a fabric of well integrated network of pedestrian, bicycle, and public transport routes. It would be very interesting to explore the relationships between these amenities and the various networks that allow people to move around the city.

The outcome of analysis could be very helpful in planning activities and events within the city as well as providing information about which parts of the city are likely to be busiest during the tourist season.

That said, the OSM data for Oxford may not make such an analysis very easy to undertake. For example, data about the bus and rail networks are encoded using the NAPTAN schema ([NAPTAN stands for National Public Transport Access Nodes](https://www.gov.uk/government/publications/national-public-transport-access-node-schema)) but this information does not always contain information about street names.


```python
print "A bus stop node without street information."
print collection.find_one({"naptan": {"$exists": 1}, "highway": "bus_stop"})
print "\n"
print "A bus stop node with street information."
print collection.find_one({"naptan.Street": {"$exists": 1}, "highway": "bus_stop"})
```

    A bus stop node without street information.
    {u'direction': u'W', u'name': u'Herschel Crescent', u'journeys': u'6', u'created': {u'changeset': u'8007524', u'version': u'6', u'uid': u'74570', u'timestamp': u'2011-04-29T22:38:18Z', u'user': u'Richard Mann'}, u'pos': [51.7249536, -1.2152904], u'visible': None, u'frequency': u'0', u'naptan': {u'Bearing': u'W'}, u'source': u'photograph', u'bus_routes': u'16A', u'_id': ObjectId('57963746d0958625140add4d'), u'type': u'node', u'id': u'16640101', u'highway': u'bus_stop'}


    A bus stop node with street information.
    {u'direction': u'W', u'name': u'Long Lane', u'journeys': u'6', u'created': {u'changeset': u'8007524', u'version': u'6', u'uid': u'74570', u'timestamp': u'2011-04-29T22:38:18Z', u'user': u'Richard Mann'}, u'pos': [51.7249171, -1.2181715], u'note': u'NAPTAN node in wrong place, and erroneously marked CUS', u'visible': None, u'frequency': u'0', u'naptan': {u'Bearing': u'W', u'Indicator': u'Opp Sheldon Way', u'BusStopType': u'CUS', u'CommonName': u'Long Lane', u'PlusbusZoneRef': u'OXFD', u'Street': u'Long Lane', u'Landmark': u'Sheldon Way', u'AtcoCode': u'340000958OPP', u'NaptanCode': u'oxfatjdj'}, u'source': u'naptan_import;photograph', u'bus_routes': u'16A', u'_id': ObjectId('57963746d0958625140add54'), u'type': u'node', u'id': u'16640114', u'highway': u'bus_stop'}


### The top 10 most common amenities


```python
results = collection.aggregate([
        {"$match": {"amenity": {"$exists": 1}}},
        {"$group": {"_id": "$amenity", "count": {"$sum": 1}}},
        {"$sort": {"count": -1}},
        {"$limit": 10}
    ])
for result in results:
    print result
```

    {u'count': 741, u'_id': u'parking'}
    {u'count': 521, u'_id': u'bicycle_parking'}
    {u'count': 332, u'_id': u'post_box'}
    {u'count': 187, u'_id': u'bench'}
    {u'count': 164, u'_id': u'pub'}
    {u'count': 163, u'_id': u'place_of_worship'}
    {u'count': 142, u'_id': u'telephone'}
    {u'count': 130, u'_id': u'restaurant'}
    {u'count': 110, u'_id': u'cafe'}
    {u'count': 102, u'_id': u'school'}


### The top 10 most common types of cuisine

For food nodes where the cuisine is known.


```python
results = collection.aggregate([
        {"$match": {"cuisine": {"$exists": 1}}},
        {"$group": {"_id": "$cuisine", "count": {"$sum": 1}}},
        {"$sort": {"count": -1}},
        {"$limit": 10}
    ])
for result in results:
    print result
```

    {u'count': 31, u'_id': u'chinese'}
    {u'count': 26, u'_id': u'sandwich'}
    {u'count': 20, u'_id': u'indian'}
    {u'count': 15, u'_id': u'coffee_shop'}
    {u'count': 11, u'_id': u'fish_and_chips'}
    {u'count': 11, u'_id': u'burger'}
    {u'count': 11, u'_id': u'pizza'}
    {u'count': 10, u'_id': u'italian'}
    {u'count': 6, u'_id': u'thai'}
    {u'count': 5, u'_id': u'asian'}


### Top 10 most commonly found places to eat


```python
results = collection.aggregate([
        {"$match": {"amenity": {"$in": ["restaurant", "cafe", "pub", "bar", "fast_food", "delicatessen"]}}},
        {"$match": {"name": {"$exists": 1}}},
        {"$group": {
                "_id": "$name",
                "amenity": {"$first": "$amenity"},
                "cuisine": {"$push": "$cuisine"},
                "count": {"$sum": 1}
            }},
        {"$project": {
                "_id": 0,
                "count": 1,
                "name": "$_id",
                "type" : {"$concat": [
                        "$amenity", " - ",
                        {"$ifNull": [{"$arrayElemAt": ["$cuisine", 0 ]}, "unknown cuisine"] }
                    ]}
            }},
        {"$sort": {"count": -1}},
        {"$limit": 10}
    ])

for result in results:
    print "Count {}: {} ({})".format(result['count'], result['name'], result['type'])
```

    Count 6: The Red Lion (pub - unknown cuisine)
    Count 5: La Croissanterie (cafe - unknown cuisine)
    Count 5: Taylors (restaurant - sandwich)
    Count 4: The White Hart (pub - unknown cuisine)
    Count 3: Mission Burrito (fast_food - mexican)
    Count 3: McDonald's (fast_food - burger)
    Count 3: Subway (fast_food - sandwich)
    Count 3: Pizza Hut (restaurant - pizza)
    Count 3: Mortons (cafe - sandwich)
    Count 3: Costa Coffee (cafe - unknown cuisine)


It is interesting to note that Asian cuisines occupy 4 of the top top types of cuisine (1: Chinese, 3: Indian, 9: Thai, 1: Asian), yet there isn't a single business offering Asian cuisine in the top 10 most commonly found places to eat. This begins to illustrate the trend that even though business offering Asian cuisine are numerous, they tend to be independent establishments rather than franchises or chains.

### The total reported bicycle parking capacity

We can't make any assumptions for records where 'capacity' is known.


```python
from bson.code import Code

result = collection.inline_map_reduce(
    Code("function() {"
         "    if (this.capacity) {"
         "        emit('total_bicycle_parking_capacity', Number(this.capacity));"
         "    }"
         "}"),
    Code("function(key, values) {"
         "    var total = 0;"
         "    for (var i = 0; i < values.length; i++) {"
         "        total += values[i];"
         "    }"
         "    return total;"
         "}"),
    query = {"amenity": "bicycle_parking"});
result
```




    [{u'_id': u'total_bicycle_parking_capacity', u'value': 8756.0}]



### The 5 streets with the most bus stops

For bus stop nodes where the street name is known.


```python
results = collection.aggregate([
        {"$match": {"naptan": {"$exists": 1}, "highway": "bus_stop"}},
        {"$group": {"_id": "$naptan.Street", "num_bus_stops": {"$sum": 1}}},
        {"$project": {"_id": 0, "num_bus_stops": 1, "street_name": "$_id"}},
        {"$sort": {"num_bus_stops": -1}},
        {"$limit": 5}
    ])
for result in results:
    print result
```

    {u'street_name': u'Oxford Road', u'num_bus_stops': 43}
    {u'street_name': u'Banbury Road', u'num_bus_stops': 40}
    {u'street_name': u'Woodstock Road', u'num_bus_stops': 33}
    {u'street_name': u'High Street', u'num_bus_stops': 23}
    {u'street_name': u'Cowley Road', u'num_bus_stops': 20}


## Conclusion

I am pleasantly surprised to see just how much information about Oxford is encoded within the OSM data. That said, the inevitable nature of crowd sourced information is clearly present. There is quite a lot of inconsistency in the data (not all illustrated in this project) as well as records that do little more than act as meta information about the creation of the data itself (such as 'fixme' and 'note').


```python
meta_keys = [key for key in unique_key_names if 'fixme' in key.lower() or 'note' in key.lower()]
percentage = '%.2f' % (float(len(meta_keys)) / len(unique_key_names) * 100)
print "There are {} 'meta' tag key names making up {}% of all unique tag key names.".format(len(meta_keys), percentage)
print "\n"
print [key for key in unique_key_names if 'fixme' in key.lower() or 'note' in key.lower()]
```

    There are 78 'meta' tag key names making up 7.72% of all unique tag key names.


    ['note2', 'Fixme:responce_2', 'frequency:note', 'note:17', 'note:14', 'note:15', 'note:13', 'note:10', 'note:11', 'notes', 'oneway:note', 'building:levels:underground:note', 'continental_geometry:note', 'parking:fixme', 'FIXME:nsl', 'note:highway', 'parking:note', 'fixme:lane', 'Note:2', 'maxspeed:note', 'FIXME', 'note:bicycle', 'note:layer', 'Fixme:2', 'fixme:responce_3', 'fixme:responce_5', 'fixme:responce_4', 'note:alt_name', 'fixme', 'note:designation', 'note:cont', 'note:crossing', 'name:note', 'highway:note', 'naptan:Notes', 'FIXME:responce', 'FIXME:cont', 'note:reply', 'note:name', 'note', 'layer:note', 'note:16', 'note:barrier', 'not:name:note', 'bicycle:note', 'note:geometry', 'note:frequency', 'note:surface', 'alignment_note', 'note:park_and_ride', 'note:0.1', 'note:0.2', 'Note', 'crossing:note', 'payment:notes', 'note:4', 'note:5', 'note:6', 'note:7', 'note:0', 'note:2', 'note:3', 'note:8', 'note:9', 'note:cycleway', 'FIXME:responce_1', 'note:12', 'cycleway:right:note', 'note:route_ref', 'note:healthcare', 'note 2', 'fee:note', 'note_3', 'note_2', 'note_1', 'cycleway:note', 'Fixme', 'fixme:license']


### Resources

The production of this project was aided by information found on the following website (various authors and contributors)

- [Udacity.com](udacity.com)
- [Docs.MongoDB.com](docs.mongodb.com)
- [API.MongoDB.com/python](api.mongodb.com/python)
- [StackOverflow.com](stackoverflow.com)
