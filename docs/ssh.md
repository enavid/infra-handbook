mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys

use agents for passphrase:

basheval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

add to ~/.bashrc یا ~/.zshrc :
bashif [ -z "$SSH_AUTH_SOCK" ] ; then
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_ed25519
fi
