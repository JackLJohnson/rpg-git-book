# Adding a build tool

In the tooling chapter of this book, we installed GNU Make with yum. We did this because GNU Make will be the build tool we use to build our entire application. This chapter will cover all the nitty gritty bits, like building data areas, message files, table, programs, service programs, binding directories, the lot.

## The `system` command in pase

The pase environment provides the `system` command which allows us to run ILE commands from a pase shell. The `system` command will run the command in a new job - this should be remembered as important when dealing with library lists and the use of QTEMP.

Some examples of the command are as follows (direct from the IBM documentation site):

1. List all of the active jobs: `system wrkactjob`
2. Create a test library: `system "CRTLIB LIB(TESTDATA) TYPE(*TEST)"`
3. Delete a library and do not write any messages: `system -q "DLTLIB LIB(TESTDATA)"`

For more, you can visit the IBM documentation on the `system` command: https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_73/rzahz/rzahzsystem.htm

## Creating a makefile for ILE projects - Simple programs

### Knowing your dependancies

Let’s start with something simple. Let’s say we have this file structure.

```
/myproject
    - src
        - programa.rpgle
        - programb.rpgle
        - programc.rpgle
        - stringlib.c
        - thecl.clle
    - headers
        - stringlib.c
        - stringlib.rpgle`
```

From your source tree, you should determine what the objects you want to build are and each dependancy is. For example:

* programa (program)
  * programa (module)
  * stringlib (module)
* programb (program)
  * programb (module)
* programc (program)
  * programc (module)
  * stringlib (module)
* thecl (program)
  * thecl (module)

From this list, you should get the impression that we build all sources into modules and then into their retrospective program objects.

### Creating your rules

This information (knowing your dependancies) will allow us to create conditions (in our makefile) based on our program dependancies. First of all, we need to setup rules for what to do for each type of source we have.

```makefile
%.rpgle:
    system "CRTRPGMOD MODULE($(BIN_LIB)/$*) SRCSTMF('./src/$*.rpgle') DBGVIEW($(DBGVIEW)) REPLACE(*YES)"

%.c:
    system "CRTCMOD MODULE($(BIN_LIB)/$*) SRCSTMF('./src/$*.c') DBGVIEW($(DBGVIEW)) REPLACE(*YES)"

%.clle:
    #Can't compile CL from IFS on all OS versions..
    -system -q "CRTSRCPF FILE($(BIN_LIB)/QSRC) RCDLEN(112)"
    system "CPYFRMSTMF FROMSTMF('./src/$*.clle') TOMBR('/QSYS.lib/$(BIN_LIB).lib/QSRC.file/$*.mbr') MBROPT(*replace)"
    system "CRTCLMOD MODULE($(BIN_LIB)/$*) SRCFILE($(BIN_LIB)/QSRC) DBGVIEW($(DBGVIEW))"

%.pgm:
    system "CRTPGM PGM($(BIN_LIB)/$*) MODULE($(patsubst %,$(BIN_LIB)/%,$(basename $^))) ENTMOD($*) REPLACE(*YES)"

all:
    @echo "Build finished!"
```

That’s all our rules setup, but let's look at the GNU Make functions and variables we are using:

* `$(BIN_LIB)` and `$(DBGVIEW)` are variables which we haven’t defined yet - that’s next.
* `$(patsubst ...` is a pattern substring function in make. We are using this to prepend the binary library for each module.
* `$(basename value)` is a function that will return the basename of a path. E.g. `myfile.rpgle` -> `myfile`.
* `$*` is used to get the current rule name that we’re working with.
* `$^` returns the values passed into the current condition.

### Dependancy list

Next, we can setup the dependancy list. **This must be placed above the rules we just created**:

```makefile
# First our variables
BIN_LIB=MYLIBRARY
DBGVIEW=*ALL

# Define what needs to get built when make is run
all: programa.pgm programb.pgm programc.pgm thecl.pgm

# %program%: depends on...

programa.pgm: programa.rpgle stringlib.c
programb.pgm: programb.rpgle
programc.pgm: programc.rpgle stringlib.c
thecl.pgm: thecl.clle

# Below are the rules.
```

### Executing GNU Make

From there, developers can call make with or without optional parameters:

* `make` - will make all (because all is the first rule)
* `make programa.pgm` will build only `programa.pgm`
* `make stringlib.c` will only build `stringlib.c`
* `make BIN_LIB=DEVLIB` will `make all`, but with the `BIN_LIB` variable as `DEVLIB`.

If you run `gmake -n` against this `makefile`, you can see the commands it would run.

```
barry$ make -n

system "CRTRPGMOD MODULE(MYLIBRARY/programa) SRCSTMF('./src/programa.rpgle') DBGVIEW(*ALL) REPLACE(*YES)"
system "CRTCMOD MODULE(MYLIBRARY/stringlib) SRCSTMF('./src/stringlib.c') DBGVIEW(*ALL) REPLACE(*YES)"
system "CRTPGM PGM(MYLIBRARY/programa) MODULE(MYLIBRARY/programa MYLIBRARY/stringlib) ENTMOD(programa) REPLACE(*YES)"
system "CRTRPGMOD MODULE(MYLIBRARY/programb) SRCSTMF('./src/programb.rpgle') DBGVIEW(*ALL) REPLACE(*YES)"
system "CRTPGM PGM(MYLIBRARY/programb) MODULE(MYLIBRARY/programb) ENTMOD(programb) REPLACE(*YES)"
system "CRTRPGMOD MODULE(MYLIBRARY/programc) SRCSTMF('./src/programc.rpgle') DBGVIEW(*ALL) REPLACE(*YES)"
system "CRTPGM PGM(MYLIBRARY/programc) MODULE(MYLIBRARY/programc MYLIBRARY/stringlib) ENTMOD(programc) REPLACE(*YES)"
# Can't compile CL from all OS versions..
system -q "CRTSRCPF FILE(MYLIBRARY/QSRC) RCDLEN(112)"
system "CPYFRMSTMF FROMSTMF('./src/thecl.clle') TOMBR('/QSYS.lib/MYLIBRARY.lib/QSRC.file/thecl.mbr') MBROPT(*replace)"
system "CRTCLMOD MODULE(MYLIBRARY/thecl) SRCFILE(MYLIBRARY/QSRC) DBGVIEW(*ALL)"
system "CRTPGM PGM(MYLIBRARY/thecl) MODULE(MYLIBRARY/thecl) ENTMOD(thecl) REPLACE(*YES)"
echo "Build finished!"
```

As a cool side note: even though we defined `stringlib.c` as a dependancy for two programs, it only built once.

## Creating a makefile for ILE projects - Service programs and binding directories

Service programs are made up of three important things:

* The module (or modules)
* The binder source
* The binding directory entry.

### Creating your rules

Let’s go ahead and create rules for creating the `.srvpgm` object, `.rpgle` module and `.bnddir` entry.

```makefile
%.rpgle:
    system "CRTRPGMOD MODULE($(BIN_LIB)/$*) SRCSTMF('./src/$*.rpgle') DBGVIEW($(DBGVIEW)) REPLACE(*YES)"

%.srvpgm:
    # We need the binder source as a member! SRCSTMF on CRTSRVPGM not available on all releases.
    -system -q "CRTSRCPF FILE($(BIN_LIB)/QSRC) RCDLEN(112)"
    system "CPYFRMSTMF FROMSTMF('./src/$*.binder') TOMBR('/QSYS.lib/$(BIN_LIB).lib/QSRC.file/$*.mbr') MBROPT(*replace)"

    system "CRTSRVPGM SRVPGM($(BIN_LIB)/$*) MODULE($(patsubst %,$(BIN_LIB)/%,$(basename $^))) SRCFILE($(BIN_LIB)/QSRC)"

%.bnddir:
    -system -q "CRTBNDDIR BNDDIR($(BIN_LIB)/$*)"
    -system -q "ADDBNDDIRE BNDDIR($(BIN_LIB)/$*) OBJ($(patsubst %.entry,(*LIBL/% *SRVPGM *IMMED),$^))"

%.entry:
    # Basically do nothing..
    @echo ""
```

### Dependancy list

Next, we need to define the dependency list rules (this goes above our rules, like last time).

```makefile
BIN_LIB=MYLIBRARY
DBGVIEW=*ALL

all: apipkg.srvpgm webcalls.srvpgm tools.bnddir

apipkg.srvpgm: apipkg.rpgle
webcalls.srvpgm: webcalls.rpgle othermod.rpgle

tools.bnddir: apipkg.entry webcalls.entry
```

### Executing GNU Make

Next, if you run `gmake -n` you can see what commands make would execute:

```
barry$ make -n

system "CRTRPGMOD MODULE(MYLIBRARY/apipkg) SRCSTMF('./src/apipkg.rpgle') DBGVIEW(*ALL) REPLACE(*YES)"
system -q "CRTSRCPF FILE(MYLIBRARY/QSRC) RCDLEN(112)"
system "CPYFRMSTMF FROMSTMF('./src/apipkg.binder') TOMBR('/QSYS.lib/MYLIBRARY.lib/QSRC.file/apipkg.mbr') MBROPT(*replace)"
system "CRTSRVPGM PGM(MYLIBRARY/apipkg) MODULE(MYLIBRARY/apipkg) SRCFILE(MYLIBRARY/QSRC)"
system "CRTRPGMOD MODULE(MYLIBRARY/webcalls) SRCSTMF('./src/webcalls.rpgle') DBGVIEW(*ALL) REPLACE(*YES)"
system "CRTRPGMOD MODULE(MYLIBRARY/othermod) SRCSTMF('./src/othermod.rpgle') DBGVIEW(*ALL) REPLACE(*YES)"
system -q "CRTSRCPF FILE(MYLIBRARY/QSRC) RCDLEN(112)"
system "CPYFRMSTMF FROMSTMF('./src/webcalls.binder') TOMBR('/QSYS.lib/MYLIBRARY.lib/QSRC.file/webcalls.mbr') MBROPT(*replace)"
system "CRTSRVPGM PGM(MYLIBRARY/webcalls) MODULE(MYLIBRARY/webcalls MYLIBRARY/othermod) SRCFILE(MYLIBRARY/QSRC)"
echo ""
echo ""
system -q "CRTBNDDIR BNDDIR(MYLIBRARY/tools)"
system -q "ADDBNDDIRE BNDDIR(MYLIBRARY/tools) OBJ((*LIBL/apipkg *SRVPGM *IMMED) (*LIBL/webcalls *SRVPGM *IMMED))"
```

As you can see, it doesn’t really do anything with the .entry rule, because really we just need the values for the .bnddir rule.
