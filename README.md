# Semantic Table Search Dataset
Resource: A Large Scale Test Corpus for Semantic Table Search

## Table Corpus
The full table corpus of 238,038 Wikipedia tables from 2013 can be found under the [table_corpus/tables_2013/](table_corpus/tables_2013/) directory. 
Alternatively it can also be downloaded via this <a href="https://www.dropbox.com/s/7vii3pdue5suxjj/tables.tar.gz?dl=0">link</a>.

The corresponding table corpus of 457,714 Wikipedia tables from 2019 can be found under the [table_corpus/tables_2019/](table_corpus/tables_2019/) directory.

We also provide a mapping of each Wikipedia page to its ID and the list of Wikipedia Tables present within it in the [wikipages_df_2013.pickle](table_corpus/wikipages_df_2013.pickle) and [wikipages_df_2019.pickle](table_corpus/wikipages_df_2019.pickle) pickled dataframes.

We additionally also provide the same tables in CSV format which can be extracted from [table_corpus/csv_tables_2013](table_corpus/csv_tables_2013) and [table_corpus/csv_tables_2019](table_corpus/csv_tables_2019).

## Query Tables
The list of all 9296 query tables can be found under the [queries/](queries/) directory.
Each query table comes from unique Wikipedia Page and each query table is parsed and presented as a list of tuples of DBpedia entities.
All query files are named based on the Wikipedia page ID from which their query table was taken from.
The mappings of the Wikipedia page ID to the Wikipedia Page name and list of Wikitables contained in it is found in under the `queries/queries_df.pickle` pickled dataframe.
We also provide our queries converted as keywords that were used by BM25 at [keyword_queries/](queries/keyword_queries/)

The [queries/](queries/) directory contains the following files/sub-directories:
* `1_tuples_per_query/`: Contains all 9296 query tables with 1 tuple per query table 
* `2_tuples_per_query/`: Contains all 9296 query tables with 2 tuples per query table 
* `5_tuples_per_query/`: Contains all 9296 query tables with 5 tuples per query table 
* `10_tuples_per_query/`: Contains all 9296 query tables with 10 tuples per query table 
* `all_tuples/`: Contains all 9296 query tables with all their parsed tuples of entities
* `queries.pickle`: A pickled dataframe that contains mappings of each Wikipedia page used in our queries to its ID, the Wikitable selected for our query as well as all the webtables present in that Wikipedia page
* `keyword_queries/`: Contains all 9296 query tables but converted into keywords for 1, 2, 5 and 10 tuples per query. The `queries.txt` file is indexed by the Wikipedia page ID (i.e., the ID of our query table) and maps to the list of keywords

## Ground-Truth (Relevance Assessments)
The ground-truth relevance assessments for all 9296 query tables can be found under the [ground_truth/](ground_truth/) directory.
There are two ground-truth relevance assessments (one with respect to the Wikipedia Categories and one with respect to the Navigation Links).
For each version we provide a .json file for each query (i.e., each Wikipedia page) and denote all its semantically relevant Wikipedia Pages for which our measure is non-zero.
In other words, given a Wikipedia page we identify all other Wikipedia pages for which there is a non-zero intersection of Wikipedia Categories or Navigation Links

We also provide the extracted Wikipedia Categories for each Wikipedia Page in [wikipage_to_categories/](ground_truth/wikipage_to_categories/) and the extracted Navigation links for each Wikipedia Page in [wikipage_to_navigation_links/](ground_truth/wikipage_to_navigation_links/)  

The [ground_truth/](ground_truth/) directory contains the following files/sub-directories:
* `wikipedia_categories/`: Contains the relevance assessments for each Wikipedia page based on the Jaccard of their Wikipedia Categories
* `navigation_links/`: Contains the relevance assessments for each Wikipedia page based on the Jaccard of their Navigation Links
* `wikipage_to_categories/`: Contains the extracted Wikipedia Categories for each Wikipedia Page in our corpus
* `wikipage_to_navigation_links`: Contains the extracted Navigation Links for each Wikipedia Page in our corpus

Below is a snippet of a subset of the ground truth tables for a query.

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/ground_truth_example.png" width="600">

### Retreiving Ground Truth of Query
Below is a code snippet of a Python function to retrieve ground truth of a query. Given a query file, it returns the query and the ground truth judgements of each table, along the tables themselves. All compresses table folders are assumed to be extracted.

```python
import json
import os
import pickle

# Returns: query, [relevance score, table]
def ground_truth(query_filename, ground_truth_folder, table_corpus_folder, pickle_mapping_file):
    query = None
    relevances = list()
    wikipages = None
    mapping = None

    with open(query_filename, 'r') as file:
        query = json.load(file)['queries']

    with open(ground_truth_folder + '/' + query_filename.split('/')[-1].split('_')[1], 'r') as file:
        wikipages = json.load(file)

    with open(pickle_mapping_file, 'rb') as file:
        mapping = pickle.load(file)

    for key, value in wikipages.items():
        table_folders = os.listdir(table_corpus_folder)
        wikipage = 'https://en.wikipedia.org/wiki/' + key
        tables = None

        for key in mapping['wikipage'].keys():
            if wikipage == mapping['wikipage'][key]:
                tables = mapping['tables'][key]
                break

        if (tables == None):
            tables = []

        for table in tables:
            for table_folder in table_folders:
                if '.' in table_folder:
                    continue

                table_files = os.listdir(table_corpus_folder + '/' + table_folder)

                if table in table_files:
                    with open(table_corpus_folder + '/' + table_folder + '/' + table, 'r') as file:
                        json_table = json.load(file)['rows']
                        relevances.append([value, json_table])

    return query, relevances
```

**Arguments**:
- `query_filename`: The absolute path to a query file
- `ground_truth_folder`: Folder of ground truth files (folders of categories of navigational links)
- `table_corpus_folder`: Folder of uncompressed table corpus folders
- `pickle_mapping_file`: Pickle file of mappings from Wikipedia page ID to Wikipedia page name its tables

An example of calling the Python snippet function using Wikipedia page categories is below:

```
query_ground_truth = ground_truth('queries/1_tuples_per_query/wikipage_13042.json', 'ground_truth/wikipedia_categories', 'table_corpus/tables', 'table_corpus/wikipages_df.pickle')
```

## DBpedia Knowledge Graph
We link Wikipedia entities in Wikipedia tables to DBpedia knowledge graph entities. Wikipedia entities in Wikipedia tables have a hyperlink referring to the Wikipedia page of the entity. Similarly, DBpedia entities have the property _foaf:isPrimaryTopicOf_ pointing the DBpedia entity to the corresponding Wikipedia entity page. We can therefore link Wikipedia table entities to DBpedia entities when both entities point to the same Wikipedia entity page.

We then use the DBpedia knowledge graph to measure similarities between pairs of entities, either by the Jaccard coefficient or by cosine similarity of knowledge graph entity embeddings. The files to download the DBpedia knowledge graph, including entity types, are found in `dbpedia_files.txt` from September 1st, 2021. The embeddings of the same DBpedia knowledge graph can be downloaded <a href="https://zenodo.org/record/6384728#.Ypm-G-5BwQ8">here</a>.

## Query Set Analsysis

**Distribution of Query Sizes**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/tuples-per-query.png">

**Query Tuple Width Distribution**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/query-tuple-width.png">

**Distribution of Relevant Tables Per Query**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/relevant-tables-per-query.png">

**Distribution of Relevant Wikipedia Pages Per Query**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/relevant-wikipages-per-query.png">

**Distribution of Categories Per Wikipedia Page**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/number-of-categories-per-wikipage.png">

**Distribution of Navigational Links Per Wikipedia Page**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/number-of-navigational-links-per-wikipage.png">

**Distribution of Wikipedia Pages per Category**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/wikipages-per-category.png">

**Embeddings Cosine Similarity Heatmap of a Selected Set of DBpedia Entities**

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/heatmap.png">
