# What is this?

This is a collection of instructions to install and configure the tools I use for development/data science/physics ...
I use WSL on Windows 11 to use things like Python, Latex, Git, C++, ...
Most of this comes from the Pep Toolbox-Workshop (https://toolbox.pep-dortmund.org/) other things come from personal experience. 
This collection is mainly for myself, so that I have everything I need at one place, but I guess it could be useful for other people too. 
And most of it should work on native Ubuntu or Linux too.

# Installation Instructions (mainly on Windows 11 + WSL)

Preferably use Windows Terminal and Visual Studio Code for everything.

## WSL and Ubuntu

- Open Windows Terminal (PowerShell) as an Administrator
- enter `wsl --install -d Ubuntu`
- after successfull completion, restart your computer
- enter a username and password for the Ubuntu login
- Update the packages via `sudo apt update && sudo apt upgrade`
- Optional: Set Ubuntu as the default shell in Windows Terminal

For more info:
https://docs.microsoft.com/de-de/windows/wsl/install 
If you are having Connection issues and using Avast: https://github.com/MicrosoftDocs/WSL/issues/481

### Useful/Essential Linux Tools

Install each via `sudo apt install <Name>`

-> `sudo apt install git make curl`

- make
- curl
- git

## Visual Studio Code

Download VSCode from here: https://code.visualstudio.com/ 
Or if you are worried about privacy, open source, tracking... use VSCodium from here: https://vscodium.com/ 
And then install it.

It's best to always open VS Code via the Ubuntu shell: `code <Path>` (or `codium <Path>`)
-> this will run VS Code with "Remote for WSL" and it will use WSL for Python etc.

Recommended Settings (in VSCode->File->Preferences->Settings):
- English as Language
- Word Wrap as "always" 

Recommended VS Code Extensions:
- Remote - WSL
- Rainbow CSV
- vscode-pdf
- Python
- C/C++
- LaTeX Workshop (disable auto compile!)
  - in VSCode->File->Preferences->Settings:  
  - search for "latex auto build clean and retry" and remove the checkmark
  - search for "latex auto build run" and set it to "never"
- Visual Studio Live Share


## Python (Scientific Python via Anaconda)

- Go to https://www.anaconda.com/products/individual
- Copy the link adress of the Linux(!) "64-Bit (x86) Installer"
- Download the Installer: `wget -O ~/Anaconda-Installer.sh <Copied-Link-Adress>`
  - For example: `wget -O ~/Anaconda-Installer.sh https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh`
- Install Anaconda via `bash ~/Anaconda-Installer.sh -b -p ~/.local/anaconda3`
  - `-b` is for batch-mode so that the `~/.bashrc` does not get cluttered
  - `-p` is to specify where Anaconda should be installed to
- Remove the Installer: `rm ~/Anaconda-Installer.sh`
- To automatically activate Anaconda (i.e. Python) add the following lines at the end of you `~/.bashrc` file:
  ```
  # Python/Anaconda:
  . "$HOME/.local/anaconda3/etc/profile.d/conda.sh"
  conda activate
  ```
  - You could for example open it with `code ~/.bashrc`

### Useful Python Packages not in Anaconda

- Uncertainties: `pip install uncertainties`

## Latex (with TexLive)

- Warning: This installation takes a long time because TexLive is a large package (~5GB)
- Download and extract the TexLive Installer: `cd ~ && curl -L http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz | tar xz`
- Install TexLive: `TEXLIVE_INSTALL_PREFIX=$HOME/.local/texlive ~/install-tl-*/install-tl`
  - Start the Installation using the option `I`
- Remove the installer: `rm -rf ~/install-tl-*`
- Add the following to your `~/.bashrc` file: 
  ```
  # LaTeX/TexLive:
  export PATH="$HOME/.local/texlive/2021/bin/x86_64-linux:$PATH"
  ```
  - You may need to adjust the path (i.e. the year)
- restart your terminal (or use `source ~/.bashrc`)
- adjust the config: `tlmgr option autobackup -- -1` and `tlmgr option repository http://mirror.ctan.org/systems/texlive/tlnet`

## C++ 

- see: https://code.visualstudio.com/docs/cpp/config-wsl 
- or better use a Makefile

- to install Eigen:
  - `sudo apt install libeigen3-dev`
  - now it could be included with `#include <eigen3/Eigen/Dense>`
  - if you want to include it with `#include <Eigen/Dense>` you need to create a symbolical link: `sudo ln -s /usr/include/eigen3/Eigen /usr/include/Eigen`

## Git configuration (for GitHub)

- Git has to be installed: `sudo apt install git`
- recommended configuration (as commands):
  ```
  git config --global user.name "Max Mustermann"
  git config --global user.email "max.mustermann@email-provider.com"
  git config --global rebase.stat true
  git config --global merge.conflictstyle diff3
  ```
- You need to use a SSH key for GitHub (https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh):
  - Create SSH-Key: `ssh-keygen -t ed25519 -C "max.mustermann@email-provider.com"`
  - Use the default path
  - Enter the password you want to use (optional)
  - Copy the public key: `cat ~/.ssh/id_ed25519.pub | clip.exe`
  - Go to GitHub (https://github.com/settings/keys) and add the key (the title is not important)
  - now you can clone the GitHub repositories using the SSH URLs (for example `git clone git@github.com:NicoJG/WSL_Instructions.git`)

## Useful Windows Configurations

in the Windows Explorer, configure:
- show file extensions
- show hidden files/folders


# WSL Bug fixes / workarounds

## Graphical Interfaces don't work (for example matplotlib)

Should not be necessary for Windows 11 but on Windows 10 it was necessary.
You need a XServer.

- Install VcXsrv (https://sourceforge.net/projects/vcxsrv/)
- in Windows search for XLaunch and choose the following configurations:
  -  Multiple Windows, Start no client, all extra settings, save configuration
  -  You may (or must perhaps) reject the firewall permissions that should pop up
- `mkdir /tmp/vcxsrv` 
- Add the following to your `~/.bashrc`:
  ```
  # XServer/VcXsrv:
  export DISPLAY=$(route.exe print | grep 0.0.0.0 | head -1 | awk '{print $4}'):0.0
  export XDG_RUNTIME_DIR="/tmp/vcxsrv"
  export LIBGL_ALWAYS_INDIRECT=1
  ```
    - if this does not work try `export DISPLAY='grep -oP "(?<=nameserver ).+" /etc/resolv.conf':0.0` or `export DISPLAY=localhost:0.0`

  - Optional: to be able to start the XServer from the command line `check_xsrv`, add the following to your `~/.bashrc`:
  ```
  # open vcxsrv if it is not running
  open_vcxsrv() {
      if ! cmd.exe /c tasklist | grep --quiet vcxsrv; then
          cmd.exe /c "C:\Users\Nico\Documents\config.xlaunch"
      fi
  }
  alias check_xsrv=open_vcxsrv
  ```

## Jupyter Lab/Jupyter Notebook on WSL

At the moment Jupyter is buggy and won't open right. 
For more information: https://stackoverflow.com/a/65133953 
For Jupyter Notebook just replace every "Lab" with a "Notebook"

- Generate the config file `jupyter server --generate-config`
- Add the following line to `~/.jupyter/jupyter_server_config.py`: `c.ServerApp.use_redirect_file = False`
  - if this does not work, try the following (see https://github.com/jupyterlab/jupyterlab/issues/10413):
  - Generate the config file `jupyter lab --generate-config`
  - Add the following line to `~/.jupyter/jupyter_lab_config.py`: `c.LabApp.open_browser = False`

# Terminal Customizations
 
The default terminal (Windows Terminal + Bash) is fine, but you can do so much more. 
I like to use zsh with oh-my-zsh (https://github.com/ohmyzsh/ohmyzsh). 
Once it is configured properly it looks amazing and has man convenient tools.

## Customize your terminal (Windows Terminal)

To look good, you first have to customize Windows Terminal
 
- Choose a color scheme from https://windowsterminalthemes.dev/ 
 - I like "Solarized Dark Higher Contrast" (https://windowsterminalthemes.dev/?theme=Solarized%20Dark%20Higher%20Contrast)
 - click on "Get Theme"
 - in Windows Terminal->Settings click "Open JSON file"
 - paste the color scheme after `"schemes": [` and add a `,` after the closing `}` (then save)
  - potentially change "background" to black (`#000000`) 
 - make the chosen color scheme the default for Ubuntu (in Settings->Ubuntu->Appearance)
- choose a background image and make it ~30% opaque (in Settings->Ubuntu->Appearance) (for example https://en.wikipedia.org/wiki/Helix_Nebula#/media/File:NGC7293_(2004).jpg)
- Change the "Bell notification style" to not audible (in Settings->Ubuntu->Advanced)
- Install a "Nerd Font" from https://www.nerdfonts.com/font-downloads 
 - I like "DejaVuSansMono NF" (`DejaVu Sans Mono Nerd Font Complete Windows Compatible.ttf`)
 - Set the Font for the Ubuntu profile (in Settings->Ubuntu->Appearance)

 
## Installing Oh My Zsh
 
- Install Zsh: `sudo apt install zsh`
- Make Zsh your default shell: `chsh -s $(which zsh)`
- Install OhMyZsh: `sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
- restart the shell (by using `exec zsh`)
- you need to copy all customizations you did to the `~/.bashrc` to `~/.zshrc`!

# Styling Zsh with Powerlevel10k

Powerlevel10k is not necessary, you could choose a theme shipped with Oh My Zsh (https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)
But I really like Powerlevel10k.

- Install Powerlevel10k: 
  - `git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k` 
  - Set `ZSH_THEME="powerlevel10k/powerlevel10k"` in `~/.zshrc`
  - `exec zsh`
  - Follow the configuration wizard (`p10k configure`) which should show up (You should have already installed a Nerd Font!)
  - (https://github.com/romkatv/powerlevel10k#oh-my-zsh)
- Powerlevel10k is fully customizable (look into `~/.p10k.zsh`)
  - a few suggestions to look into:
    ```
    POWERLEVEL9K_SHORTEN_DIR_LENGTH
    POWERLEVEL9K_LEFT_PROMPT_ELEMENTS
    POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS
    POWERLEVEL9K_TIME_FORMAT
    ```
- Choose plugins for Zsh (in `~/.zshrc`) (https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins or somewhere else via google)
  - a few suggestions to look into:
    ```
    git
    colorize
    command-not-found 
    conda-zsh-completion
    vscode
    lol
    zsh-interactive-cd
    ```
  - recommended plugins that are not included in Oh My Zsh (don't forget so change the `~/.zshrc`):
    - zsh-autosuggestions: `git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions`
    - zsh-syntax-highlighting: `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting`
- Powerlevel10k is fully customizable, a few recommendations (added to `~/zshrc`):
```

```


# Updating

## Powerlevel10k

`git -C ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k pull`

Powerlevel10k or 9k
