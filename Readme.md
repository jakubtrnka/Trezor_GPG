# Doing GPG with Trezor

## GPG

GnuPG is a complete and free implementation of the OpenPGP standard as defined by RFC4880 (also known as PGP). GnuPG allows you to encrypt and sign your data and communication. It features a versatile key management system as well as access modules for all kinds of public key directories. GnuPG, also known as GPG, is a command line tool with features for easy integration with other applications.

GPG can be considered as consisting of front-end, which is responsible for handling user requests, keyring management etc., and back-end, which involves accessing private keys, caching passphrases etc. GPG, or other application, can connect to them via sockets. These sockets are by default gpg-agent.socket, gpg-agent-extra.socket, gpg-agent-browser.socket, gpg-agent-ssh.socket, and dirmngr.socket.

* The main gpg-agent.socket is used by gpg to connect to the gpg-agent that caches passphrases for private keys.
* The intended use for the gpg-agent-extra.socket on a local system is to set up a Unix domain socket forwarding from a remote system. This enables to use gpg on the remote system without exposing the private keys to the remote system.
* The gpg-agent-browser.socket allows web browsers to access the gpg-agent.
* The gpg-agent-ssh.socket can be used by SSH. Using gpg-agent-ssh.socket SSH and GPG can share the same keyring.
* The dirmngr.socket starts a GnuPG daemon handling connections to keyservers.

another, yet optional, module is pcscd module for handling connection to Smart cards. Smart cards are hardware devices that store cryptographic keys and generate signatures or encrypt documents without the keys ever leaving the device - just like Trezor. Because OpenPGP Smart Cards have been specifically developed for GPG, they have, unlike Trezor, native support inside gpg (e. g. GPG commands for managing Smart Card, moving separate private keys into the card etc.). So in order to use Trezor with GPG, we need new module that communicates with Trezor and GPG frontent. This is called [trezor-gpg-agent](https://github.com/romanz/trezor-agent).

Although Smart Cards are better integrated into GPG, they have several drawbacks:
* They act like signing black box - whatever comes into the card will be signed and user has no control over what's happening
* They are not deterministic - GPG keys generally are randomly generated numbers and it's impossible in standard user interface to generate GPG keys in deterministic way. That makes backing up the keys much less convenient, compared to Trezor where backing it up is reduced to writing down 12 words.
* GPG defaults to RSA algorithm that uses about 20x longer keys which makes it impractical to write by hand, so the backup is usually done electronically, which is fundamentally less secure than writing it down by hand
* Trezor uses key-generation passphrase - Smart Card uses access pin so it is difficult for attacker to use stolen card, but the cryptographic keys on the chip itself are unencrypted. Trezor on the other hand uses both: access pin and passphrase that encrypts the private key making it much harder for attacker to steal the private key.

## Setting up Trezor-agent

Trezor agent is python script that handles cryptographic operations
