# Lucene Index

## Basic Concepts
* Lucene is a full-text search library in Java which makes it easy to add search functionality to an application or website.
* It does so by adding content to a full-text index. It then allows you to perform queries on this index, returning results ranked by either the relevance to the query or sorted by an arbitrary field such as a document's last modified date.
* This type of index is called an inverted index, because it inverts a page-centric data structure (page->words) to a keyword-centric data structure (word->pages).

### Inverted Index - the core building block of text search
* A kind of collection that allows us to quickly lookup a specific term and it's associated posting list.
	* posting list - a list of the numeric IDs of documents which contain that term.

* Documents -analysis-> Tokens -indexing-> Inverted Index

### Scoring - sorting documents by match quality
* by default, Lucene uses the Vector Space Model with TF-IDF weights
	* TF: Term Frequency: for each query term, how often that term appears in a document
	* IDF: Inverse Document Frequency:

### Documents
* In Lucene, a Document(collection of fields) is the unit of search and index. An index consists of one or more Documents.	
* Indexing involves adding Documents to an IndexWriter, and searching involves retrieving Documents from an index via an IndexSearcher.

### Fields
* A Field is simply a name-value pair. A Document consists of one or more Fields.
* Fields are stored separately from the index. Field storage: .fdt files
* A Field may be either:
    * indexed and stored
    * Indexed but not stored
    * or, stored but not indexed: something we like to display in the search result but not used as query parameters

	
### Segment
* Most commonly a lucene index is stored as a collection of files
	* The files are immutable
		* pros: reduce the possibility of index corruption
		* cons: update in a index is less efficient
			 when documents are added to an existing index.
* Segment
	* A logical separate set of index files
	* A single Lucene index may be split into multiple segments
	* Each segment has all the information of an independent index(e.g. inverted index of terms)
	* Each segment stores one or more of the documents in its own inverted index and field storage
	* Query with segments
		* The segments will be queried separately.
		* Lucene performs the same query on each segment but collects all the hits into one result set.

	* Lucene periodically merge segments together
		* e.g. Lucene combines the data of segments A and B to create segment C. Once segment C is fully written to disk, Lucene deletes segments A and B.
	* The ideal number of segments is usually one unless you have a very large index
	* Since the merging process is quite costly, we don't want merging performed every time the index is updated.
		* MergePolicy: findMerges() invoked when a new segment is created and returns a list of segments to merge
		* MergeScheduler: processes the merges, sometimes in separate threads

#### Delete Documents from Existing Segments
* When deleted, a document is marked for deletion. It actually gets removed only when its segment gets merged.

#### Update Documents
 * Updates require writing a new segment(existed are immutable)
	* single-doc updates are costly, bulk updates preferred
	* `writes are sequential`
	
* Segments are never modified in place
   	* filesystem-cache-friendly
   	* `lock-free!
	
### Searching
* Searching requires an index to have already been built. It involves creating a Query (usually via a QueryParser) and handing this Query to an IndexSearcher, which returns a list of Hits.

#### Faceting
* Compute value counts for docs that match a query. e.g. category counts on an ecommerce website

### Queries
* Lucene has its own mini-language for performing searches. 
* The Lucene query language allows the user to specify which field(s) to search on, which fields to give more weight to (boosting), the ability to perform boolean queries (AND, OR, NOT) and other functionality. 
* [Lucene Query Syntax](http://www.lucenetutorial.com/lucene-query-syntax.html)
* Query types: Term query, wildcard query, prefix query, fuzzy query, phrase query, range query, and boolean query.

## References
* [LuceneTutorial.com](http://www.lucenetutorial.com/basic-concepts.html)
* [Text search with Lucene (1 of 2)](https://youtu.be/x37B_lCi_gc)
* [Text search with Lucene (2 of 2)](https://youtu.be/fCK9U3L7c8U)
* [What is in a Lucene index? Adrien Grand, Software Engineer, Elasticsearch](https://youtu.be/T5RmMNDR5XI)