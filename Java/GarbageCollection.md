## Generational Garbage Collection 

* Young generation
	* Most of the newly created objects are located here. Since most objects soon become unreachable, many objects are created in the young generation, then disappear. 
	* When objects disappear from this area, we say a "minor GC" has occurred. 

* Old generation
	* The objects that did not become unreachable and survived from the young generation are copied here. 
	* It is generally larger than the young generation. As it is bigger in size, the GC occurs less frequently than in the young generation. 
	* When objects disappear from the old generation, we say a "major GC" (or a "full GC") has occurred. 

* Permanent generation (method area)
	* it stores classes or interned character strings
	* A GC may occur in this area. The GC that took place here is still counted as a major GC.
	
## Composition of the Young Generation

* The young generation is divided into 3 spaces. 
    * One Eden space
    * Two Survivor spaces

* The majority of newly created objects are located in the Eden space.
* After one GC in the Eden space, the surviving objects are moved to one of the Survivor spaces. 
* After a GC in the Eden space, the objects are piled up into the Survivor space, where other surviving objects already exist.
* Once a Survivor space is full, surviving objects are moved to the other Survivor space. Then, the Survivor space that is full will be changed to a state where there is no data at all.
* The objects that survived these steps that have been repeated a number of times are moved to the old generation.

## References

* [Understanding Java Garbage Collection](https://www.cubrid.org/blog/understanding-java-garbage-collection/)

## Further Readings

* [ZGC for Future LINE HBase by Shinya Yoshida](https://github.com/EddieChoCho/tech-talks-note/blob/master/2019/ZGC_ForFutureLineHBase.md)

 