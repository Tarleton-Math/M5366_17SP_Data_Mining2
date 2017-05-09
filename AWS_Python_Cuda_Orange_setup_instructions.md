
# Create AWS instance

Mostly from http://chrisalbon.com/jupyter/run_project_jupyter_on_amazon_ec2.html, with a few tweaks

1. create AWS account
1. launch T2.micro ubuntu 16.04 instance
1. store .pem file
1. security group, Inbound rules, "AnyWhere"
    - HTTP, TCP, port 80
    - SSH, TCP, port 22
    - Custom TCP rule, TCP, port 8888 (for Jupyter)
    - Custom TCP rule, TCP, port 5901 (for vncserver)
1. create elastic ip and associate to instance above (if not, your IP will change each time you reboot)
1. download and install putty and puttygen
1. convert .pem to .ppk using puttygen.  Store both
1. ssh into the instance
    1. open Putty
    1. Use Elastic IP address, port 22, SSH
    1. Connection -> Data -> username = ubuntu
    1. Connection -> SSH -> Auth -> set private key to the .ppk file created above
    1. Save settings  
    1. save profile
    1. open
    1. click okay if rsa popup
1. Update system software
    - sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade && sudo apt-get autoremove && sudo apt clean && sudo reboot

# Vim quick start
- i - enters insert mode, esc enters command mode
- in command mode
    - :w - saves
    - :q - quits

# Install GUI (optional, but convenient)
1. First step takes about 10 minutes
    - sudo apt-get install ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal gnome-session dconf-editor tightvncserver
1. vncserver
1. vncserver -kill :1
1. cd /home/ubuntu/.vnc/
1. sudo rm xstartup
1. sudo vim xstartup
1. paste script below called "Script /home/ubuntu/.vnc/xstartup"
1. save & exit
1. sudo chmod +x xstartup
1. Make vncserver auto launch at startup
    1. Check if "upstart" is the init process
        - sudo stat /proc/1/exe
        - If you see "upstart", skip to next step.  But if you see "systemd", run the command below.
        - sudo apt-get install upstart-sysv && sudo update-initramfs -u && sudo reboot
    1. cd /etc/init
    1. sudo vim vncserver.conf
    1. paste script below called "Script /etc/init/vncserver.conf"
    1. save & exit
    1. Should now auto launch on boot.  To start/stop manually: sudo service vncserver start/stop
1. Install TightVNCviewer (or equivalent) on your local machine
1. Connect using AWS elastic IP with ":1" appended
Script /home/ubuntu/.vnc/xstartup


#!/bin/sh 
export XKL_XMODMAP_DISABLE=1
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &

gnome-settings-daemon &
metacity &
nautilus &
gnome-terminal &
gnome-session &
gnome-panel &Script /etc/init/vncserver.conf


description "VNCserver"
author "Scott Cook"

# Stanzas
#
# Stanzas control when and how a process is started and stopped
# See a list of stanzas here: http://upstart.ubuntu.com/wiki/Stanzas

# When to start the service
#start on runlevel [2345]
start on started mountall

# When to stop the service
stop on runlevel [016]

# Automatically restart process if crashed
respawn

script
    #export PATH=/usr/bin:$PATH
    export USER="ubuntu"
    DISPLAY="1"
    DEPTH="16"
    #GEOMETRY="<WIDTH>x<HEIGHT>"
    #GEOMETRY="800x600"
    GEOMETRY="1024x768"
    #GEOMETRY="1280x1024"
    NAME="my-vnc-server"
    OPTIONS="-name ${NAME} -depth ${DEPTH} -geometry ${GEOMETRY} :${DISPLAY}"
    exec su ${USER} -c "/usr/bin/vncserver ${OPTIONS}"
end script
# Install Python
1. Find linux anaconda installer url https://www.continuum.io/downloads
1. wget "that url"
1. bash "that file" - runs installer
    - accept /home/ubuntu/anaconda3
    - accept all prompts (specifically allow it to add to system path; if you miss it, do export PATH=~/anaconda3/bin:$PATH)
1. sudo rm "that file"
1. close terminal and open a new one
1. conda update anaconda
1. mkdir /home/ubuntu/notebooks
1. Follow rest of instructions at http://chrisalbon.com/jupyter/run_project_jupyter_on_amazon_ec2.html with following changes
    - Make sure you copy your password hash for later use.  It looks like 'sha1:98ff0e580111:12798c72623a6eecd54b51c006b1050f0ac1a62d'
    - While editing config file, add c.NotebookApp.notebook_dir = u"/home/ubuntu"
1. (optional) Install Jupyter Extensions from https://github.com/ipython-contrib/jupyter_contrib_nbextensions
1. Make Jupyer auto launch at startup
    1. Check if "upstart" is the init process
        - sudo stat /proc/1/exe
        - If you see "upstart", skip to next step.  But if you see "systemd", run the command below.
        - sudo apt-get install upstart-sysv && sudo update-initramfs -u && sudo reboot
    1. cd /etc/init
    1. sudo vim jupyter.conf
    1. paste script below called "Script /etc/init/jupyter.conf"
    1. save & exit
    1. Should now auto launch on boot.  To start/stop manually: sudo service jupyter start/stop
Script /etc/init/jupyter.conf


description "Jupyter Notebook"
author "Scott Cook"

# Stanzas
#
# Stanzas control when and how a process is started and stopped
# See a list of stanzas here: http://upstart.ubuntu.com/wiki/Stanzas

# When to start the service
#start on runlevel [2345]
start on started mountall

# When to stop the service
stop on runlevel [016]

# Automatically restart process if crashed
respawn

script
    #export CUDA_ROOT=/usr/local/cuda-8.0
    #export PATH=${CUDA_ROOT}/bin:$PATH
    #export LD_LIBRARY_PATH=${CUDA_ROOT}/lib64$LD_LIBRARY_PATH
    #Use lines above if GPU & CUDA present
    
    exec /home/ubuntu/anaconda3/bin/jupyter notebook --config /home/ubuntu/.jupyter/jupyter_notebook_config.py

end script
# Install Orange
1. conda create python=3 --name orange3
1. source activate orange3
1. conda config --add channels conda-forge
1. conda install orange3
1. cd home/ubuntu
1. sudo vim orange3.launcher
1. paste script below called "Script home/ubuntu/orange3.launcher"
1. save & exit
1. sudo chmod +x orange3.launcher
1. Launch desktop gui
1. System tools -> dconf editor -> org -> gnome -> nautilus -> preferences  (if no dconf-editor, do sudo apt-get install dconf-editor)
1. Executable-text-activation -> launch
1. Double click on orange3.launcher (may take a minute to load first time)
Script home/ubuntu/orange3.launcher


#!/bin/bash
/home/ubuntu/anaconda3/envs/orange3/bin/python -m Orange.canvas


```python
# Clones Vanderplas books
! git clone https://github.com/jakevdp/WhirlwindTourOfPython.git /home/ubuntu/notebooks/jakevdp/WhirlwindTourOfPython
! git clone https://github.com/jakevdp/PythonDataScienceHandbook.git /home/ubuntu/notebooks/jakevdp/PythonDataScienceHandbook
```

# Setup rsa github access
https://help.github.com/articles/generating-an-ssh-key/

http://stackoverflow.com/questions/6565357/git-push-requires-username-and-password

1. In jupyter, open new terminal OR ssh into the instance via putty
1. git config --global --replace-all user.name {your github username}
1. git config --global --replace-all user.email {your email}
1. Below here, replace {rsafile} with /home/ubuntu/.ssh/id_rsa  This is your PRIVATE key.
1. cd /home/ubuntu/.ssh
1. ls
    - Does id_rsa already exist? If not, create by ssh-keygen -t rsa -b 4096 -N "" -C {your email} -f {rsa_file}
1. eval "$(ssh-agent -s)"
1. ssh-add {rsafile}
1. head {rsa_file}.pub - prints the PUBLIC key.
1. copy this public key to your github profile

# Make local clone of your repo

1. From github repo, click "clone or download".  Copy the *ssh* address.  Denote it {url}
1. cd /home/ubuntu/notebooks
1. git clone {url}
    - Should create subdirectory
    - Alternatively, create subdir manually.  Then append that dir to clone command.
1. git pull
1. test write privileges
    1. cd into subdir
    1. echo 'Hello World' >> testfile.md
    1. git add testfile.md
    1. git commit -m "Testing git write access"
    1. git push
    1. go to your github repo, refresh, see if changes appear

# OS commands in notebooks
You can issue OS commands within a jupyter notebook in several ways
- import os    then  os.system(commands).   See documentation on the python os module
- Lines starting with ! are sent as bash commands to the os
- Use the "magic" command %%bash to make an entire cell into a bash script

However, I usually find it easier to interact directly with the terminal.  As far as I can tell, none of these is interactive.  So if a command requires additional user input, it usually fails.  For example, you probably needed to respond "yes" to a message about rsa keys earlier.  
# alternate vncserver auto launch instructions #1

cd /etc/init.d

sudo vim vncserver


#!/bin/sh -e
### BEGIN INIT INFO
# Provides:          vncserver
# Required-Start:    networking
# Required-Stop:
# Default-Start:     3 4 5
# Default-Stop:      0 6
# Short-Description:
# Description:
### END INIT INFO

PATH="$PATH:/usr/bin/"

# The Username:Group that will run VNC
export USER="ubuntu"
#${RUNAS}

# The display that VNC will use
DISPLAY="1"

# Color depth (between 8 and 32)
DEPTH="16"

# The Desktop geometry to use.
#GEOMETRY="<WIDTH>x<HEIGHT>"
#GEOMETRY="800x600"
GEOMETRY="1024x768"
#GEOMETRY="1280x1024"

# The name that the VNC Desktop will have.
NAME="my-vnc-server"

OPTIONS="-name ${NAME} -depth ${DEPTH} -geometry ${GEOMETRY} :${DISPLAY}"

. /lib/lsb/init-functions

case "$1" in
start)
log_action_begin_msg "Starting vncserver for user '${USER}' on localhost:${DISPLAY}"
su ${USER} -c "/usr/bin/vncserver ${OPTIONS}"
;;

stop)
log_action_begin_msg "Stoping vncserver for user '${USER}' on localhost:${DISPLAY}"
su ${USER} -c "/usr/bin/vncserver -kill :${DISPLAY}"
;;

restart)
$0 stop
$0 start
;;
esac

exit 0



sudo chmod +x vncserver

sudo update-rc.d vncserver defaults 98# alternate vncserver auto launch instructions #2

cd /etc/systemd/system

sudo vim vncserver@.service


[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=ubuntu
PAMName=login
PIDFile=/home/ubuntu/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target



sudo systemctl daemon-reload

sudo systemctl enable vncserver@1.service

vncserver -kill :1

sudo systemctl start vncserver@1