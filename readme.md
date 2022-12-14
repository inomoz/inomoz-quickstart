# My quick-start archlinux installation workflow

Clone this repository and run commands in the project root directory.
Require ~ 20GB of free space (last time full installation was 13GB).

1. First, sync mirrors and install Ansible with dependencies

```shell
sudo pacman -Syy python-passlib ansible
```

2. Second, install and update the submodules, check carefully group_vars/all file (configuration).

```shell
git submodule init && git submodule update
```

3. Run the playbook as root (playbooks will use group_vars/all configuration, including username).

```shell
# install core
sudo ansible-playbook -i localhost playbook.yml --tags=base,gnupg,ssh

# import gpg key
gpg --import ../gpg.asc
rm ../gpg.asc

# Add key to sshcontrol
gpg --list-keys --with-keygrip
# find key under pub section Keygrip = ...
echo '123....' >> ~/.gnupg/sshcontrol

# Enable SSH_AGENT_PID
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
  export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
fi

# Enable tty
export GPG_TTY=$(tty)
gpg-connect-agent updatestartuptty /bye >/dev/null

# Install dotfiles & doom emacs
```shell
yadm clone git@github.com:inomoz/dotfiles.git
yadm reset --hard origin/main # be careful with this command
ls -lFa ~

git clone --depth 1 git@github.com:inomoz/doomemacs.git ~/.emacs.d
cd ~/.emacs.d/bin/
./doom install
```

# Install & configure suckless software
```shell
mkdir -p ~/Projects/suckless/
cd ~/Projects/suckless/
git clone git@github.com:inomoz/dwm.git
git clone git@github.com:inomoz/slstatus.git

cd dwm
makepkg -fsri

cd ../slstatus
make
sudo make install
```

# Install other packages
```shell
sudo ansible-playbook -i localhost playbook.yml --skip-tags=base,gnupg,ssh

# Validate emacs
./doom doctor

# Open emacs end build some packages M-x
pdf-tools-install
vterm
plantuml-download-jar

# It's recommended to reboot, to validate the installation.
reboot
```

# Info

some directories are preconfigured, you can change them if you want
for example in user.yml "/var/" used as prefix for logs

# Special thanks

- https://github.com/pigmonkey/spark

# Manual tests

ArchLinux 5.18.16-arch1-1

Some tests on complex tasks:
  Update mirrors before: `sudo pacman -Syy`

- [x] android  
  `adb devices` - show some devices connected, if they are connected
- [x] arduino  
  `arduino-cli board list`
- [x] base  
  - check pacman mirrors `sudo pacman -Syy`, must include core, extra, community, multilib
  - validate hostname `hostname -f`
  - validate group `id`
  - validate logrotate `logrotate --debug /etc/logrotate.conf`, home/...
  - run some applications: `tmux`, `git`, `ytfzf`, `ledger`, `ipython`, `sc-im`,  `bash`, `zsh`,  `krita`
    , `inkscape`, `darktable`, `blender`, `godot`, `maim`,
  - validate paccache service `systemctl status paccache.timer`
- [x] mpv
  - mpv 'https://www.youtube.com/watch?v=dQw4w9WgXcQ'
  - git clone https://github.com/inomoz/files_samples /tmp/files_samples
  - umpv 'https://www.youtube.com/watch?v=dQw4w9WgXcQ'
- [x] browsers
  - w3m, lynx
  - using dmenu: firefox, chromium, qutebrowser... can open url:
    , `xdg-open http://acid3.acidtests.org/`, `xdg-open https://www.youtube.com/watch?v=dQw4w9WgXcQ`
  - some browsers at first time require manual launch or extra configuration (tor browser)
- [?] colemak-dh  
  vconsole works, layout works, layout switch works
- [x] cups  
  - add print in cups interface: http://localhost:631/admin
  - drivers auto-copied into ~/Downloads/
  - test print page
- [x] docker  
  `docker run hello-world`, run this after re-login
- [x] gnupg
  you able push to github with gpg signature
- [x] goesimage  
  - `systemctl enable --user --now goesimage.timer`
  - `systemctl status --user goesimage.timer`
- [x] media  
  copy something to clipboard and run `qcode`
- [x] mirrorlist  
  - `sudo systemctl enable reflector-update.timer`
  - `sudo systemctl start reflector-update.service`
  - `cat /etc/pacman.d/mirrorlist`
- [x] pdf  
  `zathura path_to_pdf_file`
- [x] postgresql  
  ```shell 
  create database test
  ls /var/lib/postgresql/data/
  dropdb test
  ```
- [x] systemd-networkd  
    - `systemctl status systemd-colemak systemd-resolved`
    - `ip a`
- [x] udisks  
  you able to mount some usb devices
- [x] visidata  
  `xdg-open ...csv`
- [x] wacom    
  - `xsetwacom list devices`
  - precision in krita is working 
- [x] x  
  - xdg-dir exists `ls -l ~`
  - `vkmark -b :duration=2.0 -b vertex:interleave=true -b vertex:interleave=false -b :duration=5.0 -b cube` score ~30k
  - resolution was set correctly by autorandr
- [x] zeal
  - you need rsync docsets into ~/Documents/Zeal/docsets 
  -  `tar zcvf - zeal/ | ssh -o IdentityAgent=none inom@baikal "mkdir -p ~/Documents/zeal; cd ~/Documents/; tar xvzf -"`
  - run `zeal` and validate it, including CSS styles
  - install base docsets and custom `https://zealusercontributions.vercel.app/`

# Notes
- [ ] Sync your files
- [ ] Try to add ssh key and login (external service), git pull, etc
- [ ] Check pass and gpg stuff
- [ ] Check browser pass integration
- [ ] Sync browser bookmarks
- [ ] Validate systemd enabled services, like reflector, services list:  
- [ ] You can install lxapperance to configure theme/fonts/etc and remove it after.
- [ ] Verify and configure vappi (firefox, etc)  
- https://wiki.archlinux.org/title/Hardware_video_acceleration
- [ ] Check systemd units  
`systemctl list-unit-files --type=service`
- [ ] Configure backups and system snapshots
- [ ] Remove ansible (too heavy)
