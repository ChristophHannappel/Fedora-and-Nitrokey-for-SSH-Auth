# Fedora-and-Nitrokey-for-SSH-Auth
How to get the Nitrokey Pro working for SSH Authenticaiton on Fedora Workstation.  
I didn't find any complete Guide. So in order to save you sometime here is mine :)   
I've tested it on a fresh Fedora Workstation 35 Installation

## Requirements
* Allready initialized Nitrokey Pro 2 with GPG Keys
* Fedora Workstation 35
* Public GPG Key in a file

### Step 1: Import the Public Key
First you need to make sure your public key is imported
```bash
# Import the Key
gpg --import ./MyPublicKey.asc
```

### Step 2: Configure the gpg files: gpg.conf, gpg-agent.conf and scdaemon.conf
1. Add `use-agent` to the `~/.gnupg/gpg.conf` file.
2. Add `enable-ssh-support` and `pinentry-program /usr/bin/pinentry-curses` to the `~/.gnupg/gpg-agent.conf` file.
3. Add `pcsc-driver /usr/lib64/libpcsclite.so.1` and `disable-ccid` to the `~/.gnupg/scdaemon.conf` file.
```bash
# Add `use-agent` to the `~/.gnupg/gpg.conf` file.
echo use-agent > ~/.gnupg/gpg.conf
# Add `enable-ssh-support` and `pinentry-program /usr/bin/pinentry-curses` to the `~/.gnupg/gpg-agent.conf` file.
echo enable-ssh-support > ~/.gnupg/gpg-agent.conf
echo pinentry-program /usr/bin/pinentry-curses >> ~/.gnupg/gpg-agent.conf
# Add `pcsc-driver /usr/lib64/libpcsclite.so.1` and `disable-ccid` to the `~/.gnupg/scdaemon.conf` file.
echo pcsc-driver /usr/lib64/libpcsclite.so.1 > ~/.gnupg/scdaemon.conf
echo disable-ccid >> ~/.gnupg/scdaemon.conf
```
Now your files should look like this:  
`~/.gnupg/gpg.conf`
```conf
use-agent
```
`~/.gnupg/gpg-agent.conf`
```conf
enable-ssh-support
pinentry-program /usr/bin/pinentry-curses
```
`~/.gnupg/scdaemon.conf`
```conf
pcsc-driver /usr/lib64/libpcsclite.so.1
disable-ccid
```

### Step 3: Configure the ~/.bashrc
Append the following to your `~/.bashrc` file.
```bash
# This is a copy of the code at the man page EXAMPLES entry: man gpg-agent
#  It is important to set the environment variable GPG_TTY in your login shell, for example in the ‘~/.bashrc’ init script:
export GPG_TTY=$(tty)

# If you enabled the Ssh Agent Support, you also need to tell  ssh  about it by adding this to your init script:
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
	export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
fi

# This prevents the error "sign_and_send_pubkey: signing failed: agent refused operation": Source: https://support.nitrokey.com/t/nitrokey-ssh-git-sign-and-send-pubkey-signing-failed-agent-refused-operation/1886
gpg-connect-agent updatestartuptty /bye >/dev/null
```
Your full `~/.bashrc` file should look like this
```bash
 # .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
if [ -d ~/.bashrc.d ]; then
        for rc in ~/.bashrc.d/*; do
                if [ -f "$rc" ]; then
                        . "$rc"
                fi
        done
fi

unset rc

# It is  important to set the environment variable GPG_TTY in your login shell, for example in the ‘~/.bashrc’ init script:
export GPG_TTY=$(tty)

# If you enabled the Ssh Agent Support, you also need to tell  ssh  about it by adding this to your init script:
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
	export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
fi

# This prevents the error "sign_and_send_pubkey: signing failed: agent refused operation": Source: https://support.nitrokey.com/t/nitrokey-ssh-git-sign-and-send-pubkey-signing-failed-agent-refused-operation/1886
gpg-connect-agent updatestartuptty /bye >/dev/null
```
### Step 4: Reboot , add the SSH key and test
1. Restart your Fedora Workstation and try the following testes to validate that gpg finds your NitroKey
2. Try to show your Nitrokey Contents: `gpg --card-status`
3. Check if your public key was importet: `gpg --list-keys` 
4. Add your SSH Auth Key with the command `ssh-add`
5. Check if your public key has been added with `ssh-add -l` 
6. Connect via ssh. It should ask you for your PIN.
### Known Issues
If the Nitrokey was plugged in at boot time, you need to restart the pcscd service.
`sudo systemctl restart pcscd` 

# Sources:
* [SSH for Server Administration](https://www.nitrokey.com/documentation/applications#ssh-for-server-administration)
* [[Nitrokey] SSH / Git: sign_and_send_pubkey: signing failed: agent refused operation](https://support.nitrokey.com/t/nitrokey-ssh-git-sign-and-send-pubkey-signing-failed-agent-refused-operation/1886)
* [Fedora Yubikey GPG-Agent scdaemon issues](https://www.tomica.net/blog/2021/08/fedora-yubikey-gpg-scdaemon-issues/)
