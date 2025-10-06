# Hadoop Pseudo-Distributed Setup Guide for WSL (Ubuntu)

## Table of Contents

* [About This Project](#about-this-project)
* [Author](#author)
* [Prerequisites](#prerequisites)
* [Architecture Overview](#architecture-overview)
* [Installation Guide](#installation-guide)
    * [Step 1: Update System and Install Dependencies](#step-1-update-system-and-install-dependencies)
    * [Step 2: Download and Configure Hadoop](#step-2-download-and-configure-hadoop)
    * [Step 3: Set Up Environment Variables](#step-3-set-up-environment-variables)
    * [Step 4: Configure Hadoop XML Files](#step-4-configure-hadoop-xml-files)
    * [Step 5: Create HDFS Directories](#step-5-create-hdfs-directories)
    * [Step 6: Configure SSH for Hadoop](#step-6-configure-ssh-for-hadoop)
    * [Step 7: Format the NameNode](#step-7-format-the-namenode)
    * [Step 8: Start Hadoop Services](#step-8-start-hadoop-services)
    * [Step 9: Validate Installation](#step-9-validate-installation)
    * [Step 10: Test with Sample File](#step-10-test-with-sample-file)
* [Troubleshooting](#troubleshooting)
* [Practical Exercises](#practical-exercises)
* [Additional Resources](#additional-resources)
* [License](#license)
* [Contact](#contact)
* [Acknowledgments](#acknowledgments)
* [Changelog](#changelog)

## About This Project

This repository contains a complete, step-by-step tutorial for setting up **Apache Hadoop 3.3.6** in **Pseudo-Distributed mode (Single Node)** on **WSL (Windows Subsystem for Linux)** with Ubuntu 20.04.

I created this guide as an educational resource for my students and anyone interested in learning Hadoop from scratch. Throughout this tutorial, I document every command used, explain each component's purpose, and provide troubleshooting tips based on real-world scenarios.

What you'll learn:

* How to install and configure Hadoop in pseudo-distributed mode
* Understanding **HDFS (Hadoop Distributed File System)** architecture
* Working with **YARN** resource management
* Basic HDFS operations and file management
* How to validate your Hadoop installation

This is a **hands-on tutorial** designed for beginners with basic Linux knowledge.

## Author

**Thiago C. Soares**  
Email: thiagocsoares@gmail.com  
LinkedIn: [linkedin.com/in/thiago-csoares](https://linkedin.com/in/thiago-csoares)

Created for educational purposes to help students and developers understand Hadoop fundamentals.

## Prerequisites

Before starting this tutorial, ensure you have:

* **Windows 10/11** with WSL 2 enabled
* **Ubuntu 20.04 LTS** installed on WSL (or similar Debian-based distribution)
* At least **4GB RAM** available for WSL
* **10GB free disk space** minimum
* Basic Linux command-line knowledge
* Internet connection for downloading packages

Note: If you don't have WSL installed, follow the **official Microsoft WSL installation guide**.

## Architecture Overview

In this Single Node setup, all Hadoop daemons run on the same machine. Here's what we'll be running:

```
┌─────────────────────────────────────────────────────┐
│                 Single Node Hadoop                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────┐       │
│  │         HDFS Layer (Port 9000)          │       │
│  ├──────────────────────────────────────────┤       │
│  │  • NameNode (Metadata Management)       │       │
│  │  • DataNode (Data Storage)              │       │
│  │  • SecondaryNameNode (Checkpoint)       │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
│  ┌──────────────────────────────────────────┐       │
│  │      YARN Layer (Resource Manager)      │       │
│  ├──────────────────────────────────────────┤       │
│  │  • ResourceManager (Job Scheduling)     │       │
│  │  • NodeManager (Task Execution)         │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Key Components:**

* **NameNode**: Manages the file system namespace and metadata
* **DataNode**: Stores actual data blocks
* **SecondaryNameNode**: Creates checkpoints of the namespace
* **ResourceManager**: Manages cluster resources and scheduling
* **NodeManager**: Manages containers and monitors resource usage

## Installation Guide

### Step 1: Update System and Install Dependencies

First, update system packages and install dependencies required for Hadoop.

**Update package lists and upgrade existing packages**  
This ensures the latest security patches and software versions.

```bash
sudo apt update && sudo apt upgrade -y
```

**Install required packages**  
Install the following packages needed for Hadoop:

* `ssh`: Enables Hadoop daemons to communicate via SSH protocol
* `pdsh`: Parallel Distributed Shell for managing multiple hosts
* `openjdk-11-jdk`: Java Development Kit, required as Hadoop is written in Java
* `wget`: Tool to download files from the web
* `tar`: Utility to extract compressed archives

```bash
sudo apt install ssh pdsh openjdk-11-jdk wget tar -y
```

**Verify Java installation**  
Run the following command to confirm Java is installed correctly. It should display Java version 11.x.x.

```bash
java -version
```

**Expected output**:
```bash
openjdk version "11.0.x" 2024-xx-xx
OpenJDK Runtime Environment (build 11.0.x+x-Ubuntu-...)
OpenJDK 64-Bit Server VM (build 11.0.x+x-Ubuntu-...)
```

**Why these packages?**  
- **SSH**: Hadoop uses SSH to start and stop daemons, even in single-node mode.  
- **PDSH**: Allows executing commands across multiple machines simultaneously.  
- **Java 11**: Hadoop 3.3.6 requires Java 8 or 11 to run.  
- **wget/tar**: Essential for downloading and extracting Hadoop binaries.

### Step 2: Download and Configure Hadoop

Download the official Hadoop distribution and set up the directory structure.

```bash
# Download Hadoop 3.3.6 from Apache mirrors
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz -P ~

# Extract the downloaded archive
tar -xvzf ~/hadoop-3.3.6.tar.gz -C ~

# Rename the directory for easier access
mv ~/hadoop-3.3.6 ~/hadoop
```

**What just happened?**  
You downloaded approximately 600MB of Hadoop binaries, extracted them, and created a clean directory structure. The `~/hadoop` folder now contains all necessary files to run Hadoop.

**Directory structure overview**:
```
~/hadoop/
├── bin/          # Hadoop command-line scripts
├── etc/hadoop/   # Configuration files (we'll modify these)
├── sbin/         # Start/stop scripts for daemons
├── share/        # Hadoop JARs and libraries
└── lib/          # Native libraries
```

### Step 3: Set Up Environment Variables

Configure environment variables to make Hadoop commands accessible from any directory.

```bash
# Open the bash configuration file
nano ~/.bashrc
```

Add the following lines at the end of the file:
```bash
# Set Hadoop home directory
export HADOOP_HOME=~/hadoop
# Configure Hadoop environment variables
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
# Add Hadoop binaries and scripts to PATH
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
# Set Java home directory
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

Save and exit nano: Press `Ctrl+X`, then `Y`, then `Enter`.

```bash
# Reload the bash configuration
source ~/.bashrc

# Verify Hadoop installation
hadoop version
```

Expected output:
```bash
Hadoop 3.3.6
Source code repository https://github.com/apache/hadoop.git -r ...
Compiled by ... on 2023-xx-xxTxx:xx:xxZ
```

**Why environment variables matter**: Without setting `HADOOP_HOME` and adding it to `PATH`, you'd need to use the full path (`/home/username/hadoop/bin/hadoop`) for every command.

### Step 4: Configure Hadoop XML Files

Configure Hadoop's XML files to define system operations.

#### 4.1: Configure Java Environment

```bash
# Edit the Hadoop environment script
nano ~/hadoop/etc/hadoop/hadoop-env.sh
```

Add or uncomment:
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

#### 4.2: Configure core-site.xml

This file defines the default file system URI.

```bash
nano ~/hadoop/etc/hadoop/core-site.xml
```

Replace the content with:
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
    <description>
      The name of the default file system. A URI whose scheme and authority
      determine the FileSystem implementation.
    </description>
  </property>
</configuration>
```

**What this does**: Hadoop connects to `localhost:9000` for HDFS commands.

#### 4.3: Configure hdfs-site.xml

This file controls HDFS settings like replication and storage directories.

```bash
nano ~/hadoop/etc/hadoop/hdfs-site.xml
```

Replace the content with:
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
    <description>
      Default block replication. Since we're running single node,
      we only need one copy of each block.
    </description>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/thiagosoares/hadoop_data/hdfs/namenode</value>
    <description>
      Path on the local filesystem where the NameNode stores the namespace
      and transaction logs persistently.
    </description>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/thiagosoares/hadoop_data/hdfs/datanode</value>
    <description>
      Comma-separated list of paths on the local filesystem where
      DataNodes store their blocks.
    </description>
  </property>
</configuration>
```

**Critical note**: Replace `thiagosoares` with your actual Ubuntu username (check with `whoami`).

#### 4.4: Configure mapred-site.xml

This file specifies the MapReduce framework.

```bash
nano ~/hadoop/etc/hadoop/mapred-site.xml
```

Replace the content with:
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    <description>
      The runtime framework for executing MapReduce jobs.
      Can be local, classic, or yarn.
    </description>
  </property>
</configuration>
```

**Why YARN?** YARN separates resource management from job scheduling for efficiency.

#### 4.5: Configure yarn-site.xml

This file configures YARN services.

```bash
nano ~/hadoop/etc/hadoop/yarn-site.xml
```

Replace the content with:
```xml
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    <description>
      Shuffle service that needs to be set for MapReduce applications.
    </description>
  </property>
</configuration>
```

**What is shuffle?** The shuffle phase transfers mapper output to reducers.

### Step 5: Create HDFS Directories

Create directories for HDFS metadata and data.

```bash
# Create directory for NameNode metadata
mkdir -p ~/hadoop_data/hdfs/namenode

# Create directory for DataNode blocks
mkdir -p ~/hadoop_data/hdfs/datanode
```

**Why separate directories?**  
- **NameNode directory**: Stores critical metadata.  
- **DataNode directory**: Stores actual data blocks.

### Step 6: Configure SSH for Hadoop

Hadoop requires passwordless SSH to localhost.

```bash
# Generate SSH key pair
ssh-keygen -t rsa -P ""

# Add public key to authorized keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Start SSH service
sudo service ssh start

# Test SSH connection
ssh localhost
```

Type `exit` to return.  
**Why this step?** Hadoop uses SSH to launch daemons, even in single-node mode.

### Step 7: Format the NameNode

Initialize the HDFS file system.

```bash
# Format the HDFS filesystem
hdfs namenode -format
```

Expected output:
```bash
Storage directory /home/thiagosoares/hadoop_data/hdfs/namenode has been successfully formatted.
```

**WARNING**: Only format during initial setup or to reset HDFS (erases all data).

### Step 8: Start Hadoop Services

Start all Hadoop daemons.

```bash
# Start HDFS daemons
start-dfs.sh

# Start YARN daemons
start-yarn.sh
```

Expected output:
```bash
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes
Starting resourcemanager
Starting nodemanagers
```

### Step 9: Validate Installation

Verify that all services are running.

```bash
# List all Java processes
jps
```

Expected output:
```bash
12345 NameNode
12346 DataNode
12347 SecondaryNameNode
12348 ResourceManager
12349 NodeManager
12350 Jps
```

**Access Web Interfaces**:  
- **HDFS NameNode UI**: http://localhost:9870  
- **YARN ResourceManager UI**: http://localhost:8088  

### Step 10: Test with Sample File

Test HDFS by uploading a sample file.

```bash
# Create a test file
echo "Hello Hadoop! This is my first file in HDFS." > ~/teste.txt

# Create a user directory in HDFS
hdfs dfs -mkdir -p /user/thiagosoares

# Upload the file to HDFS
hdfs dfs -put ~/teste.txt /user/thiagosoares/

# List files in HDFS
hdfs dfs -ls /user/thiagosoares/
```

Expected output:
```bash
Found 1 items
-rw-r--r--   1 thiagosoares supergroup         44 2025-01-15 10:30 /user/thiagosoares/teste.txt
```

**Additional HDFS commands**:
```bash
# View file content
hdfs dfs -cat /user/thiagosoares/teste.txt

# Copy file from HDFS to local
hdfs dfs -get /user/thiagosoares/teste.txt ~/teste_from_hdfs.txt

# Check file status
hdfs dfs -stat "%n %r %b" /user/thiagosoares/teste.txt

# Delete file from HDFS
hdfs dfs -rm /user/thiagosoares/teste.txt
```

## Troubleshooting

**Issue 1: "Cannot start NameNode"**  
**Symptoms**:  
```bash
Starting namenodes on [localhost]
localhost: namenode is not starting
```

**Solutions**:  
- Check if port 9000 is in use:
```bash
sudo lsof -i :9000
```
- Verify `JAVA_HOME`:
```bash
echo $JAVA_HOME
```
- Check NameNode logs:
```bash
cat ~/hadoop/logs/hadoop-*-namenode-*.log
```
- Verify NameNode formatting:
```bash
ls ~/hadoop_data/hdfs/namenode/current
```

**Issue 2: "SSH Connection Refused"**  
**Symptoms**:  
```bash
localhost: ssh: connect to host localhost port 22: Connection refused
```

**Solutions**:  
- Start SSH service:
```bash
sudo service ssh start
```
- Verify SSH status:
```bash
sudo service ssh status
```
- Regenerate SSH keys if needed:
```bash
rm ~/.ssh/id_rsa*
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

**Issue 3: "HDFS Safe Mode"**  
**Symptoms**:  
```bash
org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create directory /user/xxx. Name node is in safe mode.
```

**Solutions**:  
- Check safe mode status:
```bash
hdfs dfsadmin -safemode get
```
- Wait for automatic exit or manually leave:
```bash
hdfs dfsadmin -safemode leave
```

**Issue 4: "Permission Denied" Errors**  
**Symptoms**:  
```bash
mkdir: Permission denied: user=thiagosoares, access=WRITE, inode="/":root:supergroup:drwxr-xr-x
```

**Solutions**:  
- Create directory with proper permissions:
```bash
hdfs dfs -mkdir -p /user/$(whoami)
hdfs dfs -chown $(whoami):supergroup /user/$(whoami)
```

**Issue 5: DataNode Not Starting**  
**Symptoms**: `jps` shows NameNode but no DataNode.  

**Solutions**:  
- Remove DataNode data:
```bash
rm -rf ~/hadoop_data/hdfs/datanode/*
```
- Restart services:
```bash
stop-dfs.sh
start-dfs.sh
```
- Check DataNode logs:
```bash
cat ~/hadoop/logs/hadoop-*-datanode-*.log
```

**Issue 6: "Out of Memory" Errors**  
**Symptoms**:  
```bash
Java heap space
OutOfMemoryError
```

**Solutions**:  
- Increase Java heap size:
```bash
nano ~/hadoop/etc/hadoop/hadoop-env.sh
```
Add:
```bash
export HADOOP_HEAPSIZE=2048
export HADOOP_NAMENODE_OPTS="-Xmx2g"
export HADOOP_DATANODE_OPTS="-Xmx1g"
```
- Restart Hadoop:
```bash
stop-dfs.sh
stop-yarn.sh
start-dfs.sh
start-yarn.sh
```

**Useful Debugging Commands**:
| Command | Description |
|---------|-------------|
| `jps -ml` | Check all Hadoop processes (detailed) |
| `tail -f ~/hadoop/logs/hadoop-*-namenode-*.log` | View real-time logs |
| `hdfs dfsadmin -report` | Check HDFS health |
| `hdfs fsck / -files -blocks -locations` | Verify HDFS blocks |
| `yarn node -list -all` | Check YARN node status |
| `stop-dfs.sh && stop-yarn.sh` | Stop all Hadoop services |

## Practical Exercises

### Exercise 1: File Operations
```bash
# Create a text file
echo "Your Name - Your Favorite Quote" > ~/myinfo.txt

# Create a directory structure in HDFS
hdfs dfs -mkdir -p /user/$(whoami)/exercises/ex1

# Upload the file
hdfs dfs -put ~/myinfo.txt /user/$(whoami)/exercises/ex1/

# List the file
hdfs dfs -ls /user/$(whoami)/exercises/ex1/

# View file content
hdfs dfs -cat /user/$(whoami)/exercises/ex1/myinfo.txt

# Create a copy in HDFS
hdfs dfs -cp /user/$(whoami)/exercises/ex1/myinfo.txt /user/$(whoami)/exercises/ex1/myinfo_copy.txt

# Download the copy
hdfs dfs -get /user/$(whoami)/exercises/ex1/myinfo_copy.txt ~/myinfo_downloaded.txt

# Verify files are identical
diff ~/myinfo.txt ~/myinfo_downloaded.txt
```

### Exercise 2: Working with Large Files
```bash
# Generate a 200MB file
dd if=/dev/urandom of=~/large_file.bin bs=1M count=200

# Upload to HDFS
hdfs dfs -put ~/large_file.bin /user/$(whoami)/exercises/

# Check file status
hdfs dfs -stat "%n size: %b bytes, blocks: %r" /user/$(whoami)/exercises/large_file.bin

# Check block distribution
hdfs fsck /user/$(whoami)/exercises/large_file.bin -files -blocks -locations
```

### Exercise 3: Directory Operations
```bash
# Create a nested directory structure
hdfs dfs -mkdir -p /user/$(whoami)/projects/2025/january/week1

# Create multiple test files
for i in {1..5}; do
    echo "Test file $i" > ~/test_$i.txt
    hdfs dfs -put ~/test_$i.txt /user/$(whoami)/projects/2025/january/week1/
done

# List recursively
hdfs dfs -ls -R /user/$(whoami)/projects/

# Get directory size
hdfs dfs -du -h /user/$(whoami)/projects/

# Move files
hdfs dfs -mkdir /user/$(whoami)/archive/
hdfs dfs -mv /user/$(whoami)/projects/2025/january/week1/* /user/$(whoami)/archive/

# Remove empty directories
hdfs dfs -rm -r /user/$(whoami)/projects/
```

### Exercise 4: Permissions and Ownership
```bash
# Create a directory
hdfs dfs -mkdir /user/$(whoami)/secure_data

# Check permissions
hdfs dfs -ls /user/$(whoami)/ | grep secure_data

# Set read-only permissions
hdfs dfs -chmod 444 /user/$(whoami)/secure_data

# Try to upload (should fail)
hdfs dfs -put ~/teste.txt /user/$(whoami)/secure_data/

# Restore write permissions
hdfs dfs -chmod 755 /user/$(whoami)/secure_data

# Upload should now work
hdfs dfs -put ~/teste.txt /user/$(whoami)/secure_data/
```

### Exercise 5: Monitoring and Administration
```bash
# Generate HDFS health report
hdfs dfsadmin -report > ~/hdfs_report.txt
cat ~/hdfs_report.txt

# Check filesystem
hdfs fsck / -files -blocks

# Monitor real-time stats
hdfs dfs -df -h

# View NameNode storage
hdfs dfsadmin -printTopology

# Check owned files
hdfs dfs -find / -user $(whoami)
```

### Challenge Exercise: Word Count MapReduce
```bash
# Create a large text file
for i in {1..1000}; do
    echo "Hadoop is awesome. MapReduce is powerful. HDFS is distributed." >> ~/words.txt
done

# Upload to HDFS
hdfs dfs -put ~/words.txt /user/$(whoami)/

# Run WordCount example
hadoop jar ~/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /user/$(whoami)/words.txt /user/$(whoami)/wordcount_output

# View results
hdfs dfs -cat /user/$(whoami)/wordcount_output/part-r-00000

# Download results
hdfs dfs -get /user/$(whoami)/wordcount_output ~/wordcount_results
```

Expected output:
```bash
HDFS        1000
Hadoop      1000
MapReduce   1000
awesome     1000
distributed 1000
is          3000
powerful    1000
```

## Additional Resources

**Official Documentation**:
- [Apache Hadoop Documentation](https://hadoop.apache.org/docs/)
- [HDFS Architecture Guide](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
- [YARN Architecture](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html)
- [MapReduce Tutorial](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)

**HDFS Commands**:
| Command | Description |
|---------|-------------|
| `hdfs dfs -ls <path>` | List directory |
| `hdfs dfs -mkdir <path>` | Create directory |
| `hdfs dfs -put <local> <hdfs>` | Upload file |
| `hdfs dfs -get <hdfs> <local>` | Download file |
| `hdfs dfs -cat <file>` | View file content |
| `hdfs dfs -rm <file>` | Delete file |
| `hdfs dfs -rm -r <dir>` | Delete directory |
| `hdfs dfs -cp <src> <dest>` | Copy in HDFS |
| `hdfs dfs -mv <src> <dest>` | Move in HDFS |
| `hdfs dfs -du -h <path>` | Directory size |
| `hdfs dfs -stat <file>` | File statistics |
| `hdfs dfs -tail <file>` | View last 1KB |
| `hdfs dfs -count <path>` | Count files/dirs |
| `hdfs dfs -chmod <mode> <path>` | Change permissions |
| `hdfs dfs -chown <user>:<group> <path>` | Change ownership |
| `hdfs dfsadmin -report` | Cluster report |
| `hdfs fsck <path>` | Check filesystem |
| `hdfs dfsadmin -safemode get` | Check safe mode |

**Hadoop Service Commands**:
| Command | Description |
|---------|-------------|
| `start-dfs.sh` | Start HDFS |
| `start-yarn.sh` | Start YARN |
| `stop-dfs.sh` | Stop HDFS |
| `stop-yarn.sh` | Stop YARN |
| `hadoop-daemon.sh start namenode` | Start NameNode |
| `hadoop-daemon.sh stop namenode` | Stop NameNode |
| `yarn-daemon.sh start resourcemanager` | Start ResourceManager |

**Monitoring Commands**:
| Command | Description |
|---------|-------------|
| `jps` | List Java processes |
| `jps -ml` | Detailed process list |
| `yarn node -list` | List YARN nodes |
| `yarn application -list` | List applications |
| `hdfs dfsadmin -printTopology` | Cluster topology |

**Learning Path Recommendations**:  
- **Week 1-2: HDFS Deep Dive**  
  - Learn block replication strategies  
  - Understand NameNode high availability  
  - Practice disaster recovery scenarios  
- **Week 3-4: MapReduce Programming**  
  - Write custom MapReduce jobs in Java  
  - Learn about Combiner and Partitioner  
  - Optimize MapReduce performance  
- **Week 5-6: YARN Resource Management**  
  - Configure resource pools  
  - Understand scheduling policies  
  - Monitor application performance  
- **Week 7-8: Hadoop Ecosystem**  
  - Apache Hive for SQL on Hadoop  
  - Apache Pig for data flow scripting  
  - Apache HBase for NoSQL storage  

**Books and Courses**:  
- "Hadoop: The Definitive Guide" by Tom White (O'Reilly)  
- Coursera: "Big Data Specialization" by UC San Diego  
- Udemy: "Ultimate Hands-On Hadoop" by Frank Kane  
- edX: "Big Data Fundamentals" by IBM  

**Community and Support**:  
- [Stack Overflow - Hadoop Tag](https://stackoverflow.com/questions/tagged/hadoop)  
- [Apache Hadoop Mailing Lists](https://hadoop.apache.org/mailing_lists.html)  
- Hadoop Users Group  

## Stopping Hadoop Services

Shut down services gracefully to avoid data corruption:

```bash
# Stop YARN services
stop-yarn.sh

# Stop HDFS services
stop-dfs.sh

# Verify all processes stopped
jps
```

## Starting Fresh (Complete Reset)

To reset your Hadoop installation:

```bash
# Stop all services
stop-yarn.sh
stop-dfs.sh

# Remove HDFS data directories
rm -rf ~/hadoop_data/hdfs/namenode/*
rm -rf ~/hadoop_data/hdfs/datanode/*

# Remove logs (optional)
rm -rf ~/hadoop/logs/*

# Format NameNode
hdfs namenode -format

# Start services again
start-dfs.sh
start-yarn.sh
```

**WARNING**: This deletes ALL data in HDFS!

## Known Issues and Limitations

**WSL-Specific Issues**:  
- **Memory Limitations**: WSL 2 defaults to 50% of total RAM (max 8GB on Windows 10). Configure in `.wslconfig`.  
- **Network Access**: WSL 2 uses a virtual network adapter. If web UIs don't load, try localhost or WSL's IP (`ip addr`).  
- **File System Performance**: Avoid storing HDFS data on `/mnt/c/`. Use Linux filesystem (`~/`).  

**Single Node Limitations**:  
- No fault tolerance  
- No data redundancy (replication=1)  
- Limited performance  
- No load balancing  
- Perfect for learning and development  

**Next Steps**: Check out the upcoming "Hadoop Multi-Node Cluster Setup" tutorial or simulate multiple nodes using Docker.

## Contributing and Feedback

I welcome:  
- Bug reports: Open an issue on GitHub  
- Suggestions: Ideas to improve the tutorial  
- Questions: Ask in the Discussions section  

Note: This is an educational resource, not a production guide.

## License

Copyright © 2025 Thiago C. Soares  

This work is licensed under a [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License](https://creativecommons.org/licenses/by-nc-nd/4.0/).  

**You are free to**:  
- Share: Copy and redistribute the material  

**Under these terms**:  
- Attribution: Give appropriate credit and link to the license  
- NonCommercial: Do not use for commercial purposes  
- NoDerivatives: Do not distribute modified versions  

**Citation format**:  
Soares, T.C. (2025). Hadoop Single Node Setup on WSL (Ubuntu).  
Retrieved from https://github.com/thiagosoares/hadoop-single-node-tutorial

## Contact

Thiago C. Soares  
Email: thiagocsoares@gmail.com  
LinkedIn: [linkedin.com/in/thiago-csoares](https://linkedin.com/in/thiago-csoares)  
GitHub: [github.com/thiagosoares](https://github.com/thiagosoares)

## Acknowledgments

This tutorial is based on:  
- Official Apache Hadoop documentation  
- Years of teaching Big Data courses  
- Student feedback and questions  
- My learning journey with distributed systems  

Special thanks to:  
- My students who tested these instructions  
- The Apache Hadoop community  
- All open-source contributors  

## Changelog

**Version 1.0.0 (January 2025)**  
- Initial release  
- Complete installation guide for Hadoop 3.3.6  
- WSL/Ubuntu 20.04 specific instructions  
- 5 practical exercises  
- Comprehensive troubleshooting section  

**If This Helped You**:  
- Star this repository on GitHub  
- Share with fellow students/developers  
- Leave feedback in the Discussions  
- Email me your success story!  

Made with love for Big Data learners  
Happy Hadooping!  

Last updated: October 2025
