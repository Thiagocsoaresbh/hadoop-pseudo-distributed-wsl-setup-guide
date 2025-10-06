
# Hadoop Single Node Setup on WSL (Ubuntu)

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

---

## About This Project

This repository contains a complete, step-by-step tutorial for setting up **Apache Hadoop 3.3.6** in **Single Node mode** on **WSL (Windows Subsystem for Linux)** with Ubuntu 20.04.

I created this guide as an educational resource for my students and anyone interested in learning Hadoop from scratch. Throughout this tutorial, I document every command I used, explain what each component does, and provide troubleshooting tips based on real-world scenarios.

What you'll learn:

* How to install and configure Hadoop in pseudo-distributed mode
* Understanding **HDFS (Hadoop Distributed File System)** architecture
* Working with **YARN** resource management
* Basic HDFS operations and file management
* How to validate your Hadoop installation

This is a **hands-on tutorial** designed for beginners with basic Linux knowledge.

---

## Author

**Thiago C. Soares**
Email: thiagocsoares@gmail.com
LinkedIn: [linkedin.com/in/thiago-csoares](https://linkedin.com/in/thiago-csoares)

Created for educational purposes to help students and developers understand Hadoop fundamentals.

---

## Prerequisites

Before starting this tutorial, make sure you have:

* **Windows 10/11** with WSL 2 enabled
* **Ubuntu 20.04 LTS** installed on WSL (or similar Debian-based distribution)
* At least **4GB RAM** available for WSL
* **10GB free disk space** minimum
* Basic Linux command-line knowledge
* Internet connection for downloading packages

Note: If you don't have WSL installed, follow the **official Microsoft WSL installation guide**.

---

## Architecture Overview

In this Single Node setup, all Hadoop daemons run on the same machine. Here's what we'll be running:

┌─────────────────────────────────────────────────────┐
│                   Single Node Hadoop                │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────┐       │
│  │         HDFS Layer (Port 9000)           │       │
│  ├──────────────────────────────────────────┤       │
│  │  • NameNode (Metadata Management)        │       │
│  │  • DataNode (Data Storage)               │       │
│  │  • SecondaryNameNode (Checkpoint)        │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
│  ┌──────────────────────────────────────────┐       │
│  │      YARN Layer (Resource Manager)       │       │
│  ├──────────────────────────────────────────┤       │
│  │  • ResourceManager (Job Scheduling)      │       │
│  │  • NodeManager (Task Execution)          │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
└─────────────────────────────────────────────────────┘


**Key Components:**

* **NameNode**: Manages the file system namespace and metadata
* **DataNode**: Stores actual data blocks
* **SecondaryNameNode**: Creates checkpoints of the namespace
* **ResourceManager**: Manages cluster resources and scheduling
* **NodeManager**: Manages containers and monitors resource usage

---

## Installation Guide

### Step 1: Update System and Install Dependencies

First, I updated my system packages and installed all necessary dependencies that Hadoop requires to run properly.

```bash
# Update package lists and upgrade existing packages
# This ensures we have the latest security patches and software versions
sudo apt update && sudo apt upgrade -y

# Install required packages:
# - ssh: Hadoop daemons communicate via SSH protocol
# - pdsh: Parallel Distributed Shell for managing multiple hosts
# - openjdk-11-jdk: Java Development Kit (Hadoop is written in Java)
# - wget: Tool to download files from the web
# - tar: Utility to extract compressed archives
sudo apt install ssh pdsh openjdk-11-jdk wget tar -y

# Verify Java installation
# This should display Java version 11.x.x
java -version
Expected output:

openjdk version "11.0.x" 2024-xx-xx
OpenJDK Runtime Environment (build 11.0.x+x-Ubuntu-...)
OpenJDK 64-Bit Server VM (build 11.0.x+x-Ubuntu-...)
Why these packages?

SSH: Hadoop uses SSH to start and stop daemons on different nodes (even in single-node mode)

PDSH: Allows executing commands across multiple machines simultaneously

Java 11: Hadoop 3.3.6 requires Java 8 or 11 to run

wget/tar: Essential tools for downloading and extracting Hadoop binaries

Step 2: Download and Configure Hadoop
Now I downloaded the official Hadoop distribution and set up the directory structure.

Bash

# Download Hadoop 3.3.6 from Apache mirrors
# The -P flag specifies the download directory (home folder)
wget [https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz](https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz) -P ~

# Extract the downloaded archive
# -x: extract files
# -v: verbose mode (show progress)
# -z: handle gzip compression (.gz files)
# -f: specify the archive filename
# -C: change to directory before extracting
tar -xvzf ~/hadoop-3.3.6.tar.gz -C ~

# Rename the directory to a simpler name for easier access
# This makes our path cleaner: ~/hadoop instead of ~/hadoop-3.3.6
mv ~/hadoop-3.3.6 ~/hadoop
What just happened? I downloaded approximately 600MB of Hadoop binaries, extracted them, and created a clean directory structure. The ~/hadoop folder now contains all necessary files to run Hadoop.

Directory structure overview:

~/hadoop/
├── bin/          # Hadoop command-line scripts
├── etc/hadoop/   # Configuration files (we'll modify these)
├── sbin/         # Start/stop scripts for daemons
├── share/        # Hadoop JARs and libraries
└── lib/          # Native libraries
Step 3: Set Up Environment Variables
I configured the system environment variables so Hadoop commands are accessible from anywhere in the terminal.

Bash

# Open the bash configuration file with nano editor
nano ~/.bashrc
Add the following lines at the end of the file:

Bash

# Set Hadoop home directory
export HADOOP_HOME=~/hadoop
# Configure Hadoop environment variables
# These variables tell Hadoop where to find its components
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
# Add Hadoop binaries and scripts to system PATH
# This allows us to run hadoop commands from any directory
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
# Set Java home directory
# Hadoop needs to know where Java is installed
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
Save and exit nano: Press Ctrl+X, then Y, then Enter

Bash

# Reload the bash configuration to apply changes immediately
# Without this, we'd need to close and reopen the terminal
source ~/.bashrc

# Verify Hadoop installation
hadoop version
Expected output:

Hadoop 3.3.6
Source code repository [https://github.com/apache/hadoop.git](https://github.com/apache/hadoop.git) -r ...
Compiled by ... on 2023-xx-xxTxx:xx:xxZ
Why environment variables matter: Environment variables are like shortcuts that tell your system where to find programs. Without setting HADOOP_HOME and adding it to PATH, I would need to type the full path /home/username/hadoop/bin/hadoop every time I want to run a Hadoop command.

Step 4: Configure Hadoop XML Files
This is the most critical step. I configured the core Hadoop XML files to define how the system operates.

4.1: Configure Java Environment
Bash

# Edit the Hadoop environment script
nano ~/hadoop/etc/hadoop/hadoop-env.sh
Find and uncomment (or add) this line:

Bash

# Tell Hadoop where Java is installed
# This prevents "JAVA_HOME not set" errors
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
4.2: Configure core-site.xml
This file defines the default file system URI and other core configurations.

Bash

nano ~/hadoop/etc/hadoop/core-site.xml
Replace the content with:

XML

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
What this does: When I run HDFS commands, Hadoop knows to connect to localhost:9000 instead of the local file system.

4.3: Configure hdfs-site.xml
This file controls HDFS-specific settings like replication and storage directories.

Bash

nano ~/hadoop/etc/hadoop/hdfs-site.xml
Replace the content with:

XML

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
Critical note: Replace thiagosoares with your actual Ubuntu username! You can check your username with:

Bash

whoami
4.4: Configure mapred-site.xml
This file specifies the MapReduce framework to use.

Bash

nano ~/hadoop/etc/hadoop/mapred-site.xml
Replace the content with:

XML

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
Why YARN? YARN separates resource management from job scheduling, making Hadoop more efficient and allowing it to run non-MapReduce workloads.

4.5: Configure yarn-site.xml
This file configures YARN services.

Bash

nano ~/hadoop/etc/hadoop/yarn-site.xml
Replace the content with:

XML

<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    <description>
      Shuffle service that needs to be set for MapReduce applications.
    </description>
  </property>
</configuration>
What is shuffle? In MapReduce, the "shuffle" phase is where output from mappers is transferred to reducers. YARN's NodeManager needs to provide this service.

Step 5: Create HDFS Directories
I created the directories where HDFS will store metadata and actual data.

Bash

# Create directory for NameNode metadata
# The NameNode stores file system structure, permissions, and block locations here
mkdir -p ~/hadoop_data/hdfs/namenode

# Create directory for DataNode blocks
# The actual file content (in 128MB blocks) is stored here
mkdir -p ~/hadoop_data/hdfs/datanode
Why separate directories?

NameNode directory: Contains critical metadata. If corrupted, you lose track of all your data.

DataNode directory: Contains actual data blocks. In multi-node clusters, you'd have multiple DataNode directories for redundancy.

Step 6: Configure SSH for Hadoop
Hadoop requires passwordless SSH to localhost to start and manage its daemons.

Bash

# Generate SSH key pair
# -t rsa: Use RSA algorithm
# -P "": Empty passphrase (passwordless)
# Press Enter for all prompts to use default settings
ssh-keygen -t rsa -P ""

# Add your public key to authorized keys
# This allows SSH to localhost without password
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Set proper permissions (SSH requires these specific permissions)
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Start SSH service (required in WSL)
sudo service ssh start

# Test SSH connection to localhost
# Type 'yes' if prompted about host authenticity
ssh localhost
You should now be connected via SSH to your own machine. Type exit to return.

Why this step? Even though we're running everything on one machine, Hadoop's start scripts use SSH to launch daemons. Without passwordless SSH, Hadoop will fail to start.

Step 7: Format the NameNode
Before first use, I formatted the NameNode to initialize the HDFS file system.

Bash

# Format the HDFS filesystem
# This creates the initial directory structure in the NameNode directory
# WARNING: Only do this once! Formatting again will erase all HDFS data
hdfs namenode -format
Expected output (look for these lines):

Storage directory /home/thiagosoares/hadoop_data/hdfs/namenode has been successfully formatted.
⚠️ IMPORTANT WARNING: Formatting the NameNode is like formatting a hard drive - it erases everything! Only run this command:

During initial setup

When you want to completely reset HDFS and lose all data

Step 8: Start Hadoop Services
Now comes the moment of truth - starting all Hadoop daemons!

Bash

# Start HDFS daemons (NameNode, DataNode, SecondaryNameNode)
# This script uses SSH to start each daemon
start-dfs.sh

# Start YARN daemons (ResourceManager, NodeManager)
start-yarn.sh
Expected output:

Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes
Starting resourcemanager
Starting nodemanagers
What's happening? These scripts are launching Java processes in the background that will handle all Hadoop operations. Each daemon has a specific role in the Hadoop ecosystem.

Step 9: Validate Installation
I verified that all Hadoop services are running correctly.

Bash

# List all Java processes
# This shows all running Hadoop daemons
jps
Expected output (process IDs will differ):

12345 NameNode
12346 DataNode
12347 SecondaryNameNode
12348 ResourceManager
12349 NodeManager
12350 Jps
If you see all 5 processes (excluding Jps), congratulations! Hadoop is running!

Access Web Interfaces
Hadoop provides web UIs to monitor your cluster:

HDFS NameNode UI:
http://localhost:9870
Here I can see: HDFS health and capacity, Live/dead DataNodes, File system browser, NameNode logs

YARN ResourceManager UI:
http://localhost:8088
Here I can see: Running applications, Cluster metrics (memory, CPU), Application history, NodeManager status

Screenshots placeholder:
Add screenshots of both web interfaces to the docs/images/ folder

Step 10: Test with Sample File
Finally, I tested HDFS by uploading a sample file and verifying it's stored correctly.

Bash

# Create a simple test file in my home directory
echo "Hello Hadoop! This is my first file in HDFS." > ~/teste.txt

# Create a user directory in HDFS
# -p flag creates parent directories if they don't exist
# IMPORTANT: Replace 'thiagosoares' with your username
hdfs dfs -mkdir -p /user/thiagosoares

# Upload the file from local filesystem to HDFS
# -put copies files from local to HDFS
hdfs dfs -put ~/teste.txt /user/thiagosoares/

# List files in HDFS to confirm upload
hdfs dfs -ls /user/thiagosoares/
Expected output:

Found 1 items
-rw-r--r--   1 thiagosoares supergroup         44 2025-01-15 10:30 /user/thiagosoares/teste.txt
Additional HDFS commands to try:

Bash

# View file content directly from HDFS
hdfs dfs -cat /user/thiagosoares/teste.txt

# Copy file from HDFS back to local filesystem
hdfs dfs -get /user/thiagosoares/teste.txt ~/teste_from_hdfs.txt

# Check file status and replication
hdfs dfs -stat "%n %r %b" /user/thiagosoares/teste.txt

# Delete file from HDFS
hdfs dfs -rm /user/thiagosoares/teste.txt
Understanding HDFS paths:

Local filesystem: /home/thiagosoares/teste.txt

HDFS filesystem: /user/thiagosoares/teste.txt
These are two completely separate file systems!

Troubleshooting
Here are common issues I encountered and how I solved them:

Issue 1: "Cannot start NameNode"
Symptoms:

Starting namenodes on [localhost]

localhost: namenode is not starting

Solutions:

Check if port 9000 is already in use:

Bash

sudo lsof -i :9000
# If something is using port 9000, kill it or change Hadoop's port
Verify JAVA_HOME is set correctly:

Bash

echo $JAVA_HOME
# Should output: /usr/lib/jvm/java-11-openjdk-amd64
Check NameNode logs:

Bash

cat ~/hadoop/logs/hadoop-*-namenode-*.log
Verify NameNode was formatted:

Bash

ls ~/hadoop_data/hdfs/namenode/current
# Should contain files like VERSION, fsimage_*, edits_*
Issue 2: "SSH Connection Refused"
Symptoms:

localhost: ssh: connect to host localhost port 22: Connection refused

Solutions:

Start SSH service:

Bash

sudo service ssh start
Verify SSH is running:

Bash

sudo service ssh status
Re-test SSH connection:

Bash

ssh localhost
If still failing, regenerate SSH keys:

Bash

rm ~/.ssh/id_rsa*
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
Issue 3: "HDFS Safe Mode" - Can't Write Files
Symptoms:

org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create directory /user/xxx. Name node is in safe mode.

What is Safe Mode? HDFS enters safe mode during startup while checking block replication. In safe mode, the filesystem is read-only.

Solutions:

Check safe mode status:

Bash

hdfs dfsadmin -safemode get
Wait for automatic exit (usually 30 seconds):

Bash

# Watch the status
watch -n 1 hdfs dfsadmin -safemode get
Manually leave safe mode (not recommended in production):

Bash

hdfs dfsadmin -safemode leave
Issue 4: "Permission Denied" Errors
Symptoms:

mkdir: Permission denied: user=thiagosoares, access=WRITE, inode="/":root:supergroup:drwxr-xr-x

Solutions:

Create directory with proper permissions:

Bash

hdfs dfs -mkdir -p /user/$(whoami)
hdfs dfs -chown $(whoami):supergroup /user/$(whoami)
Always work in your user directory:

Bash

# Good practice: use /user/yourname/
hdfs dfs -put file.txt /user/$(whoami)/
Issue 5: DataNode Not Starting
Symptoms:

jps shows NameNode but no DataNode

Web UI shows 0 live nodes

Solutions:

Check if DataNode directory has incorrect cluster ID:

Bash

# Remove DataNode data (will not affect uploaded files, only local cache)
rm -rf ~/hadoop_data/hdfs/datanode/*

# Restart DataNode
stop-dfs.sh
start-dfs.sh
Verify DataNode logs:

Bash

cat ~/hadoop/logs/hadoop-*-datanode-*.log
Issue 6: "Out of Memory" Errors
Symptoms:

Java heap space

OutOfMemoryError

Solutions:

Increase Java heap size for HDFS:

Bash

nano ~/hadoop/etc/hadoop/hadoop-env.sh
Add:

Bash

export HADOOP_HEAPSIZE=2048
export HADOOP_NAMENODE_OPTS="-Xmx2g"
export HADOOP_DATANODE_OPTS="-Xmx1g"
Restart Hadoop:

Bash

stop-dfs.sh
stop-yarn.sh
start-dfs.sh
start-yarn.sh
Useful Commands for Debugging
Command	Description
jps -ml	Check all Hadoop processes (detailed list)
tail -f ~/hadoop/logs/hadoop-*-namenode-*.log	View real-time logs
hdfs dfsadmin -report	Check HDFS health
hdfs fsck / -files -blocks -locations	Verify HDFS blocks
yarn node -list -all	Check YARN node status
stop-dfs.sh && stop-yarn.sh	Stop all Hadoop services

Exportar para as Planilhas
Practical Exercises
Now that your Hadoop cluster is running, try these hands-on exercises:

Exercise 1: File Operations
Practice basic HDFS commands:

Bash

# 1. Create a text file with your name and favorite quote
echo "Your Name - Your Favorite Quote" > ~/myinfo.txt

# 2. Create a directory structure in HDFS
hdfs dfs -mkdir -p /user/$(whoami)/exercises/ex1

# 3. Upload the file
hdfs dfs -put ~/myinfo.txt /user/$(whoami)/exercises/ex1/

# 4. List the file and note the file size
hdfs dfs -ls /user/$(whoami)/exercises/ex1/

# 5. View the file content
hdfs dfs -cat /user/$(whoami)/exercises/ex1/myinfo.txt

# 6. Create a copy of the file in HDFS
hdfs dfs -cp /user/$(whoami)/exercises/ex1/myinfo.txt /user/$(whoami)/exercises/ex1/myinfo_copy.txt

# 7. Download the copy back to local filesystem
hdfs dfs -get /user/$(whoami)/exercises/ex1/myinfo_copy.txt ~/myinfo_downloaded.txt

# 8. Verify files are identical
diff ~/myinfo.txt ~/myinfo_downloaded.txt
Expected result: The diff command should return nothing (files are identical)

Exercise 2: Working with Large Files
Learn how HDFS handles large files by creating and uploading a bigger dataset:

Bash

# 1. Generate a 200MB file with random data
dd if=/dev/urandom of=~/large_file.bin bs=1M count=200

# 2. Upload to HDFS
hdfs dfs -put ~/large_file.bin /user/$(whoami)/exercises/

# 3. Check file status (note the block count)
hdfs dfs -stat "%n size: %b bytes, blocks: %r" /user/$(whoami)/exercises/large_file.bin

# 4. Use fsck to see block distribution
hdfs fsck /user/$(whoami)/exercises/large_file.bin -files -blocks -locations
Questions to answer:

How many blocks is your file split into? (Hint: 128MB default block size)

Where are the blocks located?

What's the replication factor?

Exercise 3: Directory Operations
Master directory management:

Bash

# 1. Create a nested directory structure
hdfs dfs -mkdir -p /user/$(whoami)/projects/2025/january/week1

# 2. Create multiple test files
for i in {1..5}; do
    echo "Test file $i" > ~/test_$i.txt
    hdfs dfs -put ~/test_$i.txt /user/$(whoami)/projects/2025/january/week1/
done

# 3. List recursively
hdfs dfs -ls -R /user/$(whoami)/projects/

# 4. Get directory size
hdfs dfs -du -h /user/$(whoami)/projects/

# 5. Move all files to a new location
hdfs dfs -mkdir /user/$(whoami)/archive/
hdfs dfs -mv /user/$(whoami)/projects/2025/january/week1/* /user/$(whoami)/archive/

# 6. Remove empty directories
hdfs dfs -rm -r /user/$(whoami)/projects/
Exercise 4: Permissions and Ownership
Practice HDFS security:

Bash

# 1. Create a directory
hdfs dfs -mkdir /user/$(whoami)/secure_data

# 2. Check default permissions
hdfs dfs -ls /user/$(whoami)/ | grep secure_data

# 3. Change permissions to read-only
hdfs dfs -chmod 444 /user/$(whoami)/secure_data

# 4. Try to create a file (should fail)
hdfs dfs -put ~/teste.txt /user/$(whoami)/secure_data/

# 5. Restore write permissions
hdfs dfs -chmod 755 /user/$(whoami)/secure_data

# 6. Now upload should work
hdfs dfs -put ~/teste.txt /user/$(whoami)/secure_data/
Exercise 5: Monitoring and Administration
Learn administrative commands:

Bash

# 1. Generate HDFS health report
hdfs dfsadmin -report > ~/hdfs_report.txt
cat ~/hdfs_report.txt

# 2. Check filesystem
hdfs fsck / -files -blocks

# 3. Monitor real-time stats
hdfs dfs -df -h

# 4. View NameNode storage
hdfs dfsadmin -printTopology

# 5. Check which files you own
hdfs dfs -find / -user $(whoami)
Challenge Exercise: Word Count MapReduce
Run a classic MapReduce job:

Bash

# 1. Create a large text file
for i in {1..1000}; do
    echo "Hadoop is awesome. MapReduce is powerful. HDFS is distributed." >> ~/words.txt
done

# 2. Upload to HDFS
hdfs dfs -put ~/words.txt /user/$(whoami)/

# 3. Run the built-in WordCount example
hadoop jar ~/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /user/$(whoami)/words.txt /user/$(whoami)/wordcount_output

# 4. View the results
hdfs dfs -cat /user/$(whoami)/wordcount_output/part-r-00000

# 5. Download results to local
hdfs dfs -get /user/$(whoami)/wordcount_output ~/wordcount_results
Expected output:

HDFS    1000
Hadoop  1000
MapReduce 1000
awesome 1000
distributed 1000
is      3000
powerful 1000
What just happened? You ran your first MapReduce job! The WordCount program:

Read the input file from HDFS

Split it into words (Map phase)

Counted occurrences (Reduce phase)

Wrote results back to HDFS

Additional Resources
Official Documentation
Apache Hadoop Documentation

HDFS Architecture Guide

YARN Architecture

MapReduce Tutorial

Useful Commands Reference
HDFS Commands	Description
File operations	
hdfs dfs -ls <path>	List directory
hdfs dfs -mkdir <path>	Create directory
hdfs dfs -put <local> <hdfs>	Upload file
hdfs dfs -get <hdfs> <local>	Download file
hdfs dfs -cat <file>	View file content
hdfs dfs -rm <file>	Delete file
hdfs dfs -rm -r <dir>	Delete directory
hdfs dfs -cp <src> <dest>	Copy in HDFS
hdfs dfs -mv <src> <dest>	Move in HDFS
File information	
hdfs dfs -du -h <path>	Directory size
hdfs dfs -stat <file>	File statistics
hdfs dfs -tail <file>	View last 1KB
hdfs dfs -count <path>	Count files/dirs
Permissions	
hdfs dfs -chmod <mode> <path>	Change permissions
hdfs dfs -chown <user>:<group> <path>	Change ownership
Administration	
hdfs dfsadmin -report	Cluster report
hdfs fsck <path>	Check filesystem
hdfs dfsadmin -safemode get	Check safe mode

Exportar para as Planilhas
Hadoop Service Commands	Description
Start services	
start-dfs.sh	Start HDFS
start-yarn.sh	Start YARN
Stop services	
stop-dfs.sh	Stop HDFS
stop-yarn.sh	Stop YARN
Individual daemon control	
hadoop-daemon.sh start namenode	Start NameNode
hadoop-daemon.sh stop namenode	Stop NameNode
yarn-daemon.sh start resourcemanager	Start ResourceManager

Exportar para as Planilhas
Monitoring Commands	Description
jps	List Java processes
jps -ml	Detailed process list
yarn node -list	List YARN nodes
yarn application -list	List applications
hdfs dfsadmin -printTopology	Cluster topology

Exportar para as Planilhas
Learning Path Recommendations
After mastering this tutorial, I recommend:

Week 1-2: HDFS Deep Dive

Learn about block replication strategies

Understand NameNode high availability

Practice disaster recovery scenarios

Week 3-4: MapReduce Programming

Write custom MapReduce jobs in Java

Learn about Combiner and Partitioner

Optimize MapReduce performance

Week 5-6: YARN Resource Management

Configure resource pools

Understand scheduling policies

Monitor application performance

Week 7-8: Hadoop Ecosystem

Apache Hive for SQL on Hadoop

Apache Pig for data flow scripting

Apache HBase for NoSQL storage

Books and Courses
"Hadoop: The Definitive Guide" by Tom White (O'Reilly)

Coursera: "Big Data Specialization" by UC San Diego

Udemy: "Ultimate Hands-On Hadoop" by Frank Kane

edX: "Big Data Fundamentals" by IBM

Community and Support
Stack Overflow - Hadoop Tag

Apache Hadoop Mailing Lists

Hadoop Users Group

Stopping Hadoop Services
When you're done working with Hadoop, properly shut down all services:

Bash

# Stop YARN services
stop-yarn.sh

# Stop HDFS services
stop-dfs.sh

# Verify all processes stopped
jps
# Should only show "Jps" process
Important: Always stop services gracefully to avoid data corruption!

Starting Fresh (Complete Reset)
If you need to completely reset your Hadoop installation:

Bash

# 1. Stop all services
stop-yarn.sh
stop-dfs.sh

# 2. Remove HDFS data directories
rm -rf ~/hadoop_data/hdfs/namenode/*
rm -rf ~/hadoop_data/hdfs/datanode/*

# 3. Remove logs (optional)
rm -rf ~/hadoop/logs/*

# 4. Format NameNode
hdfs namenode -format

# 5. Start services again
start-dfs.sh
start-yarn.sh
⚠️ WARNING: This will delete ALL data stored in HDFS!

Known Issues and Limitations
WSL-Specific Issues
Memory Limitations

WSL 2 has memory limits set by Windows

Default is 50% of total RAM (max 8GB on Windows 10)

Can be configured in .wslconfig file

Network Access

WSL 2 uses a virtual network adapter

Port forwarding is usually automatic but may fail

If web UIs don't load, try localhost or WSL's IP (ip addr)

File System Performance

Accessing Windows files from WSL is slower

Keep Hadoop data in Linux filesystem (~/)

Don't store HDFS data on /mnt/c/

Single Node Limitations
This setup is NOT production-ready:

❌ No fault tolerance (single point of failure)

❌ No data redundancy (replication=1)

❌ Limited performance (single machine)

❌ No load balancing

✅ Perfect for learning and development

Next Steps: Multi-Node Cluster
Want to expand to multiple nodes? Check out my next tutorial:

Coming soon: "Hadoop Multi-Node Cluster Setup"
For now, you can simulate multiple nodes using Docker containers!

Contributing and Feedback
While this is primarily an educational resource, I welcome:

Bug reports: If commands don't work, open an issue

Suggestions: Ideas for improving the tutorial

Questions: Ask in the Discussions section

However, please note:

This is a learning resource, not a production guide

I update the tutorial based on student feedback

Focus is on clarity over enterprise features

License
Copyright © 2025 Thiago C. Soares

This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License.

You are free to:

Share — copy and redistribute the material in any medium or format

Under the following terms:

Attribution — You must give appropriate credit, provide a link to the license, and indicate if changes were made (but since NoDerivatives, you cannot make changes)

NonCommercial — You may not use the material for commercial purposes

NoDerivatives — If you remix, transform, or build upon the material, you may not distribute the modified material

This means:

✅ Use this tutorial for personal learning

✅ Share the link to this repository

✅ Reference this work in academic papers (with citation)

✅ Use in classroom settings (with attribution)

❌ Repost this content on your own blog/site without permission

❌ Sell this tutorial or include it in paid courses

❌ Modify and redistribute altered versions

❌ Remove author attribution

Citation format:
Soares, T.C. (2025). Hadoop Single Node Setup on WSL (Ubuntu).
Retrieved from https://github.com/thiagosoares/hadoop-single-node-tutorial

Contact
Thiago C. Soares
Email: thiagocsoares@gmail.com
LinkedIn: linkedin.com/in/thiago-csoares
GitHub: github.com/thiagosoares

Acknowledgments
I created this tutorial based on:

Official Apache Hadoop documentation

Years of teaching Big Data courses

Student feedback and common questions

My own learning journey with distributed systems

Special thanks to:

My students who tested these instructions

The Apache Hadoop community

All open-source contributors

Changelog
Version 1.0.0 (January 2025)

Initial release

Complete installation guide for Hadoop 3.3.6

WSL/Ubuntu 20.04 specific instructions

5 practical exercises

Comprehensive troubleshooting section

If This Helped You
If this tutorial was helpful:

Star this repository on GitHub

Share with fellow students/developers

Leave feedback in the Discussions

Email me your success story!

Made with love for Big Data learners
Happy Hadooping!

Last updated: January 2025