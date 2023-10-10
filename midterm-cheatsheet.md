# Midterm Cheatsheet

### Todos:
1) Notes - Tuesday
2) Discussions - Tuesday
3) Vitamins - Tuesday
4) Exams - Tuesday, Wednesday, Thursday


## Disk and Files

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

> 
