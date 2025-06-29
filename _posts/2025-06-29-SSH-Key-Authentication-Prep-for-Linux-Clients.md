`ssh-keygen -t ecdsa -C "<Insert Your Label Here>"

`-t ecdsa` - this sets the encryption algorithm
`-C "<Insert Your Label Here>"` - This is just to add a label to your key

When prompted- save your key to a file (You can just press enter keep default location)

When prompted- set a passphrase for extra security (optional)

**Default Locations:**
	Private key: ~/.ssh/id_ed25519
	Public key: ~/.ssh/id_ed25519.pub

The .ssh directory is a hidden directory, to see it run- `ls -a`

To view the content of the pub key file use the `cat` command. Short for concatenate, this command will print the contents of a file to the terminal.

`cat ~/.ssh/id_ecdsa.pub` -print the content of the pub key so you can copy it to the Windows Server
