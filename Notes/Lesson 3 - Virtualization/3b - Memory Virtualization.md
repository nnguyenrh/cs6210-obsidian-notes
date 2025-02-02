## 1. Memory Virtualization Introduction 

A bulk of our discussion on virtualization centers around memory systems. The reason is quite simple–the memory hierarchy is crucial to performance. Even device virtualization is heavily reliant on how memory system is virtualized and made available to the operating systems running on top of the hypervisor. Let's begin with techniques for **virtualizing memory management** in a performance-conscious manner.

## 2. Memory Hierarchy 

![[Pasted image 20250119130505.png]]

Let us recall the Memory Hierarchy. L1, L2, and L3 caches are physically tagged, so there is no need to handle them differently when virtualizing the memory hierarchy. However, handling the virtual page translation between virtual and physical memory, is much less straightforward. 

## 3. Memory Subsystem Recall 

![[Pasted image 20250119130513.png]]

Recall that in any modern operating system, each process exists inside of its own protection domain, which corresponds to a separate hardware address space. Additionally, the operating system maintains a **page table** on behalf of each of these processes. 

The page table is the operating system's data structure that stores the **mapping** between the **virtual page numbers** and the **physical pages** where those virtual pages are contained in the main memory of the hardware. 

Physical memory is contiguous, starting from zero to whatever is the maximum limit of the hardware capabilities are. But the virtual address space of a given processor course is **not contiguous** in physical memory. Rather, it is scattered all over the physical memory. 

This is the advantage of **page based memory management**. While a process perceives its virtual address as contiguous, this is not necessarily reflected in the physical mapping of those virtual pages to the physical pages in the main memory.

## 4. Memory Management and Hypervisor 

![[Pasted image 20250119130547.png]]

By contrast, under a virtualized system, the hypervisor sits between the guest operating system and the hardware. Individual processes run inside of each operating system, and each process is executed under its own protection domain. 

What that means is that in each of these operating systems, there is a **distinct page table** that the operating system maintains on behalf of the processes that are running in that operating system. 

**Does the hypervisor know about the page tables maintained on behalf of the processes that are running in each one of these individual operating systems?** The answer is no. Windows and Linux are simply **protection domains** distinct from one another so far as the hypervisor is concerned. 

Therefore, the fact that each of Windows and Linux within them contain application of a processes is something that the hypervisor doesn't know about at all.

## 5. Memory Manager Zoomed Out 

In the same virtualized system, each guest operating system believes that they are accessing a contiguous section of physical memory. 

But unfortunately, physical memory, or **machine memory**, is in the control of the **hypervisor**, and not in the control of any one operating system. The physical memory of each of these operating systems is itself an illusion and is not contiguous in terms of the machine memory the hypervisor controls. 

![[Pasted image 20250119130708.png]]

Refer to the image above. Windows has physical memory that is divided 2 regions: $R_1$ and $R_2$. Suppose that $R_1$ has a number of pages, 0 through $q$, and R2 has a number of pages starting from $q+1$ through $N$. This is the physical memory allocated to Windows.

However, notice how $R_1$ occupies hypervisor-controlled machine memory.  $R_2$ is not contiguous with respect to $R_1$ in machine memory.

Linux also has two regions of physical memory, which we will once again label $R_1$ and $R_2$.  And it has a total capacity of M + 1 physical page frames all of which zero through L are contiguous in the machine memory. L+1 through M are contiguous in machine memory but they're not contiguous with respect to the other region $R_1$. **Why would this happen?** 

The hypervisor is not being nasty to the operating system. Even if all of the N plus 1 pages of Windows and N plus 1 pages of Linux would be contiguous in the machine memory they cannot all start at 0 because the physical memory the real physical memory which we're calling machine memory is the only thing that is contiguous. 

And that has to be partitioned between these two guys and therefore the starting point for the physical memory, the illusion of the physical memory so far as a particular operating system, cannot be 0. Also the memory requirements of operating systems are dynamic and bursting. 

When Windows started out initially with Q plus 1 pages and later on it needed additional memory then it's going to **request the hypervisor** and at that point it may not be possible for the hypervisor to give another region which is contiguous with the previous region that Windows already has. 

So these are the reasons why the **physical memory** of an operating system it still becomes an **illusion** in terms of how it is actually contained in the machine memory.

## 6. Zooming Back In 

![[Pasted image 20250119130846.png]]

Zooming back in to what is going on within a given operating system we already know that the **process address space** for an application is an illusion in the sense that the virtual memory space of this process is **contiguous** but in terms of physical memory they are not contiguous. 

And the page table data structure that the operating system maintains on behalf of the processes is the one that supports this illusion so far as this process is concerned. 

By mapping the virtual page number of the process using the page table for this particular process into the physical page number where a particular virtual page may be contained in physical memory. 

This is a setting in a non-virtualized operating system. So the **page table** serves as the **broker** to convert a virtual page number to a physical page number. 

In a virtualized setting we have **another level of indirection**.

![[Pasted image 20250119131608.png]]

This physical page number of the operating system has to be mapped to the machine memory or the machine page numbers. We'll call the pages in the machine memory which is the real thing as MPN short for Machine Page Numbers. And this goes from zero through some max which is the total memory capacity that you have in your hardware. 

This data structure the page table is a traditional page table that gives a mapping between virtual page number and the physical page number. And this is a traditional page table. The mapping between the physical page number and the machine page number that is PPN to MPN mapping is kept in another page table which is called **shadow page table** S-PT. 

![[Pasted image 20250119131715.png]]

So now in a virtualized setting there's a two-step translation process to go from VPN to MPN. 

The page table maintained by the guest operating system is the one that translates VPN to PPN and then there is this additional page table called a shadow page table that converts the PPN to MPN. 

## 7. Quiz - Who Keeps PPN MPN Mapping 

In a fully virtualized system, where is the PPN to MPN mapping stored?
- Guest OS 
- Hypervisor

In a paravirtualized system, where is the PPN to MPN mapping stored? 
- Guest OS 
- Hypervisor 

---

In the case of a fully virtualized hypervisor the guest operating system has no knowledge of machine pages. It thinks that it's physical memory is contiguous because it is thinking that it is running on bare metal nothing has been changed in the operating system to run on top of a hypervisor in a fully virtualized setting. And therefore it is the **responsibility of the hypervisor** to maintain the PPN to MPN mapping. 

In a para virtualized setting on the other hand **the guest operating system knows that it is not running on bare metal**. It knows that there's a hypervisor in between it and the real hardware. And it knows therefore that its physical memory is going to be fragmented in the machine memory. **So the mapping PPN to MPN can be kept in either the guest OS or the Hypervisor.** But usually it is kept in the **guest operating system**. We'll talk more about that later on.

## 8. Shadow Page Table 

![[Pasted image 20250119131930.png]]

Let's understand what exactly this shadow page table is and what it is. In many architectures for example Intel's X86 family the CPU uses the page table for address translation. What that means is presented with the virtual address the CPU first looks up the TLB to see if there is a match for the virtual page number contained in this virtual address. If there is a match it's a hit and it can translate this virtual address to the physical address. If it is a miss CPU knows where in memory the page table data structure is kept by the operating system. And therefore what it does it goes to the page table which is in main memory and retrieves the specific entry which will give it the translation from the virtual page number to the physical page number. And once it gets that it'll stash it in the TLB as well and be able to generate the physical address that is specified by this particular virtual address. 

So that's the way the CPU does the translation in many architectures. So in other words both the TLB and the page table are data structures that the architecture uses for address translation. Page table is also a data structure that is set by the operating system for enabling the processor to do this translation. So in other words the hardware page table is really the Shadow page table in the virtualized setting if the architecture is going to use the page table for address translation.

## 9. Efficient Mapping (Full Virtualization)

![[Pasted image 20250119132042.png]]

As I mentioned in a fully virtualized setting the guest operating system has no idea about machine pages. It thinks that the physical page number that it is generating is the real thing. But it is not. 

And therefore there is two levels of indirection:
- In the guest OS, mapping from virtual page to physical page
- The hypervisor mapping physical page to machine page using the shadow page table
	- Recall that the shadow page table may be the real hardware page table that the CPU uses as well, and is the data structure that is maintained by the hypervisor to translate the PPN to MPN. 

On every memory access of a process of the guest operating system, the virtual address has to be converted to a machine address.  We would like to make this process more efficient because it's happening on every memory access by **getting rid of one level of indirection** that is this **translation by the guest operating system**. 

How do we avoid the additional level of indirection applied where the virtual page number is converted into a physical page number by the guest operating system, and then looked up in the shadow table by the hypervisor to generate the machine page number?

Remember that the guest operating system establishes the initial mapping between a virtual page number and a physical page number by creating an entry in the page table for the process that is generating this virtual address in the first place.  **Updating the page table is a privileged instruction**. Therefore, when the guest OS attempts to update the page table to establish a mapping between VPN and PPN, it'll result in a trap called by the hypervisor. Then, the hypervisor will note that this particular VPN corresponds to a specific entry in the shadow page table. 

So the guest OS **is not actually updating the page table**. The hypervisor is responsible for the mapping between VPN and MPN, which is stored in the shadow page table.
- Note that the SPT can either be the hardware page table (if the processor is using the page table for its address translation) or it could be the TLB. 
- In either case the hypervisor is going to **install the translation from VPN to MPN** into the **TLB** and the **hardware page table**. 

So long as the translation has already been installed in the TLB and the hardware page table, the hypervisor, without the intervention of the guest operating system, can **translate the virtual page number of a user level process running on top of the guest operating system** directly to the **machine based number** using the TLB and the hardware page table. 

This is a trick to **make address translation efficient** because it's extremely crucial that on every memory access we don't go through the guest top rating system to do the address translation it's just not acceptable. And this is the trick that is used in VMware ESX server that is implemented on top of Intel hardware.

## 10. Efficient Mapping (Para Virtualization)
![[Pasted image 20250119132337.png]]

In a para-virtualized setting on the other hand the operating system knows that its physical memory is not contiguous. And therefore this burden of efficient mapping can be shifted into the guest operating system itself. 

So now the guest operating system is going to maintain contiguous physical memory makes it simpler in terms of all the other subsystems to do that. But it is also going to know that its notion of physical memory is not the same as machine memory and so it will map the **discontiguous physical memory** to real hardware pages. So that burden of doing the PPN to MPN mapping can be pushed into the guest operating system in a para-virtualized setting. 

So on an architecture like Intel where the page table is a data structure of the operating system and it is also used by the hardware to do the address translation, the responsibility of allocating and managing the hardware page table data structure can be shifted into the guest operating system. **In a fully virtualized setting it's not possible to do that because the operating system in a fully virtualized setting is unaware of the fact that it is not running on bare metal.** 

But in a paravirtualized setting since it is possible to do that it is more efficient to push this efficient mapping handling into the guest operating system. 

For example in Xen which is a paravirtualized hypervisor it provides a set of Hypercalls for the guest operating system to tell the Hypervisor about changes to the hardware page table. 

![[Pasted image 20250119132449.png]]

So for instance there is a call that says `create PT` and this allows a guest operating system to allocate and initialize a page frame that it has previously acquired a real page frame that it is previously acquired from the hypervisor as a hardware resource. It can target that physical page frame as a page table data structure. 

Recall that each guest operating system would have received a bunch of physical memory from the hypervisor at the beginning of establishing its footprint on the hypervisor. And so it can use one of those real physical memories to host a page table data structure on behalf of a new process that it launches now. 

So anytime a new process starts up in the guest operating system the guest operating system will make a hypercall to xen saying. Please create a page table for me and this is the page frame that I'm giving you to use as the page table.

So when the guest operating system has to operate this particular process which got launched then it can make another hypercall to the hypervisor saying "please switch to page table and here is the location of the page table." The hypervisor doesn't know about all these processes. All it understands is that there's a hypercall that says change the page table from whatever it used to be to this new page table. And that essentially results in this guest operating system switching the address space of the currently running process on the the bare hardware on the bare metal to P1 by this switch page table hypercall. 

Xen will do that appropriate thing of setting the hardware register of the processor to point to this page table data structure. In response to this hypercall from the guest operating system. If the process P1 one were to page fault at some point of time the page fault would be handled by the guest operating system. We'll talk about how it does that later on. But once Xen handles that page fault for P1 and says oh this particular virtual page of this process is now going to correspond to a physical frame that I own, I'm going to tell the hypervisor that **the mapping in the page table has to be set for this translation that I just established for the faulted page for this process.** So there's another hypercall that's available for updating a given page table data structure. And using this, the guest operating system can deal with modifications to the base table data structure. 

All the things that an operating system would have to do in a normal setting on bare metal, you have to be able to do in the setting where you have the hypervisor sitting between the real hardware and the guest operating system. And the three things that are required to be done in the conflicts of memory management in a para virtualized setting is:
- being able to create a brand new hardware address space for a newly launched process which involves creating a page table. That's a hypercall that's available. 
- When you do a context switch you want to switch the page table. That's a hypercall that's available when you do a context switch in the guest operating system from P1 to P2 the guest can tell the hypervisor that the page table to use from now on is such and so. That's the way the guest can do a context switch from one process to another. 
- And thirdly since not all of the address space or the memory footprint of a process would be physical memory if the currently running process were to incur at page four that has to be handled by the guest operating system. In handling that it establishes a mapping between the missing virtual page for this process and the physical frame in which the contents of the page is now contained. That mapping has been put into the page table for this particular process. Again that's something that only the hypervisor can do because it is a privileged operation happening inside the hardware. And for that purpose the hypervisor provides a hyper call that allows a guest operating system to update the base table data structure. 

So at the outset I said that handling virtual memory is a thorny issue. Doing the mapping from virtual to physical on every memory access without the intervention of the guest operating system is the key to good performance. And it can be done both in fully virtualized and paravirtualized setting by the tricks that we talked about just now.

## 11. Dynamically Increasing Memory 

![[Pasted image 20250119132807.png]]

The next thing we are going to talk about is how can we dynamically increase the amount of physical memory that's available to a particular operating system running on top of the hypervisor?  

As I mentioned memory requirements tend to be bursty and therefore the hypervisor has to be able to allocate real physical memory or machine memory on demand to the requesting guest operating systems on top of it. 

Let's look at this picture here. Let's assume that this is the total amount of machine memory that's available to the hypervisor. And the hypervisor has divvied up the available machine memory among these two operating systems. 

So the house, meaning the hypervisor, has no spare memory at all. It is completely divided up and given to these two guys. 

**What if the windows operating system experiences a burst in memory usage and therefore requires more memory from the hypervisor?** It may happen because of some resource hungry application that had been started up in windows maybe a video streaming application that is gobbling up a lot of physical memory and therefore windows needs more memory and comes to the hypervizor asking for additional hardware resources. But unfortunately the bank is empty. 

![[Pasted image 20250119132922.png]]

What the hyper-visor could do is recognize that: well maybe this  other operating system doesn't need all of the physical resources I allocated it. So I'm going to grab back a portion of the physical memory that Linux has. And once I get back this portion of physical memory that I previously allocated to Linux I can then give it to Windows to satisfy its sudden hunger for more memory. **Well this principle of robbing Peter to pay Paul can lead to unexpected and anomalous behavior of applications running on the guest operating system**. A standard approach of course would be to coax one of the guest operating system in this case perhaps Linux to give up some of its physical memory voluntarily to satsify the needs of a peer that is currently experiencing memory pressure.

## 12. Ballooning

![[Pasted image 20250119133002.png]]

That's the idea behind a technique that I'm going to describe to you called ballooning. The idea is to have a special device driver installed in every guest operating system. So even if it is a fully virtualized setting since device drivers can be installed on the fly with the cooperation of the guest operating system. The hypervisor can install the device driver which is called a balloon in the operating system. 

And this balloon device driver is the key for managing memory pressures that maybe experienced by a virtual machine or a desktop operating system in a virtualized setting. 

Let's say that the house needs more memory suddenly. And this may be a result of another guest operating system saying that it needs more memory. So what the hypervisor would do is it'll contact one of the guest operating system that currently is not actively using all of its memory.


![[Pasted image 20250119133046.png]]

And talk to this balloon driver that it has installed through a private channel that exists between the hypervisor and the balloon driver. So this is something that only the hypervisor knows about because this is a special device driver that the hypervisor has installed. It knows how to get to this device driver and tell it to do something. And in this case what the hypervisor is going to do is **tell this balloon device driver to inflate.** 

What that means is that this balloon device driver is going to make requests to the guest operating system saying I need more memory. I need more memory I need more memory. And the guest operating system will of course give this balloon driver the memory that it is requesting. And since the amount of physical memory that's available to the guest operating system is finite, if one process in this case the balloon driver is making requests from the guest saying give me more memory, guest has to necessarily make room for that by **paging out to the disk unwanted pages from the total memory footprint of all the processes that are currently running in this guest operating system.** 

![[Pasted image 20250119133214.png]]

And once it is done swapping out of pages and perhaps even entire processes out onto the disc, that's the way it can make room for the request that is coming from this balloon driver. 

Now once the balloon driver has gotten all this extra memory out of this guest operating system. It can return those physical memories. The real physical memory or the machine memory back to the hypervisor. So we started with this house needing more machine memory. And the way the house got that more machine memory is by **contacting its balloon driver** installed in one of the guest operating systems that is not actively using all of its resources. **Asking the balloon to inflate** and inflating the balloon is essentially meaning that we are **acquiring more memory from the guest operating system**. So you can see visually that the footprint of the balloon goes up because of the inflation. That means it's got a bigger memory footprint. And all of this memory footprint is extra resources that it can simply return to the hypervisor. It's not going to use it. It just wants to get it so that it can return it to the hypervisor. So that's this path here. 

So the opposite of what I've just described is a situation where the house needs less memory. Or in other words it has more memory to give away to guest operating system. So maybe it is this guest that requested more memory in the first place And in that case it wants the guest to get to the memory that it wanted. And the way it can do that is as follows. 

![[Pasted image 20250119133322.png]]

Once again the hypervisor through it's private channel will contact the balloon driver and tell the balloon driver to deflate the balloon. By deflate what is being instructed to the balloon driver is to contract its memory footprint. So if it contracts its memory footprint by deflating, **it is actually releasing memory into the guest operations**. So available physical memory. In the guest operating system is going to increase because of the fact that the balloon is deflating and giving out the memory that is currently sitting on. 

![[Pasted image 20250119133509.png]]

So now the guest operating system has more memory to play with. That's the effect of the balloon deflating is that the guest operating system has more memory. Which means that it can page in from the disk the working set of processes that are executing on this guest operating system. So that those processes can have more memory resources than they've been able to because of the balloon occupying a lot of the memory. 

So this technique of ballooning assumes **cooperation with the guest operating system**. So that based on the need of the hour the hypervisor can work with the guest operating system implicitly, without the guest really knowing about it because it is all happening through the balloon driver that has been installed by the hypervisor in each of the guest operating systems. And this technique of ballooning is **applicable to both fully virtualized and para-virtualized environments** to ease the **over commitment of memory by the hypervisor to the guests**. I mean it's sort of line airline reservations. You always notice that in airline reservation the airlines tend to sell more seats than what they have with the hope that someone is not going to show up. That's the same sort of thing that is being done here that there is a finite amount of physical resources available with the hypervisor. And it is doling it out to all the guest operating systems. And what it wants to do is it wants to be able to reacquire some of those resources to satisfy the needs of another guest operating system that is experiencing a memory pressure at any point of time.

## 13. Sharing Memory Across Virtual Machines 

![[Pasted image 20250119133534.png]]

Memory is a precious resource. You don't want to waste it. You want protection for the virtual machines from one another. But at the same time if there's an opportunity to share memory so that the available physical resource can be maximally utilized you want to be able to do that. So the question is **can we share memory across virtual machines without affecting the integrity of these machines?** Because protection is important but without affecting the protection can we actually enhance the utility of the available physical memory in the hyperviser? And the answer is yes think about it. 

You may have one instance of Linux contained in a virtual machine maybe hosted by me. And maybe you have the same Linux hosted in another virtual machine virtual machine number two. And in both of our virtual machines the memory footprint of the operating system is exactly the same. And even the applications that run on it are going to be exactly the same in terms of the contents, because of the same operating system it is just that we have two different instances. And let's say that we have Firefox running on this VM and similarly this Firefox running on this VM. 

This copy of Linux is going to have a page table unique for this particular Firefox instance. Similarly this instance of Linux is going to have a page-table deal structure unique for this instance of Firefox. And if all things are equal in terms of versions and so on then a particular virtual page of the Firefox instance running here and the Firefox instance running here is going to have the same content whether we are talking about this VM or this VM. **And so there is an opportunity to make both of those page table entires to point to the same machine page.** If we can do that then we are avoiding duplication. We're not comprimising safety but we're avoiding duplication. **This is particularly true for the core pages.** The core pages are immutable. So the core pages for this Firefox instance and this Firefox instance could actually share the same page in physical memory. And if it does that you're using the resources taht much more effectively in a virtualized setting. 

**One way of doing the sharing is by a cooperation between the virtual machines and the hypervisor**. In other words the guest OS has **hooks** that allows the hypervisor to mark pages "copy on write" and have the PPNs point to exactly the same machine page. And this way if a page is going to be written into by the operating system it'll result in a fault and we can make a copy of the page that is currently being shared across two different virtual machines. That is one way of doing it. That is **with the cooperation between the hypervisor and the virtual machines**. So that we can with their connivance **share machine pages across virtual machines** but mark the **entries in the individual page tables as copy on write** meaning that so long as they reach here perfectly fine. The minute any one of these guys wants to write into a particular page at that point you make a copy of it and make these two instances point to different machine pages so that is one way of doing it.

## 14. VM Oblivious Page Sharing 

![[Pasted image 20250119133811.png]]

An alternative is to achieve the same effect but completely oblivious to the guest operating system and this is used in VMware ESX server. **The idea is to use content-based sharing.** And in order to support that VM Ware has a data structure which is a hash table kept in the hypervisor. **And this hash table data structure contains a content hash of the machine pages.** 

So for instance this entry is saying that. For virtual machine number three. For it's physical page which is at this address 43f8 there is a machine page that hosts this physical page number of VM3 and that's contained in machine page number 123b and the content hash that is if you hash the contents of this memory page you get a signature. That signature is the content hash. That content hash is stored in this data structure. 

Now let's see how this data structure is used for doing VM-oblivious page sharing in the ESX Server. 

![[Pasted image 20250119133856.png]]

We want to know if there is some page in VM2 which is content wise the same as this page contained in VM3. **In particular we want to know if this physical page number.; PPN 2868 of VM 2 which is mapped to this machine page1096 is content-wise the same as this guy.** So how do we find that out? What we do is we take the contents of this machine page 1096 and create a content hash. So that's the content hash that you're going to generate a particular algorithm that the hypervisor is going to run to create a content hash. So we create a content hash for this page 1096 that corresponds to PPN 2868 of VM 2. 

Now we take this content hash and **look through hypervisor's data structure to see if there is a match** between this content hash that I created for this page and any page currently in the machine memory. Well we have a match. We have a match between the content hash for this page and the content hash of the page comtained in VM 3 43f8 which is mapped to MPN 123b. 

![[Pasted image 20250119134012.png]]

So now we've got this match. Can we know that this page and this page are exactly the same? **No.** It's only a hint at this point that this pages content hash is the same as this because this content hash for 123b was taken at some point of time. And we created this data structure to represent this as a hint frame which has a particular content hash. 

Now VM3 could have been using this page actively and modified it and if it has modified it then this content hash that we have in this data structure **may no longer be valid**. And therefore even though we've got a match it's only a hint not an absolute. It's only a hint. 

So once we have a match then we want to **do a full comparison** between these two guys. Make sure that these two guys are exactly the same full comparison upon match.

## 15. Successful Match 

![[Pasted image 20250119134059.png]]

If the content hash of 1096 and 123b are exactly the same then **we can modify the PPN to MPN mapping** in VM2 for the page 2868 which used to point to 1096 we can now make it point to 123b because they both are exactly the same content. 

![[Pasted image 20250119134125.png]]

And once we have done that then we **increment the reference count** to this hash table entry to 2 indicating that there are 2 different virtual machines that map to the same machine page 123b. And we're also going to remember that these two mappings between PPN 2868 to 123b 43f8 to 123b are "copy on write" entries indicating that they can share this page so long as these 2 virtual machines are reading it. But if any one of them tries to write it at that point for the integrity of the system **you have to make a copy of this page** and change the mappings for those PPNs to go to different MPNs. That's the reason that we want to do this copy on write. 

And now we can free-up page number 1096. So there is one more page frame that's available for the house in terms of allocation. 

Because all of these things that I mentioned just now are fairly labor-intensive operations. So you don't want to do this when there is active usage of the system. So scanning the pages that is, going through all of a virtual machine's pages to see if pages that are contained in a virtual machine may already be present in the machine memory reflecting the contents of some of the virtual machine, that kind of scanning you want to do it as a **background activity** of the server when it is lightly loaded. Looking for such matches and mapping the virtual machines to share the same machine page and freeing up machine memory for allocation by the hypervisor. 

And the important thing that you have to notice is that as opposed to the earlier mechanism that I mentioned where I said that with the cooperation of the guest operating system the hypervisor can get into the page table data structures inside the guest operating systems. No such thing here. **It is completely done oblivious to the guest operating systems** and therefore there is no **change that needs to be made to the guest operating systems** in order to do the sharing in an oblivious way. 

And this technique is **applicable to both fully virtualized as well as the paravirtualized environments**. Because basically all that we are saying is that let's go through the memory contents of a virtual machine and see if the memory contents of the virtual machine any particular page frames can be shared with other virtual machines and if so let's do that. And free up some memory. That's the idea can be applied to both fully virtualized and para virtualized settings.

## 16. Memory Allocation Policies 

Up to now we talked about mechanisms for virtualizing the memory resource. In particular for dealing with dynamic memory pressure and sharing machine pages across VM's. **A higher level issue is the policies we have to use for allocating and reclaiming memory from the domains to which we've allocated them in the first place.** Ultimately the **goal of virtualization** is **maximizing the utilization of the resources**. Memory is a precious resource. 

### Share-based approach

Virtualized environments may use different policies for memory allocation. One can be a pure share based policy. The idea here is you pay less you get less. That's the idea. 

So if you have a service level agreement with the data center then the data center gives you a certain amount of resources based on the amount of dollars you put on a table. So that is a pure share based approach. 

**The problem with a share based approach is of course the fact that it could lead to hoarding.** If a virtual machine gets a bunch of resources and it's not really using it it's just wasting it. 

### Working-set based approach

Now the desired behavior is if the working-set of a virtual machine goes up, you give it more memory. If it's working-set shrinks get back the memory so that you can give it to somebody else. So working-set-based approach would be the saner approach. But at the same time if I paid money I need my resources. 

### Dynamic idle-adjusted shares approach

So one thing that can be done is sort of put these two ideas together in implementing a dynamic idle-adjusted shares approach. In other words you're going to tax the guys that are hoarders. So you tax the idle pages more than active pages. If I've given you a bunch of resources if you're actively using it more power to you. But if you're hoarding it I'm going to tax you. I'm going to take away the resources that I gave you. And you may not even notice it because you're not using it anyway. So that's the idea in this dynamic idle-adjusted shares approach. 

And now what is this tax? Well we could make the tax rate 0% that is plutocracy meaning you paid for it you got it you can sit on it I'm not going to tax you that's one approach. Or I could make the tax 100% meaning that if you've got some resources and you're not using it I'm going to take all of it away. So that's the wealth redistribution. Sort of a socialistic policy use it or lose it. 

And in other words if you make the tax 100% we are ignoring shares altogether. Now something in between is probably the best way to do it so for instance if you use a tax rate of 50% or 75% saying if you have idle pages then the tax rate is 50% there's a 50% chance I'll take it away. And that's what is being done in the VMware ESX server today in terms of how to allocate memory to the domains that need it. You are going to use share based approach but if you're not actively using it we're going to take it away from you. **And of course we'll give it back to you if you start using it and that's the reason why you don't want to make the tax 100% but have it somewhat smaller.** So by having a tax rate that is not quite 100% but maybe 50% or 75% we can reclaim most of the idle memory from the VM's that are not actively using it. But at the same time since we're not taxing idle pages 100% it allows for sudden working set increases that might be experienced by a particular domain. Suddenly a domain starts needing more memory at that point it may still have some reserves in terms of the idle pages that I did not take away from that particular domain. So the key point is that you don't want to tax at 100% because this allows for sudden working set increases that may be there in a virtual machine that happens to be idle for some time but suddenly work picks up.