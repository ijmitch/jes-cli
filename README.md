# Imagining a better set of JES functions for ZOAU

## Initial thought - how it started

Using `jls`, `ddls` and `pjdd` means I can look at JES output, but have to re-type 
the jobname in to do `ddls` and then the step and DD names for `pjdd`.

```
12:58:38 IMITCHE@WINMVS2C ~ → jls '/IMITCHE'
IMITCHE  IJMPRTDU JOB07689 CC        0000
IMITCHE  IJMPRTDU JOB07674 JCLERR       ?
IMITCHE  IJMPRTDU JOB07673 JCLERR       ?
IMITCHE  IJMPRTDU JOB07400 CC        0000
IMITCHE  IJMPRTDU JOB07393 CC        0000
IMITCHE  IJMPRTDU JOB07390 CC        0000
IMITCHE  IJMPRTDU JOB07082 CC        0000
IMITCHE  IJMPRTDU JOB98542 CC        0000
IMITCHE  IJMPRTDU JOB98518 CC        0000
IMITCHE  IJMPRTDU JOB98459 CC        0000
12:58:52 IMITCHE@WINMVS2C ~ → ddls JOB98459
JES2     -        JESMSGLG UA      133    21   2   1217
JES2     -        JESJCL   V       136    69   3   3980
JES2     -        JESYSMSG VA      137    35   4   2037
IPCSDUMP -        IPCSTOC  VBA     137     3 103    382
IPCSDUMP -        IPCSPRNT VBA     137    47 104   3371
IPCSDUMP -        SYSTSPRT VBA     137    61 105   3373
12:59:20 IMITCHE@WINMVS2C ~ → pjdd JOB98459 IPCSDUMP IPCSPRNT | nvim -
```

This example pipes some IPCS output into `nvim` for browsing (could equally be `nano` or
another way to browsing.

My first thought was to imagine a TUI for all this (perhaps using [Rich](https://github.com/Textualize/rich),
but I'm not expert in that to even choose the best option).

But thinking about it some more, perhaps just a simpler implementation of *autocomplete* would be another option.
(Maybe using [argcomplete](https://github.com/kislyuk/argcomplete).)

Something like:
```shell
jes dd JOB07689
       JOB07674
       JOB07673 
       JOB07400 
       JOB07393 
       JOB07390
       JOB98459 
```
where `jls` is used to find the suggestions for the jobnames then once one is picked:

```
jes dd JOB98459 JES2
                IPCSDUMP
```
`ddls` is used to find suggestions for steps, and finally:

```
jes dd JOB98459 IPCSDUMP IPCSTOC
                         IPCSPRNT
                         SYSTSPRT
```
means I can pick the data set I want to see.

Ending up with:

```
jes dd JOB98459 IPCSDUMP IPCSPRNT
```

(The 'verb' could equally be `cat` rather than `dd` - I'm very open to the right choice.)

And, of course, I can add `| nvim -` to redirect into an edit buffer rather than just stdout.

## What else?

Having dreamt up `jes` as a CLI with autocompletion capabilities, I began thinking of more functions...

### Submitting JCL

Yes, we have USS `submit` and ZOAU `jsub` - and you can capture the jobname into an environment variable
so that the last one is available for

```
ddls $MYJOB
pjdd $MYJOB jes2 sysmsglg
```

Assuming autocompletion is a good thing...

```
jes sub <path-to-folder-of-jcl-source>/cicsa.jcl
                                      /cicsb.jcl
                                      /sysdump.jcl
```
and then a `jes dd ...` as above would discover the job or a more sophisticated scheme of remembering the jobnames.

Then I thought about some 'context' for the jobs to help relate simple jobnames to their purpose.

```
jes sub <path-to-folder-of-jcl-source>/cicsa.jcl -o <path>
```
where `-o <path>` would optionally create a folder for the job (defaulting to use the jobname
as the new folder's name) which could then be used in:

```
jes cp JOB98459 IPCSDUMP IPCSPRNT -o <path>
```
to copy the data out of JES and into the zFS location so it's easier to work with.

(Which is not much more than `jes dd JOB98459 IPCSDUMP IPCSPRNT > <path>/JOB98459/IPCSPRNT.txt` or similar but with less typing.)

Of course, I'm then responsible for deciding the fate of that job's folder in zFS.
