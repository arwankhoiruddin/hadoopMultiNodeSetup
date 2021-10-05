# Do these steps on all nodes

## Install JDK and Hadoop

```
sudo apt-get update
sudo apt-get install openjdk-8-jdk openjdk-8-jre
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.1.1/hadoop-3.1.1.tar.gz
tar -xvfz hadoop-3.1.1.tar.gz
mv hadoop-3.1.1 /usr/local/hadoop
```

## Update `etc/environment`

```
update-alternatives --display java
java - auto mode
  link best version is /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
  link currently points to /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
  link java is /usr/bin/java
  slave java.1.gz is /usr/share/man/man1/java.1.gz
/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java - priority 1081
  slave java.1.gz: /usr/lib/jvm/java-8-openjdk-amd64/jre/man/man1/java.1.gz
```
As seen here, the `JAVA_HOME` should be set to `/usr/lib/jvm/java-8-openjdk-amd64/jre`

```
nano /etc/environment
```

## Add path to Hadoop and add `JAVA_HOME`

```
PATH="/usr/local/hadoop/bin:/usr/local/hadoop/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64/jre"
```

## Add hadoop user

```
adduser hadoop
usermod -aG hadoop hadoop
chown hadoop:root -R /usr/local/hadoop
chmod g+rwx -R /usr/local/hadoop
```

## Enable remote login in all slaves

```
nano /etc/ssh/sshd_config 
PermitRootLogin prohibit-password to PermitRootLogin yes 
PasswordAuthentication no to PasswordAuthentication yes
```
Restart ssh

```
sudo service ssh restart
```


# Do these for Master only

## Login as hadoop user

```
su hadoop
```

## Create ssh key

```
ssh-keygen -t rsa
```

## Copy SSH to all slaves

```
ssh-copy-id hadoop@hadoop1
ssh-copy-id hadoop@hadoop2
```

# Configuring Hadoop

Here, `conf` means the following path: `/usr/local/hadoop/etc`

## conf/masters (`master` only)

```
nano masters
```
Fill with the name of the master, in my case my server is `hadoop1`

## conf/slaves (`master` only)

```
nano slaves
```
Fill with master and all slaves. Because I only have one slave node, I will have two entries only

```
hadoop1
hadoop2
```

## conf/*-site.xml (all machines)

### core-site.xml

```
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://hadoop1:9000</value>
  </property>
</configuration>
```

### hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/usr/local/hadoop/data/nameNode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/usr/local/hadoop/data/dataNode</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
</configuration>
```
