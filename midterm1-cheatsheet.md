# Midterm Cheatsheet

### Todos:
1) Notes - Monday
2) Midterm Review Session - Tuesday
3) Discussions - Tuesday
4) Vitamins - Tuesday (or Thursday)
5) Exams - Tuesday, Wednesday, Thursday


## Disk and Files

> should maybe talk about disk and how the physical hardware works, that felt important.

> The basic unit of data for relational databases is a record (row).

> These records are organized into relations (tables) and can be modified, deleted, searched, or created in memory.

> The basic unit of data for disk is a page, the smallest unit of transfer from disk to memory and vice versa.

> Based on the
relation’s schema and access pattern, the database will determine: (1) type of file used, (2) how
pages are organized in the file, (3) how records are organized on each page, (4) and how each record
is formatted.

> A heap file is a file type with no particular ordering of pages or of the records on pages
and has two main implementations: LinkedList and Page Directory.

> In the linked list implementation, each data page contains records, a free space tracker, and
pointers (byte offsets) to the next and previous page

> The Page Directory implementation differs from the Linked List implementation by only using a
linked list for header pages. Each header page contains a pointer (byte offset) to the next
header page, and its entries contain both a pointer to a data page and the amount of free
space left within that data page.

> A sorted file is a file type where pages are ordered and records within each page are
sorted by key(s).

> Searching through sorted files takes logN I/Os where N = # of pages
since binary search can be used to find the page containing the record. Meanwhile, insertion, in
the average case, takes logN + N I/Os since binary search is needed to find the page to write to
and that inserted record could potentially cause all later records to be pushed back by one. On
average, N / 2 pages will need to be pushed back, and this involves a read and a write IO for each
of those pages, which results in the N I/Os term.

> For all problems in this course, you should
ignore the I/O cost associated with reading/ writing the file’s header pages when the underlying
file implementation is not provided in the question. On the other hand, you must include the I/O
cost associated with reading/ writing the file’s header pages when a specific file implementation
(i.e., heap file implemented with a linked list or page directory) is provided in the question.

> Record types are completely determined by the relation’s schema and come in 2 types: Fixed
Length Records (FLR) and Variable Length Records (VLR). FLRs only contain fixed length
fields (integer, boolean, date, etc.), and FLRs with the same schema consist of the same number of
bytes. Meanwhile, VLRs contain both fixed length and variable length fields (eg. varchar), resulting in each VLR of the same schema having a potentially different number of bytes. VLRs store
all fixed length fields before variable length fields and use a record header that contains pointers to
the end of the variable length fields.

> Regardless of the format, every record can be uniquely identified by its record id - [page #, record
number on page].

> Pages containing FLRs always use page headers to store the number of records currently on the page.

> If the page is unpacked, the page header typically stores an additional bitmap that breaks the
page into slots and tracks which slots are open or taken.

> For VLRs, each page uses a page footer
that maintains a slot directory tracking slot count, a free space pointer, and entries.

> The
footer starts from the bottom of the page rather than the top so that the slot directory has room
to grow when records are inserted.
The slot count tracks the total number of slots. This includes both filled and empty slots. The
free space pointer points to the next free position within the page. Each entry in the slot directory
consists of a [record pointer, record length] pair.

> I/O cost of Page Directory is at max number of header pages at max (read header) + 3 (1 read data + 1 write data + 1 write header) = N + 3

For the below apply the following schema: <br>
1) name VARCHAR(12)
2) student BOOLEAN(1)
3) birthday DATE(8)
4) state VARCHAR(2)

> The minimum size, in bytes, of a record with varchars and a header page of 5 bytes is the cost of the header page added with all fixed size bytes because the VARCHARS could be null values.

This would be 5 (header) + 1 (boolean) + 8 (date) = **14**.

> The maximum size, in bytes, of a record with varchars and a header page of 5 bytes is the cost of the header page. 

This would be 5 (header) + 12 (VARCHAR(12)) + 1 (boolean) + 8 (date) + 2 (VARCHAR(2))
= **28**

> What is the size of the slot directory if 4 VLRs are inserted into an empty page? The slot directory contains a slot count, free space pointer, and entries, which are record
pointer, record size pairs. Since pointers are just byte offsets within the page, the size of the
directory is 40 bytes.
4 (slot count) + 4 (free space) + (4 (record pointer) + 4 (record size)) * 4 (# records)
= 4 + 4 + 8*4 = **40**

> The worst case time to see if there is free space in a LinkedList implementation is when you have to read the header page alongside all of the free pages. Free pages + 1.

> True or False: Assume you are working with a page directory implementation. All data pages
must be examined in the worst case to find a page with enough free space for a new insertion. **False**




## B+ Trees

> d is the order of a B+ tree. Each node can have between d and 2d entries inclusive, assuming no deletes happen.

> In between each entry of an inner node is a child node. Since there are at most 2d entries in a node, inner nodes may have at most 2d + 1 child pointers. This is also called the tree's fanout.

> They keys in the children to the left of an entry must be less than the entry while the keys in the children to the right must be greater than or queal to the entry

> All leaves are at the same depth and have between d and 2d entries.

> There are 3 alternatives to storing the actual records in the tree:

1) In the Alternative 1 scheme, the leaf pages are the data pages. Rather than containing
pointers to records, the leaf pages contain the records themselves.

2) In the Alternative 2 scheme, the leaf pages hold pointers to the corresponding records.

3) In the Alternative 3 scheme, the leaf pages hold lists of pointers to the corresponding records.
This is more compact than Alternative 2 when there are multiple records with the same leaf
node entry.

> Clustered/unclustered refers to how the data pages are structured.

> Because the leaf pages are the actual data pages for Alternative 1 and keys are sorted on the index leaf pages, Alternative 1 indices are clustered by default. Therefore, unclustering only applies to Alternative 2 or 3. I think this might be because of pointers to the records rather than holding the actual records

> Unclustered indices required that you look inside data pages eing pointed to in order to retrieve all the records associated with these keys.

> Clustered indices have data pages that are sorted on the same index on which you've built the B+ tree. This does not mean the data pages are sorted exactly, just that keys are roughly in the same order as data. Better for IOs because better caching strategy with records nearby in the same node happening to be on the same data page. Also could be pointers tbh, im not really sure about the pointers thing.

> Unclustered has 1 I/O per record. Clustered has 1 I/O per page of records.

> Counting IOs general procedure:

1) Read the appropriate root-to-leaf path.
2) Read the appropriate data page(s). If we need to read multiple pages, we will allot a read IO for each page. In addition, we account for clustering for Alt. 2 or 3 (see below.)
3) Write data page, if you want to modify it. Again, if we want to do a write that spans multiple data pages, we will need to allot a write IO for each page.
4) Update index page(s).

> Bulk Loading: the insertion procedure to construct a B+ tree from scratch to not traverse the tree every time we insert, but insert as we are down in the leaf node.

> The maximum number of entries we can add without increasing the height is the total capacity of a height 2, order d = 1 tree minus the current number of entries. The total capacity is (2d)(2d + 1)^h = (2)(3^2) = 18. The current number of entries is 6 because the entries are in the leaf nodes. Therefore, we can add a maximum of 18 − 6 = 12.

> For bulkloading, 1. Leaf nodes do not fill up to 2*d+1 and split, but rather, fill up to be 1 record more than fillFactor full, then "splits" by creating a right sibling that contains just one record (leaving the original node with the desired fill factor). fillFactor should ONLY be used for determining how full leaf nodes are (not inner nodes), and calculations should round up, i.e. with d=5 and fillFactor=0.75, leaf nodes should be 8/10 full.

## Buffer Management

> The buffer manager is responsible for managing pages in memory and processing page requests from the file and index manager. Remember, space on memory is limited, so we cannot afford to store all pages in the buffer pool. The buffer manager is responsible for the eviction policy, or choosing which pages to evict when space is filled up. When pages are evicted from memory or new pages are read in to memory, the buffer manager communicates with the disk space manager to perform the required disk operations.

> Memory is converted into a buffer pool by partitioning the space into frames that pages can be
placed in. A buffer frame can hold the same amount of data as a page can (so a page fits perfectly
into a frame). To efficiently track frames, the buffer manager allocates additional space in memory
for a metadata table.

> The table tracks 4 pieces of information:

1. Frame ID that is uniquely associated with a memory address
2. Page ID for determining which page a frame currently contains
3. Dirty Bit for verifying whether or not a page has been modified
4. Pin Count for tracking the number of requestors currently using a page

> When pages are reqested from the buffer manager and the page already exists within memory, the page's pin count is incremented and the page's memory address is returned.

> If the page does not exist in the buffer pool and there is still space, the next empty frame is found and the page is read into that frame. The page's pin count is set to 1 and the page's memory address is returned. IN the case, where the page does not exist and there are no empty frames left, a replacement policy must be used to determine which page to evict.

> Replacement policy depends on page access patterns and the optimal policy is chosen by counting page hits. 

> A **page hit** is when a requested page can be found in memory without having to go to disk.

> Each **page miss** incurs an additional IO cost, so a good eviction policy is critical for performance. 

> The **hit rate** for an access pattern is the number of page hits / (number of page hits + page misses) or page hits / number of accesses.

> Additionally, if the evicted page has the dirty bit set, the page is written to disk to ensure that updates are persisted. The dirty bit is set to 1 if and when  a page is written to with updates in memory. The dirty bit is set to 0 once the page is written back to disk.

> Once the requestor completes its workload, it is responsible for telling the buffer manager to decrement the pin count associated with pages that it previously used.

> LRU Cache Replacement Policy is used to evict least recently used unpinned pages which have a pin count of 0 and the lowest value in the Last Used column. This Last Used colum is added to metadata and measures the latest time at which a page's pin count is decremented.

> Implementing LRU normally can be costly. But the Clock policy provides an alternative implementation that approximates LRU using a ref bit (recently referenced) column in the metadata table and a clock hand variable to track the current frame in consideration.

> Using a ref bit reduces the cost of both space and time. Only a single bit per frame is needed instead of multiple bits to store a Last Used value. With respect to time, the buffer manager just follows the clock hand.

> In LRU, the buffer manager has to spend time searching for the minimum value in the Last Used column.

1) Iterate through frames within the table, skipping pinned pages and wrapping around to frame 0 upon reaching the end, until the first unpinned frame with ref bit = 0 is found.
2) During each iteration, if the current frame’s ref bit = 1, set the ref bit to 0 and move the clock hand to the next frame
3) Upon reaching a frame with ref bit = 0, evict the existing page (and write it to disk if the dirty bit is set; then set the dirty bit to 0), read in the new page, set the frame’s ref bit to 1, and move the clock hand to the next frame

> If accessing a page currently in the buffer pool, the clock policy sets the page’s ref bit to 1 without moving the clock hand.

> LRU sucks when you are trying to read a set of pages S where |S| > buffer pool. When repeatedly accessing this set, we must evict constantly the pages we might want to use next and because of this we get 0 cache hits.

> MRU Replacement is much better for this purpose of sequential scanning performance.

> A frame is defined as the size of a disk page.

> The requester of the page (file/index management code) sets the dirty page.

> The dirty bit is used to track whether the page has been modified before it’s written back to disk. The pin is used to track popularity

> Reference bits are not used for LRU policy. They are used for the Clock policy.

> Since we can fit all pages in memory, we do not need to evict pages during a sequential scan.

> The pin count is decremented by whoever requested the page to signify that they are done with it

## Sorting

> When counting I/Os, we ignore any potential caching done by the buffer manager. This implies that once we unpin the page and say that we are done using it, the next time we attempt to access the page it will always cost 1 I/O. I guess this is a worst case scenario so Big(IO) not Big(O) LOL.

> Two Way External Merge Sort: first step of our sorting algorithm should be to sort the records on each individual page. We’ll call this first phase the “conquer” phase because we are conquering individual pages.

> After this, let’s start merging the pages together using the merge algorithm from merge sort. We’ll call the result of these merges sorted runs. A sorted run is a sequence of pages that is sorted.

> IO Cost: First, notice that each pass over the data will take 2 ∗ N I/Os where N is the number of data pages. This is because for each pass, we need to read in every page and write back every page after modifying it. Therefore we need ⌈log2(N)⌉ merging passes, and 1 + ⌈log2(N)⌉ passes in total. 2N ∗ (1 + ⌈log2(N)⌉) I/Os. In total, we have two input buffers and 1 output buffer for a total of 3 pages required in each merging pass. We have more than 3 frames in memory for sroting.

> Let's assume we have B buffer pages available to us in memory. Rather than just sorting individual pages, let’s load B pages and sort them all at once into a single sorted run. We have B bufferframes available to us, but we need 1 for the output buffer. This means that we can have B-1 input buffers and can thus merge together B-1 sorted runs at a time. 2N ∗ (1 + ⌈logB−1⌈N/B⌉⌉) I/Os

> You are trying to sort the Students table which has 1960 pages with 8 available buffer pages. 

1) How many sorted runs will be produced after each pass?

The first pass loads all 8 buffer pages with data pages at a time and outputs sorted runs until each page is part of a sorted run. This means that we will have 1960 / 8 = 245 sorted runs of 8 pages after pass 0.

2) How many pages will be in each sorted run for each pass?

8 - 1 = 7. (B - 1), we need 1 for the output

3) How many I/Os does the entire sorting operation take?

4 * 2 * 1960 = 15,680 I/Os

> What is the minimum number of buffer pages that we need to sort 1000 data pages in two passes?

1000 / (B)(B-1) <=1, B = 33

## Hashing

> The first partitioning pass will hash each record to B - 1 partitions. A partition is a set of pages such that for particular hash functions, the values on the pages all hash to the same hash value. B - 1 partitions and 1 input buffer. 

> For the partitions that are too big, we simply repartition them using a different hash function than we used in the first pass. Why a different hash function? If we reused the original function, every value would hash to its original partition so the partitions would not get any smaller. We can recursively partition as many times as necessary until all of the partitions have at most B pages.

> Analysis of External Hashing. Property 1 says that we must read in every page during the first partitioning pass. This comes straight from the algorithm. Property 2 says that during a partitioning pass we will write out at least as many pages as we read in. This comes directly from the explanation above - we may create additional pages during a partitioning pass. Property 3 says that we will not read in more pages than what we wrote out during the partitioning pass before. In the worst case, every partition from pass i will need to be repartitioned, so this would require us to read in every page. In most cases, however, some partitions will be small enough to fit in memory, so we can read in fewer pages than we produced during the previous pass. Property 4 says that the number of pages we will build our hash table out of is at least as big as the number of data pages we started with. This comes from the fact that the partitioning passes can only increase the number of data pages, not decrease them.

>  How many IOs does it take to hash a 500 page table with B = 10 buffer pages? Assume that we use perfect hash functions. 

500 / 9 = 56 pages of data in each partition. Total IOs is 500 + 56 * 9 = 500 + 504 = 1004. But the 56 pages cannot be fit into memory, so we must recursively partition 56 / 9 = 7 pages each. These will fit in memory. 81 * 7 = 567 pages as output and 504 pages as input makes a total of 1071 IOs. FInal conquer step involves feeding 567 pages read to write to disk. 567 + 567 = 1134. 1004 + 1071 + 1134 = 3209 IOs in total.

> We want to hash a 30 page table using B=6 buffer pages. Assume that during the first partitioning pass, 1 partition will get 10 data pages and the rest of the pages will be divided evenly among the other partitions. Also assume that the hash function(s) we use for recursive partitioning are perfect. How many IOs does it take to hash this table?

1 partition of 10 data pages. The rest of the pages will be be divided evenly. B = 6 so we have 5 partitions. 1 with 10 and 4 with (30 - 10) / 4 or 5 each. The partition with 10 must be further divided to get 5 partitions of 2 each. The 2 each must be conquered and the 5 each must be conquered. This is 


# Sorting

General External Merge Sort: B=4, N=8


Cost = 2N * (1 + ⌈ logB-1(⌈N/B⌉) ⌉)
	= 2(8) * (1 + ⌈ log3(2) ⌉)
	= 16 * (1 + 1)
	=  32 I/Os  ✓


logB-1(N/B) ≤ p − 1 where p is number of passes.

passes: 1 + ceil(logb-1(n/b))

IOs = 2 * N * passes^^^

add boolean algebra

add spacial indexing discussion to cheatsheet