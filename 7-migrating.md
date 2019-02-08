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

Where the resulting layout in the IFS could be very similar.
