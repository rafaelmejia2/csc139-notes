## **How do we support a large address space with  (potentially) a lot of free space between the stack and the heap?**
To solve this problem, an idea was born, and it is called **segmentation**.  The idea is simple: instead of having just one base and bounds pair in our MMU, why not have a base and bounds pair per
logical **segment** of the address space? A segment is just a contiguous portion of the address space of a particular length, and in our canonical address space, we have three logically-different segments: code, stack, and heap. What segmentation allows the OS to do is to place each one
of those segments in different parts of physical memory, and thus avoid filling physical memory with unused virtual address space.

What if we tried to refer to an illegal address (i.e., a virtual address of 7KB or greater), which is beyond the end of the heap? You can imagine what will happen: the hardware detects that the address is out of bounds, traps into the OS, likely leading to the termination of the offending process. And now you know the origin of the famous term that all C programmers learn to dread: the segmentation violation or segmentation fault.

## **Which Segment Are We Referring To?**
The hardware uses segment registers during translation. How does it know the offset into a segment, and to which segment an address refers? One common approach, sometimes referred to as an explicit approach, is to chop up the address space into segments based on the top few bits
of the virtual address; this technique was used in the VAX/VMS system. In our example above, we have three segments; thus we need two bits to accomplish our task. If we use the top two bits of our 14-bit virtual address to select the segment, our virtual address looks like this:

![[Pasted image 20250511140356.png]]

In our example, then, if the top two bits are 00, the hardware knows the virtual address is in the code segment, and thus uses the code base and bounds pair to relocate the address to the correct physical location.

If the top two bits are 01, the hardware knows the address is in the heap, and thus uses the heap base and bounds. Let’s take our example heap virtual address from above (4200) and translate it, just to make sure this is clear. The virtual address 4200, in binary form, can be seen here:

![[Pasted image 20250511140504.png]]

Thus, if base and bounds were arrays (with one entry per segment),
the hardware would be doing something like this to obtain the desired
physical address:
```C
// get top 2 bits of 14-bit VA
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
// now get offset
Offset = VirtualAddress & OFFSET_MASK
if (Offset >= Bounds[Segment])
	RaiseException(PROTECTION_FAULT)
else
	PhysAddr = Base[Segment] + Offset
	Register = AccessMemory(PhysAddr)
```

## **What About The Stack?**
Thus far, we’ve left out one important component of the address space: the stack. The stack has been relocated to physical address 28KB in the diagram above, but with one critical difference: it grows backwards (i.e., towards lower addresses). In physical memory, it “starts” at 28KB1 and grows back to 26KB, corresponding to virtual addresses 16KB to 14KB; translation must proceed differently.

The first thing we need is a little extra hardware support. Instead of just base and bounds values, the hardware also needs to know which way the segment grows (a bit, for example, that is set to 1 when the segment grows in the positive direction, and 0 for negative).

## **Support for Sharing**
Specifically, to save memory, sometimes it is useful to **share** certain memory segments between address spaces. In particular, **code sharing** is common and still in use in systems today.
To support sharing, we need a little extra support from the hardware, in the form of **protection bits.**

## **Fine-grained vs. Coarse-grained Segmentation**
We can think of this segmentation as **coarse-grained**, as it chops up the address space into relatively large, coarse chunks. 

However, some early systems were more flexible and allowed for address spaces to consist of a large number of smaller segments, referred to as **fine-grained** segmentation.
	Supporting many segments requires even further hardware support, with a segment table of some kind stored in memory

## **OS Support**
The general problem that arises is that physical memory quickly becomes full of little holes of free space, making it difficult to allocate new segments, or to grow existing ones. We call this problem **external fragmentation**.

One solution to this problem would be to compact physical memory by rearranging the existing segments.

A simpler approach might instead be to use a free-list management algorithm that tries to keep large extents of memory available for allocation. There are literally hundreds of approaches that people have taken, including classic algorithms like **best-fit** (which keeps a list of free spaces
and returns the one closest in size that satisfies the desired allocation to the requester), **worst-fit**, **first-fit**, and more complex schemes like the **buddy algorithm**. 

