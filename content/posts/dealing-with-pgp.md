+++
title = 'Dealing With PGP'
date = 2024-10-07T20:25:38+01:00
draft = false
+++

So, you're going to use PGP (or GPG, I will be using PGP to refer to both, since they both use OpenPGP, which is a standard) for some reason. I'm not sure why you would want to [do this](https://www.latacora.com/blog/2019/07/16/the-pgp-problem/), and to be honest, I'm not going to ask, because there might be an edge case that calls for it. I can at least tell you how to do this properly without the risk of private keys leaking out.

## What am I going to need for this?

So, to get through this without risking a leakage of keys, you'll need:

- An air-gapped machine. An old laptop will work fine. Disable the Wi-fi, and if possible, remove the card, speakers and microphone to minimise leakage. Extreme? Maybe, but it'll work.

- Two USB drives. One with TAILS, and one empty one. Make it ext4, or maybe exFAT, if you need Windows to read it for some reason. If you really want to, you could LUKS encrypt it, but that might be overkill.

- Dice. Yes, I'm serious. You'll need these to generate passphrases.

- A copy of the EFF's diceware wordlist.

- An OpenGPG smartcard. Prefereably a Nitrokey 3 model, as they're cheaper than Yubikeys. Yubikeys can work in a pinch, but they are expensive.

- A notepad or a piece of paper. This is how you'll write down your passphrases until you can get to a password manager.

- A shredder, to destroy the paper after copying the passphrases into it. Trust me, do not have that paper lying around!

## A dire warning before we begin

I have been informed, late in the day that the mantainer of GnuPG has decided to break with OpenPGP and create his own standard. Do I need to explain why [it's a bad idea?](https://blog.pgpkeys.eu/security-issues-librepgp-2024-08.html). Hopefully, I don't, so we need to do some prepwork beforehand to prevent disaster.

## Prepwork?

Very much so. Don't worry, it's pretty easy to deal with. When you're on the TAILS desktop, launch the terminal, then  cd into the .gnupg directory, and then use your favorite text editor to edit gpg.conf with all this:

```
use-agent
personal-digest-preferences SHA512
cert-digest-algo SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES ZLIB BZIP2 ZIP Uncompressed
personal-cipher-preferences AES256 AES192 TWOFISH CAMELLIA256 AES
weak-digest SHA1
openpgp
```

Do note that you will need to comment out the default options that this replaces, since RiseUP hasn't updated their guide in forever and a day. Only comment out options that you're changing, just to make things less of a pain in the arse.

Save it, and then try listing the keys again. No config error should pop up. If it does, check the config for typos.

Done all that? Good. We've fixed the major issues at play.

## How do I generate the bloody key?

Okay, type in `gpg --full-generate-key --expert`, and use either number 11 (That's ECC, but setting your own capabilities",if your smartcard supports ECC and Curve 25519. If not, go with number 8 (RSA, but setting your own capabilities), and go with 4096 key size. Either way, make sure that Certify is the only capability that it has. We'll be generating subkeys later. Don't set this key to expire, either.

Once you've gone through the user ID screen (please [don't use the comment](https://web.archive.org/web/20190507155816/https://debian-administration.org/users/dkg/weblog/97) part!), You'll be asked to set your passphrase. While waiting for bootup, it might be a good idea to use the dice to generate at least a four word passphrase. Five would be a little stronger, I feel. Don't take too long entering it, otherwise it will time out.

Done? Okay. Now, we will be editing that key. Enter `gpg --expert --edit-key` along with your fingerprint or part of the ID. It'll be fine. Just run `showpref` while you're there to make sure that CAST5 and AEAD are out.

## Subkeys? What are you talking about?!

Okay. Subkeys are pretty useful. They can keep your main secret key safe, if you back it up and remove it from the keyring. We'll talk about this later on. For now, we'll focus on the subkeys.

Go ahead and enter `addkey`. If your main key is an ECC key, you'll want to go with 10 and 12 for the signing and encryption subkey. If it's RSA, then go with 4 and 6. For the authentication key? ECC users choose 11 and set Authentication as the capability, while RSA users go with 8 and do the same thing.

When it comes to key validity time, I personally go with 4 years, but you can choose how long you want. You can always extend it later if need be.

Now, once you're done, enter `save` to exit and update your key.

## Now what?

We're going to make a backup of both the private key and the subkeys. If you lose your yubikey, and get another one, you can recover from the disaster.

To do this, run `gpg -o whatever-sec.asc --export-secret-keys --armor`, then your key ID. You will need to enter your passphrase, so do that.

Once that's done, export your public key with `gpg -o whatever-pub.asc --export-keys --armor`, then your key ID. You'll be needing this later. While you're at it, I don't see any harm in running `gpg -o whatever-sec-sub.asc --export-secret-subkeys --armor` too, for a backup copy of the sub keys.

Now, plug in your smartcard. Double-check it's being seen with `gpg --card-status`. It should be working fine. Now, enter `gpg --edit-key` and your key ID once more, but this time, type `key 1` to select the first subkey.

Now enter `keytocard`, and put the first key in slot 1. After that, type `key 1` to deselect, then repeat this with `key 2`, then `key 3`. Done all that? Okay.

Move the backups you've made onto your USB flash drive. Make sure you unmount it safely, too. Maybe considerstoring it and getting FuwaMoco to guard it, or something. I dunno. Just keep it safe!

Now, you'll want to get the stubs for the subkeys. To do that, run `gpg -o whatever-sec-sub-stub.asc --export-secret-subkeys --armor`, then.. you guessed it, your key ID. That way, the stubs should be there if you need them. Although in my testing, GPG seems to be smart enough to put two and two together, but it never hurts, I say.

Now, time for a bit of testing, I feel! Backups safe and protected? Good.

We are going to run `gpg --delete-secret-key` then your Key ID. Comfirm the deletion. Then..we're going to re-import the stubs back into the keyring. As if we're setting up a new laptop or something of the sort.

To do that, run `gpg --import whatever-sec-sub-stub.asc`, then insert your smartcard dongle, and run `gpg --card-status`. It should link with the stubs and the public key and it should work fine. Back up your stub file too, and put it on the drive with your other backups too.

## So, is that everything!

It should be! Alongside compatability with other OpenPGP implementations, it should be just fine!
