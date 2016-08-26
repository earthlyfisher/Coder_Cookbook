复制文件是一个普通的操作，对应java不同版本的实现，延伸了多种实现方式，通过以下，一一分析：
##通过普通IO流处理
```java
public static void copyInputStreamToOutputStream(
            InputStream inputStream, OutputStream outputStream,
            int fileAvailable) throws IOException {
        if (fileAvailable <= COPY_FILE_SIZE) {
            byte[] bytes = new byte[fileAvailable];
            new DataInputStream(inputStream).readFully(bytes);
            outputStream.write(bytes);
            outputStream.flush();
        } else {
            byte[] bytes = new byte[COPY_FILE_SIZE];
            int len = inputStream.read(bytes);
            while (len > 0) {
                outputStream.write(bytes, 0, len);
                outputStream.flush();
                len = inputStream.read(bytes);
            }
        }
    }
```
##通过`ByteBuffer`处理
```
public static void copyFileByByteBuffer(String srcFileName , String dstFileName , 
			ByteBuffer byteBuffer) throws IOException {
		File srcFile = new File(srcFileName);
		File dstFile = new File(dstFileName);
		if(!srcFile.exists()) throw new RuntimeException("src file not exists!");
		if(dstFile.exists()) throw new RuntimeException("dst file exists!");
		FileChannel inFileChannel = null;
		FileChannel outFileChannel = null;
		FileInputStream in = new FileInputStream(srcFile);
		FileOutputStream out = new FileOutputStream(dstFile);
		try {
			inFileChannel = in.getChannel();
			outFileChannel = out.getChannel();
			int size = inFileChannel.read(byteBuffer);
			while(size > 0) {
				byteBuffer.flip();
				outFileChannel.write(byteBuffer);
				byteBuffer.clear();
				size = inFileChannel.read(byteBuffer);
			}
		}finally {
			closeStream(in , out);
			byteBuffer.clear();
		}
	}
```
##通过channel处理
```java
public static void copyFileByChannel(String srcFileName , String dstFileName) throws IOException {
		File srcFile = new File(srcFileName);
		File dstFile = new File(dstFileName);
		if(!srcFile.exists()) throw new RuntimeException("src file not exists!");
		if(dstFile.exists()) throw new RuntimeException("dst file exists!");
		FileChannel inFileChannel = null;
		FileChannel outFileChannel = null;
		FileInputStream in = new FileInputStream(srcFile);
		FileOutputStream out = new FileOutputStream(dstFile);
		try {
			long position = 0;
			inFileChannel = in.getChannel();
			outFileChannel = out.getChannel();
			while(position < inFileChannel.size()) {
				long realCopy = outFileChannel.transferFrom(inFileChannel , position , COPY_FILE_SIZE);
				position += realCopy;
			}
			//inFileChannel.transferTo(0, srcFile.length(), outFileChannel);//最大支持2G
		}finally {
			closeStream(in , out);
		}
	}
```
##通过MappedByteBuffer处理
```java
public static void copyFileByMappedByteBuffer(String srcFileName , String dstFileName) throws IOException {
		FileChannel inFileChannel = new RandomAccessFile(srcFileName , "r").getChannel();
		FileChannel outFileChannel = new RandomAccessFile(dstFileName , "rw").getChannel();
		try {
			long fileSize = inFileChannel.size();
			long position = 0;
			while(position < fileSize) {
				long copyFileSize = Math.min((fileSize - position), COPY_FILE_SIZE);
				MappedByteBuffer mappedByteBuffer = outFileChannel.map(MapMode.READ_WRITE,  position ,  copyFileSize);//copyFileSize最大支持2^31-1
				inFileChannel.read(mappedByteBuffer);
				position += mappedByteBuffer.position();
			}
		}finally {
			closeStream(outFileChannel);
		}
	}
```
当然以上处理，会生成大量的对象，内存使用会急速上升.
```java
	public static void copyFileByFiles(String srcFileName, String dstFileName) {
		try {
			System.out.println("copy start...");
			Files.copy(Paths.get(srcFileName), Paths.get(dstFileName), StandardCopyOption.REPLACE_EXISTING);
			System.out.println("copy end...");
		} catch (Exception e) {
			e.printStackTrace();
		}
	```
