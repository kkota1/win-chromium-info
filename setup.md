# modified windows setup
Due to bloat/omissions in official

Build takes forever the first time and then will speed up for minor changes due to autoninja

## System requirements

* A 64-bit Intel machine with at least 8GB of RAM. More than 16GB is highly
  recommended.
* At least 100GB of free disk space on an NTFS-formatted hard drive. FAT32
  might not work, as some of the Git packfiles are larger than 4GB.
  ⚠️ kkota1 note: I had git lfs installed and fat32 (default windows) appears worked fine. https://git-lfs.com/
* An appropriate version of Visual Studio, as described below.
* Windows 10 or newer.

## Setting up Windows

### Visual Studio

[Visual Studio 2022](https://learn.microsoft.com/en-us/visualstudio/releases/2022/release-notes) (>=17.0.0)
is preferred. 

Add the "Desktop development with C++" component and the "MFC/ATL support" sub-components during install.

- Add 10.0.20348.0 Windows 10 SDK during install. Complete VS22 installation

Control Panel → Programs → Programs and Features → Select the "Windows
Software Development Kit" → Change → Change → Check "Debugging Tools For
Windows" → Change.

## Install `depot_tools`

Download the [depot_tools bundle](https://storage.googleapis.com/chrome-infra/depot_tools.zip)
and extract it somewhere (eg: C:\src\depot_tools). Use “Extract all…” from the context menu

Add depot_tools directory to the start of your system-level environment variables PATH (must be ahead of any installs of Python.).

Also, add a DEPOT_TOOLS_WIN_TOOLCHAIN environment variable in the same way, and set
it to 0. This tells depot_tools to use your locally installed version of Visual
Studio (by default, depot_tools will try to use a google-internal version).


`set vs2022_install=C:\Program Files\Microsoft Visual Studio\2022\Community` or wherever it is

From a cmd.exe shell, run:

```shell
$ gclient
```

## Check python install

After running gclient open a command prompt and type `where python` and
confirm that the depot_tools `python.bat` comes ahead of any copies of
python.exe. Failing to ensure this can lead to overbuilding when
using gn - see [crbug.com/611087](https://crbug.com/611087).

Open 'App execution aliases' section of Control Panel and unticking the boxes next to anything python

## Get the code

Configure Git if you haven't:

```shell
$ git config --global user.name "My Name"
$ git config --global user.email "my-name@chromium.org"
$ git config --global core.autocrlf false
$ git config --global core.filemode false
$ git config --global branch.autosetuprebase always
```


```shell
$ mkdir chromium && cd chromium
```

```shell
$ fetch chromium --no-history
```

You should configure your PC so that it doesn't sleep or hibernate during the fetch or else errors may occur. If errors occur while fetching sub-repos then you can start over, or you may be able to correct them by going to the chromium/src directory and running this command:

```shell
$ gclient sync
```

The rest of the instructions should be followed in src

```shell
$ cd src
```

*Optional*: You can also [install API
keys](https://www.chromium.org/developers/how-tos/api-keys) if you want your
build to talk to some Google services, but this is not necessary for most
development and testing purposes.

## Setting up the build

Chromium uses [Ninja](https://ninja-build.org) as its main build tool along with
a tool called [GN](https://gn.googlesource.com/gn/+/main/docs/quick_start.md)
to generate `.ninja` files. You can create any number of *build directories*
with different configurations. To create a build directory:

```shell
$ gn gen out/Default
```

### Faster builds

```shell
gn gen out/Default --args="is_component_build = true is_debug = true symbol_level = 1 blink_symbol_level = 0 v8_symbol_level = 0"
```

## Build Chromium

Build Chromium (the "chrome" target) with Ninja using the command:

```shell
$ autoninja -C out\Default chrome
```

`autoninja` is a wrapper that automatically provides optimal values for the
arguments passed to `ninja`.

You can get a list of all of the other build targets from GN by running
`gn ls out/Default` from the command line. To compile one, pass to Ninja
the GN label with no preceding "//" (so for `//chrome/test:unit_tests`
use ninja -C out/Default chrome/test:unit_tests`).

## Run Chromium

Once it is built, you can simply run the browser:

```shell
$ out\Default\chrome.exe
```

(The ".exe" suffix in the command is actually optional).

### Editing and Debugging With the Visual Studio IDE

You can use the Visual Studio IDE to edit and debug Chrome, with or without
Intellisense support.

#### Using Visual Studio Intellisense

If you want to use Visual Studio Intellisense when developing Chromium, use the
`--ide` command line argument to `gn gen` when you generate your output
directory (as described on the [get the code](https://dev.chromium.org/developers/how-tos/get-the-code)
page):

```shell
$ gn gen --ide=vs out\Default
$ devenv out\Default\all.sln
```

GN will produce a file `all.sln` in your build directory. It will internally
use Ninja to compile while still allowing most IDE functions to work (there is
no native Visual Studio compilation mode). If you manually run "gen" again you
will need to resupply this argument, but normally GN will keep the build and
IDE files up to date automatically when you build.

The generated solution will contain several thousand projects and will be very
slow to load. Use the `--filters` argument to restrict generating project files
for only the code you're interested in. Although this will also limit what
files appear in the project explorer, debugging will still work and you can
set breakpoints in files that you open manually. A minimal solution that will
let you compile and run Chrome in the IDE but will not show any source files
is:

```
$ gn gen --ide=vs --filters=//chrome --no-deps out\Default
```

You can selectively add other directories you care about to the filter like so:
`--filters=//chrome;//third_party/WebKit/*;//gpu/*`.

There are other options for controlling how the solution is generated, run `gn
help gen` for the current documentation.
