# SSHAgent

L'agent SSH evite de taper sa passphrase à chaque fois que l'on se connecte en SSH sur une machine.

## Préparation

Dans la console "sous-systeme Debian" sur Windows

```bash
mkdir ~/scripts/
editor ~/scripts/ssh_agent.sh
```

```ssh_agent.sh
#!/bin/bash
#set -x
export SSH_AUTH_SOCK=/tmp/ssh-agent.$HOSTNAME.sock
if [ ! -f $SSH_AUTH_SOCK ]; then
    ssh-add -l 2>/dev/null >/dev/null
    out=$?
    if [ "$out" -ge "2" ]; then
        ssh-agent -a "$SSH_AUTH_SOCK" >/dev/null
        ssh-add
    fi
fi
```

## Ajout au bashrc

```bash
editor ~/.bashrc
```
Ajouter cette ligne à la fin
```bash
source ~/scripts/ssh_agent.sh
```
