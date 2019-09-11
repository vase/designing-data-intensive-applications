# Chapter 3. Storage and Retrieval

## Summary

# Database basic function:
  * Store data when you give it
  * Give data when you ask for it
      > Simple example of data structre: Get and set mehtod (key-value sotre)
  * Set function - good performance, becasue just need to append after last
  * Get function - bad performance, needs to scan entire database from beginning to end
      > bad performance of "get" method solved using index
  * Index is an additional data structure that does not effect the content of the database
  * Index slows down write, speed up query. Need to find the balance
  * Storage engines is categorised into 2 main catogories:
    * Optimized for transsaction processing (OLTP - Online Transaction Processing)
    * Optimized for analytics (OLAP - Online Analytical Processing)

# Hash Index
  * Keeps an in memory hash map where every key is mapped to a byte of offset in the data file
  ![1](https://user-images.githubusercontent.com/35839199/64672742-de3d6e80-d49e-11e9-9c8a-f7eed8358f56.png)
  * In the book, the author explains how to prevent databse from running out of space if data is only added by appending into databse.
  * Once segment reaches a certain amount of space,the segment will be closed and new writes will be appended to a new segment. After compaction, old segments are deleted
  * Compaction: throwing away dupplicate keys in the log through merging segments
  ![2](https://user-images.githubusercontent.com/35839199/64672990-94a15380-d49f-11e9-9a57-752ede9efdfe.png)
  * When doing compaction, check most recent segment for the key first, followed by second most recent segment and so on. 
  * Compacted segment only takes most updated data, therefore reads are done on compacted segment which reduces lookup overhead.

Advantage of append-only log
  * Appending is much faster than random writes
  * Concurrency and reash revocery is much simpler the old and new data are spliced together as no data is overwritten.
  
# SSTables and LSM-Trees
  * Sorted String Table (SSTable) is where the sequence of key-value pairs is sorted by key
  * Advantages:
    * Efficient merging of segments
    ![3](https://user-images.githubusercontent.com/35839199/64674649-ddf3a200-d4a3-11e9-8a1c-d8da70db2144.png)
    * Efficient lookup of particular key. Only have to search between shaded area
    ![4](https://user-images.githubusercontent.com/35839199/64674735-0c717d00-d4a4-11e9-8636-37d9a069e497.png)
    
Constructing and maintaining SSTalbes
* When a write comes in, add it to an in-memory balanced tree data structure (red-black tree). This in-memory tree is often called a memtable
* Writes to disk as SSTable when memtable gets bigger than some treashold
* To serve read request, start finding key from most recent segment to least reent segment
* Run Merging and compaction from time to time
> Log-Structured Merged Tree: Keeping a cascade of SSTables that are merged in the background

# B-Trees
* Only similarity with SSTables is that key-value pair is sorted by key
* B-Trees breaks databse down into fixed sized blocks or pages where each page can be identified using an address or location.
![5](https://user-images.githubusercontent.com/35839199/64676106-5d36a500-d4a7-11e9-8e90-e1bd783bcdb2.png)
* Branching factor: number of references to child pages in one page
* Updating values in B-Trees is just search and replace.
* Adding a new key is also the same. However if there is not enough free space on the page, the page splits into half to be able to accomodate the new key.
![6](https://user-images.githubusercontent.com/35839199/64676316-e221be80-d4a7-11e9-9aae-b82ebf3f93f7.png)

Making B-Trees Reliable
* Basic underlying write operation in B-tree is to overwrite a page on disc
* Can be dangerous because database crash during write operation can end up in corrupted index
* Solved with write ahead log (WAL). Makes sure modfication must be written before it can be applied to the pages.
* Insead of using WAL, some databases uses copy0on-write scheme where modified pae is written in different location

# Comparison of B-Trees and LSM Trees

## Downside of B-Trees
* B-trees has many writes. At least twice. Once to WAL. Once to the tree.
* B-Trees has many writes not suitable for SSD as SSD has limited writes
* B-Trees: Write amplification offurs where one write on database will result in multiple writes on database over the course of the database lifetime.
* B-Trees: Write amplification directly affect performance
* LSM trees can be compressed better as there in fragmentation in B-trees

## Downsides of LSM Trees
* Compaction interfere read and write
* LSM has high write throughput even it can whistand higher write throughput than B-trees. The disk finite write bandwidth needs to be shared between initial write and the compaction threads running in background
* LSM has many same keys in one segment

# Other indexing structures
  * Clustered index - storing all row data within the index
  * Nonclusered index - storing only references of data within the index
  * Multi column indexes - concatinated indexes which combines several fields into one key. a more general way of querying several columns at once.
  *B-trees and lsm tree is not able to answer that kind of query but one option is to translate 2d location into a single number using a space-filling curve and use regular B-tree indexing.
  
  # Fuzzy indexes
  * Comming up results with similar query like google search
  * Requres linguistic analysis
  
  # Why keeping everyting in memory?
  * RAM has become cheaper over time
  * New ways of database backup with battery-powered TAM
  * Files on disc can easily be backed up, inspected and analyzed by external utilities
  * Removes all overheads acssociated with managing on disk data structures
  * Avoid the overheads of encoding in-memory data structures in a form that can be written to disk
  
# Online Transaction Processing (OLTP)
Allowing clients to make low-latenct reads and writes as opposed to batch processing jobs which only run preiodically

# Online Analytical Processing (OLAP)
A seperate database optimized for analytics called a data warehouse
* Contain read only data that id derived from OLTP systems through ETL process

# Stars and snowflakes
* made up of fact table and dimension table
* in snowflake data structure, dimension table have further branch of dimension tables

# Column Oriented Storage
Store all values of colums together in a file to make query easier
* Can be compressed through column compression (bitmap indexed storage)
* Allows vectorized processing

* Sorting in column oriented storage needs to be stored by row at the same time as sorting columns independently will not make sense.
* Sorting colum oritented storage can do further compression of repeated values

> We have learn that storing data differently has their own advantages. Since we are already having redundant databases as backup, it is wise to store backup data in different ways to make full use.

# Data Cubes: Materialized view
Have data cube t to compute aggregrations every time new data is added in to reduce query time by analysts.
![7](https://user-images.githubusercontent.com/35839199/64683466-de953400-d4b5-11e9-91ae-ac9fa473ca68.png)
