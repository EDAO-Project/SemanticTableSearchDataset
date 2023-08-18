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
The ground-truth relevance assessments for all 9296 and 2560 query tables for the 2013 and 2019 Wikipedia tables, respecitvely, can be found under the [ground_truth/](ground_truth/) directory.
There are two ground-truth relevance assessments (one with respect to the Wikipedia Categories and one with respect to the Navigation Links).
For each version we provide a .json file for each query (i.e., each Wikipedia page) and denote all its semantically relevant Wikipedia Pages for which our measure is non-zero.
In other words, given a Wikipedia page we identify all other Wikipedia pages for which there is a non-zero intersection of Wikipedia Categories or Navigation Links

We also provide the extracted Wikipedia Categories for each Wikipedia Page in [wikipage_to_categories(2013)/](ground_truth/2013/wikipage_to_categories/) and [wikipage_to_categories(2019)/](ground_truth/2019/wikipage_to_categories/), and the extracted Navigation links for each Wikipedia Page in [wikipage_to_navigation_links(2013)/](ground_truth/2013/wikipage_to_navigation_links/) and [wikipage_to_navigation_links(2019)/](ground_truth/2019/wikipage_to_navigation_links/).

The [ground_truth/](ground_truth/) directory contains the following files/sub-directories for the 2013 and 2019 Wikipedia tables snapshots:
* `wikipedia_categories/`: Contains the relevance assessments for each Wikipedia page based on the Jaccard of their Wikipedia Categories
* `navigation_links/`: Contains the relevance assessments for each Wikipedia page based on the Jaccard of their Navigation Links
* `wikipage_to_categories/`: Contains the extracted Wikipedia Categories for each Wikipedia Page in our corpus
* `wikipage_to_navigation_links`: Contains the extracted Navigation Links for each Wikipedia Page in our corpus

Below is a snippet of a subset of the ground truth tables for a query, where tables that cannot be found by keyword search are marked with red borders.

<img src="https://github.com/EDAO-Project/SemanticTableSearchDataset/blob/main/figures/gt_example.png" width="600">

### Retreiving Ground Truth of Query
Below is a code snippet of a Python function to retrieve ground truth of a query. Given a query file, it returns the query and the ground truth judgements of each table, along the tables themselves. All compresses table folders are assumed to be extracted.

```python
import json
import os
import pickle

# Returns: query, [relevance score, table]
def ground_truth(query_filename, ground_truth_folder, pickle_mapping_file):
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

    for key in mapping['wikipage'].keys():
        wiki_id = mapping['wikipage'][key].split('/')[-1]

        if wiki_id in wikipages.keys():
            relevance = wikipages[wiki_id]

            for table in mapping['tables'][key]:
                relevances.append([relevance, table])

    return query, relevances
```

**Arguments**:
- `query_filename`: The absolute path to a query file
- `ground_truth_folder`: Folder of ground truth files (folders of categories of navigational links)
- `pickle_mapping_file`: Pickle file of mappings from Wikipedia page ID to Wikipedia page name and its tables

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

## Reproducing Evaluation Results
We first describe the steps to setup this benchmark.
Then, we describe how to execute the benchmark.

### Setup
First, decompress the necessary files using the following commands

```bash
cd ground_truth/2013/wikipage_to_categories/
tar -xf wikipage_to_categories.tar.gz
rm wikipage_to_categories.tar.gz
cd ../wikipage_to_navigation_links/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; done
```

Concatenate the Wikipage-to-navigational links into a single JSON file using the following Python script

```python
import os
import json

files = os.listdir('.')
concat = dict()

for file in files:
    if not '.json' in file:
        continue

    with open(file, 'r') as handle:
        lst = json.load(handle)

        for wikipage in lst.keys():
            concat[wikipage] = lst[wikipage]

with open('wikipage-to-navigational-link.json', 'w') as handle:
    json.dump(concat, handle, indent = 4)
```

This Python script can also be used to concatenate the JSON files for the 2019 ground truth.

Now, enter the 2019 ground truth which requires a bit more work.

```bash
cd ../../2019/navigational_links/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; mv ${F:0:-7}/* . ; rmdir ${F:0:-7} ; done
cd ../wikipedia_categories/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; mv ${F:0:-7}/* . ; rmdir ${F:0:-7} ; done
cd ../wikipage_to_categories/
tar -xf wikipage_to_categories.tar.gz
rm wikipage_to_categories.tar.gz
cd ../wikipage_to_navigation_links/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; done
cd ../../../
```

Now, the ground truth is fully prepared.
We will now move on to the two table corpora from 2013 and 2019.
First, extract to table-to-entity links files.

```bash
cd table_corpus/
tar -xf tableIDToEntities_2013.ttl.tar.gz
tar -xf tableIDToEntities_2019.ttl.tar.gz
rm tableIDToEntities_*.ttl.tar.gz
```

Run the following commands to extract the tables on CSV format first.

```bash
cd csv_tables_2013/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; mv csv_tables_2013/${F:0:-7}/* . ; done && rm -rf csv_tables_2013
cd ../csv_tables_2019/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; mv csv_tables_2019/${F:0:-7}/* . ; done && rm -rf csv_tables_2019
```

Now, run the following commands to extract the annotated JSON tables.

```bash
cd ../tables_2013/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; mv ${F:0:-7}/* . ; rmdir ${F:0:-7} ; done
cd ../tables_2019/
for F in ./* ; do tar -xf ${F} ; rm ${F} ; mv ${F:0:-7}/* . ; rmdir ${F:0:-7} ; done
cd ../../
```

The benchmark files are now fully extracted.
We can now move on to setting up the actual bechmark.

We evaluate two baselines on this benchmark: BM25 and TURL.
We first setup BM25.
Run the following commands with root access to install the necessary packages.

```bash
apt-get update
apt-get install software-properties-common -y
add-apt-repository ppa:deadsnakes/ppa
apt update
apt install python3.8.10
apt-get install curl tar net-tools nano htop wget openjdk-11-jdk -y
pip3 install pandas elasticsearch==5.5.3 scipy joblib tqdm nltk==3.4.5
```

Run the following commands to extract BM25 and download Elasticsearch.

```bash
tar -xf bm25.tar.gz
rm bm25.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.tar.gz
tar -xf elasticsearch-5.3.0.tar.gz
rm elasticsearch-5.3.0.tar.gz
mv elasticsearch-5.3.0 bm25/
```

Process the JSON table corpus into a single JSON file that we will use to construct the BM25 indexes.

```bash
cd bm25/
mkdir -p bm25_tables/
python3 json_converter.py --input_dir ../table_corpus/tables_2013/ --output_dir bm25_tables/ \
--table_id_to_entities_path ../table_corpus/tableIDToEntities_2013.ttl
```

You can substitute the `2013` with `2019` if you want to use the more recent Wikipedia tables.
We can now load the BM25 indexes.

```bash
elasticsearch-8.5.3/bin/elasticsearch &
sleep 2m
python3 indexer.py --index_name bm25_index --input bm25_tables/tables.json
cd ..
```

To stop the Elasticsearch instance running BM25 when you are done, run `kill $!`.

// TODO: Insert here instructions to setup benchmarking of TURL.

The benchmark is now fully prepared, and we can move on to execution the benchmark.

### Benchmark Execution
To execute BM25 over the queries in this benchmark for top-100, run the following commands.

```bash
cd bm25/
mkdir results/
python3 pool_ranker.py --output_dir ./results/ --input_dir ../queries/2013/1_tuples_per_qury/ --index_name bm25_index --topn 100
```

To use the queries based on the 2019 Wikipedia tables, substitute `2013` with `2019`.
The queries in this example are 1-tuple queries.
Change the number `1` to either `2`, `5`, or `10` to increase query size.
The results will be stores in the `results/` folder.

// TODO: Insert here instructions to run TURL on our queries.

Finally, we can start evaluating the results using NDCG and recall.
Create a Python3 script that contains the function for retrieving ground truth from the beginning of this README document.
Add the following function to retrieve the scores from BM25 and TURL.

```python
# Parameters:
#   - query_id: Name of query file
#   - tuples: Size of query
#   - k: Top-K value
#   - gt_tables: List of ground truth tables
#   - results_path: Path to results file
#   - approach: Either 'BM25' or 'TURL' depending on the result to retrieve scores from
#   - only_tables: If true, only the list of resuling tables are returns (use this to measure recall)
# Returns ranked list of scores for each corpus table
def results(query_id, tuples, k, gt_tables, results_path, approach = None, only_tables)
    if approach is None or (approach != 'BM25' and approach != 'TURL') or not os.path.exists(results_path):
        return None

    with open(path, 'r') as f:
        predicted = dict()

        if approach == 'BM25':
            for line in f:
                split = line.split('\t')
                qid = 'wikipage_' + split[0]
                table = split[2]
                score = float(split[4])

                if (qid == query_id.replace('.json', '')):
                    predicted[table] = score

        else:
            first = True

            for line in f:
                if first:
                    first = False
                    continue

                split = line.strip().split(',')
                predicted[split[0]] = float(split[1])

        scores = {table:0 for table in gt_tables}

        for table in predicted:
            scores[table] = predicted[table]

        sort = list(sorted(predicted.items(), key = operator.itemgetter(1), reverse = True))

        if only_tables:
            return [e[0] for e in sort][0:k]

        return list(scores.values())
```

We use the following function to compute recall.

```python
def recall(tables, gt):
    count = 0

    for table in tables:
        if table in gt:
            count += 1

    if len(tables) == 0:
        return 0

    if count > len(gt):
        return 1.0

    return float(count) / len(gt)
```

Now, to compute NDCG and recall, collect ground truth, load the ground truth into the corpus tables, collect baseline results, and measure recall and NDCG.

```python
import os
import numpy as np
from sklearn.metrics import ndcg_score

k = 100
query_file = 'queries/013/1_tuples_per_query/wikipage_192952.json'
gt_folder = 'round_truth/2013/wikipedia_categories/'
corpus = 'table_corpus/tables_2013/'
mapping = 'table_corpus/wikipages_df_2013.pickle'
gt = ground_truth(query_file, gt_folder, mapping)
gt_relevance_scores = {table:0 for table in os.listdir(corpus)}

for relevance in gt[1]:
    gt_relevance_scores[relevance[1]] = relevance[0]

gt_tables = gt[1]
gt_tables.sort(reverse = True, key = lambda rel: rel[0])
gt_tables = [table[1] for relevance in gt_tables][0:k]

bm25_results_ndcg = results('wikipage_207751', 1, k, None, 'bm25/results/content.txt', approach = 'BM25', False)
bm25_results_recall = results('wikipage_207751', k, 100, None, 'bm25/results/content.txt', approach = 'BM25', True)
ndcg_score = ndcg_score(np.array([list(gt_relevance_scores.values())]), np.array([bm25_results_ndcg]), k = k)
recall_score = recall(bm25_results_recall, gt_tables)
```

The variables `ndcg_score` and `recall_score` hold the NDCG and recall values for this example, respectively.
