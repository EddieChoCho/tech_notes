# Elasticsearch

## Precision and Recall are inversely related

* Precision: I want all the retrieved results to be a perfect match to the query, even if it means returning less
  documents.

```
Precision = True positives / (True positives + False positives)
```

* Recall: I want to retrieve results eve. if documents may not be a perfect match to the query.(quantity)

```
Recall = True positives / (True positives + False negatives)
```

## What is a score?

* The score is a value that represents how relevant a document is to that specific query
* A score is computed for each document that is a hit

* Term Frequency(TF): determines how many times each search term appears in a document. If search terms are found in
  high frequency in a document, the document is considered more relevant to the search query.
* Inverse Document Frequency(IDF): IDF diminishes the weight terms that occur very frequently in the document set and
  increases the weight of terms that occur rarely!