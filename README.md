# ansible-bootstrap
This is a (slightly stripped down) production example of an Ansible
bootstrapping role & playbook.

The role assumes the following in `ansible.cfg`:
```
[privilege_escalation]
become=True
become_user=root
```

It is designed to support SSH pipelining:
```
[ssh_connection]
pipelining = True
```

This requires a dance of using paramiko for the first task to get rid of
`requiretty` from the sudo configuration.

You will need two public keys in `roles/bootstrap/files/` named
`ansible.rsa.key.pub` and `ansible.ed25519.key.pub`.  It sets them both
up to facilitate using RSA now (in support of older systems) but preparing for
a move to ed25519.  If you don't want both, you can remove the reference to
one in the role tasks.  Configure the private key in `ansible.cfg`:
```
[defaults]
private_key_file = /YOUR/PATH/TO/ansible.rsa.key
```

## Usage

If the system already has a user named `ansible` with sudo access and a
known password:
```
  ansible-playbook bootstrap.yaml \
    -e target=EXAMPLE.DOMAIN --ask-pass --ask-become-pass
```

If you wish to use a different user, specify it with `-u`:
```
  ansible-playbook bootstrap.yaml -u USERNAME \
    -e target=EXAMPLE.DOMAIN --ask-pass --ask-become-pass
```

If you wish to use a different user and have SSH key-based access, omit
`--ask-pass`:
```
  ansible-playbook bootstrap.yaml -u USERNAME \
    -e target=EXAMPLE.DOMAIN --ask-become-pass
```

If the system does not have sudo installed (e.g. Debian 10 out of the box),
but you have a user which can use SSH _and_ have a root password, specify a
`become_method` of `su` and, when prompted, specify the root password as the
become password (which may be different from the SSH user's password):
```
  ANSIBLE_BECOME_METHOD=su ansible-playbook bootstrap.yaml \
    -u USERNAME -e target=EXAMPLE.DOMAIN --ask-pass --ask-become-pass
```
