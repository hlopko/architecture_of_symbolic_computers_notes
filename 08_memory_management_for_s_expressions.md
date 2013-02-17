# Memory Management for S-Expressions #

Many cells are allocated by the SECD programs, and there is no possibility to
reuse existing cells yet. We need a so called ***Garbage Collector***.

A cell can be in 3 states:

* Free
* Allocated
* Garbage

## Allocation ##

	allocate(f) =
		if no-more-free-space(f)
		then allocate(garbage-collect(f))
		else get-next-free-cell(f)
		where
			no-more-free-space(f) = T if no free cells available
			and garbage-collect(f) collects all garbage cells
			and get-next-free-cell(f) get next available free cell

### Free Cells in Consecutive Locations ###

This is the simplest approach. The F registers stores a memory address below
which all cells are already used and above which all free cells are present.

### Free Cells Individually Marked ###

Each cell contains in its tag field an extra bit - the ***mark bit***,
which has two values - *free* and *allocated*. Initially all cells are marked
free, and are marked allocated when used. If some garbage collection mechanism
determines that the allocated cell is an fact garbage, it can set the flag to
free again and the cell will be used by allocation again.

Allocate function is more complicated now, it has to scan through memory
checking the mark bits.

### Linked Free List ###

A garbage collector linked all free cells into a ***free list***. The
F register points to the first cell of the list. Allocation is made by getting
the car(F) and F is replaced by cdr(F).

## Mark-Sweep Collection ##

The basic approach is to mark all cells as free and then trace through all
cells which are in use (all cells referenced by any of the SECD registers) and
mark them. 

The basic problem is that we are running out of memory, how can we afford to
scan all registers? Simple solution is to reserve a finite length
memory for a stack, and when the stack overflows, we can start over and seach
for black cells which have white children. Improvement would also be to
remember the smallest address that is 'forgotten'.

Entirely different technique called ***Deutsch-Schorr-Waite algorithm*** avoids
the stack altogether by reversing the pointers. An additional information is
needed in each cell - ***direction tag***.

	mark(i, parent, c) = "color all cells accessible from i as c"
		if mark-bit(i) = c
		then backup(i, parent, c)
		else let ix = color(i, c)
			 in if tag(i) = terminal
			 then backup(i, parent, c)
			 else let child = car(i) 
				  in mark(child, rplacdtag((rplaca(i, parent), A) c))

	backup(child, parent, c) = "back up from child"
		if null(parent) then c
		else let dtag = direction-tag(parent)
		     and pcar = car(parent)
			 and pcdr = cdr(parent)
			 in if dtag = A "see which parent field was reversed"
				then "reset parent's car and then reverse parents cdr"
					mark(pcdr, 
						 rplactag(
							rplacd(rplaca(parent, child), pcar), D), 
						 c)
				else "reset parent's cdr and back up"
					backup(rplacd(parent, child), pcdr, c)


## Parallel Mark-Sweep ##

Above solutions are so called *stop the world* algorithms. The execution has to
be stopped in order to collect garbage. An incremental variation of mark-sweep
exists. A third color has to be added - ***gray***. Simple marking algorithm is
now:

	mark(i, c) = "color all cells accessible from i as c"
		if mark-bits(i) = c "stop on marked cell"
		then next-gray(0, c) "and start scan from 0"
		elseif mark-bits(i) = gray
		then mark(cdr(color(i, c)), c)
		else let ix = color(i, c)
			 in if tag(i) = terminal
			 then next-gray(0, c)
			 else let iy = color(car(i), gray)
				  in mark(cdr(ix), c)
	where next-gray(i, c) =
		if i > top-of-memory
		then c
		elseif mark-bits(i) = gray
		then mark(i, c)
		else next-gray(i + 1, c)

* White - represents free or potentially free cells
* Gray - represents cells touched by the gc, but their car or cdr has not yet
  been touched
* Black - represents cells that have been fully traced

*Note: mutator cannot arbitrarily change links from black cells.*

## Reference Counts ##

Another solution is to keep a count of references pointing to the cell in its
header. Whenever a new reference is created (e.g. using cons function), the count is
incremented, whenever the cell is abandoned (e.g. rplaca function), the count is decremented.

It has its drawbacks:
* inability to guarantee a bounded amount of time on each operation that
  modifies the count
* large number of bits required in the header
* loops can cause instabilities

## Compacting Collectors ##

Instead of having a linked list of free cells, a garbage collector actually
moves the cells in the memory to ensure consecutive free space. ***Baker's
algorithm*** is one of more often used, even in production systems.

The basic idea of the algorithm is to split the memory into two equally large
halves - ***hemispaces***, and to use only one of them. The one currently in
use is called ***fromspace*** and the other is called ***tospace***. When the
fromspace is completely full, the living cells are evacuated into
***tospace***. When evacuation is finished, the spaces switch and old fromspace
becomes new tospace. As a biproduct of evacuation, cells are nicely compacted.

Role of the F register of the SECD Machine takes ***CP*** (***Creation
Pointer***). On the opposite end there is the ***EP*** (***Evacuation
Pointer***). 

Evacuating a cell means moving its exact copy to tospace. All existing cells
which point to this cells don't know the cell moved. This is solved by setting
***invisile pointer*** or ***reference pointer***. Any reference to this cell
is invisibly forwarded into tospace. 

*Note: As I already know about Baker quite a bit, I skipped a lot of stuff from the
book. If interested, read the fantastic paper **Uniprocessor garbage collection
techniques** by Paul Wilson where you'll find pretty much everything you need
to know about garbage collection.*

### Region Collectors ###

Baker has issues:

* There is no distinction between very long-lived data (such as the root of the
  environment) and very short-lived data (such as top of the stack). Both are
  copied at each flip.
* The nice performance characteristics fall apart when actual storage in use
  nears or exceeds one-half of the total available storage.

Both issues can be solved by splitting the memory into ***regions***. A region
is quite small, few pages of about 1K to 4K each. 

*Note: Again, check the paper by Paul WIlson*

A problem is, that there may be pointers from one region to another, and after
freeing a condemned region such older regions may be left with a ***dangling
pointer***. The most common solution to this is to append to each region an
***entry table*** which contains an entry for each forward pointer reference
from an earlier region to this one. This entry includes an invisible pointer to
the appropriate cell in the region and a backward pointer to the cell in the
earlier region making the forward reference. The actual forward pointer in this
latter cell is to the entry table entry, not the desired cell. 

## Alternative List Representations ##

Ever since their invention, s-expressions have been a magnet for clever
implementations other than the simple tag-car-cdr representations used to this
point. These ideas fall into three categories:

* Methods to compact the amount of storage needed to represent a s-expression
* Techniques for implementing tag bits
* Approaches to list representation that avoid pointers altogether

### List Compaction ###

In many cases, the space taken up by the car and cdr pointer exceeds the number
of bits of the information pointed to. This was recognized early on, and
techniques called ***car coding*** and ***cdr coding*** were developed.

***Car coding*** comes from the observation, that most of the car cells point
to constants. Significant savings are thus possible if the constant value
replaces car pointer. This requires an extra tag bit to permit the machine to
distinguish a car pointer from car constant. 

For ***cdr coding***, very often car pointer points to the next word in memory.
Second often occurence is cdr pointing to nil. We need two bits to express
this information. 

So up until now, each cell holds in its tag field:

* Memory management bits (2)
* Indicators of the type of value stored in the value field
* A representation of what the cdr of this field would look like if it were
  a cons cell


### Tag Implementation Techniques ###

In short, we have 3 approaches:

* Store tags together with the data
* Store tags on one place and data elsewhere
* Throw away the tags completely and store different objects into different
  areas of the memory (first 1mb of memory contains only integer constants,
  then 1mb of floating point numbers etc.)

### Pointer Avoidance ###

It is not true that the only way to implement s-expressions is with pointers.
This section gives several interesting approaches based on explicit labelling.

* The root node of an s-expression is assigned the number 1
* For each nonterminal, if its label is k:
	* the label for the node's car subnode is 2 * k
	* the label for the node's cdr subnode is 2 * k + 1

Now whole s-expression can be stored in an array.

Without the proof, the following is an algorithm that takes any composition of
cars and cdrs and computes the binary form of such an index as follows:

1. Start with a leading 1
2. For the sequences of **a**s and **d**s in `c{ad}+r` create a sequence of
   1 for each d and 0 for each a.
3. Reverse this sequence
4. Append this sequence to the leading 1.

For example, the function car translates into binary number 10, or index 2.
Cadaddr should return an element at index 58 (111010).





