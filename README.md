# Product-Content Based Filtering Recommendation System
# Data Set: Multi-aspect Reviews of Beer.

## Data Analysis
* Metadata of the data
    - `beer/name`: The name of the beer.
    - `beer/beerId`: Unique identifier for each beer.
    - `beer/brewerId`: Unique identifier for the brewer of the beer.
    - `beer/ABV`: Alcohol By Volume (ABV) of the beer, indicating the percentage of alcohol present in the beverage.
    - `beer/style`: Style of the beer, such as India Pale Ale (IPA), Bohemian Pilsener, Kölsch, etc.
    - `review/appearance`: Rating for the appearance of the beer, typically on a scale from 0 to 5.
    - `review/aroma`: Rating for the aroma of the beer, typically on a scale from 0 to 5.
    - `review/palate`: Rating for the palate of the beer, typically on a scale from 0 to 5.
    - `review/taste`: Rating for the taste of the beer, typically on a scale from 0 to 5.
    - `review/overall`: Overall rating of the beer, typically on a scale from 0 to 5.
    - `review/time`: Timestamp of the review.
    - `review/profileName`: Username or profile name of the reviewer.
    - `review/site`: Website where the review was posted.
    - `review/id`: Unique identifier for each review.

* One beer (distinguished by its name) can have more than one beerId (and more than one corresponding brewerId).
* A beer having one beerId can have both: a non-NaN ABV value corresponding to a beerId and a NaN ABV value corresponding to another beerId.
 Both `ratebeer` & `beeradvocate` have identical column names conveying the same info but the columns maybe of different data type:


|                | review/appearance  |    review/aroma     |   review/palate    |    review/taste     |    review/overall    |
| :------------: | :----------------: | :-----------------: | :----------------: | :-----------------: | :------------------: |
|   `ratebeer`   | `string`,eg: "4/5" | `string`,eg: "6/10" | `string`,eg: 3"/5" | `string`,eg: "6/10" | `string`,eg: "13/20" |
| `beeradvocate` |  `float` (max: 5)  |  `float` (max: 5)   |  `float` (max: 5)  |  `float` (max: 5)   |   `float` (max: 5)   |

* Strategy is to convert the `ratebeer` data values to number after and scaling them down to a scale of 0 to 5.


* Number of reviews per beer

|       |         |
| :---: | :-----: |
| count | 160803  |
| mean  |  28.05  |
|  std  | 131.60  |
|  min  |  1.00   |
|  25%  |  1.00   |
|  50%  |  4.00   |
|  75%  |  12.00  |
|  max  | 5906.00 |


* Filtering data based on the minimum number of reviews for each beer:

| Threshold (>) |  Unique Beers  |  Total Reviews   | Unique Beer Styles |
| :-----------: | :------------: | :--------------: | :----------------: |
|       1       | 119317 (74.2%) | 4469291 (99.08)  |        179         |
|       4       | 72540 (45.1%)  | 4341357 (96.244) |        179         |
|       5       | 65078 (40.4%)  | 4304047 (95.417) |        179         |
|      10       | 43912 (27.3%)  | 4142054 (91.826) |        179         |
|      12       | 39164 (24.35%) | 4087609 (90.619) |        179         |
|      20       | 27462 (17.1%)  | 3900701 (86.475) |        178         |
|      50       | 13777 (8.56%)  | 3465012 (76.816) |        177         |
|      60       | 11871 (7.38%)  | 3359776 (74.483) |        170         |
|      100      |  7912 (4.92%)  | 3052487 (67.671) |        165         |
|      200      |  4376 (2.71%)  | 2555620 (56.656) |        163         |
|      500      |  1605 (0.99%)  | 1689674 (37.459) |        146         |
|     1000      |  590 (0.36%)   | 983847  (21.81)  |        125         |

* Number of reviews per user

|       |          |
| :---: | :------: |
| count |  60786   |
| mean  |  74.20   |
|  std  |  358.04  |
|  min  |   1.00   |
|  25%  |   1.00   |
|  50%  |   3.00   |
|  75%  |  17.00   |
|  max  | 16364.00 |

* Filtering data based on the minimum number of users for each beer:


| Threshold (>) | Unique Profiles |  Total Reviews   | Unique Beer Styles |
| :-----------: | :-------------: | :--------------: | :----------------: |
|       1       | 41898 (68.92%)  | 4491889 (99.58%) |        179         |
|       4       | 26958 (44.34%)  | 4451723 (98.69%) |        179         |
|       5       | 24741 (40.70%)  | 4440638 (98.44%) |        179         |
|      10       | 18705 (30.77%)  | 4394208 (97.41%) |        179         |
|      12       | 17321 (28.49%)  | 4378352 (97.06%) |        179         |
|      20       | 13901 (22.86%)  | 4323556 (95.84%) |        179         |
|      50       |  9323 (15.33%)  | 4176513 (92.59%) |        179         |
|      60       |  8590 (14.13%)  | 4135868 (91.68%) |        179         |
|      100      |  6666 (10.96%)  | 3984283 (88.32%) |        179         |
|      200      |  4258 (7.00%)   | 3640468 (80.70%) |        179         |
|      500      |  2034 (3.34%)   | 2931985 (65.0%)  |        179         |
|     1000      |  1012 (1.66%)   | 2211079 (49.01%) |        179         |


## Feature Extraction
* Data analysis revealed that we have a catalogue of 60k unique products with 19k unique users and 4.5 million reviews.
* No metadata of products is in the dataframe. Instead, textual reviews are provided.
* The reviews are very detailed and describe various aspects of the product.
* Therefore, those reviews are mined to extract keywords: a pair of (adjective, noun) describing sentiment and aspect. These pair along with their frequency becomes the feature of the product.
* The reviews were first filtered for non-english sentences. Around 48k reviews in French, German, Spanish were filtered out.
* For mining the pairs, we used SpaCy along with it's `en_core_lg` model.
* The dependency graph of each review sentence was tokenized adn parsed to painstakingly extract pairs from every review.
* The whole corpus of reviews was parsed forun 13.99 lakh unique pairs. 
* We employed various methods to reduce the number of features without losing the semantics significance of each pair:
  * basic frequency based filtering algorithm 
  * training GloVe model on review corpus and clustering similar pairs
  * perfomring SVD on the feature matrix
* We reduced the number of features from 13.99 lakh to 43k to 14k to 2500 to 1587

## Representing Item/Users
* Due to the lack of product metadata we decided to extract keywords from product reviews and represent products based on the keywords in their reviews and the numerical rating provided.
* Users were repsented on a per product basis (represented for each product they review) by using the keyword in their reviews. This lead to an extremely sparse matrix as compared to the product matrix.

## Content Based System:
* We followed a kNN based approad:
  * Given a user, we find her most liked items (rating >=4)
  * For each item in candidate set, we find it's similarity for all items in user liked item list. Then we take the average of those simmilarity and that becomes the score of that candidate item.
  * This process is repeated for all candidate items and the top-k item with highest similarity are recommended. 

## Novelty/Insights:
* Special attention was given to keep the code as efficient as possible given the scale of the data
  * Scale of the data: 59k unique products with total of 4.5M reviews
  * Dataframe was saved as a parquet file instead of csv/json.
  * Intermediate dictionaries were saved as pickle file instead of recalculating them on each run
  * Resource heavy calculations were delegated to cloud computing and IRLP lab systems
* Parsing reviews using dependency graphs for feature extraction (sentiments-aspect) using spaCy pre-trained models (16+ hours of processing time)
* Training using GloVe model. Used 3 different methods for training:
  * Explicitly on review corpus (unigrams)
  * Explicitly on review corpus (bigrams) (6 hours of processing time)
  * On review corpus along with added vocabulary of GloVe 840B 300D model
* Reducing the dimension by clustering the sentiment-aspect pairs which are semantically identical using GloVe model
* Scaling down the frequency and discounting terms which occur in too many reviews using tf-idf

## Evaluation:
* Instead of using Jaccard coefficient we used Overlap coefficient of (sentiment, aspect) pairs between recommended and liked items
* Based on the given aspects by the user we are filtering our catalog and generating the candidate set
* Intra list diversity
* Serendipity (Unexpectedness)
* Review summarization of the recommended items
* Front-end implementation on a subset of raw data during the computation of the more advanced model
* Future direction: will also try to deploy the advanced model 
