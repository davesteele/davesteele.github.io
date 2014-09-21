---
layout: post
title:  "Anatomy of a GPG Key"
date:   2014-09-20 21:29:00
categories: gpg
---

I have a confession to make. I had a fairly hard time understanding all of the ins and outs of managing keys using the gnupg tool gpg. Pretty much all of the documentation is procedural - how to use the tool to accomplish some specific tasks. Many questions that I had were tangential to the particular procedure, and therefore not covered where I needed it to be.

For me, the key to understanding how to work with gpg was to understand the packet structure of the underlying OpenPGP Message Format ([RFC4880](http://tools.ietf.org/html/rfc4880)), which defines how gpg messages, signatures, and key material are stored. The goal of this post is to grease the skids for the next guy, by tying the key storage format to the RFC definition, and to the associated gpg commands and parameters.

### TL;DR

Here are some takeaways I would have loved to have had going into this:

* Most key parameters are stored in the self signature. That means they can be changed at will by the key owner without affecting the status of external key signatures.
* Subkeys need only be self-signed (which is automatic). Trust from external signatures is provided transitively.


## Some Terms
It's best that you have an understanding of data encryption and data signing using public key cryptography before you read this. You should also know about key signing and the web of trust. Oh, and also binary-to-hexadecimal conversion for one (small) part. Having said that, let's be clear on a couple of terms:

* Master key vs. subkey - A PGP key may contain other information in addition to the key itself. A subkey is a key that is stored as a sub-component of another key. The Master key is the top level key.
* Public key - This post is working with the published version of the key structure. Therefore, only public keys are described (the ones that encrypt and create signatures). Your local version of your key also includes the associated private keys (for decryption and signature verification), to define the key pair.
* Key structure - Part of the challenge of understanding gpg key management documentation is the flexibility in the definition of the word 'key'. It can refer to a specific private or public key, or to a particular key pair, or to the OpenPGP structure that defines a suite of information associated with a key or set of keys. I will use the term "key/public key" and "key structure" to distinguish between the possible interpretations. Key pairs and private keys will not come up here. "Key structure' is not a term you will see outside of this post.
* Key ID - A hexadecimal string that identifies a key (usually the master key).
* UID, or User ID - The name and email of the user is stored in one or more UID entries, stored under the Master key.
* Certification vs. signing - 'Signing' is an action against arbitrary data. 'Certification' is the signing of another key. Ironically, the act of certifying a key is universally called "key signing". Just embrace the contradiction.
* Key packet - 'Packet' is the term used by RFC4880 to identify a component of the message format. Messages and keys structures are made up of packets and subpackets of various types.

## My Key

Following is an annotated and edited dump of my key (actually 'key structure', per my parlance), originally generated with:

    gpg -a --export "David Steele" | gpg --list-packets --verbose

where "David Steele" matches a UID for my key. Substitute your name or master key id to see your key structure. Add the '\-\-debug 0x02' option to the second gpg invocation to see the entire contents, including the binary key data (thanks [superuser.com][]).

[superuser.com]: http://superuser.com/questions/696941/human-readable-dump-of-gpg-public-key

This is a pretty standard published key structure, which is to say that it contains a master certification/signing public key, with a public subkey dedicated to encryption (GPG always creates a separate encryption subkey to the master, to avoid [problems][]). I also have an extra public signing subkey with an expiration date, for a couple of [reasons][].

[problems]: http://serverfault.com/questions/397973/gpg-why-am-i-encrypting-with-subkey-instead-of-primary-key
[reasons]: https://wiki.debian.org/Subkeys#Why.3F

I've linked aspects of the key dump to explanation paragraphs below.

-----

> \# [Master key](#MasterKey)  
> :public key [packet](#PacketType):  
> &nbsp;&nbsp;&nbsp;&nbsp;[version 4](#PacketVer), [algo 1](#KeyAlg), created [1281838967](#DatesExpir), expires 0  
> &nbsp;&nbsp;&nbsp;&nbsp;pkey[0]: [4096 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;pkey[1]: [17 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;[keyid: 8A3171EF366150CE](#KeyId)  

> \# [User ID #1](#UID1)  
> :user ID packet: "David Steele <daves@users.sourceforge.net\>"  

> \# [Endorsing Signatures](#EndorsingSigs)  
> \# [Self Signature](#SelfSig)  
> :signature packet: algo 1, keyid 8A3171EF366150CE  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1281838967, md5len 0, [sigclass 0x13](#SigClass)  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 8, begin of digest f9 34  
> &nbsp;&nbsp;&nbsp;&nbsp;[hashed subpkt](#SigSubpacket) 2 len 4 (sig created 2010-08-15)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 27 len 1 ([key flags](#KeyFlags): 03)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 11 len 4 ([pref-sym-algos](#PrefSymAlgos): 9 8 7 3)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 21 len 4 ([pref-hash-algos](#PrefHashAlgos): 10 9 8 11)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 22 len 4 (pref-zip-algos: 2 3 1 0)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 30 len 1 ([features](#Features): 01)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 23 len 1 ([key server preferences](#KeyServPrefs): 80)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID 8A3171EF366150CE)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [4092 bits]  
> \# [External Signatures](#ExternalSigs)  
> :signature packet: algo 17, keyid F7EBEE8EB7982329  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1284226833, md5len 0, sigclass 0x10  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 2, begin of digest fe c2  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 2 len 4 (sig created 2010-09-11)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID F7EBEE8EB7982329)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [160 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [157 bits]  
> :signature packet: algo 17, keyid DABDF1E4C88B0C9C  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1349019663, md5len 0, sigclass 0x13  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 2, begin of digest 1b fd  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 2 len 4 (sig created 2012-09-30)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID DABDF1E4C88B0C9C)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [160 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [158 bits]  

> \# User ID #2  
> :user ID packet: "David Steele <dsteele@gmail.com\>"  
> :signature packet: algo 1, keyid 8A3171EF366150CE  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1322364912, md5len 0, sigclass 0x13  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 2, begin of digest e6 05  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 2 len 4 (sig created 2011-11-27)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 27 len 1 (key flags: 03)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 11 len 5 (pref-sym-algos: 9 8 7 3 2)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 21 len 5 (pref-hash-algos: 8 2 9 10 11)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 22 len 3 (pref-zip-algos: 2 3 1)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 30 len 1 (features: 01)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 23 len 1 (key server preferences: 80)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID 8A3171EF366150CE)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [4096 bits]  
> :signature packet: algo 17, keyid F7EBEE8EB7982329  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1322413422, md5len 0, sigclass 0x10  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 2, begin of digest b4 2e  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 2 len 4 (sig created 2011-11-27)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID F7EBEE8EB7982329)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [157 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [159 bits]  

> \# [Encryption subkey](#EncryptSubkey)  
> :public sub key packet:  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, algo 1, created 1281839112, expires 0  
> &nbsp;&nbsp;&nbsp;&nbsp;pkey[0]: [4096 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;pkey[1]: [17 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;keyid: 2DC87C4C0D929394  
> :signature packet: algo 1, keyid 8A3171EF366150CE  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1281839112, md5len 0, sigclass 0x18  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 8, begin of digest 87 3b  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 2 len 4 (sig created 2010-08-15)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 27 len 1 (key flags: 0C)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID 8A3171EF366150CE)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [4096 bits]  

> \# [Signing subkey](#SigningSubkey)  
> :public sub key packet:  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, algo 1, created 1408105689, expires 0  
> &nbsp;&nbsp;&nbsp;&nbsp;pkey[0]: [4096 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;pkey[1]: [17 bits]  
> &nbsp;&nbsp;&nbsp;&nbsp;keyid: 627EBB290A817A82  
> :signature packet: algo 1, keyid 8A3171EF366150CE  
> &nbsp;&nbsp;&nbsp;&nbsp;version 4, created 1408105689, md5len 0, sigclass 0x18  
> &nbsp;&nbsp;&nbsp;&nbsp;digest algo 2, begin of digest 62 1d  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 2 len 4 (sig created 2014-08-15)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 27 len 1 (key flags: 02)  
> &nbsp;&nbsp;&nbsp;&nbsp;hashed subpkt 9 len 4 (key expires after 5y0d0h0m)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 16 len 8 (issuer key ID 8A3171EF366150CE)  
> &nbsp;&nbsp;&nbsp;&nbsp;subpkt 32 len 540 (signature: v4, class 0x19, algo 1, digest algo 2)  
> &nbsp;&nbsp;&nbsp;&nbsp;data: [4094 bits]  

-----


## <a name="MasterKey"></a>Master Key

The first packet in a published OpenPGP/gpg key structure is the master signing/certification public key. The overall key structure is referenced by the Key ID of this key.

### <a name="PacketType"></a>Packet Type

The 'types' of packets in an OpenPGP key structure or message are defined in Section 5 of the RFC ([RFC4880-5][]). The master signing public key uses a packet with a 'tag' value of 6 ([RFC4880-5.5.1.1][]). The tag values are not shown in the gpg key dump - just the resulting type.

[RFC4880-5]: http://tools.ietf.org/html/rfc4880#section-5
[RFC4880-5.5.1.1]: http://tools.ietf.org/html/rfc4880#section-5.5.1.1


### <a name="PacketVer"></a>Packet Version

The RFC defines both 'version 3' and 'version 4' packet types. For key packets, a modern gpg will only create version 4 packets ([RFC4880-5.5.2][]). You should avoid working with version 3 keys - there are known weaknesses with the format.

[RFC4880-5.5.2]: http://tools.ietf.org/html/rfc4880#section-5.5.2

### <a name="KeyAlg"></a>Key Algorithm

The "algo" parameter in the dump identifies the encryption algorithm associated with the key packet. The list of required and possible algorithms is listed in the  "Constants" section of the RFC ([RFC4880-9.1][]). 

[RFC4880-9.1]: http://tools.ietf.org/html/rfc4880#section-9.1

That section states that 'DSA' (for signing) and 'Elgamal' (for encryption) can be expected to be the most available across implementations and versions. My key uses RSA (encrypt or sign), which is the current default in gpg:

    $ gpg --gen-key
    gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    
    Please select what kind of key you want:
       (1) RSA and RSA (default)
       (2) DSA and Elgamal
       (3) DSA (sign only)
       (4) RSA (sign only)
    Your selection? 1
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 4096
    ...

The information stored for the key (pkey[0] and pkey[1] in this case) is dependent on the algorithm.

The length of the key is important. Mine is 4096 bits, based on the current [recommendation][KeyLengthRecommendation] from Debian (as an exercise, you can verify that the key listed in that document is RSA).

[KeyLengthRecommendation]: http://keyring.debian.org/creating-key.html

### <a name="DatesExpir"></a>Dates and Expiration

The dates in the key structure dump are [Unix epochs][epoch]. Convert to human-readable with:

[epoch]: http://en.wikipedia.org/wiki/Unix_time

    $ date -d @1281838967
    Sat Aug 14 22:22:47 EDT 2010

The 'expires' value of '0' means that the key itself has no expiration date. Note that we will find out shortly that there are multiple ways to express key expiration. This mechanism is part of the key definition - changing it would change the Key ID for the Master key, which would in turn invalidate all signatures for the key structure.

Perhaps for that reason, gpg does not use this field to define the expiration date for a generated key. It is expressed in the key self-signature, as shown later.

There has been a fair amount of mulling about the right strategy for key expiration. My sense it that the consensus is that opinion on expiring Master keys is mixed, encryption subkeys should have no expiration, and signing subkeys should.

### <a name="KeyId"></a>Key ID

The RFC defines a 160-bit 'fingerprint' for a key, which is typically expressed as a hexadecimal string, divided into 10 4-character groups ([rfc4880-12.2][]). When validating keys for key signing, the fingerprint is used.

[RFC4880-12.2]: http://tools.ietf.org/html/rfc4880#section-12.2

    $ gpg --fingerprint "David Steele"
    pub   4096R/366150CE 2010-08-15
          Key fingerprint = AE0D BF5A 92A5 ADE4 9481  BA6F 8A31 71EF 3661 50CE
    uid                  David Steele <dsteele@gmail.com>
    uid                  David Steele <daves@users.sourceforge.net>
    sub   4096R/0D929394 2010-08-15
    sub   4096R/0A817A82 2014-08-15 [expires: 2019-08-14]

The key structure dump is expressing this fingerprint as a 'key id', taking the last 16 characters of that fingerprint (again, [rfc4880-12.2][]).

The gpg program muddies the waters a bit by using the last 8 characters of the fingerprint as its definition of the key id, shown on the 'pub' line for the fingerprint call above. It is using the definition of key id from section 3.3 ([rfc4880-3.3][]).

[RFC4880-3.3]: http://tools.ietf.org/html/rfc4880#section-3.3

Going one step further down the rabbit hole, in some contexts this value needs to have "0x" prepended ('0x366150CE'). I've run across this in a key server search function.

The key id is a shorthand method for referring to a particular key or key structure. The 8-character version is the primary mechanism for referring to a particular key.

The Key ID of the Master public key ('366150CE' in this case) is used to refer to some of its own subkeys, such as the associated private signing key, as well as the encryption subkey.

The fingerprint/key id is a hash of the entire key packet, and only the key packet. It is invalidated (changed) if any information in the key packet is changed, but is unaffected by any changes in any other packets.

## <a name="UID1"></a>User ID

The user ID packet defines a name/email address that is associated with the key structure ([RFC4880-5.11][]). The gpg program will store it in [RFC2822][] format ("David Steele <dsteele@gmail.com\>") based on the name and email you provide it when you generated the key.

[RFC2822]: http://tools.ietf.org/html/rfc2822#section-3.4

[RFC4880-5.11]: http://tools.ietf.org/html/rfc4880#section-5.11

The key structure can have more than one user id. For instance, if you want to use the key structure with more than one email account, multiple user ids would be needed.

### <a name="EndorsingSigs"/>Endorsing Signatures

A user id packet is followed by one or more 'signature packets'. 

This is a key point - the key signature covers the contents of the master key packet and the last-encountered user id packet. So, the signature is a certification by the signer that the user identified in the user id is who he says he is, and that he claims ownership of the key.

A key signature only covers one user id. Separate signatures are needed if certification is desired for multiple user ids.

Signatures are identified by the 'signature type', shown as 'sigclass' in the packet structure dump. Types/classes 0x10 through 0x13 are key signatures ([RFC4880-5.2.1][]).

[RFC4880-5.2.1]: http://tools.ietf.org/html/rfc4880#section-5.2.1

Key signatures can be created using the 'sign-key' command in the key edit mode. 

### <a name="SelfSig"/>Self Signature

The 'self signature' is a key signature generated by the master key being signed. It serves as verification that the user id is valid for that key. For version 4 packets, it also provides a number of parameters to be associated with the key/user id pair ([RFC4880-5.2][]).

[RFC4880-5.2]: http://tools.ietf.org/html/rfc4880#section-5.2

Self signatures are generated automatically by gpg, as keys are generated as parameters are changed.

### <a name="SigClass"/>Signature Class

Again, the key signature type, or 'sigclass', identifies the signature as a key signature. It also claims to provide a level of assurance of that certification, from "does not make any particular assertion" to "substantial verification of the claim of identity ([RFC4880-5.2][]).

It appears that they are largely unused for key validation purposes. The Debian key signing guidlines recommends using '[casual][]', which I believe maps to 0x12. However, the RFC states:

[casual]: https://www.debian.org/events/keysigning

> Most OpenPGP implementations make their "key signatures" as 0x10
> certifications.  Some implementations can issue 0x11-0x13
> certifications, but few differentiate between the types.

I have both 0x10 and 0x13 key signatures in my key structure.


### <a name="SigSubpacket"/>"Endorsing Signature" Subpackets

Here is where things got interesting for me, and enlightening. The key self signature contains subpackets with additional information about the key and how it is to be used ([RFC4880-5.2][]). That means such information is validated by the master key, but that the information is unaffected by (and unaffecting of) other externally-generated key signatures. Some of these subpackets are detailed in the following sections, with the gpg mechanisms for manipulation. The full list of subpackets is in [RFC4880-5.2.3.1][]

[RFC4880-5.2]: http://tools.ietf.org/html/rfc4880#section-5.2
[RFC4880-5.2.3.1]: http://tools.ietf.org/html/rfc4880#section-5.2.3.1

#### <a name="KeyFlags"/>Key Flag Subpacket

I've said a couple of times now that the master key is used for signing, and there is a separate encryption subkey. Well, it is the 'keyflags' field that defines these roles for the keys ([RFC4880-5.2.3.21][]).

[RFC4880-5.2.3.21]: http://tools.ietf.org/html/rfc4880#section-5.2.3.21

The flag data is reflected in the output of gpg when you invoke \-\-edit-key:

    $ gpg --edit-key "David Steele"
    gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    
    Secret key is available.
    
    pub  4096R/366150CE  created: 2010-08-15  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    sub  4096R/0D929394  created: 2010-08-15  expires: never       usage: E   
    sub  4096R/0A817A82  created: 2014-08-15  expires: 2019-08-14  usage: S   
    [ultimate] (1). David Steele <dsteele@gmail.com>
    [ultimate] (2)  David Steele <daves@users.sourceforge.net>
    
The 'usage' characters map to the key flags as follows:

Flag | gpg character | Description
:--- | :-----------: | :----------
0x01 | "C"           | Key Certification
0x02 | "S"           | Sign Data
0x04 | "E"           | Encrypt Communications
0x08 | "E"           | Encrypt Storage
0x10 |               | Split key
0x20 | "A"           | Authentication
0x80 |               | Held by more than one person

If you look in my key structure dump, you'll see that my master key's key flag is 0x03, which is Key Certification plus Sign Data. The encryption subkey is 0x0C, which is Encrypt Communications plus Encrypt Storage. The signing subkey is 0x02, or Sign Data.

There is not much to work with on this flag, under the normal gpg mode. I don't believe that it is editable in gpg, and the values are created automatically on key creation. Also, there is no option on subkeys to create a key which can both sign and encrypt.

It is possible to set these flags on creation, if you use the gpg "\-\-expert" option along with "\-\-edit-key". Additional key algorithms are shown on the edit "addkey" command which permit toggling some key flags.

#### <a name="PrefSymAlgos"/>Preferred Symmetric Algorithms

First, be aware that when you use public key algorithms, you are actually managing a symmetric encryption key, which is used to do the actual work of encrypting your message. The why of that is out of scope here.

The defined list of algorithms is at [RFC4880-9.2][]. You will probably not need to work with this list. I note with curiosity that TripleDES, the only required algorithm, is not on my list.

[RFC4880-9.2]: http://tools.ietf.org/html/rfc4880#section-9.2

#### <a name="PrefHashAlgos"/>Preferred Hash Algorithms

The hash algorithms available are defined at [RFC4880-9.4][]. These are used for signatures. MD5 should definitely not be in the list. For compatibility, SHA-1 should be in the list somewhere. Note that this is not current [Debian practice][] (see the section on gpg.conf).

[Debian practice]: http://keyring.debian.org/creating-key.html

#### <a name="Features"/>Key Features

"Features" doesn't yet serve much purpose ([RFC4880-5.2.3.24][]).

[RFC4880-5.2.3.24]: http://tools.ietf.org/html/rfc4880#section-5.2.3.24

#### <a name="KeyServPrefs"/>Key Server Preferences

The key server preferences subpacket is also a bit light ([RFC4880-5.2.3.17][])

[RFC4880-5.2.3.17]: http://tools.ietf.org/html/rfc4880#section-5.2.3.17

### <a name="ExternalSigs"/>External Key Signatures

Following the self signature are the externally generated key signatures. You get these when you import a copy of your key that has been signed remotely and exported by a person who is certifying your key structure/user id. Note that I show two signatures from the same user/key, ('F7EBEE8EB7982329'), certifying my two user id's independently.

## <a name="EncryptSubkey"/>Encryption 

You can see that this was created at the same time as the master key. As mentioned previously, this is created automatically with \-\-gen-key.

The parameters have already been discussed.

## <a name="SigningSubkey"/>Signing Subkey


This signing subkey was created in 2014. It, and the encryption subkey, are only self-signed. That alone allows them to inherit the trust from the top level master keys defined by the other master key signatures.

The new wrinkle in this key is an expiration subpacket on the self signature, which puts a time limit on the certification ([RFC4880-5.2.3.6][]).

[RFC4880-5.2.3.6]: http://tools.ietf.org/html/rfc4880#section-5.2.3.6

----------

Should you have made it this far, I hope you have found this useful. Please let me know if you find any issues with the explanations.

