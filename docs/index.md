# Introduction

[Duplicacy](https://duplicacy.com) is state-of-the-art backup tool that has extensive cloud support. It also supports local disks and your own **sftp** servers.     

Duplicacy is available as a web-based GUI or as a command line tool. 
The software does require a license but the **command-line interface** (CLI) version is free for personal use.  
  
We’ll be using the **CLI** version for this tutorial.

# Installation    

**STEP 1:**   

Pre-compiled binaries are available for Linux, macOS, and Windows directly from the Duplicacy [GitHub](https://github.com/gilbertchen/duplicacy/releases) repository.

Download the latest version for your system.
~~~
$ wget https://github.com/gilbertchen/duplicacy/releases/download/v2.7.2/duplicacy_linux_x64_2.7.2
~~~

**STEP 2:**    

Make the file executable.
~~~
$ chmod +x duplicacy_linux_x64_2.7.2
~~~
   
**STEP 3:**  

Rename the file to **duplicacy**.
~~~
$ mv duplicacy_linux_x64_2.7.2 duplicacy
~~~

**STEP 4:**  

Change the file permissions.
~~~
$ chmod 755 duplicacy
~~~

**STEP 5:**  

Move the file to the **/usr/bin** directory.
~~~
$ sudo mv duplicacy /usr/bin/
~~~

# Initialize the remote storage and repository

The **duplicacy init** command is used to initialize the remote storage and the backup directory.
~~~
duplicacy init [command options] <snapshot id> <storage url>
~~~
The `<snapshot id>` refers to the name you want to give to your backup job.    
The `<storage url>` refers to the remote server and directory path for your backups.

We will only be using the following options:
~~~
-encrypt, -e          encrypt the storage with a password
-storage-name <name>  assign a name to the storage
~~~

An example scenario will make things easier to understand.

1. The sftp server hostname: **nacho.local**
2. Directory to be backed up: **/home/curt/Photos** 
3. Backup directory on the **sftp** server: **photobackup**
4. Name of backup job: **photoBackup**. This refers to the `<snapshot id>` option.
5. Name of remote storage: **nachoStorage**. This refers to the `-storage-name <name>` option.

For me, **sftp://curt@nacho.local/remotePhotos** would be the complete `<storage url>`.    

**STEP 1:**      

Initialize the remote storage and repository. Run the following command from the directory you are backing up:
~~~
$ duplicacy init -e -storage-name nachoStorage photoBackup sftp://curt@nacho.local/photobackup
~~~
You will see the following prompts:
~~~
Enter SSH password:
Enter the path of the private key file:
Enter storage password for sftp://curt@nacho.local/remotePhotos:
Re-enter storage password:
~~~

1. Hit `enter` to leave the password blank if you use **ssh** keys.       
2. Type in the full path to your **private ssh** key. For me, I would enter **/home/curt/.ssh/id_rsa** for the path.  
3. Type in a storage password you choose to use.

**STEP 2:**    

**Duplicacy** creates a configuration folder named **.duplicacy** in the current directory after the initialization is finished.   

Copy your **private ssh** key to this directory. Here is how I would copy it on my system:
~~~
$ cp /home/curt/.ssh/id_rsa /home/curt/Photos/.duplicacy/
~~~

**STEP 3:**    

Edit the **.duplicacy/preferences** file with a plain text editor to change the **"keys":**&nbsp; line.
~~~
$ vim .duplicacy/preferences
~~~

Default setting:
~~~
"keys": null,
~~~

Change the line to this:
~~~
"keys": {
            "ssh_key_file": ".duplicacy/id_rsa"
         },
~~~

# Running the backup

**STEP 1:**    

Run the **duplicacy backup** command from the backup directory.
~~~
$ duplicacy backup -stats
~~~

You will be prompted to enter the storage password. 
Enter the storage password you created when you initialized the repository.

Use the `DUPLICACY_<STORAGENAME>_PASSWORD` environment variable if you do not want to type in the password every time you run the backup command.  

* You can run the **export** command or specify this variable in a **bash** script if you want to use this.

* For example, if your storage password was `ThisIsNotAVerySecurePassword`, you would run the following from your terminal or add it to a **bash** script:
~~~
export DUPLICACY_NACHOSTORAGE_PASSWORD="ThisIsNotAVerySecurePassword"
~~~

**NOTE**: You need to use capitalization for the remote storage name regardless of the actual name of the storage. This means that `nachoStorage` would be written as `NACHOSTORAGE`.     
  
# Restoring backups    

**STEP 1:**