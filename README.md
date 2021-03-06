![](https://raw.githubusercontent.com/fastrizwaan/flatpak-wine32/main/Screenshots/right-click.png)

Main branch does not include mono and gecko msi files to save space.

winetricks when asked to install (as described below in wintricks section) will prompt for installing wine-mono, you should install it then, to install dotnet35, xna31 etc.

# Run wine 32bit apps using flatpak in Centos 7 or any distro with flatpak support:
```
wget https://github.com/fastrizwaan/flatpak-wine32/raw/main/io.github.flatpak-wine32.flatpak
sha256sum io.github.flatpak-wine32.flatpak ; #check sum below
```

`209878a1a9b58fa147d49e639499cb0f1fcde398eafbbbfe98af97dceb8d3397  io.github.flatpak-wine32.flatpak`

```
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo -y
# Install required runtime, see bottom for nvidia GPU
sudo flatpak install                                               \
org.freedesktop.Platform/x86_64/20.08                              \
org.freedesktop.Platform.GL.default/x86_64/20.08                   \
org.freedesktop.Platform.VAAPI.Intel/x86_64/20.08                  \
org.freedesktop.Platform.openh264/x86_64/2.0                       \
runtime/org.freedesktop.Platform.Compat.i386/x86_64/20.08          \
org.freedesktop.Platform.GL32.default/x86_64/20.08 -y

flatpak --user install io.github.flatpak-wine32.flatpak -y

flatpak run io.github.flatpak-wine32 game.exe ; #replace game.exe with your exe
```

#### Nvidia gpu 32 bit drivers need to be installed, if game complains about D3D or Opengl or GL install nvidia drivers see bottom for Nvidia Section
```
NVERSION=$(nvidia-settings -q all |grep OpenGLVersion|grep NVIDIA|sed 's/.*NVIDIA \(.*\) /nvidia-\1/g'|sed 's/\./-/g')

sudo flatpak install flathub org.freedesktop.Platform.GL32.$NVERSION -y

```
By Default, all programs are installed in ~/.wine, use ` WINEPREFIX=~/.mywine flatpak run io.github.flatpak-wine32 Setup.exe ` to install in ` ~/.mywine ` directory.

# winetricks
![](https://raw.githubusercontent.com/fastrizwaan/flatpak-wine32/main/Screenshots/winetricks_availabe.png)

### Three ways to run wintricks in flatpak-wine32
#### 1. run from flatpak
```
flatpak run --command=bash io.github.flatpak-wine32
winetricks <whatever>
```
#### 2. run directly from shell
```
flatpak run --command=winetricks io.github.flatpak-wine32 ; #for winetricks GUI

flatpak run --command=winetricks io.github.flatpak-wine32 d3dx9; #for winetricks directx
flatpak run --command=winetricks io.github.flatpak-wine32 xna31; #for winetricks xna framework
flatpak run --command=winetricks io.github.flatpak-wine32 xinput; #for winetricks xinput xbox joystick support
flatpak run --command=winetricks io.github.flatpak-wine32 vcrun2008; #for winetricks vcruntime

```
#### 3. by aliasing winetricks
```
alias winetricks="flatpak run --command=winetricks io.github.flatpak-wine32"
echo 'alias winetricks="flatpak run --command=winetricks io.github.flatpak-wine32"' >> ~/.bash_aliases 
winetricks xinput ;#now flaptak winetricks is run
```
# flatpak-wine32
wine build with runtime 20.08 i386, provides wine to Centos like distros

a big thanks to https://github.com/Pobega for making his fightcade flatpak manifest with i386 compat

## Build flatpak-wine32.flatpak on your own

```
git clone https://github.com/fastrizwaan/flatpak-wine32.git
cd flatpak-wine32;

# Install runtime and Sdk to build flatpak
sudo flatpak install \
runtime/org.freedesktop.Sdk/x86_64/20.08                           \
org.freedesktop.Platform/x86_64/20.08                              \
runtime/org.freedesktop.Sdk.Compat.i386/x86_64/20.08               \
runtime/org.freedesktop.Sdk.Extension.toolchain-i386/x86_64/20.08  \
org.freedesktop.Platform.GL.default/x86_64/20.08                   \
org.freedesktop.Platform.VAAPI.Intel/x86_64/20.08                  \
org.freedesktop.Platform.openh264/x86_64/2.0                       \
runtime/org.freedesktop.Platform.Compat.i386/x86_64/20.08          \
org.freedesktop.Platform.GL32.default/x86_64/20.08

flatpak-builder --force-clean build-dir/ io.github.flatpak-wine32.yml
flatpak-builder --install --user --force-clean build-dir/ io.github.flatpak-wine32.yml 
```

Download example installer:
` wget https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v7.9.3/npp.7.9.3.Installer.exe `

## Usage (install/run windows exe software)
### 1. From inside Flatpak:
```
flatpak run --command=bash io.github.flatpak-wine32
which wine
wine ~/Downloads/npp.7.9.3.Installer.exe
```
### install notepadpp 32 bit installer from inside flatpak runtime
`WINEPREFIX=~/.wine-npp wine ~/Downloads/npp.7.9.3.Installer.exe`


### 2. As commandline Argument to Flatpak 

` flatpak run io.github.flatpak-wine32  ~/Downloads/npp.7.9.3.Installer.exe `

### to install in a separate wine prefix (here replace .TTT with your own)

` WINEPREFIX=~/.TTT flatpak run io.github.flatpak-wine32 ~/Downloads/npp.7.9.3.Installer.exe `

#### 3. Alias it for convenience:


```
alias wine32="flatpak run io.github.flatpak-wine32"
alias winetricks="flatpak run --command=winetricks io.github.flatpak-wine32"
echo 'alias wine32="flatpak run io.github.flatpak-wine32"' >> ~/.bash_aliases
echo 'alias winetricks="flatpak run --command=winetricks io.github.flatpak-wine32"' >> ~/.bash_aliases 

```
### with the alias command

` WINEPREFIX=~/.notepadpp wine32 ~/Downloads/npp.7.9.3.Installer.exe `


## Create flatpak

```
flatpak-builder --repo="repo" --force-clean build-dir/ io.github.flatpak-wine32.yml 
flatpak --user remote-add --no-gpg-verify "io.github.flatpak-wine32" "repo"
flatpak build-bundle "repo" "io.github.flatpak-wine32.flatpak" io.github.flatpak-wine32 stable --runtime-repo="https://flathub.org/repo/flathub.flatpakrepo"
```

## Install flatpak
` flatpak install --user io.github.flatpak-wine32.flatpak `


## For Nvidia Users, install your version of Nvidia drivers
check you nvidia drivers version from nvidia-settings app and install accordingly
```
flatpak remote-ls flathub | grep nvidia

#flatpak install flathub org.freedesktop.Platform.GL32.nvidia-MAJORVERSION-MINORVERSION
sudo flatpak install flathub org.freedesktop.Platform.GL32.nvidia-460-39
#or
sudo flatpak install flathub org.freedesktop.Platform.GL32
```

### To stop / kill a flapak running program, if it hangs; like killall -KILL <program name>
` flatpak kill io.github.flatpak-wine32`
