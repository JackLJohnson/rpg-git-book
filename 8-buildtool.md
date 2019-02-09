# Adding a build tool

In the tooling chapter of this book, we installed GNU Make with yum. We did this because GNU Make will be the build tool we use to build our entire application. This chapter will cover all the nitty gritty bits, like building data areas, message files, table, programs, service programs, binding directories, the lot.

## The `system` command in pase

The pase environment provides the `system` command which allows us to run ILE commands from a pase shell. The `system` command will run the command in a new job - this should be remembered as important when dealing with library lists and the use of QTEMP.

Some examples of the command are as follows (direct from the IBM documentation site):

1. List all of the active jobs: `system wrkactjob`
2. Create a test library: `system "CRTLIB LIB(TESTDATA) TYPE(*TEST)"`
3. Delete a library and do not write any messages: `system -q "DLTLIB LIB(TESTDATA)"`

For more, you can visit the IBM documentation on the `system` command: https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_73/rzahz/rzahzsystem.htm

## Creating a makefile for ILE projects (part 1)

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

### Creating your first rules

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

## Creating a makefile for ILE projects (part 2)
