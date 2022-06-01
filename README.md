# Semantic Table Search Dataset
Resource: A Large Scale Test Corpus for Semantic Table Search

## Table Corpus
The full table corpus of 238,038 Wikipedia tables can be downloaded via this <a href="https://www.dropbox.com/s/7vii3pdue5suxjj/tables.tar.gz?dl=0">link</a>.

## Query Tables
The list of all 9296 query tables can be found under the [queries/](queries/) directory.
Each query table comes from unique Wikipedia Page and each query table is parsed and presented as a list of tuples of DBpedia entities.
All query files are named based on the Wikipedia page ID from which their query table was taken from.
The mappings of the Wikipedia page ID to the Wikipedia Page name and list of Wikitables contained in it is found in under the [queries.pickle](queries/wikipages_df.pickle) pickled dataframe.
We also provide our queries converted as keywords that were used by BM25 at [keyword_queries/](queries/keyword_queries/)

The directory contains the following files/sub-directories:
* `1_tuples_per_query/`: Contains all 9296 query tables with 1 tuple per query table 
* `2_tuples_per_query/`: Contains all 9296 query tables with 2 tuples per query table 
* `5_tuples_per_query/`: Contains all 9296 query tables with 5 tuples per query table 
* `10_tuples_per_query/`: Contains all 9296 query tables with 10 tuples per query table 
* `all_tuples/`: Contains all 9296 query tables with all their parsed tuples of entities
* `queries.pickle`: A pickled dataframe that contains mappings of each Wikipedia page used in our queries to its ID, the Wikitable selected for our query as well as all the webtables present in that Wikipedia page
* `keyword_queries/`: Contains all 9296 query tables but converted into keywords for 1, 2, 5 and 10 tuples per query. The `queries.txt` file is indexed by the Wikipedia page ID (i.e., the ID of our query table) and maps to the list of keywords


## Ground-Truth (Relevance Assessments)