# Lightweight Directory Access Protocol(LDAP)

## The Directory Information Tree(DIT)
* The DIB is composed of a set of entries organized hierarchically in a tree structure known as the Directory Information Tree (DIT); specifically, a tree where vertices are the entries.

* The arcs between vertices define relations between entries.  If an arc exists from X to Y, then the entry at X is the immediate superior of Y, and Y is the immediate subordinate of X.  
* An entry's superiors are the entry's immediate superior and its superiors.
* An entry's subordinates are all of its immediate subordinates and their subordinates.

* Similarly, the superior/subordinate relationship between object entries can be used to derive a relation between the objects they represent.  DIT structure rules can be used to govern relationships between objects.

## Structure of an Entry
* Attributes: An entry consists of a set of attributes that hold information about the object that the entry represents.
	* User Attributes: represent user information
	* Operational Attributes: represent operational and/or administrative information
	* Attribute Types [RFC4519](https://tools.ietf.org/html/rfc4519)	

* The attribute type governs whether the attribute can have multiple values, the syntax and matching rules used to construct and compare values of that attribute, and other functions. 

* No two values of an attribute may be equivalent.  
	* Two values are considered equivalent if and only if they would match according to the equality matching rule of the attribute type.

## Naming of Entries
### Relative Distinguished Name
* Each entry is named relative to its immediate superior.  This relative name, known as its Relative Distinguished Name (RDN), is composed of an unordered set of one or more attribute value assertions (AVA) consisting of an attribute description with zero options and an attribute value.  
* These AVAs are chosen to match attribute values (each a distinguished value) of the entry.
* An entry's relative distinguished name must be unique among all immediate subordinates of the entry's immediate superior (i.e., all siblings).
* The following are examples of string representations of RDNs:
	```
	UID=12345
	OU=Engineering
	CN=Kurt Zeilenga+L=Redwood Shores
	```
* The last is an example of a multi-valued RDN; that is, an RDN composed of multiple AVAs.

### Distinguished Name (DN)
* An entry's fully qualified name, known as its Distinguished Name (DN), is the concatenation of its RDN and its immediate superior's DN.  
* A Distinguished Name unambiguously refers to an entry in the tree.  
* The following are examples of string representations of DNs
	```
	UID=nobody@example.com,DC=example,DC=com
	CN=John Smith,OU=Sales,O=ACME Limited,L=Moab,ST=Utah,C=US
	```
      
### Alias Name
* TBD...

## Object Class
* TBD...

## References
* [RFC4512 - LDAP: Directory Information Models](https://tools.ietf.org/html/rfc4512)
* [RFC4519 - LDAP: Schema for User Applications](https://tools.ietf.org/html/rfc4519)
* [LDAP wiki](https://ldapwiki.com/wiki/Main)