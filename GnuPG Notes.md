Generating a Key
================

Simple. Issue this:

` gpg --gen-key`

Here's a transcript of what happens next. It's all nice and interactive
:)

` gpg (GnuPG) 1.4.5; Copyright (C) 2006 Free Software Foundation, Inc.`  
` This program comes with ABSOLUTELY NO WARRANTY.`  
` This is free software, and you are welcome to redistribute it`  
` under certain conditions. See the file COPYING for details.`  
` `  
` Please select what kind of key you want:`  
` (1) DSA and Elgamal (default)`  
` (2) DSA (sign only)`  
` (5) RSA (sign only)`  
` Your selection? 5`  
` RSA keys may be between 1024 and 4096 bits long.`  
` What keysize do you want? (2048) `  
` Requested keysize is 2048 bits`  
` Please specify how long the key should be valid.`  
` 0 = key does not expire`  
` `<n>`  = key expires in n days`  
` `<n>`w = key expires in n weeks`  
` `<n>`m = key expires in n months`  
` `<n>`y = key expires in n years`  
` Key is valid for? (0) 1y`  
` Key expires at Tue 20 Dec 2011 10:52:20 AM CST`  
` Is this correct? (y/N) y`  
` `  
` You need a user ID to identify your key; the software constructs the user ID`  
` from the Real Name, Comment and Email Address in this form:`  
` "Heinrich Heine (Der Dichter) `<heinrichh@duesseldorf.de>`"`  
` `  
` Real name: Nikhil Anand`  
` Email address: anand.nikhil@gmail.com`  
` Comment: `  
` You selected this USER-ID:`  
` "Nikhil Anand `<anand.nikhil@gmail.com>`"`  
` `  
` Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O`  
` You need a Passphrase to protect your secret key.`  
` `  
` We need to generate a lot of random bytes. It is a good idea to perform`  
` some other action (type on the keyboard, move the mouse, utilize the`  
` disks) during the prime generation; this gives the random number`  
` generator a better chance to gain enough entropy.`  
` ...+++++`  
` .+++++`  
` gpg: key C236FD2B marked as ultimately trusted`  
` public and secret key created and signed.`  
` `  
` gpg: checking the trustdb`  
` gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model`  
` gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u`  
` gpg: next trustdb check due at 2011-12-20`  
` pub   2048R/C236FD2B 2010-12-20 [expires: 2011-12-20]`  
` Key fingerprint = 8E05 7113 DF16 CB7A E7A5  0422 A8E4 0177 C236 FD2B`  
` uid                  Nikhil Anand `<anand.nikhil@gmail.com>  
` `  
` Note that this key cannot be used for encryption.  You may want to use`  
` the command "--edit-key" to generate a subkey for this purpose.`

Key Rings
=========

Keys are stored in files called "key rings".

-   The `secring.gpg` file is the key ring that stores secret keys.
-   The `pubring.gpg` file is the key ring that stores public keys.

They're most likely stored in the `.gnupg` folder in your home
directory.

Viewing Keys
============

### Viewing Public Keys

Here's the naive approach:

` [root@example ~]# `**`gpg` `--list-keys`**  
` /root/.gnupg/pubring.gpg`  
` ------------------------`  
` pub   1024D/F24F1B08 2002-04-23 [expired: 2004-04-22]`  
` uid                  Red Hat, Inc (Red Hat Network) `<rhn-feedback@redhat.com>  
` `  
` pub   2048R/C236FD2B 2010-12-20 [expires: 2011-12-20]`  
` uid                  Nikhil Anand `<anand.nikhil@gmail.com>

There's nothing paranoid about checking public key fingerprints if the
issuer has provided them. For this, use the `--fingerprint` flag:

` [root@example ~]# `**`gpg` `--fingerprint`**  
` /root/.gnupg/pubring.gpg`  
` ------------------------`  
` pub   1024D/F24F1B08 2002-04-23 [expired: 2004-04-22]`  
` Key fingerprint = D8CC 06C2 77EC 9C53 372F  C199 B1EE 1799 F24F 1B08`  
` uid                  Red Hat, Inc (Red Hat Network) `<rhn-feedback@redhat.com>  
` `  
` pub   2048R/C236FD2B 2010-12-20 [expires: 2011-12-20]`  
` Key fingerprint = 8E05 7113 DF16 CB7A E7A5  0422 A8E4 0177 C236 FD2B`  
` uid                  Nikhil Anand `<anand.nikhil@gmail.com>

### Viewing Private Keys

` [root@example ~]# `**`gpg` `--list-secret-keys`**  
` /root/.gnupg/secring.gpg`  
` ------------------------`  
` sec   2048R/C236FD2B 2010-12-20 [expires: 2011-12-20]`  
` uid                  Nikhil Anand `<anand.nikhil@gmail.com>

Sharing Your Key(s)
===================

You share your public keys by exporting them to a human-readable ASCII
file. Let's use the public keyring from the previous section.

` [root@example ~]# `**`gpg` `-a` `--export` `"Nikhil` `Anand"` `>`
`Nikhil_Anand.gpg`**

This produces a file which looks like this:

` -----BEGIN PGP PUBLIC KEY BLOCK-----`  
` Version: GnuPG v1.4.5 (GNU/Linux)`  
` `  
` mQENBE0PieMBCACoEUNmHGj8TQ+Kq69PrDEN20D0p83ReN+njEUTUbFmnHvtN7Mh`  
` wVytH+pncl1F8Ji5ZbD0iHq/m8ki51QAKVPSLbk1AQ7CpGp7jUWdKlrpzSdJCea6`  
` DL4gy7feARavZSFGhoHoMuLP+gil0w2jMqzLq42DJ5qbGqMIwsZzBQoMf0dDsnf2`  
` 6Llc1HADIpz1gAe6fScmqIHmV2Q3JwtjtkK/gP0roJm8aJaYRWUawdFu/4SipjjM`  
` trBTkBwS2IJ+T0QUUiKWVR7gH33qd164gERjq5GVyfvchps1e9EVxEB++zHnX41j`  
` /klLKLh0Q+X0lGq1211tUFcvcRhlNgiB1/iBABEBAAG0JU5pa2hpbCBBbmFuZCA8`  
` YW5hbmQubmlraGlsQGdtYWlsLmNvbT6JATwEEwECACYFAk0PieMCGwMFCQHhM4AG`  
` CwkIBwMCBBUCCAMEFgIDAQIeAQIXgAAKCRCo5AF3wjb9K7plB/9Z3ZBG5UzcTaU0`  
` IycHMhN0tMHYEnl5qZYv2Vp4HDOR+xiooULvw13eUggp7e/QQiwNpxroDw+YYG1n`  
` b7CpSqYnZlMhwuTgab2ec/tFa12pEY+e7fCPFV8RJOKe2h18stuNvbbHzdC22wFz`  
` CxTavVVAKt+mNtbfNzI6OkPywGEdRB7ZSkOwq09FZbtLN0/HMtww53Du9nmX3T6r`  
` EiyONJfVz34SC3jXkrIk7rdAbTueBqmAOoWqT7o6ZXioCMGXEHPxvTs0fnZ77dnn`  
` V65qtZoAO74aUczvQtUIbFWB67T/ooZS4gYLovHn0NlU3qaE0g3avjZJN/mb7dGw`  
` nFRQ1H9f`  
` =VY6t`  
` -----END PGP PUBLIC KEY BLOCK-----`

Never, ever, ever share your private key!

Importing Keys
==============

### Other Public Keys

Simple. Let's say I wanted to import my friend Scott's public key which
he's uploaded/emailed to me.

` [root@example ~]# `**`gpg` `--import` `Scott.gpg`**  
` [root@example ~]# `**`gpg` `--fingerprint`**  
` /root/.gnupg/pubring.gpg`  
` ------------------------`  
` pub   1024D/F24F1B08 2002-04-23 [expired: 2004-04-22]`  
`       Key fingerprint = D8CC 06C2 77EC 9C53 372F  C199 B1EE 1799 F24F 1B08`  
` uid                  Red Hat, Inc (Red Hat Network) `<rhn-feedback@redhat.com>

` pub   2048R/C236FD2B 2010-12-20 [expires: 2011-12-20]`  
`       Key fingerprint = 8E05 7113 DF16 CB7A E7A5  0422 A8E4 0177 C236 FD2B`  
` uid                  Nikhil Anand `<anand.nikhil@gmail.com>

` pub   1024D/910620BF 2010-05-12`  
`       Key fingerprint = B3B6 A608 6012 F724 52C3  03F4 D085 AAC6 9106 20BF`  
` uid                  Scott `<Scott@chocolatine.org>  
` sub   4096g/29673670 2010-05-12`

Make sure you verify the fingerprint! In this case, Scott emailed me
what to expect.

### Other Private Keys

` [root@example ~]# `**`gpg` `--allow-secret-key-import` `--import`
`private.key`**

You may want to do this to import your private key on systems other than
your own. This allows you to use your public/private keypair across many
other systems. *However*, this is not ideal at all; you can see why NFS
mounting homedirs is a good idea :)

Encrypting and Decrypting Data
==============================

### Encryption

Let's say I (Nikhil Anand) wanted to send something to my friend Scott,
whose key I imported in the previous section.

` [root@example ~]# `**`gpg` `--encrypt` `--local-user` `"Nikhil`
`Anand"` `--recipient` `"Scott"` <filename>**

When you're more of a Yoda with GPG, you can significantly shorten the
longopts version above with something more terse:

` [root@example ~]# `**`gpg` `-e` `-u` `"Nikhil` `Anand"` `-r` `"Scott"`
<filename>**

### Decryption

Let's say Scott and I have exchanged our public keys. He's now sent me a
super-secret message (encrypted with my public key) I want to decrypt
(with my private key):

`[root@example ~]# `**`gpg` `--decrypt` <filename>**
