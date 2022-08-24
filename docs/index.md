## Introduction

[Duplicacy](https://duplicacy.com) is state-of-the-art backup tool that has extensive cloud support. It also supports local disks and your own SFTP servers.     

Duplicacy is available as a web-based GUI or as a command line tool. 
The software does require a license but the command-line interface (CLI) version is free for personal use.  
  
We’ll be using the **CLI** version for this tutorial.

## Installation    

**STEP 1:**   

Pre-compiled binaries are availble for Linux, macOS, and Windows directly from the Duplicacy [GitHub](https://github.com/gilbertchen/duplicacy/releases) repository.

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

Rename the file to `duplicacy`.
~~~
$ mv duplicacy_linux_x64_2.7.2 duplicacy
~~~

**STEP 4:**  

Change the file permissions.
~~~
$ chmod 755 duplicacy
~~~

**STEP 5:**  

Move `duplicacy` to the `/usr/bin` directory.
~~~
$ sudo mv duplicacy /usr/bin/
~~~

## Initialize the remote storage and repository

The **duplicacy init** command is used to initialize the remote storage and the backup directory.
~~~
duplicacy init [command options] <snapshot id> <storage url>
~~~
The `<snapshot id>` refers to the name you want to give to your backup job.    
The `<storage url>` refers to the remote server and directory path for your backups.

We will only use the following options:
~~~
-encrypt, -e          encrypt the storage with a password
-storage-name <name>  assign a name to the storage
~~~

**Example scenario:**

You have an `sftp` server named **nacho.local** that you want to use to backup your **Photos** directory.      
You have a directory named **remotePhotos** on the **nacho.local** `sftp` server. This is the directory to be used for the backups.    
You can connect to the **nacho.local** `sftp` server using `ssh` keys.    
You want to name the backup job **photoBackup**. This refers to the `<snapshot id>` option.    
You want to name the remote storage **nachoStorage**. This refers to the `-storage-name <name>` option.    

> For me, I would use **sftp://curt@nacho.local/remotePhotos** for the `<storage url>`.

**STEP 1:**      

Initialize the remote storage and repository.
~~~
$ duplicacy init -e -storage-name nachoStorage photoBackup sftp://curt@nacho.local/remotePhotos
~~~
You will see the following prompts:
~~~
Enter SSH password:
Enter the path of the private key file:
Enter storage password for sftp://curt@nacho.local/remotePhotos:
Re-enter storage password:
~~~
`Enter SSH password`:&nbsp; Hit the `enter` key on your keybord to leave the password blank if you use `ssh` keys.       

`Enter the path of the private key file`:&nbsp; Type in the full path to your **private** `ssh` key. 
> For me, I would enter **/home/curt/.ssh/id_rsa** for the path.  

`Enter storage password`:&nbsp; Type in the storage password you wish to use.

**STEP 2:**    

**Duplicacy** creates a configuration folder named `.duplicacy` after the initialization is completed.   

Copy your **private** `ssh` key to this directory. Here is how I would copy it on my system:
~~~
$ cp /home/curt/.ssh/id_rsa /home/curt/Photos/.duplicacy/
~~~

**STEP 3:**    

Edit the `.duplicacy/preferences` file to change the `"keys":` line. Use a plan text editor such as `vim` or `nano`.
~~~
$ vim .duplicacy/preferences
~~~

Default setting:
~~~
"keys": null,
~~~

Change `"keys": null,` to this:
~~~
"keys": {
            "ssh_key_file": ".duplicacy/id_rsa"
         },
~~~
> Use a text editor such as `vim` or `nano` to edit the file because spacing is **important**.

## Running the backup command

**STEP 1:**    

Run the `duplicacy backup` command from the backup directory.
~~~
$ duplicacy backup -stats
~~~

You will be prompted to enter the storage password. 
Enter the storage password you created when you initialized the repostiory.

If you do not want to enter the storage password every time you run the backup, you can use the `DUPLICACY_<STORAGENAME>_PASSWORD` environment variable. If you want to automate your backups by running them from a `bash` script via `cron`, you will need to also use this variable.    

If your storage password is `ThisIsNotAVerySecurePassword`, the command would like this in our example scenario:
~~~
export DUPLICACY_NACHOSTORAGE_PASSWORD="ThisIsNotAVerySecurePassword"
~~~
**NOTE**: You need to use capitalization for the remote storage name regardless of the actual name of the storage. This means that `nachoStorage` needs to be written as `NACHOSTORAGE`.

You can run this shell command before you run the `duplicacy backup` command or add it to a `bash` script.
