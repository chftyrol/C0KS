# C0KS
## Chief's Zero Knowledge Storage

* * *

### USAGE<a name="USAGE"></a>

As this is a command line utility, it is run by launching in a terminal the command

**c0ks** [_options_] _action_

where _options_ and _action_ are explained in detail in the sections **OPTIONS** and **ACTIONS** of this document.

To obtain the desired results from using C0KS, configure it following the **CONFIGURATION** section before running it.

### DESCRIPTION<a name="DESCRIPTION"></a>

_**C0KS**_ stands for Chief’s Zero Knowledge Storage. It was born out of a necessity to give users of online file hosting services a chance to be in control of their data in “the cloud”.

In essence it is a simple duplicity wrapper with a couple of extra features. Duplicity works by taking backups of a directory: each backup is split up into volumes (each of max 200M), which are signed and encrypted using GnuPG and then uploaded to a specified remote. Duplicity uses the rsync utility, which enables is to only process and send those parts of files that have changed since the last backup. This minimizes the traffic and space usage. What is stored in the remote are only encrypted tarballs of the data (for full backups) or encrypted tar archives of the new files and deltas of old backups (for incremental backups)

The use of encryption ensures that the remote location cannot know anything about what is being uploaded and hosted, except for the size of the volumes and when they are uploaded. Hence the zero knowledge part of the name C0KS. Moreover, before being encrypted the volumes are signed with the user’s private PGP key, which makes sure that when he later accesses his data he can be sure it was not tampered with, by checking the cryptographic signature.

C0KS adds to Duplicity a _diff feature_, which enables the user to clearly visualize the differences between the remote and local data, without having to download and process it all: it notifies about new local files absent in the remote and viceversa, as well as changes to local or remote files. The user can then decide what to do, namely synchronizing the remote with the local storage or pulling the more recent data from the remote to the local storage.

### OPTIONS<a name="OPTIONS"></a>

**-c** (No Colors)

Disable colors in the output.

**-h** (Show Help)

Print a help message, useful for a quick reminder of the available options and actions.

**-L** _file_ (Specify Log File)

Use _file_ as log file for C0KS.

**-n** (Dry Run)

Run the program without performing any action, but merely simulating what would be done. This is most useful combined with the **-v** option to enable verbose output.

**-v** (Verbose Output)

Enable verbose output. This also makes duplicity more verbose, which can make the output quite noisy.

**-w** _dir_ (Specify Local Target Directory)

Sets _dir_ as the local directory where the data is locally stored.

### ACTIONS<a name="ACTIONS"></a>

**push**

On the first run, **push** creates a new full backup of the specified local directory. On subsequent runs incremental backups are performed. If a file was modified locally its changes will be uploaded. If a file that was in the remote was locally deleted it will be deleted in the remote as well. If a new file was created locally it will be uploaded to the remote (after being processed, encrypted and signed).

When done print a summary of what has been done.

**pull**

Replace the contents of the local storage with that of the remote, downloading (decrypting and verifying) the data to the local storage directory. Note that at the moment it is not possible to simply pull the files that are new or updated on the remote and place them in the local storage directory. Only a full restore of the remote to the local storage is supported for now. Implementation of partial restores has already been planned.

**status**

Show some information about the remote such as: date and time of first and last backups, number of backups performed, number of contained volumes, as well as a table of backup sets.

**diff**

Show delta of remote and local storage, without downloading the whole backup. The output highlights: new local files to be uploaded to remote, new remote files to be pulled to the local storage directory, files that changed on the remote and files that changed in the local storage.

### CONFIGURATION<a name="CONFIGURATION"></a>

Options and actions should be satisfactory for everyday usage of C0KS. Nonetheless the program should be configured carefully before running it, to obtain the desired results. Parameters that require configuration are at least: address and password of the remote, encryption and signing keys, local storage directory. Other parameters may be configured as well.

Parameters are configured by directly changing their values in the source of the script.

**ENCRYPT_KEY**

ID or fingerprint of the GnuPG key to be used to encrypt files in the remote.

**SIGN_KEY**

ID or fingerprint of the GnuPG key to be used to sign files in the remote (can be equal to ENCRYPT_KEY)

**GPG_AGENT_CACHE_DURATION**

Determines maximum duration of a push action. Defaults to 1 week.

**REMOTE_ADDR**

The address of the remote, using the appropriate protocol (ftp, webdav, ecc) using the syntax [protocol]://[user]@[server]

e.g. If Sharon used box.com as a remote storage provider (which supports the webdav protocol) and she was registered with the email address sharon.valerii@galactica.com she would have to set something like:

**REMOTE_ADDR="webdavs://sharon.valerii@galactica.com@dav.box.com/dav/c0ks"**

e.g. If Sharon used her account sharon.valerii at the cylon server for ftp and her path there was /baseship/boomer she would have to set something like:

**REMOTE_ADDR="sftp://sharon.valerii@cylon.com/baseship/boomer"**

Duplicity currently supports the following solutions as remotes: local file storage, scp/ssh, ftp, rsync, HSI, WebDAV, Tahoe-LAFS, and Amazon S3\. For details on how to specify the REMOTE_ADDR in cases not covered here, refer to the Duplicity documentation.

**PASS_ENTRY**

If the password manager utility _pass_ is installed and you have an entry in _pass_ for the remote you are going to use (ftp server, webdav account, etc.) set this variable to the name of the _pass_ entry. e.g. If

**# pass show c0ks_remote**

shows the password to your remote, set this variable to _c0ks_remote_.

**LOG_FILE**

Where you want to save the session log file. Set to **/dev/null** if you don’t want logs.

**STORAGE_NAME**

A name for your C0KS. This acts as an identifier for your set of backups.

**DUPLICITY_LOG_FD**

Used to redirect output between stdout and log file. Do NOT change this, unless you know what you are doing.

**DUPLICITY_OPTIONS**

Specify some sensible Duplicity options. Do NOT change this, unless you know what you are doing.

**LOCAL_TARGET_DIR**

Path of the directory where you want to store the local copy of the data. This can also be specified with the **-w** option.

**EXCLUDE**

List here optional subdirs of **LOCAL_TARGET_DIR** that you don’t want included in the backups. Each entry should be preceded by --exclude.

e.g. **EXCLUDE="--exclude ${LOCAL_TARGET_DIR}/useless_stuff --exclude ${LOCAL_TARGET_DIR}/unnecessary_dir"**

### A NOTE ON PASSWORDS<a name="A NOTE ON PASSWORDS"></a>

C0KS makes use of two distinct passwords: the one to access the remote storage and the one to unlock the GnuPG private key. The latter should be asked by the _gpg-agent_ pinentry utility or directly by Duplicity when needed. Since **push** actions can take a long time to complete (for large full backups), a new gpg-agent instance is started when c0ks is run and is closed when c0ks exits. The maximum time the _gpg-agent_ daemon will be kept running is **GPG_AGENT_CACHE_DURATION** and defaults to a week.

The password to the remote storage can be specified in two ways: if you use the password manager _pass_ by setting the configuration variable **PASS_ENTRY**; otherwise you can specify the password by setting the environment variable **C0KS_PASSWORD** when launching c0ks. e.g.

**# C0KS_PASSWORD="itsinthefrakkingship" c0ks push**

### NOTICE<a name="NOTICE"></a>

C0KS’s author is not responsible for any data loss or damage to your system. Even though highly unlikely to happen, keep in mind this project is still in its testing phase. For more advanced operations on the storage, such as restoring specific backup or deleting backups you should use Duplicity directly. In passing, I thank Duplicity and its community for working at such a wonderful project which inspired me to write this little tool.

* * *
