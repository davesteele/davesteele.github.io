---
layout: post
title:  "Intermediate GPG"
date:   2015-08-01 21:44:00
categories: gpg
---

Let's say you have been using  GnuPG for some amount of time. You've generated a key for yourself, know how to put it on a keyserver, understand what public keys are used for, know what the 'web of trust' is, and the difference between encryption and signing.

Then, at some point, you have a sudden need to understand the workings of gpg in more detail. This is typically because you are about to attend your first [key signing](https://en.wikipedia.org/wiki/Key_signing_party), or because you start looking into why people keep talking about the need for '[subkeys](https://wiki.debian.org/Subkeys)'.

I found that transition bumpy. Here is a short list of topics that I believe everyone should understand for intermediate-level gpg operations.

### How to keep straight the various things called 'keys'
 
When you first created your "key" using "gpg --gen-key", you actually made a "key certificate". Key certificates typically contain multiple public key pairs, plus other information.
 
At the top of the key certificate is the 'Primary' (or 'Master') key pair, which is used for signing and certification  ("Certification" is the process of signing keys).
 
The Primary key pair has one or more User IDs (UID) associated with it. The UID is a just a string (e.g. "David Steele &lt;dsteele@gmail.com&gt;") that defines a person/entity that has access to the secret half of the key pairs.
 
Most key certificates store a separate "subkey" pair to be used for encryption. The clear divide between signing and encryption keys is done to avoid a [vulnerability](http://serverfault.com/questions/397973/gpg-why-am-i-encrypting-with-subkey-instead-of-primary-key).

So, in gpg, the word 'key' may refer to a key certificate, to one of the key pairs in the certificate, or to one of the public/private keys in a key pair, depending on the context. If you understand the roles that each of these entities provides, you should be able to keep the references straight. The rest of this document is intended to help with that.

### Key signatures explicitly cover only a single Primary public key plus a User ID
 
When someone signs your 'key', the data included in the signature operation consists only of the public Primary key in your certificate, plus one UID. No other certificate data is included in the data being signed.

This is important to know, because it means that a key signature against your key certificate/UID will remain valid for that UID as long as the Primary key and the UID have not been revoked, regardless of how the rest of the certificate is amended.

It also means that, if you have more than one UID in your certificate, multiple independent signatures are required to cover all of them.

### Trust for other keys in the certificate is provided by way of 'self signatures' using the Primary key

Since external key signatures only provides trust as far as the Primary key, another mechanism is needed to prove that the other keys in the certificates are trusted. This is provided by 'self signatures' - signatures of the other keys made using the Primary key. That provides a path of trust from the person who signed your key, through the Primary key, to each of the subkeys.

Self signatures are automatically created whenever keys are manipulated in your certificate. That is why gpg asks for the Primary key passphrase whenever you work with subkeys.

This prevalent use of self signatures means that the only time you need to worry about losing your position in the web of trust is when you add a UID or you transition to a new certificate. You are free to make any other changes without affecting your trust level.

Hearkening back to the multiple meanings of 'key' - when you attend a key signing, you are collecting signatures against your key (Primary key) in order to provide web-of-trust verifyability for the keys (Primary and subkeys) that make up your key (certificate). Make sense?

### In general, nothing is ever deleted from a key certificate
 
This is true because it is enforced by key servers. Deleting a key in your local copy of your key certificate will have no effect on the data stored and provided by public key servers.

If what you want to do is delete a key or a UID, what you do instead is create a [revocation entry](https://www.debian-administration.org/article/450/Generating_a_revocation_certificate_with_gpg) for the key/UID using 'gpg --edit-key', and republish your key certificate with 'gpg --send-key'. (Note that good key hygiene calls for pre-generating a [revocation certificate](https://www.debian-administration.org/article/450/Generating_a_revocation_certificate_with_gpg) at key creation time, and keeping it around in case it is needed)

### Key signatures generally live only in the certificate holding the key being signed
 
When someone signs your Primary key, they do so by downloading your key certificate, and then performing a signing operation with, 'gpg --sign-key'. They then send you the entire copy of your certificate, including this new signature. When you import this, gpg recognizes the duplication, and just adds the new signature to your local copy of the certificate.

### A key certificate may be referenced in gpg using the key id/fingerprint of any of the enclosed keys

By convention, your certificate is identified by the key id/fingerprint of your Primary key. But, this is just a convention - references to any of your subkeys may be used instead.

Consider my current key:


    $ gpg --edit-key "AE0D BF5A 92A5 ADE4 9481  BA6F 71EF 3661 50CE"
    ...
    pub  4096R/366150CE  created: 2010-08-15  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    sub  4096R/0D929394  created: 2010-08-15  expires: never       usage: E   
    sub  4096R/0A817A82  created: 2014-08-15  expires: 2019-08-14  usage: S   
    [ultimate] (1). David Steele <dsteele@gmail.com>
    [ultimate] (2)  David Steele <daves@users.sourceforge.net>


This key has a Primary key with the key id '366150CE', which can be used for certification (usage 'C', again for signing keys) and for signing (usage 'S'). The encryption key , '0D929394' (usage 'E') was created automatically with the certificate. I added a signing-only subkey, '0A817A82', some time later.

If I wish to encrypt data using my encryption key, I can actually use any one of these key ids to identify the key certificate. That is, on my computer, each of these commands perform exactly the same operation, encryption using key '0D929394':


    $ gpg --encrypt --recipient 366150CE foo
    $ gpg --encrypt --recipient 0D929394 foo
    $ gpg --encrypt --recipient 0A817A82 foo
    $ gpg --encrypt --recipient dsteele@gmail.com foo


Note that I am using short ids to reference keys to be used for encryption - in practice that is considered a [bad idea](https://www.debian-administration.org/users/dkg/weblog/105). Fingerprints should be used instead ('gpg --list-keys --fingerprints &lt;key reference&gt;').

### But, gpg will select the key to use automatically, regardless of the identifying key id

Again, you can use the key id of any of the keys in your certificate to identify the key certificate. But, once gpg has picked out the certificate, it ignores the specific subkey you may have specified, and instead automatically selects the key to use.

For the key listed above, there are two keys which may be used for signing, '366150CE' and '0A817A82'. Regardless of which key id may be reference in 'gpg --sign', the key '0A817A82' is always used. This is because gpg will always select the [newest valid key](http://support.gpgtools.org/discussions/problems/8919-force-subkey-for-signing#19 Jun,%202013%2012:15%20PM).

It is possible to override automatic key selection by appending the key id/finger print with an [exclamation mark](https://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html) on the command line.
 
### You should consider a number of configuration options before creating and signing keys
 
The definition of what constitutes adequate encryption complexity changes over time. If you would like to be on the leading edge of that wave, consider the following:

* 'gpg --gen-key' using the default key types ('RSA and RSA') and increase the key length from the default to a length of 4096 bits
* strengthen the hash algorithm used for signing

The mechanics for both of these changes are described on the [Debian keyring website](http://keyring.debian.org/creating-key.html). If you do update the signing hash algorithm, you should update all of the self signatures in the key certificate.

Note that 'fixing' the key length of your primary would invalidate all of the web-of-trust signatures in your key certificate. It amounts to creating a new certificate.

### Certificate configuration information is stored in the Primary key self signature

Key options, such as the preferred signing algorithm, are stored as a part of the self signature. many 'gpg --edit-key' operations may therefore result in a new self signature being appended to a key.

### Subkeys can be useful, but you don't have to deal with that before the keysigning

If you are like I was, you may be contemplating key signing parties and subkeys at the same time, and wondering how the two topics may interact. The answer is, they don't. You may decide to generate your subkeys before or after a key signing. The validity of the new subkey is identical either way.  

### 'Trust' vs. 'Validity'

The gpg application has precise definitions for these terms, which doesn't always match general usage. A key's 'validity' is defined by the web of trust provided by distributed key signatures, while 'trust' is defined on your local keyring using the 'trust' command under 'gpg --edit-key'. The interaction of these two terms is described in detail in the [GNU Privacy Handbook](https://www.gnupg.org/gph/en/manual/x334.html#AEN384).

### More detail is available.

For more detail on what key certificates look like, and how gpg works, see [Anatomy of a GPG Key](https://davesteele.github.io/gpg/2014/09/20/anatomy-of-a-gpg-key/).

