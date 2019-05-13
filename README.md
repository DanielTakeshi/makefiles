# Testing Makefiles

Here's some basic C compilation, and then running the resulting C "executable"
file `a.out`:

```
gcc test.c ; ./a.out
```

or do this if you'd rather compile with a more interesting name:

```
gcc -o hithere test.c  ; ./hithere
```

If you look at the three files that we have with 'hello' in some form, then to
compile:

```
gcc -o hellomake hellomake.c hellofunc.c -I.
```

and to run, `./hellomake`. The above actually compiles *both* of the .c files.

Note that if you don't use that period at the end, you'd get:

```
$ gcc -o hellomake hellomake.c hellofunc.c -I
gcc: error: missing path after ‘-I’
```

Because it needs the include for gcc to find the `hellomake.h` file.

But ideally, we don't want to type or re-use the command each time. Or, if we
have many .c files, if we only change one out of many, we don't want to compile
all of them. The solution is to use Makefiles.


## Makefiles

Use Makefiles for containing the compilation rules. Usually we just type `make`
if a `Makefile` exists. Here's an easy way to do this:

```
CC=gcc
CFLAGS=-I.

hellomake: hellomake.o hellofunc.o
     $(CC) -o hellomake hellomake.o hellofunc.o
```

To run this, put these three `hello`-related files in their own clean
directory. In this repository they might not be set that way as I prefer more
sophisticated Makefiles which use `include` and `src` directories. :-)

And we get this, where here I'm showing the stuff in the directory. It seems
like the make will go through and print the resulting individual compiler
commands (i.e., the stuff that starts with `gcc` in stdout). It creates *two*
object files ending in .o and then the actual executable in `hellomake`. And
also note that `-I.` is right after the compiler, i.e., the `CFLAGS` argument
is automatically implicitly "inserted" there.

```
$ ls -lh
total 24K
-rw-rw-r-- 1 daniel daniel  117 May 13 09:25 hellofunc.c
-rw-rw-r-- 1 daniel daniel  111 May 13 09:25 hellomake.c
-rw-rw-r-- 1 daniel daniel   57 May 13 09:24 hellomake.h
-rw-rw-r-- 1 daniel daniel   98 May 13 09:36 Makefile
-rw-rw-r-- 1 daniel daniel 2.6K May 13 09:40 README.md
$ make
gcc -I.   -c -o hellomake.o hellomake.c
gcc -I.   -c -o hellofunc.o hellofunc.c
gcc -o hellomake hellomake.o hellofunc.o
$ ls -lh
-rw-rw-r-- 1 daniel daniel  117 May 13 09:25 hellofunc.c
-rw-rw-r-- 1 daniel daniel 1.5K May 13 09:42 hellofunc.o
-rwxrwxr-x 1 daniel daniel 8.5K May 13 09:42 hellomake
-rw-rw-r-- 1 daniel daniel  111 May 13 09:25 hellomake.c
-rw-rw-r-- 1 daniel daniel   57 May 13 09:24 hellomake.h
-rw-rw-r-- 1 daniel daniel 1.4K May 13 09:42 hellomake.o
-rw-rw-r-- 1 daniel daniel   98 May 13 09:36 Makefile
-rw-rw-r-- 1 daniel daniel 2.6K May 13 09:40 README.md
```

And this is the more sophisticated Makefile with the new file directory:

```
IDIR =../include
CC=gcc
CFLAGS=-I$(IDIR)

ODIR=obj
LDIR =../lib

LIBS=-lm

_DEPS = hellomake.h
DEPS = $(patsubst %,$(IDIR)/%,$(_DEPS))

_OBJ = hellomake.o hellofunc.o
OBJ = $(patsubst %,$(ODIR)/%,$(_OBJ))


$(ODIR)/%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: $(OBJ)
	$(CC) -o $@ $^ $(CFLAGS) $(LIBS)

.PHONY: clean

clean:
	rm -f $(ODIR)/*.o *~ core $(INCDIR)/*~
```

See [the tutorial][1] for the steps involved in getting to this. Some reminders
to myself:

- Use `CC` and `CXX` for defining C and C++ compilers, respectively (i.e., put
  `CC := gcc` at the top of `Makefile`). Usually on Ubuntu, these would be gcc
  and g++.

- Another special "make" keyword: `CFLAGS` represents the list of flags to pass
  to the compilation command (and presumably the same for `CXXFLAGS` and C++).
  Usually, we'll use `-I` for representing any "dependencies", and note that we
  put the directory right after it. Recall that we used `-I.` for the current
  directory. But you could use `-I../include` like we have above.

- If you get a "missing separator" error, usually that means the Makefile has
  whitespaces instead of tabs. That's annoying for my vim, so be sure to have
  an easy way to type the TAB key. [See this question][4] for an easy vim
  solution.

- You'll see stuff like this in the Makefile:
  ```
  output: inputs
      commandToExecute
  ```
  which specify the "rules". The "inputs" are also viewed as "dependencies"
  because we need then to generate the "output" after running the
  "commandToExecute"! For example:

  ```
  %.o : %.c $(DEPS)
  ```

  for the output:inputs thing means that all the files ending with .o depend
  upon the .c version fo those files, and all of the stuff in the DEPS variable
  (actually, called "macro"). Incidentally, the percent sign `%` is how we can
  refer to all files matching some expression, similar to the asterisk `*` in
  the command line.

- For using constants or macros, use the `$(...)` syntax, where `...` is
  replaced with the appropriate name.

- There are special macros, `$@` and `$^` representing the left and right sides
  of the colon sign (i.e., inputs versus outputs).

- If things are working after doing `make`, then subsequent `make` commands
  will just say that the files are "up to date".

- `#` means comments, though there aren't any in the above Makefile.

- Not shown the above, but you can define boolean variables in Makefiles, such
  as `NO_OPENGL := True` if you'd rather not let people go through the pains of
  OpenGL. Then in the Makefile, you can do things like:

  ```
  ifdef NO_OPENGL:
      # put command here.
  endif
  ```

  for if the command is even defined (could be commented out), or this:

  ```
  ifndef NO_OPENGL:
      # put command here.
  endif
  ```

  if ... you guessed it, the command is *not* defined. But it seems like this
  is discouraged in Makefiles (and C/C++ code, for that matter) since it
  hinders readability.

- Use a `make clean` command to remove .o object files.

- You can use special functions like `patsubst` in Makefiles to manipulate
  text. [See this for a reference][6].



## References

Useful tutorials:

- [Tutorial][1]
- [Tutorial][5]

Useful questions:

- [Question][2]
- [Question][3]
- [Question][7]

[1]:http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/
[2]:https://stackoverflow.com/questions/653807/determining-c-executable-name
[3]:https://stackoverflow.com/questions/920413/make-error-missing-separator
[4]:https://stackoverflow.com/questions/4781070/how-to-insert-tab-character-when-expandtab-option-is-on-in-vim
[5]:http://www.cs.ucr.edu/~nael/cs153/lectures/C-tutorial/makefile.html
[6]:https://stackoverflow.com/questions/32176074/function-patsubst-in-makefile
[7]:https://stackoverflow.com/questions/448910/what-is-the-difference-between-the-gnu-makefile-variable-assignments-a
