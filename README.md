# Sign your commits

It is good practice to sign your commits, the reason you sign a commit is to show that a commit was created by you.
By default git accepts whatever author you provide it with, so anyone can pretend to be you.
If someone would get access to your repository, for example by stealing your SSH-key, or cracking your password, they can commit whatever they want.
Of course having an password protected SSH-key and a strong password with 2FA is good protection, but it still does not validate that you created the commit.

The default way this is handled is by creating a GPG key, you can see this as your personal signature that you sign your work with.

The default tool to use is [GnuPG](https://www.gnupg.org/), and is available on every common operating system.

If you want to get it done right away, there is an excellent tutorial on [GitHub](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) or [Gitlab](https://docs.gitlab.com/ee/user/project/repository/gpg_signed_commits/).

This article helps you understand what GPG is and what else you can do it with it.

## Installation
If not yet installed, you can install GPG using any common Linux package manager, as well as [HomeBrew](https://formulae.brew.sh/formula/gnupg)
```bash
brew install gpg
```

## Creating a key
To create a new key, run the following command:
```bash
gpg --full-gen-key
```
The defaults are sensible, make sure you pick a strong password, and use a real and email address.
This key is meant to identify you after all.

After you complete the questions, you have generated a personal keypair.
You can run the following commands to see the keys:
```bash
gpg --list-keys
```
```bash
gpg --list-secret-keys
```

Always protect your secret key, no one can have this but you.

## How to use
These are some common use cases:

### Sign a file, output a test.doc.asc signature file:
```bash
gpg --clearsign test.doc
```

### Export your public key, to share with someone. 

This example shows how to find your key by email. 
You can have multiple keys for the same email address, GPG will pick the first one then.
It is safer to do it by ID as shown further down in the article, but works just fine if you only have one key.
Note: the --armor flag tells GPG to output in base64 encoded data, by default GPG outputs binary data. 

```bash
gpg --export --armor your@email.com > your-public-key.txt
```

### Export your private key for a backup, please handle this securely:
```bash
gpg --export-secret-keys --armor your@email.com > your_super_secret_private_key.txt
```

### Import someone else's key:
```bash
gpg --import someone_elses_public_key.txt
```

### Encrypt a file asymmetrically, and sign it. 

For this you need to import the recipients public key first:
```bash
gpg --encrypt --sign --recipient someone@else.com doc.txt
```

### Compress a file, sign it, and output in binary format:
```bash
gpg --clearsign test.doc
```

## Signing commits

What we want to achieve is to sign our commits using our private key, so it can be validated remotely using our public key.

A safe way to identify a GPG key is to use the longer format, the default short format is sensitive to collision attacks.
Therefore we use the longer 64bit/16hex character ID. To find our ID:

You'll first see "sec", then the encryption method used, then after the forward slash, you find the 16 character long key ID.
```bash
gpg --list-secret-keys --keyid-format=long
```

To tell git to use this as your ID:
```bash
git config --global user.signingkey {keyid}
```

To tell git to sign your commits by default:
```bash
git config --global commit.gpgsign true
```

On the remote repository there will be a GPG key link in your settings. Just like the public key of an SSH keypair, you need to add the GPG public key here.
You can export the key by email or ID. It's safer by ID, because you can have multiple keys with the same email address.

Note: the ID of the secret key is the same as that of the public key, they're a pair after all.
```bash
gpg --export --armor your_long_key_id
```
The next time you push code to the remote repository of your choice, you should see an indication the commit was verified.

## Tips:

### Adjusting cache timeout of your private key password 
By default when you are prompted for your password, the password is cached for 10 minutes. After 10 minutes you will be prompted again.
You can adjust this timeout by creating the following file if it does not exist:
```bash
mkdir ~/.gnupg/
cd ~/.gnupg/
touch gpg-agent.conf
```

Here you can set two values: 
- default-cache-ttl: the timeout in seconds after the last gpg activity. Default is 600sec.
- max-cache-ttl: the timeout in seconds after you last entered your password. Default is 2 hours.

For example, if you want to set the timeout for both to an hour:
```bash
default-cache-ttl 3600
max-cache-ttl 3600
```

After  this you need to restart the gpg agent:
```bash
gpgconf --kill gpg-agent
gpg-agent --daemon --use-standard-socket
```

## Further reading

- [ Why sign git commits ]( https://withblue.ink/2020/05/17/how-and-why-to-sign-git-commits.html )
- [ Changing name and author in git ]( https://www.git-tower.com/learn/git/faq/change-author-name-email )
- [ Getting started with gpg ]( https://www.redhat.com/sysadmin/getting-started-gpg )
- [ How to sign messages ]( https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages )
- [ What is GPG encryption and do you need it? ]( https://www.liquidweb.com/kb/is-gpg-still-useful-in-todays-insecure-world/ )

Spot a mistake, missing information? Submit a [PR](https://github.com/ptrck0/gpg-howto).
