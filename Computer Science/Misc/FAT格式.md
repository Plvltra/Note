结构图示
![[Pasted image 20250306181552.png]]

- volume: A logically contiguous sector address space as specified in the relevant standard for recording.(volume由若干cluster组成)
- cluster: A unit of allocation comprising a set of logically contiguous sectors. Each cluster within the volume is referred to by a cluster number “N”. All allocation for a file must be an integral multiple of a cluster.(cluster由若干sector组成)
- sector: A unit of data that can be accessed independently of other units on the media
- **Reverved Region**: 包含BPB(BIOS Parameter Block) structure, 相当于FAT系统的元信息
- **FAT Region**: 记录clusterId与clusterId的链接关系(例如一个文件由许多cluster组成, 记录以单向链表为数据结构的信息)
![[Pasted image 20250306182750.png]]
- **Root Directory Region**(FAT32没有): 包含根目录的信息, FAT32在Data段，由RootCluster记录
- **File & Directory(Data) Region**: 又叫数据区，包含cluster的实际内容。对于目录项，cluster由子项的directory entry组成，例如一个目录下有十个文件，则cluster包含十个directory entry。对于文件，cluster包含文件数据。
