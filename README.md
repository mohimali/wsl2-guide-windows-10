# WSL2 Guide on windows 10

WSL2 will enable you to use the linux sub system on Windows 10. This should also work on Windows 11.

# Notes before starting

If you ever get firewall notifications for either docker, ubuntu, any other applications that you install, it's important to tick both of these boxes and allow access

![](images/firewall.png)

Its important you learn the following as we will be referring  to these terms often.
 
- WSL2 stands for Windows Subsystem for linux. Always use version 2. It lets you use linux operating systems such as ubuntu as close to native as possible. https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux
- Display Servers enable you to run GUI apps from a linux shell. https://en.wikipedia.org/wiki/Windowing_system#Display_server
- Windows 11 Wayland is one implementation of a Display Server . It comes packaged with Windows 11. Windows 10 does not, therefore the `DISPLAY` export isn't needed but you can override this if you wish to use your own X Server if you are on Windows 11.
- Its recommended you run intellij from within wsl2 to get a more native linux experience. It also indexes faster. 
- Use ~/.bashrc if your using bash or ~/.zshrc for zsh users.
- Modern operating systems use 64 bit apps to make use of more then 4gb of ram. Remember x86 is the same as 32bit. Try and use 64bit applications where possible. In a windows title bar if its blank it is usually 64bit(if you are running a 64bit Windows 10), ensure it doesnt say x86 especially for powershell.

# Pre-requisites 
- WSL 2 is only available in Windows 10, Version 1903, Build 18362 or higher.

## Install WSL2
In a powershell admin (64 bit not x86) run the following. First command will install chocolatey, a package manager. 
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Install WSL and the recommended Ubuntu distro. Once ubuntu-2004 is installed run it from your start menu and follow the instructions.

```
wsl --install
choco install wsl-ubuntu-2004
```

Microsoft terminal is very useful and nice to use
```
choco install microsoft-windows-terminal
```
Once ubuntu-2004 is installed run it from your start menu and follow the instructions.
You can now use the windows terminal from the start menu. All below commands unless explicitly mentioned should use the windows terminal running the ubuntu20.04 distro. 

## Install sdk man a java package manager
```
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk list java
sdk install java 11.0.11.hs-adpt
java --version
```

## Install homebrew as it's useful for installing things like maven. Homebrew is not just for mac!
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install maven
mvn --version
```

## Install docker 
Note this will be updated later to use the free version of docker due to license restrictions. For now, you can use docker for windows
https://docs.docker.com/desktop/windows/install/

Tick all boxes for any firewall windows for this application

![](images/docker-wsl2-enable.png)
![](images/docker-ubuntu-enable.png)


Restart your windows terminal and test docker via the following commands
```
docker run hello-world
docker ps
```

## WSLG Apps
We can use a display server to run gui apps. For testing purposes we will be using gedit and Nautilus(file explorer)

Install your display server from https://sourceforge.net/projects/vcxsrv/

Create a windows shortcut to point to your display server. You can use the one inside the scripts folder as its set up with the below arguments. By running this, it should run your display server in background. Tick all boxes for any firewall windows for this application

![](images/vc-shortcut.png)

```
"C:\Program Files\VcXsrv\vcxsrv.exe" :0 -ac -terminate -lesspointer -multiwindow -clipboard -wgl -swcursor -dpi 144
```

Install gui apps and run them. You can use Ctrl+Z/Ctrl+C to exit them within terminal if they do not shut down properly. The DISPLAY export lets you 

Store the below in your .bashrc or .zshrc file. This will point to the display server(VcXsrv)
```
export DISPLAY="`grep nameserver /etc/resolv.conf | sed 's/nameserver //'`:0"
```

```
sudo apt install nautilus
sudo apt install gedit
nautilus
gedit
```

You can also install intellij with the following commands.
```
cd
mkdir scripts
```

For community intellij users
```
wget https://download.jetbrains.com/idea/ideaIC-2021.2.3.tar.gz

tar -zxf ideaIU-2021.2.2.tar.gz
```

For intellij ultimate users
```
wget https://download.jetbrains.com/idea/ideaIU-2021.2.3.tar.gz
tar -zxf ideaIU-2021.2.3.tar.gz
```
Ensure you add your libraries on your ubuntu shell, for things like fonts etc
```
sudo apt install libcups2 libpangocairo-1.0-0 libatk-adaptor libxss1 libnss3 libxcb-keysyms1 x11-apps libgbm1
```

I usually create an alias to run this via one of the below commands. Use the correct path. I usually store this in either the `.bashrc`  or `.zshrc`
```
export LIBGL_ALWAYS_INDIRECT=1
alias idea='/home/mohim/scripts/idea-IU-212.5284.40/bin/idea.sh $1 > /dev/null 2>&1 &'
```

Then you can run `idea` in your ubuntu terminal

## Integration with VS Code
VS Code probably has the best integration with wsl2. 
- Install VS Code - https://code.visualstudio.com/
- Install the plugin for WSL2
![](images/vs-code-wsl2-plugin.png)

Then if you restart your pc/terminal you can run `code dir` within the windows terminal and it will open up your folders/files in vs code code.

## Running the Display server as a task when you log in
If you don't want to run the display server every time you start your pc you can setup a basic task

Open up task scheduler from the start menu. Add a basic task with these instructions and arguments. This will run every time you log in. Its import to untick the boxes in red even they are greyed out.

![](images/task-scheduler-trigger.png)

![](images/task-scheduler-trigger.png)

![](images/task-scheduler-untick-these-boxes.png)

## Open windows explorer via terminal

Add the following alias to your .bashrc or .zshrc. Then you can use `open dir` to open a specific directory in windows file explorer
```
alias open="explorer.exe"
```

# Troubleshooting
This part will cover any issues you have


## WSL2 Not starting on bootup
If you have issues running wsl2 on bootup then you can disable fastboot. https://stephenreescarter.net/wsl2-network-issues-and-win-10-fast-start-up

## WSL2 Internet issues with vpn 
Use CMD or postman/internet browser for vpn related requests. For now the vpn isn't compatible with the ubuntu terminal. There are alternatives that you can do by editing the /etc/resolv.conf file but this only works with some corporate vpns.  

## Intellij not starting
- Check your $DISPLAY variable to ensure it points to the WSL2 
- Check you are running VcXsrv. The shortcut is inside the scripts folder. I hardly advise you set it up in a task so it auto starts on login.

# Intellij has missing icons and fonts

Go to help > Edit Custom VM options on intellij and add the following line
```
-Dremote.x11.workaround=false

```

## Intellij looks blurry
Only do these steps if your intelliJ looks blurry. I had the same issue on a 4k monitor.
1. Right click the following files and go to properties -> compatibility > Change high DPI Scaling > High DPI Scaling override Application
```
C:\Program Files\VcXsrv\vcxsrv.exe
C:\Program Files\VcXsrv\xlaunch.exe
```

Then you can add the following to your .bashrc or .zshrc file. This profile works for me on a 4k monitor with 175% windows display scaling. Note only do the below if you have a 4k monitor. The numbers may change
```
export GDK_SCALE=2
export GDK_DPI_SCALE=0.75
export QT_SCALE_FACTOR=2
```

## Scrolling in nano via mouse
Add the following alias to your .bashrc or .zshrc. Reload this in your terminal using source ~/.bashrc
```
alias nano="nano --mouse"
```

# TODO

- Add more wsl2 tips from existing files
- paste images and files to intellij and nautilus
- Map wsl2 as a network drive for easy exploring within file explorer
- get rid of scaling errors with nautilus
- scale pictures on readme a little smaller
- how to shortcut intellij on desktop from wsl2
- intellij open from terminal at a particular folder path within wsl2
- add file explorer to terminal path

