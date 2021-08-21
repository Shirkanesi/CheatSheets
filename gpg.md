# Git(Hub) keys - CHEATSHEET
Note: Some of the commands used won't work in Command Prompt or PowerShell. The should be run using `GitBash`
# GPG - Keys (Commit-Signing)
## List your installed keys
`$ gpg --list-secret-keys --keyid-format=long` will show you a list of all keys installed. For each key you'll see something like `rsa4096/<id>` this id will later be important. This is is also, what [GitHub](https://github.com/settings/keys) will show you as the _Key ID_.

## Export private and public key from key-store / key-ring
To export your public key from your local `gnupg` into C:\\\\ (/c/)

`$ gpg --output /c/public.pgp --armor --export <id>`,

exporting the private key works the same:

`$ gpg --output /c/private.pgp --armor --export-secret-key <id> `.

This command will probably ask you for your key's passphrase.

Those two files will be the same as you get when generating new keys. Those files, especially the private key should never be leaked to anyone without proper reason.

## Getting information about a .pgp-file
To get some more information about a public-key you can simply use:

`$ gpg <keyfile>`

## Generating a new key-pair
TODO
`$ gpg --full-gen-key`. This will start an automated assistant helping you to create the proper key. For git you normally want to use RSA&RSA (time of writing the default option `1`). Then you'll need to specify the key-length. Rule of thumb: the longer the better (`4096`). Now you'll be asked, whether your gpg-key should expire at any point in the future. After that you need to specify your name and email (note: this email will be used to identify you later!).

Note: the keys will be generated during this process, however, in order to upload the public key to GitHub, you need to export the public key as described above.

## Change your passphrase
`$ gpg --edit-key <id>`, then `passwd`. Now the command should tell you exactly what to do next.

### Verifying everything works:
Just go ahead and create an empty git-repostory, add a file and commit using `-S` to sign your commit:

```
git init
echo example > example.txt
git add .
git commit -S -m "Sign-Test"
git log --show-signature
```
The `git log` should now show you the key-id and author details.

# SSH-Keys (client-authentication)
## Get the SHA256-Hash of a ssh-key
To get the SHA-256-hash of a ssh-key stored on your local system, you can use 

`$ ssh-keygen -l -E sha256 -f /path/to/key/id_ed25519`

You can use this hash to identify a key on e.g. [GitHub](https://github.com/settings/keys).

## Retrieving a SSH public key from private key
`$ ssh-keygen -y `

This will ask you for the key-file the key is stored in (e.g. id_ed25519). After entering the passphrase the command will spit out the public `ssh-rsa`-String.
This key can be directly pasted into [GitHub](https://github.com/settings/keys).

## Creating a new SSH-key
`$ ssh-keygen` this will generate a new key and ask you where to store. Also you'll be asked, whether you want to set a passphrase. The public-key will be stored along with the private key; filename will end with `.pub` (alternatively, it can later be recovered with the command mentioned above).

# Use GPG4Win as your GPG-Program for `git`
In order to use GPG4Win, you need to edit your `.gitconfig`.

```toml
[gpg]
	# Use GPG4Win
	program = C:\\Program Files (x86)\\GnuPG\\bin\\gpg.exe
	# Use the git-bash-style gpg
    # program = gpg
```
Shortcut: `$ git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe"`

## Auto-Signing commits
Add the following entry to your `.gitconfig`
```toml
[commit]
	gpgsign = true
```
Shortcut: `$ git config --global commit.gpgsign true`

## Auto-Signign tags
Add the following entry to your `.gitconfig`
```toml
[tag]
	gpgSign = true
```
Shortcut: `$ git config --global tag.gpgSign true`