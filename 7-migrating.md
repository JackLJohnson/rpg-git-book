# Starting the migration

This chapter will cover what migration means, what tools can be used to migrate source members, handling changes to the copy books, compiling sources from the IFS (editors and command line) and why iProjects is not used.

## What migration means

Migration actually means copying the source member contents to the IFS. Only one person has to do this step. Once all the source is migrated into git, is when normal development. If you let someone edit the source members while migrating the source to the IFS, you will have to re-copy the changed members.

For example, your library, source physical files and members might look like this:

```
DEVLIB
  - QRPGLESRC
    - PROGRAMA.RPGLE
    - PROGRAMB.RPGLE
    - PROGRAMC.RPGLE
  - QSQLSRC
    - CUSTOMERS.SQL
    - INVENTORY.SQL
  - QCLLESRC
    - STARTJOB.CLLE
  - QCMDSRC
    - STARTJOB.CMD
```

Where the resulting layout in the IFS could be very similar:

```
/home
  /barry
    /.git
    /qrpglesrc
      programa.rpgle
      programb.rpgle
      programc.rpgle
    /qsqlsrc
      customers.sql
      inventory.sql
    /qcllesrc
      startjob.cmd
    /qcmdsrc
      startjob.cmd
```

**Notes about migrating to the IFS**:

1. You will lose the TEXT column that source members have, which is usually used for describing what the source is. Although, you can still put that in the program as a comment.
2. The type of the source member becomes the extension when in the IFS.
3. Files and directories of sources are usually stored as all lowercase.
4. It is recommend you retain the 10 character limit on programs, commands, modules, etc - any source related to Db2 for i doesn't matter as much as Db2 for i and most ILE languages support 'long names'
5. Sources on the IFS should be stored as encoding 1208 (UTF-8).

## Tools used for migration

Initially migrating the source code can be the hardest part of the entire process, but once it's done: it's done. There are many ways to do it, but this will only describe two.

### 1. Manually migrating

All a migration consists of is moving source members to the IFS. To our benefit, the `CPYTOSTMF` command exists, which can be used to copy a source member to a stream file. For example:

```
CPYTOSTMF FROMMBR('/QSYS.lib/DEVLIB.lib/QRPGLESRC.file/PROGRAMA.mbr') TOSTMF('/home/barry/project/qrpglesrc/programa.rpgle') STMFOPT(*REPLACE) STMFCCSID(1208)
```

On the basis of this command, you would have to run this command for each source member you want to migrate.

### 2. Using the migrate tool

There is an open source migrate tool, simply named 'migrate', which automates the copying of source members into a directory. It also creates the streamfiles with the correct extensions.

To use the migrate tool, you will need to clone it and build it manually.

```
git clone https://github.com/worksofliam/migrate.git
cd migrate
gmake
```

Building this solution will create the `MIGRATE` library and inside is the `MIGSRCPF` command. `MIGSRCPF` has three simple parameters.

![](./images/migsrcpf.PNG)

If we had a library with source physical files and wanted to migrate them into a new project directory, we would have to run the command once to migrate the source physical file. It will copy the source member into the IFS as a 1208 (UTF-8) streamfile. If the file or folder it tries to create already exists, it will fail.

```
MKDIR DIR('/home/BARRY/myproj')
MIGSRCPF LIBRARY(TESTPROJ) SOURCEPF(QRPGLESRC) OUTDIR('/home/BARRY/myproj')
MIGSRCPF LIBRARY(TESTPROJ) SOURCEPF(QRPGLEREF) OUTDIR('/home/BARRY/myproj')
MIGSRCPF LIBRARY(TESTPROJ) SOURCEPF(QCLLESRC)  OUTDIR('/home/BARRY/myproj')
```

This would create three directories in `/home/BARRY/myproj` like the following:

```
/home
  /BARRY
    /myproj
      /qrpglesrc
        /somesource.rpgle
        /somesource.rpgle
      /qrpgleref
        /whatever.rpgle
      /qcllesrc
        /pgm1.clle
        /pgm2.clle
        /pgm3.clle
```

Note that it will create all directories and files with lowercase names.

### Other possible ways.

You could potentially create an iProject in RDi based on a library and then have a local copy of all the source (which you can then put into a git repository later). You can also use the SPF Clone tool in ILEditor to clone a libraries source members on to your local machine (which can also be put into a git repository later).
