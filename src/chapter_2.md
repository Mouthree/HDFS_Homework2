# HDFS_java

## 先开Hadoop

```shell
cd /usr/local/hadoop
./sbin/start-dfs.sh #启动hadoop
```

## 之后在本地再新建一个文件

```shell
vim hdfs_java.txt
```

进入之后按`i`开始编辑, 按`esc`退出编辑, 输入`:wq`保存并退出

---

## `(1)`向HDFS中存入文件,如果有则选择追加还是覆盖

进入eclipse

```shell
cd /usr/local/eclipse
./eclipse 
```

## [点击查看如何新建项目](#关于如何创建hdfs项目)

在项目中写入如下代码

```java
import java.io.IOException;
import java.util.Scanner;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.FileReader;
import java.io.BufferedReader;

public class one {
	public static void main(String[] args) throws Exception {
		Scanner scanner = new Scanner(System.in);
		System.out.println("input local address");
		String file_address = scanner.nextLine();
		System.out.println("input file name");
		String file_name = scanner.nextLine();
		System.out.println("input folder address");
		String folder_address = scanner.nextLine();
		one m = new one();
		m.upload_file(file_address, file_name, folder_address);
		scanner.close();
	}
	
	public void upload_file(String file_address, String file_name, String folder_address) throws Exception {
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path local_file = new Path(file_address + "/" + file_name);
        Path over_file = new Path(folder_address);
        
        if (fs.exists(new Path(folder_address + "/" + file_name))) {
            System.out.println("The file is already there");
            System.out.println("Choose append or overwrite  |  (a, o)");
            Scanner scanner = new Scanner(System.in);
            String t = scanner.nextLine();
            if(t.equals("a")) {
            	BufferedReader br = null;
            	FSDataOutputStream out = fs.append(new Path(folder_address + "/" + file_name));
            	br = new BufferedReader(new FileReader(file_address + "/" + file_name));
                String line;
                while ((line = br.readLine()) != null) {
                    out.writeBytes(line + "\n");
                }
                out.close();
                br.close();
                System.out.println("successfully append");
            }else if(t.equals("o")) {
            	fs.copyFromLocalFile(local_file, over_file);
            	System.out.println("successfully overwrite");
            }
            scanner.close();
        } else {
        	fs.copyFromLocalFile(local_file, over_file);
        	System.out.println("OK");
        }
        fs.close();
	}
}
```

### 流程
1. 首先输入要`上传的文件`的`本地路径`

2. 接着输入要上传的文件名, 要带后缀

3. 最后输入要传入的hdfs的路径

4. 新开一个命令行窗口, *直接右键之前打开的,然后点击新建即可*

5. 输入`cd /usr/local/hadoop`进入hadoop文件夹, 接着输入`hdfs dfs -cat /user/fileTest/hdfs_java.txt`来查看操作是否成功,成功了则会显示文件内容

6. 第一次上传会直接结束,再次运行程序会提示覆盖还是追加,根据文本提示操作


### 实机演示

<video controls src="hdfs_1.mp4" title="Title"></video>

### [代码详解](#1)

---

## `(2)`从HDFS上下载文件到本地,如果重名则重命名再下载

首先创建文件二

输入如下代码

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Scanner;
import java.nio.file.Files;
import java.nio.file.Paths;
public class two {
	public static void main(String[] args) throws Exception {
		Scanner scanner = new Scanner(System.in);
		System.out.println("input folder address");
		String folder_address = scanner.nextLine();
		System.out.println("input file name");
		String file_name = scanner.nextLine();
		System.out.println("input local address");
		String file_address = scanner.nextLine();
		two m = new two();
		m.download_file_local(file_address, file_name, folder_address);
		scanner.close();
	}
	
	public void download_file_local(String file_address, String file_name, String folder_address) throws Exception {
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path local_address = new Path(file_address);
        Path hdfs_file = new Path(folder_address + "/" + file_name);
        if(Files.exists(Paths.get(file_address + "/" + file_name))) {
        	System.out.println("The file is already there");
        	System.out.println("input new file name");
        	Scanner scanner = new Scanner(System.in);
        	String new_file_name = scanner.nextLine();
        	Path new_local = new Path(file_address + "/" + new_file_name);
        	fs.copyToLocalFile(false, hdfs_file, new_local, true);
        	scanner.close();
        	System.out.println("successfully download");
        }else {
        	fs.copyToLocalFile(false, hdfs_file, local_address, true);
        	System.out.println("OK");
        }
	}
}
```

### 流程

1. 在上一部新建的命令行窗口中输入`rm -f hdfs_java.txt`来删除之前创建的文件,由于在实验一中我们上传了一个文件到hdfs中,所以不用担心文件没了

2. 输入`ls`查看是否删除成功

1. 首先输入要下载的文件在hdfs中的文件夹

2. 输入要下载到本地的文件名

3. 输入要下载到本地的路径

4. 再次来到之前新建的命令行窗口中,输入`ls`查看本地文件内容是否下载成功

5. 再次运行代码, 根据提示输入新的文件名,再次按照`上一步`操作,可以看到重命名的文件

### 实机演示

<video controls src="hdfs_2.mp4" title="Title"></video>

### [代码详解](#2)

---

## `(3)`将HDFS中指定文件的内容输出到终端中

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.BufferedReader;
import org.apache.hadoop.fs.FSDataInputStream;
import java.io.InputStreamReader;
public class three {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path hdfsPath = new Path(hdfsFilePath);
        FSDataInputStream in = fs.open(hdfsPath);
        BufferedReader br = new BufferedReader(new InputStreamReader(in));
        String line;
        while ((line = br.readLine()) != null) {
        	System.out.println(line);
        }
        br.close();
        in.close();
        System.out.println("");
        System.out.println("----------------------");
        System.out.println("successfully out");
        scanner.close();
	}
}
```

### 流程

1. 按照提示输入要显示的文件在hdfs中的路径

2. 直接运行即可

### 实机演示

<video controls src="hdfs_3.mp4" title="Title"></video>

### [代码详解](#3)

---

## `(4)`显示HDFS中指定的文件的读写权限、大小、创建时间、路径等

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FileStatus;
public class four {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path hdfsPath = new Path(hdfsFilePath);
        FileStatus fileStatus = fs.getFileStatus(hdfsPath);
        System.out.println("file address: " + fileStatus.getPath());
        System.out.println("file size: " + fileStatus.getLen() + " 字节");
        System.out.println("block size: " + fileStatus.getBlockSize() + " 字节");
        System.out.println("number of copies: " + fileStatus.getReplication());
        System.out.println("user: " + fileStatus.getOwner());
        System.out.println("grouping: " + fileStatus.getGroup());
        System.out.println("permission: " + fileStatus.getPermission());
        System.out.println("change time: " + fileStatus.getModificationTime());
        System.out.println("visit time: " + fileStatus.getAccessTime());
        scanner.close();
	}
}

```

### 流程

1. 按照提示输入要显示的文件在hdfs中的路径

### 实机演示

<video controls src="hdfs_4.mp4" title="Title"></video>

### [代码详解](#4)



---

## `(5)`给定HDFS中某一个目录，输出该目录下的所有文件的读写权限、大小、创建时间、路径等信息，如果该文件是目录，则递归输出该目录下所有文件相关信息

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FileStatus;
public class five {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path hdfsPath = new Path(hdfsFilePath);
        FileStatus[] fileStatuses = fs.listStatus(hdfsPath);
        for (FileStatus fileStatus : fileStatuses) {
	        System.out.println("file address: " + fileStatus.getPath());
	        System.out.println("file size: " + fileStatus.getLen() + " 字节");
	        System.out.println("block size: " + fileStatus.getBlockSize() + " 字节");
	        System.out.println("number of copies: " + fileStatus.getReplication());
	        System.out.println("user: " + fileStatus.getOwner());
	        System.out.println("grouping: " + fileStatus.getGroup());
	        System.out.println("permission: " + fileStatus.getPermission());
	        System.out.println("change time: " + fileStatus.getModificationTime());
	        System.out.println("visit time: " + fileStatus.getAccessTime());
	        System.out.println("");
	        System.out.println("---------------------");
	        System.out.println("");
        }
        scanner.close();
	}
}
```

### 流程

1. 按照提示输入文件夹路径

### 实机演示

<video controls src="hdfs_5.mp4" title="Title"></video>

### [代码详解](#5)



---

## `(6)`提供文件路径,对其创建或删除,创建时自动创建路径

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class six {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path file_address = new Path(hdfsFilePath);
        System.out.println("delete or create | (d/c)");
        String input_choose = scanner.nextLine();
        if(input_choose.equals("d")) {
        	if(fs.exists(new Path(hdfsFilePath))) {
        		fs.delete(file_address, true);
        		System.out.println("successfully delete");
        	}else {
        		System.out.println("no file");
        	}
        }else if(input_choose.equals("c")) {
        	fs.create(file_address, true, fs.getConf().getInt("io.file.buffer.size", 4096));
        	System.out.println("OK");
        }
        scanner.close();
	}
}
```

### 流程

1. 按照提示输入**hdfs**中的文件路径,根据文本提示操作


### 实机演示

<video controls src="hdfs_6-1.mp4" title="Title"></video>

### [代码详解](#6)

---

## `(7)`HDFS创建或删除目录,创建时如果没有就自动新建,删除时如果目录不为空则问用户

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FileStatus;

public class seven {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path file_address = new Path(hdfsFilePath);
        System.out.println("delete or create | (d/c)");
        String input_choose = scanner.nextLine();
        if(input_choose.equals("d")) {
        	if(fs.exists(new Path(hdfsFilePath))) {
        		FileStatus[] fileStatuses = fs.listStatus(file_address);
        		if(fileStatuses.length == 0) {
        			fs.delete(file_address, true);
        			System.out.println("successfully delete");
        		}else {
        			System.out.println("This folder is not empty, do you want to delete? (y/n)");
        			String t = scanner.nextLine();
        			if(t.equals("y")) {
        				fs.delete(file_address, true);
        				System.out.println("successfully delete");
        			}else {
        				System.out.println("OK");
        			}
        		}
        	}else {
        		System.out.println("no file");
        	}
        }else if(input_choose.equals("c")) {
        	fs.create(file_address, true, fs.getConf().getInt("io.file.buffer.size", 4096));
        	System.out.println("OK");
        }
        scanner.close();
	}
}

```

### 流程

0. 懒得写了,直接看`实机演示`, 后面的几个也都一样

### 实机演示

<video controls src="HDFS_7.mp4" title="Title"></video>

### [代码详解](#7)

---

## `(8)`向HDFS中指定文件追加内容，由用户指定内容追加到原有文件开头或结尾

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.OutputStream;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import org.apache.hadoop.io.IOUtils;
public class eight {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path file_address = new Path(hdfsFilePath);
        if(fs.exists(new Path(hdfsFilePath))) {
        	System.out.println("add head or tail | (h/t)");
        	String t = scanner.nextLine();
        	if(t.equals("h")) {
        		System.out.println("input what you want add");
        		String a = scanner.nextLine();
        		OutputStream out = fs.create(new Path(hdfsFilePath.replace(".", "_mouthree" + ".")));
        		out.write(a.getBytes());
        		out.write("\n".getBytes());
        		BufferedReader reader = new BufferedReader(new InputStreamReader(fs.open(new Path(hdfsFilePath))));
        		while ((a = reader.readLine()) != null) {
                    out.write(a.getBytes());
                    out.write("\n".getBytes());
                }
        		IOUtils.closeStream(reader);
                IOUtils.closeStream(out);
                fs.delete(new Path(hdfsFilePath), true);
                fs.rename(new Path(hdfsFilePath.replace(".", "_mouthree" + ".")), new Path(hdfsFilePath));
                System.out.println("OK");
        	}else {
        		OutputStream out = fs.append(file_address);
        		System.out.println("input what you want add");
        		String a = scanner.nextLine();
        		out.write(a.getBytes());
        		out.write("\n".getBytes());
        		out.close();
        		System.out.println("OK");
        	}
        }else {
        	System.out.println("no file");
        }
        scanner.close();
	}
}
```

### 流程

### 实机演示

<video controls src="HDFS_8.mp4" title="Title"></video>

### [代码详解](#8)

---

## `(9)`删除HDFS中指定的文件

```java

import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class nine {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        if(fs.exists(new Path(hdfsFilePath))) {
        	fs.delete(new Path(hdfsFilePath), true);
        	System.out.println("OK");
        }else {
        	System.out.println("no file");
        }
        scanner.close();
	}
}

```

### 流程

### 实机演示

<video controls src="hdfs_9.mp4" title="Title"></video>

### [代码详解](#9)

---

## `(10)`删除HDFS中指定目录，由用户指定目录中如果存在文件时是否删除目录

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
public class ten {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path file_address = new Path(hdfsFilePath);
		if(fs.exists(new Path(hdfsFilePath))) {
    		FileStatus[] fileStatuses = fs.listStatus(file_address);
    		if(fileStatuses.length == 0) {
    			fs.delete(file_address, true);
    			System.out.println("successfully delete");
    		}else {
    			System.out.println("This folder is not empty, do you want to delete? (y/n)");
    			String t = scanner.nextLine();
    			if(t.equals("y")) {
    				fs.delete(file_address, true);
    				System.out.println("successfully delete");
    			}else {
    				System.out.println("OK");
    			}
    		}
    	}else {
    		System.out.println("no file");
    	}
		scanner.close();
	}
}
```

### 流程

### 实机演示

同第七题,直接输入要删除的文件夹名就行

### [代码详解](#10)

---

## `(11)`在HDFS中，将文件从源路径移动到目的路径

```java
import java.util.Scanner;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class eleven {
	public static void main(String[] args) throws Exception {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		System.out.println("input where you want move");
		String hdfsOverPath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        Path file_address = new Path(hdfsFilePath);
        Path file_over_address = new Path(hdfsOverPath);
        if(fs.exists(file_address) && fs.exists(file_over_address)) {
        	if(fs.rename(file_address, file_over_address)) {
        		System.out.println("successfully move");
        	}else {
        		System.out.println("ERROR");
        	}
        }else {
        	System.out.println("some folder is not in there");
        }
        scanner.close();
	}
}

```

### 流程

### 实机演示

<video controls src="hdfs_11.mp4" title="Title"></video>

### [代码详解](#11)

---

## `(12)`实现按行读取,一次返回一行,如果空则返回 "空"

### 注意:该实验需要创建`两个`class

### twelve.java的内容

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import java.io.IOException;
import java.util.Scanner;

public class twelve {
	public static void main(String[] args) throws IOException {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        FileSystem fs = FileSystem.get(conf);
        MyFSDataInputStream.cat(fs, hdfsFilePath);
        System.out.println("-----------------");
        System.out.println("");
        System.out.println("OK");
        scanner.close();
	}
}
```

### MyFSDataInputStream的内容

```java
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.*;

public class MyFSDataInputStream extends FSDataInputStream {

	public MyFSDataInputStream(InputStream in) {
		super(in);
		// TODO Auto-generated constructor stub
	}
	
	public static String readline(BufferedReader br) throws IOException {
		char[] data = new char[1024];
		int off = 0;
		while (br.read(data, off, 1) != -1) {
			if (String.valueOf(data[off]).equals("\n") ) {
				off += 1;
				break;
			}		
			off += 1;
		}
		if (off> 0) {
			return String.valueOf(data);
		} else {
			return null;
		}
	}
	public static void cat(FileSystem fs, String FilePath) throws IOException {
		Path remotePath = new Path(FilePath);
		FSDataInputStream in = fs.open(remotePath);
		BufferedReader br = new BufferedReader(new InputStreamReader(in));
		String line = null;
		while ( (line = MyFSDataInputStream.readline(br)) != null ) {
			System.out.println(line);
		}
		br.close();
		in.close();
		fs.close();
	}
}
```

### 流程

### 实机演示

<video controls src="hdfs_12.mp4" title="Title"></video>

### [代码详解](#12)

---

## `(13)`用`java.net.URL`和`org.apache.hadoop.fs.FsURLStreamHandlerFactory`编程完成输出HDFS中指定文件的文本到终端中

```java
import java.io.IOException;
import java.util.Scanner;
import java.net.URL;
import org.apache.hadoop.conf.Configuration;
import java.io.InputStream;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;

public class thirteen {
	static{
        URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
    }
	public static void main(String[] args) throws IOException {
		System.out.println("input hdfs file address");
		Scanner scanner = new Scanner(System.in);
		String hdfsFilePath = scanner.nextLine();
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS","hdfs://localhost:9000");
        conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
        conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER");
        conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.enable", true);
        InputStream in = new URL("hdfs://localhost:9000" + hdfsFilePath).openStream();
        IOUtils.copyBytes(in, System.out, 2048,false);
        IOUtils.closeStream(in);
        scanner.close();
        System.out.println("-----------------");
        System.out.println("");
        System.out.println("OK");
	}
}
```

### 流程

### 实机演示

<video controls src="hdfs_13.mp4" title="Title"></video>

### [代码详解](#13)

---
---

# 附录一 : 一些基础操作

## 关于如何创建HDFS项目

在进入eclipse之后,按照如下视频操作

<video controls src="java_f_1.mp4" title="Title"></video>

实验几,命名就叫几,例如实验二, 项目名字叫`HDFS_2`, 类名叫`two`

---
---

# 附录二 : 代码逻辑分析

## 1

<video controls src="code_1.mp4" title="Title"></video>

---

## 2

<video controls src="code_2.mp4" title="Title"></video>

---

## 3

<video controls src="code_3.mp4" title="Title"></video>

---

## 4

<video controls src="code_4.mp4" title="Title"></video>

---

## 5

<video controls src="code_5.mp4" title="Title"></video>

---

## 6

<video controls src="code_6.mp4" title="Title"></video>

---

## 7

<video controls src="code_7.mp4" title="Title"></video>

---

## 8

<video controls src="code_8.mp4" title="Title"></video>

---

## 9

实现和之前一样,不讲了

---

## 10

同上

---

## 11

<video controls src="code_11.mp4" title="Title"></video>

---

## 12

<video controls src="code_12.mp4" title="Title"></video>

---

## 13

<video controls src="code_13.mp4" title="Title"></video>

---
