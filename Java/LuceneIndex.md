## Lucene Index

### Basic Concepts
* Lucene is a full-text search library in Java which makes it easy to add search functionality to an application or website.
* It does so by adding content to a full-text index. It then allows you to perform queries on this index, returning results ranked by either the relevance to the query or sorted by an arbitrary field such as a document's last modified date.
* This type of index is called an inverted index, because it inverts a page-centric data structure (page->words) to a keyword-centric data structure (word->pages).

#### Documents
* In Lucene, a Document is the unit of search and index. An index consists of one or more Documents.
* Indexing involves adding Documents to an IndexWriter, and searching involves retrieving Documents from an index via an IndexSearcher.

#### Fields
* A Field is simply a name-value pair. A Document consists of one or more Fields.

#### Searching
* Searching requires an index to have already been built. It involves creating a Query (usually via a QueryParser) and handing this Query to an IndexSearcher, which returns a list of Hits.

#### Queries
* Lucene has its own mini-language for performing searches. 
* The Lucene query language allows the user to specify which field(s) to search on, which fields to give more weight to (boosting), the ability to perform boolean queries (AND, OR, NOT) and other functionality. 
* [Lucene Query Syntax](http://www.lucenetutorial.com/lucene-query-syntax.html)

## References
* [LuceneTutorial.com](http://www.lucenetutorial.com/basic-concepts.html)