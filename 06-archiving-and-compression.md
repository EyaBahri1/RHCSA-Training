<h1 align="center" style="color: red;">Archiving & Compression</h1>

## Introduction
👋 In this section, we will explore how to archive and compress directories in a Red Hat Linux environment.

### Theory:
<p align="center">
  <img src="images/Cap.JPG" alt="cap" style="width: 400px;"/>
</p>

* `tar -cvf archivename.tar directory` → gather files into a single archive. (Step 1)
* `tar -tvf archivename.tar` → list the contents of an archive.

Compress the resulting archive using gzip or bzip2: (Step 2)
* `gzip archivename.tar` → compress with gzip.
* `gzip -d archivename.tar.gz` → decompress a gzip archive.
* `bzip2 archivename.tar` → compress with bzip2.
* `bzip2 -d archivename.tar.bz2` → decompress a bzip2 archive.

Perform both steps at once (archiving + compression):
* `tar -cvzf archive.tar.gz directory` → archive and compress using gzip.
* `tar -cvjf archive.tar.bz2 directory` → archive and compress using bzip2.
* `tar -cvzf archive.tar.gz -C /source/dir .` → create archive from a specific directory (`.` = relative paths, no absolute path stored in archive).
* `ls -lh` → check the file size.

Extract an archive:
* `tar -xvf archivename.tar` → extract a tar archive.
* `tar -xvzf archivename.tar.gz` → extract a gzip-compressed archive.
* `tar -xvjf archivename.tar.bz2` → extract a bzip2-compressed archive.
* `tar -xvzf archivename.tar.gz -C /target/directory` → extract to a specific directory.

> **Options summary:**
> `-c` create · `-x` extract · `-t` list · `-v` verbose · `-f` file · `-z` gzip · `-j` bzip2 · `-C` change directory

---

## Lab 06

#### Q0. Create an archive named "documents.tar.gz" containing all files in the "/home/user/documents" directory:
```bash
tar -cvzf document.tar.gz /home/user/documents
```

#### Q1. Compress the entire "/var/log" directory into a file named "logs\_backup.tar.bz2":
```bash
tar -cvjf logs_backup.tar.bz2 /var/log
```

#### Q2. Make a compressed backup of the "/etc" directory and save it as "etc\_backup.tar.gz":
```bash
tar -cvzf etc_backup.tar.gz /etc
```

<p style="text-align: right;">
  <a href="https://github.com/halekammoun/RHCSA-Training/blob/main/README.md#table-des-matieres">Back to Table of Contents</a>
</p>
