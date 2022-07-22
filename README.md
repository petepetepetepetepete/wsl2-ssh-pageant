# wsl2-ssh-pageant

## Motivation
I use a Yubikey to store a GPG key pair and I like to use this key pair as my SSH key too. GPG on Windows exposes a Pageant style SSH agent and I wanted a way to use this key within WSL2.

## How to use with WSL2

### Prerequisite
In order to use `wsl2-ssh-pageant` you must have installed `socat` and `ss` on your machine.

For example, on Ubuntu you can install these by running: `sudo apt install socat iproute2`

### Installation
1. Install `wsl2-ssh-pageant.exe` into Windows file system
    ```
    make
    WINDESTDIR="$(wslpath -u "$(wslvar HOMEDRIVE)$(wslvar HOMEPATH)")/.ssh/"
    mkdir -p $WINDESTDIR
    chmod +x wsl2-ssh-pageant.exe
    cp wsl2-ssh-pageant.exe $WINDESTDIR
    ```
2. Add one of the following to your shell configuration (for e.g. `.bashrc`, `.zshrc` or `config.fish`). For advanced configurations consult the documentation of your shell.

#### Bash

*SSH:*
```bash
export SSH_AUTH_SOCK="$HOME/.ssh/agent.sock"
if ! ss -a | grep -q "$SSH_AUTH_SOCK"; then
  rm -f "$SSH_AUTH_SOCK"
  wsl2_ssh_pageant_bin="$(wslpath -u "$(wslvar HOMEDRIVE)$(wslvar HOMEPATH)")/.ssh/wsl2-ssh-pageant.exe"
  if test -x "$wsl2_ssh_pageant_bin"; then
    (setsid nohup socat UNIX-LISTEN:"$SSH_AUTH_SOCK,fork" EXEC:"$wsl2_ssh_pageant_bin" >/dev/null 2>&1 &)
  else
    echo >&2 "WARNING: $wsl2_ssh_pageant_bin is not executable."
  fi
  unset wsl2_ssh_pageant_bin
fi
```

*GPG:*
```bash
export GPG_AGENT_SOCK="$HOME/.gnupg/S.gpg-agent"
if ! ss -a | grep -q "$GPG_AGENT_SOCK"; then
  rm -rf "$GPG_AGENT_SOCK"
  wsl2_ssh_pageant_bin="$(wslpath -u "$(wslvar HOMEDRIVE)$(wslvar HOMEPATH)")/.ssh/wsl2-ssh-pageant.exe"
  gpg_config_path="$(wslvar LOCALAPPDATA | sed -e 's,\\,/,g')"
  if test -x "$wsl2_ssh_pageant_bin"; then
    (setsid nohup socat UNIX-LISTEN:"$GPG_AGENT_SOCK,fork" EXEC:"$wsl2_ssh_pageant_bin --gpgConfigBasepath '$gpg_config_path/gnupg' --gpg S.gpg-agent" >/dev/null 2>&1 &)
  else
    echo >&2 "WARNING: $wsl2_ssh_pageant_bin is not executable."
  fi
  unset wsl2_ssh_pageant_bin
fi
```

## Troubleshooting

### wsl2-ssh-pageant won't work with Pageant > 0.74
See https://github.com/BlackReloaded/wsl2-ssh-pageant/issues/23 for details. Current workaround is to install putty 0.74

### Smartcard is detected in Windows and WSL, but ssh-add -L returns error
If this is the first time you using yubikey with windows with gpg4win, please follow the instructions in the link
https://developers.yubico.com/PGP/SSH_authentication/Windows.html

| Make sure ssh support is enabled in the `gpg-agent.conf` and restart `gpg-agent` with the following command

```
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
```

### Agent response times are very slow
If ssh,ssh-add,gpg etc are very slow (~15-25 seconds) check that wsl2-ssh-pageant resides on the windows file system. This is due to an issue with the WSL interop documented [here](https://github.com/BlackReloaded/wsl2-ssh-pageant/issues/24) and [here](https://github.com/microsoft/WSL/issues/7591)

## Credit

Some of the code is copied from benpye's [wsl-ssh-pageant](https://github.com/benpye/wsl-ssh-pageant). This code shows how to communicate to pageant.
