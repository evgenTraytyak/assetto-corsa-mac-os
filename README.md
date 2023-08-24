# (WIP) Tutorial on how to run Assetto Corsa (with Logitech G29) on Mac OS Sonoma using Wine and Game Porting Toolkit

## Requirements:
- You must run the latest Beta of macOS Sonoma (Beta 6 and above)!
- Device with Apple Silicon chip (M1, M1 Pro, M1 Max, M2, M2 Pro, M2 Max, M2 Ultra)

We will install the x86 version Homebrew in order to be able to use Apple's modified version of `Wine` and to install the Windows version Steam.

We will make sure that our existing environment (and the Apple silicon version of Homebrew we need for 'serious' work) remains undisturbed.

## Step-by-step installation:

- Open a terminal (or iTerm2)
- Make sure that rosetta is installed by entering:
```
softwareupdate --install-rosetta --agree-to-license
```
Now your Mac is able to execute `x86_64` code. This is the basis for all the following installations.

- Now switch to a `x86` shell by entering:

```bash
arch -x86_64 zsh 
```

Type `arch` again, to make sure that you are using Intel. It should **not** show `arm64`, but `i386`

Now, from a terminal that uses `x86_64` arch, install homebrew for x86:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

This will install the Intel x86 version of homebrew to `/usr/local`. If you already installed homebrew
for Apple Silicon, then this version resides in `/opt/homebrew`. This guide will assume that the Apple Silicon
homebrew is your important version, and will remain the default.

In order not to mess up the two homebrew versions, we should create an alias for the Intel homebrew:

```bash
alias brew86=/usr/local/bin/brew
```

Note: if you are following Apple's readme, make sure to replace all instances of `brew` in Apple's doc with `brew86` from now on.

Now we install Apple's homebrew tap:

```bash
brew86 tap apple/apple http://github.com/apple/homebrew-apple
brew86 -v install apple/apple/game-porting-toolkit
```

Now create a directory that will contain all the Windows stuff, Steam, and any games you download. Here we'll use '~/Win10',
which is our `wine` prefix (Apple uses the example of `my-game-prefix`, we'll go with `Win10`):

```bash
mkdir ~/Win10
WINEPREFIX=~/Win10 `brew86 --prefix game-porting-toolkit`/bin/wine64 winecfg
```

Note: we use the alias `brew86` we defined above.

This should start the `winecfg` program, a small setup for our `wine` environment. Select `Windows 10`, and `Apply`, `OK`.

- Now open the Game Porting Toolkit that you downloaded above.

<img src="https://github.com/domschl/WinSteamOnMac/blob/main/Resources/gameporting.png" width="60%">

From your Terminal, it should be available at:

```bash
ls /Volumes/Game\ Porting\ Toolkit-1.0/
```

Make sure you see the files of the toolkit and then:

```bash
ditto /Volumes/Game\ Porting\ Toolkit-1.0/lib/ `brew86 --prefix game-porting-toolkit`/lib/
```

This copies the required Apple libraries into your `wine` installation.

Now copy the Windows setup of Steam (we assume you downloaded into the default ~/Downloads` folder) into your new Windows drive:

```bash
cp ~/Downloads/SteamSetup.exe ~/Win10/drive_c
```

Now enter your `Win10` directory and note the full path of this directory:

```bash
cd ~/Win10
pwd
```

It should say something like `/Users/you-user-name/Win10`. This is your `wine`-prefix, which you need to use below (instead of `your-user-name` placeholder).




## Part about `winetricks`

Let's install [`winetricks`](https://formulae.brew.sh/formula/winetricks) to configure the necessary libraries for Assetto Corsa.

```bash
brew install winetricks
```

After installation we should run `winetricks` with:

```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/etraytyak/Win10 WINE=/usr/local/opt/game-porting-toolkit/bin/wine64 DISPLAY=foobar winetricks --gui=zenity
```


Check that wine switched back to Windows 10 from Windows2003 after installation...


Apple provides a few scripts to start games, which we will not use: `gameportingtoolkit`, `gameportingtoolkit-no-esync`, and `gameportingtoolkit-no-hud`. Open those script with your favorite editor to learn how to configure `wine`. The `Readme.rtf` or the Game Porting Toolkit explains the differences.

We use what we learned from inspecting the scripts and directly start the Steam setup with:

```bash
cd ~/Win10/drive_c
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/your-user-name/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 SteamSetup.exe
```

Make sure to replace `your-user-name` above with the output of the `pwd` command.

This should now start the Steam setup process. Be patient.

For me, it stayed with a black windows with no content for quite some time. After waiting patiently, I aborted the setup (be sure not to abort too early!), and started Steam directly:

```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEPREFIX=/Users/your-user-name/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files\ \(x86\)/Steam/Steam.exe
```

This again took some time, but then the Steam logo appeared!

Login, and start installing Assetto Corsa from your library.





## Part about Logitech software and drivers

How to run Logitech G Hub (need to use some extra env variables and flags cause it's a Electron app)
```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEDLLOVERRIDES=libglesv2.dll=d WINEPREFIX=/Users/etraytyak/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files/LGHUB/lghub.exe --disable-gpu --no-sandbox --single-process
```

If it crashes with error like this:
```
0184:err:dbghelp_msc:codeview_process_info Unknown CODEVIEW signature 00000000 in module L"dxgi"
 1: 0000000142E10B06 class v8::MaybeLocal<class v8::Object> __cdecl node::Buffer::New(class v8::Isolate
* __ptr64,char * __ptr64,unsigned __int64,void (__cdecl*)(char * __ptr64,void * __ptr64),void * __ptr64)
+45414
 2: 0000000142E108DB class v8::MaybeLocal<class v8::Object> __cdecl node::Buffer::New(class v8::Isolate
* __ptr64,char * __ptr64,unsigned __int64,void (__cdecl*)(char * __ptr64,void * __ptr64),void * __ptr64)
+44859
```
Try to remove `--single-process` flag, only:

```bash
MTL_HUD_ENABLED=0 WINEESYNC=1 WINEDLLOVERRIDES=libglesv2.dll=d WINEPREFIX=/Users/etraytyak/Win10 /usr/local/opt/game-porting-toolkit/bin/wine64 Program\ Files/LGHUB/lghub.exe --disable-gpu --no-sandbox
```

