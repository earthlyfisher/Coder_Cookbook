##direct-buffer,non-direct-buffer
   Direct vs Non-direct vs MappedByteBuffer in Java: Direct buffers are
	 allocated outside heap and they are not in control of Garbage Collection
	 while non-direct buffers are simply a wrapper around byte arrays, located
	 inside heap. Memory mapped files can be accessed by using
	 MappedByteBuffer, which is also a direct buffer. One more thing to
	 remember is that default order of bytes in ByteBuffer is BIG_ENDIAN,
	 which means the bytes of a multibyte value are ordered from most
	 significant to least significant.
	 
	 Key advantage of Memory Mapped File is that operating system takes care
	 of reading and writing and even if your program crashed just after
	 writing into memory. OS will take care of writing content to File.

```java
	@Test
	public void testIo() throws IOException {
		int count = 10485760; // 10 MB

		RandomAccessFile memoryMappedFile = new RandomAccessFile("D:\\test\\largeFile.txt", "rw");

		// Mapping a file into memory
		MappedByteBuffer out = memoryMappedFile.getChannel().map(FileChannel.MapMode.READ_WRITE, 0, count * 100);

		// Writing into Memory Mapped File
		for (int i = 0; i < count; i++) {
			out.put((byte) 'A');
		}

		System.out.println("Writing to Memory Mapped File is completed");

		// reading from memory file in Java
		for (int i = 0; i < 10; i++) {
			System.out.print((char) out.get(i));
		}
		System.out.println();
		System.out.println("Reading from Memory Mapped File is completed");

	}
```
**Difference between Direct and Mapped ByteBuffer in Java**
![](../image/Difference between Direct and Mapped ByteBuffer in Java.jpg)
