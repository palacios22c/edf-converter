# Edf2Mat© Matlab Toolbox

Converts EyeLink 1000 Edf files into Matlab

```txt
  Adrian Etter
  Marc Biedermann

  University of Zurich
  Department of Economics
  Schönberggasse 1
  CH-8001 Zurich
```

E-Mail: <it@econ.uzh.ch>


## Abstract

Edf2Mat is a Matlab Toolbox for easy conversion of EyeLink Edf result files. For fast verification of valid data, there is an included plot function, which displays eye movement,  pupil size and a heatmap of the eye movement. There are a few examples in the example file which help analyze eye data.

## Terms and Conditions

### Copyright

Copyright © 2007-2024 Adrian Etter, Marc Biedermann, University of Zurich. All rights reserved.

This document may be copied, modified, reproduced and redistributed for educational and personal use as long as the original author is mentioned and cited.

MATLAB® is a registered Trademark of MathWorks, Inc.™ (<http://www.mathworks.com>).
EyeLink® is a registered trademark of SR Research Ltd., Mississauga, Ontario, Canada (<http://www.sr-research.com>)

### Acknowledgment

You are allowed to use this software for free, but please acknowledge if you are using this software to process Edf-files:

The conversion of the EyeLink® 1000 Edf files was done with the Edf2Mat Matlab Toolbox designed and developed by Adrian Etter and Marc Biedermann at the University of Zurich.

### License

Edf2Mat Toolbox is Licensed under the MIT License.
The Edf2Mat Toolbox uses slightly modified code (Kovach, 2011) from C. Kovach 2007.

## Installation

### Requirements

#### On Windows: Matlab

- Ensure the [Visual C++ Redistributable for Visual Studio 2015](https://www.microsoft.com/en-us/download/details.aspx?id=48145) is installed
- Ensure the [Visual C++ Redistributable for Visual Studio 2008](https://www.microsoft.com/en-us/download/details.aspx?id=29) is installed
- Both installers require admin rights, see [User-based installation (without admin rights)](#user-based-installation-without-admin-rights) for what is really needed

#### On Mac

the edfapi.framework must be in `/Library/Frameworks`. The Library can be found in the Package. Attention: If the Zip file was unpacked on windows first, and then copied, the folder structure will be broken. The zip-file must be extracted on a Mac in order to work. Otherwise the symlinks will be broken.

on os x in `/Library/Frameworks` it should look like the following

```txt
edfapi.framework/
├── Headers -> Versions/Current/Headers
├── Resources -> Versions/Current/Resources
├── Versions
│   ├── A
│   │   ├── Headers
│   │   │   ├── edf.h
│   │   │   ├── edf_data.h
│   │   │   └── edftypes.h
│   │   ├── Resources
│   │   │   └── Info.plist
│   │   └── edfapi
│   └── Current -> A
└── edfapi -> Versions/Current/edfapi
```

NOTE:

- Intel chips: use `edfapi.Framework-3.1.zip` (later version isn't tested yet)
- Apple Silicon chips: use `edfapi.Framework-4.2.762.0.zip`
- No admin rights to write to `/Library/Frameworks`? See [User-based installation (without admin rights)](#user-based-installation-without-admin-rights)

### Files needed

- The Edf2Mat Class
- All files in the private folder
- All Dlls in the private folder

### User-based installation (without admin rights)

The toolbox itself never needs administrator rights – only two things in the
default instructions above do: writing the framework to `/Library/Frameworks`
on macOS (that folder is owned by `root`), and installing the Visual C++
redistributables on Windows.

| Step | Needs admin? | User-based alternative |
| --- | --- | --- |
| Toolbox files (`@Edf2Mat`, `+edfmex`) | no | keep them anywhere in your home directory |
| Adding the toolbox to the Matlab search path | `savepath` writes into `matlabroot` | `startup.m` in your user work folder |
| macOS: `edfapi.framework` | yes, for `/Library/Frameworks` | `~/Library/Frameworks` + repointing the mex file |
| Windows: VC++ 2008 x64 redistributable | yes | none, ask IT once |
| Windows: VC++ 2015 redistributable | yes | usually already covered by Matlab itself |

#### Quick installation

1. **Toolbox** – put the repository into a folder you can write, e.g.
   `~/Documents/MATLAB/edf-converter`, and add the folder that *contains*
   `@Edf2Mat` to the search path – not `@Edf2Mat` itself. `savepath` does not
   work without admin rights; use a `startup.m` in your user work folder
   instead (`userpath` in Matlab tells you where that is):

   ```matlab
   % file: <userpath>/startup.m
   addpath('/Users/<you>/Documents/MATLAB/edf-converter');
   ```

2. **macOS only** – unpack the framework into your home directory and repoint
   the mex file once (run from the repository root, on a Mac so the symlinks
   survive):

   ```bash
   mkdir -p ~/Library/Frameworks

   # Apple Silicon – Intel: edfapi.framework-3.1.zip
   unzip -o "@Edf2Mat/private/edfapi.framework-4.2.762.0.zip" -d ~/Library/Frameworks

   install_name_tool -change \
       "/Library/Frameworks/edfapi.framework/Versions/A/edfapi" \
       "$HOME/Library/Frameworks/edfapi.framework/Versions/A/edfapi" \
       "@Edf2Mat/private/edfimporter.mexmaca64"
   ```

   On Intel the same command is needed for `edfimporter.mexmaci64` (and for
   `edfimporter_pre11.mexmaci64` on macOS < 11), but with
   `" @executable_path/../Frameworks/edfapi.framework/Versions/A/edfapi"` as
   the old path – the leading blank is part of the stored string.

   `install_name_tool` is part of the Xcode Command Line Tools
   (`xcode-select -p`). Without them there is no user-based option left, see
   the details below.

3. **Windows only** – ask your IT department once for the
   [Visual C++ Redistributable for Visual Studio 2008](https://www.microsoft.com/en-us/download/details.aspx?id=29)
   (x64). The rest is normally already in place.

4. **Check the installation** with the shipped example file:

   ```matlab
   edf = Edf2Mat('eyedata.edf');
   ```

##### Troubleshooting

| Message | Cause |
| --- | --- |
| `Library not loaded: /Library/Frameworks/edfapi.framework/...` | framework not found – step 2 missing or the mex file was replaced |
| `Invalid MEX-file ...: The specified module could not be found` (Windows) | missing VC++ redistributable, see step 3 |
| `Undefined function 'Edf2Mat'` | search path missing – step 1 |
| conversion works, but the old framework is still being used | an outdated copy of `@Edf2Mat` in the current folder wins over the search path – check with `which Edf2Mat` |
| `install_name_tool -change` reports nothing and the path is unchanged | the old path did not match exactly – mind the leading blank in the Intel variant |

#### TL;DR – technical details

##### Why the framework cannot simply be moved

Unpacking the framework into `~/Library/Frameworks` alone is *not* enough,
because none of the lookup mechanisms of `dyld` ever considers that folder:

- `edfimporter.mexmaca64` (Apple Silicon) stores the absolute path
  `/Library/Frameworks/edfapi.framework/Versions/A/edfapi`. Being a recent
  binary it gets no fallback search at all – `dyld` tries exactly that path and
  nothing else.
- `edfimporter.mexmaci64` / `edfimporter_pre11.mexmaci64` (Intel) store an
  `@executable_path` based path and only work today because old binaries still
  get the legacy fallback list `/Library/Frameworks:/System/Library/Frameworks`.

Repointing the mex file with `install_name_tool` is therefore the only way to
relocate the framework.

##### Why `DYLD_FALLBACK_FRAMEWORK_PATH` is not an option

Setting `DYLD_FALLBACK_FRAMEWORK_PATH` to `~/Library/Frameworks` does **not**
work: the `matlab` launcher is a shell script, and macOS removes all `DYLD_*`
variables as soon as a protected system binary such as `/bin/sh` is started.
They never arrive in the Matlab process – `getenv('DYLD_FALLBACK_FRAMEWORK_PATH')`
returns empty inside Matlab, while an ordinary variable passed the same way
survives. This is not a matter of the hardened runtime; the Matlab binary is
signed with `flags=0x0(none)`.

##### Code signature and durability of the rewrite

- `install_name_tool` refreshes the ad-hoc signature of the mex file itself, so
  no manual re-signing is needed – `codesign --verify` passes afterwards. Should
  it ever complain, repair the file with `codesign --force --sign - <mexfile>`.
- If the old path does not match exactly, `install_name_tool` changes nothing
  and reports no error.
- The rewrite lives in the binary and is lost whenever the mex file is replaced
  (`git pull`, fresh clone, `edfmex.build()`). Repeat step 2 after such an
  update.
- Check the result with:

  ```bash
  otool -L "@Edf2Mat/private/edfimporter.mexmaca64" | grep edfapi
  codesign --verify "@Edf2Mat/private/edfimporter.mexmaca64" && echo "signature ok"
  ```

##### Search path details

- `savepath` writes `pathdef.m` into the Matlab installation
  (`toolbox/local/pathdef.m`), and that file is read-only there – so the saved
  path is not available without admin rights.
- `userpath` can still be empty on a fresh installation. In that case set it
  once (a user preference, no admin rights needed) and restart Matlab:

  ```matlab
  userpath('/Users/<you>/Documents/MATLAB');
  ```

- Use an absolute path in `addpath`, not `fullfile(userpath, ...)`, so the line
  in `startup.m` also works while `userpath` is still unset.
- `matlab -batch` and other non-desktop sessions neither add the `userpath`
  folder nor run its `startup.m`. Scripts started that way have to call
  `addpath` themselves.

##### Windows runtime dependencies

- `edfimporter.mexw64` needs `VCRUNTIME140.dll` (VC++ 2015) and the Universal
  CRT (`api-ms-win-crt-*.dll`). The UCRT is part of Windows 10/11, and
  `VCRUNTIME140.dll` normally ships with Matlab, so this requirement is often
  already satisfied.
- `edfapi64.dll` needs `MSVCR90.dll`/`MSVCP90.dll` and requests them as the
  side-by-side assembly `Microsoft.VC90.CRT`. There is no per-user installer
  for it – the VC++ 2008 x64 redistributable has to be installed once with
  admin rights. This is the single request to hand to your IT department.
- The 32-bit chain (`edfimporter.mexw32`, `edfapi.dll`) additionally needs
  `MSVCR100.dll` (VC++ 2010) and is only relevant for very old 32-bit Matlab
  releases.

##### Verification status

The macOS steps were verified end-to-end with Matlab R2026a on Apple Silicon:
after repointing, `lsof` on the running Matlab process confirms that the
framework is loaded from `~/Library/Frameworks` and that the system-wide copy is
not touched. A quarantine flag on the unpacked framework (as set by browsers on
downloaded archives) does not prevent loading, since the framework is signed by
SR Research. The Windows dependencies were read from the import tables of the
shipped binaries, not verified on a Windows machine.

### Edf2Mat with Linux using wine

Edf2Mat on Linux with `wine` relies on the Windows executable `edf2asc.exe`. Check the additional comments about it:

- Matlab needs to be run without admin privileges (no sudo). This is a safety feature of `wine`.
- The user running the Matlab instance needs permission to read and write the edf files.

## How to use Edf2Mat – Toolbox

There is an `Example.m` script. Have a look at it.
Type `help` for help

```bash
  help Edf2Mat


    Edf2Mat is a converter to convert Eyetracker data files to
    Matlab file and perform some tasks on the data

    The new procedure uses code from SR-Research that returns all info of
    the edf and not just part of it. The new routine is based on the work
    of C. Kovach 2007 and is only for non-commercial use!



    Syntax: Edf2Mat(filename);
          Edf2Mat(filename, verbose);


   Inputs:
      filename:           must be of type *.edf
      useOldProcedure:    If you want to use the old procedure with
                          edf2asc.exe, you can set this argument to
                          true, default is false
      verbose:            logical, can be true or false, default is true.
                          If you want to suppress output to console,
                          verbose has to be false


  The Basic functionality is as follows:
  Convert Edf File

  edf1 = Edf2Mat('fMRI_Results_sub_025_270712EYE25r1.edf');
```

Calling the Edf2Mat with a filename converts the given edf file to a Matlab structure, which will be available in the Matlab workspace.
In order to save the produced structure to a matfile, just call “save(edf1)”, whereas edf1 is the variable assign when calling the Edf2Mat Class.

### Plot

The Edf2Mat class has its own plot functionality to plot the content. It’s more for a fast forward validation of data than actually the way you should plot your data.

```matlab
plot(edf1);
```

![alt text](./images/plotEdf.png "Example of the function plot(edf1)")

#### Plot last 2000 Elements

In order to plot eye movement only in a specified time range, the Matlab builitin plot command could be used as following:

```matlab
figure();
plot(edf1.Samples.posX(end - 2000:end), edf1.Samples.posY(end - 2000:end), 'o');
```

#### Plot the pupil size

To simply plot the pupil size for a given time window, the pupil size array can be accessed as stated in the next line.

```matlab
figure();
plot(edf1.Samples.pa(2, end - 500:end));
```

#### Plot just the heatmap of the eye movement

```matlab
plotHeatmap(edf1);
```

![alt text](./images/heatmapExample.png "Example of the function plotHeatmap(edf1)")

## Bibliography

Kovach, C. (2011, 01 12). SR Research. Retrieved from SR Research Support: [https://www.sr-support.com/showthread.php?255-Import-of-EDF-file-into-Matlab&p=6781#post6781](https://www.sr-support.com/showthread.php?255-Import-of-EDF-file-into-Matlab&p=6781#post6781)

## Development

### Create Mex files

Run the following command to create the mex file for the current system architecture.

```matlab
% for debug add "true" as argument
edfmex.build()
```
