On macOS 15, gpg (GnuPG) 2.4.8, libgcrypt 1.11.1.

```bash
# Import someone's key and verify fingerprint.
gpg --import their_key.pub.asc
gpg --fingerprint someone@example.com

# Encrypt a message. If you use your email address, you'll encrypt
# the message for yourself :) Long and short args.
echo "Secret lunch plans at 2pm." | gpg --encrypt --armor --recipient someone@example.com > encrypted.txt
echo "Secret lunch plans at 2pm." | gpg -ear somebody@example.com > encrypted.txt

# Decrypt
gpg --decrypt encrypted.txt
gpg -d encrypted.txt
```

### Setup

```bash
# Generating a key. This is interactive. Used the defaults, except
# a key size of 4096 bytes.
gpg --full-generate-key

# If you messed up, remove the key
gpg --delete-secret-and-public-keys you@example.com

# Export the key (for your website or email signature; --armor makes it ASCII).
gpg --armor --export you@example.com > my_key.pub.asc

# Export the PRIVATE key (be careful!)
gpg --armor --export-secret-keys you@example.com > my_key.priv.asc

# On another machine, import if you'd like
gpg --import my_key.pub.asc
gpg --import my_key.priv.asc

# List all public and private keys
gpg --list-keys
gpg --list-secret-keys
```
