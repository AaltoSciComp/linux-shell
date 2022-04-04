Bonus material
==============

Parts that did not fit.

[FIXME: should be moved to another tutorial *SSH: beyond login*]

SSH keys and proxy (*bonus section*)
------------------------------------
* SSH is the standard for connecting to remote computers: it is
  both powerful and secure.
* It is highly configurable, and doing some configuration will make
  your life much easier.

SSH keys and proxy jumping makes life way easier. For example, logging
on to Triton from your Linux workstation or from kosh/lyta.

For PuTTY (Windows) SSH keys generation, please consult section "Using public keys for SSH authentication" at [#putty-sshkeys]_

On Linux/Mac: generate a key on the client machine

::

 ssh-keygen -o  # you will be prompted for a location to save the keys, and a passphrase for the keys. Make sure passphrase is strong (!)
 ssh-copy-id aalto_login@triton.aalto.fi   # transfer file to a Triton, or/and any other host you want to login to

From now on you should be able to login with the SSH key instead of
password. When SSH key added to the ssh-agent (once during the login
to workstation), one can login automatically, passwordless.

Note that same key can be used on multiple different computers.

SSH proxy is yet another trick to make life easier: allows to jump
through a node (in OpenSSH version 7.2 and earlier ``-J`` option is
not supported yet, here is an old recipe that works on Ubuntu
16.04). By using this, you can directly connect to a system (Triton)
through a jump host (kosh):  On the client side, add to
``~/.ssh/config`` file (create it if does not exists and make it
readable by you only)::

 Host triton triton.aalto.fi
     Hostname triton.aalto.fi
     ProxyCommand ssh YOUR_AALTO_LOGIN@kosh.aalto.fi -W %h:%p





.. [#putty-sshkeys] https://the.earth.li/~sgtatham/putty/0.70/htmldoc/
