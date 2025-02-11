# wsl2-ssh-bridge

## Motivation

I use a Yubikey to store a GPG key pair and I like to use this key pair as my
SSH key too. GPG on Windows exposes a Pageant style SSH agent and I wanted a way
to use this key within WSL2.

## How to use with WSL2

### Prerequisite

In order to use `wsl2-ssh-bridge` you must have installed `socat` and `ss` on
your machine.

For example, on Ubuntu you can install these by running the following command:

```bash
sudo apt install --yes socat iproute2
```

### Installation

1. Download latest version from
[release page](https://github.com/KerickHowlett/wsl2-ssh-bridge/releases/latest)
and copy `wsl2-ssh-bridge.exe` to your windows home directory (or other location
within the windows file system). Then simlink to your `$HOME/.ssh` directory for
easy access

    ```bash
    windows_destination="/mnt/c/Users/Public/Downloads/wsl2-ssh-bridge.exe"
    linux_destination="$HOME/.ssh/wsl2-ssh-bridge.exe"
    wget -O "$windows_destination" "https://github.com/KerickHowlett/wsl2-ssh-bridge/releases/latest/download/wsl2-ssh-bridge.exe"
    # Set the executable bit.
    chmod +x "$windows_destination"
    # Symlink to linux for ease of use later
    ln -s $windows_destination $linux_destination
    ```

2. Add one of the following to your shell configuration (for e.g. `.bashrc`,
`.zshrc` or `config.fish`). For advanced configurations consult the
documentation of your shell.

#### Bash/Zsh

*SSH:*

```bash
export SSH_AUTH_SOCK="$HOME/.ssh/agent.sock"
if ! ss -a | grep -q "$SSH_AUTH_SOCK"; then
  rm -f "$SSH_AUTH_SOCK"
  wsl2_ssh_bridge_bin="$HOME/.ssh/wsl2-ssh-bridge.exe"
  if test -x "$wsl2_ssh_bridge_bin"; then
    (setsid nohup socat UNIX-LISTEN:"$SSH_AUTH_SOCK,fork" EXEC:"$wsl2_ssh_bridge_bin" >/dev/null 2>&1 &)
  else
    echo >&2 "WARNING: $wsl2_ssh_bridge_bin is not executable."
  fi
  unset wsl2_ssh_bridge_bin
fi
```

*GPG:*

```bash
export GPG_AGENT_SOCK="$HOME/.gnupg/S.gpg-agent"
if ! ss -a | grep -q "$GPG_AGENT_SOCK"; then
  rm -rf "$GPG_AGENT_SOCK"
  wsl2_ssh_bridge_bin="$HOME/.ssh/wsl2-ssh-bridge.exe"
  if test -x "$wsl2_ssh_bridge_bin"; then
    (setsid nohup socat UNIX-LISTEN:"$GPG_AGENT_SOCK,fork" EXEC:"$wsl2_ssh_bridge_bin --gpg S.gpg-agent" >/dev/null 2>&1 &)
  else
    echo >&2 "WARNING: $wsl2_ssh_bridge_bin is not executable."
  fi
  unset wsl2_ssh_bridge_bin
fi
```

#### Fish

*SSH:*

```fish
set -x SSH_AUTH_SOCK "$HOME/.ssh/agent.sock"
if not ss -a | grep -q "$SSH_AUTH_SOCK";
  rm -f "$SSH_AUTH_SOCK"
  set wsl2_ssh_bridge_bin "$HOME/.ssh/wsl2-ssh-bridge.exe"
  if test -x "$wsl2_ssh_bridge_bin";
    setsid nohup socat UNIX-LISTEN:"$SSH_AUTH_SOCK,fork" EXEC:"$wsl2_ssh_bridge_bin" >/dev/null 2>&1 &
  else
    echo >&2 "WARNING: $wsl2_ssh_bridge_bin is not executable."
  end
  set --erase wsl2_ssh_bridge_bin
end
```

*GPG:*

```fish
set -x GPG_AGENT_SOCK "$HOME/.gnupg/S.gpg-agent"
if not ss -a | grep -q "$GPG_AGENT_SOCK";
  rm -rf "$GPG_AGENT_SOCK"
  set wsl2_ssh_bridge_bin "$HOME/.ssh/wsl2-ssh-bridge.exe"
  if test -x "$wsl2_ssh_bridge_bin";
    setsid nohup socat UNIX-LISTEN:"$GPG_AGENT_SOCK,fork" EXEC:"$wsl2_ssh_bridge_bin --gpg S.gpg-agent" >/dev/null 2>&1 &
  else
    echo >&2 "WARNING: $wsl2_ssh_bridge_bin is not executable."
  end
  set --erase wsl2_ssh_bridge_bin
end
```

## Troubleshooting to Most Frequent Problems

### Smart Card is detected in Windows and WSL, but `ssh-add -L` returns error

If this is the first time you using YubiKey with Windows and Gpg4Win, please
follow these instructions from Yubico's Official Website:
<https://developers.yubico.com/PGP/SSH_authentication/Windows.html>

| Make sure ssh support is enabled in the `gpg-agent.conf` and restart
`gpg-agent` with the following command

```bash
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
```

### Agent response times are very slow

If you find commands like `ssh`, `ssh-add`, or `gpg` are slow (i.e., about
15-25 seconds) check that `wsl2-ssh-bridge` resides on Windows file system.
This is due to an issue with the WSL interop documented
[here](https://github.com/BlackReloaded/wsl2-ssh-bridge/issues/24)
and [here](https://github.com/microsoft/WSL/issues/7591)

## Credit

1. The original branch I forked this repo from:
[BlackReloaded/wsl2-ssh-pageant](https://github.com/BlackReloaded/wsl2-ssh-pageant)
2. Some of BlackReloaded's code is copied from
[benpye/wsl-ssh-pageant](https://github.com/benpye/wsl-ssh-pageant).
This code shows how to communicate to pageant.
