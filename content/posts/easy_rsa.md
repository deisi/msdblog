---
title: "In House Certificate Authority with Eeasy RSA"
date: 2020-02-04T12:47:36+01:00
draft: false
tags: ["Security", "HowTo"]
images: ["/easyrsa/easy_rsa.png"]
---

![img](/easyrsa/easy_rsa.png)

Over the years I grew a fairly sophisticated network at home. Currently I have a
x86 server and a raspberry pi running 24/7. On the x86 Server there are many
services like homeassistant, tvheadend, kodi, mopidy, snapcast and a couple
more. The raspberry pi is used in my bathroom to play music and measure
humidity and temperature. Anyway, most of the services in a local network run
without encryption. Take mopidy e.g. it doesn't even have any documentation on
how to encrypt its connection and that is not even bad, because why would you
want to encrypt the control of you music system. Also because mopidy is
controlled via a web-page, it is relatively "easy" to add an encryption layer to
it using a reverse proxy setup.

So there is the question why would you want encryption throughout your local
network and honestly the best answer I have, is because I want to. Somehow I
like encryption. I like it, when the little lock in the top left corner of my
browser is green even if I just change the music or I look at the temperature
in the bathroom. It gives me some kind of satisfaction.

To do this, you could go with self singed certificates all over the place. This
would probably be the easiest, but if we want easy, we wouldn't talk about
encryption throughout the local network in the first place, right?. Thus we
**Need** a Certificate Authority or also called CA.

Now you *can* use OpenSSl for this, but somehow I could never get used to
commands like:

```
openssl req -new -sha256 -key domain.com.key -out domain.com.csr
```

Quite frankly, the command line interface of openssl is outright horrible. I
guess its very flexible and the guys who made it have done an awesome Job, if
you consider what it is used for. However it always felt very verbose for my use
case.

Thanks to god (or just the developers of OpenVPN). You can use **easy_rsa** for
this.

# What we need
- 1. CA consists out of a private key and a self singed public key that you have
  to transport somehow on all your devices. It is super important, that you
  protect the private key of your CA as good as possible.
- 2. For all the services you want to encrypt, you need another private key and
  from this private key, you will generate a *certificate singing request* often
  just called csr.
- 3. This csr needs to be transported over to your CA and subsequently singed by
  the CA. The singed csr is then called a *certificate* or often just crt.
- 4. This certificate must be transported back to the machine you run your
  service on and can then be used to start your encrypted service.

# EasyRSA

## Setting up a CA
Decide a folder where you want to store your CA. For security reasons I would
recommend to do it with root under and not as a normal user, but that is
personal taste.

```
mkdir -p /etc/easy-rsa/ca
cd /etc/easy-rsa/ca
```

Download the [openssl-easyrsa.cnf](/static/easyrsa/openssl-easyrsa.cnf) and put it in the same folder.

Next step is to initiate a new PKI and create a CA key with a self singed
certificate. with:

```
easyrsa init-pki
easyrsa build-ca
```

You **must** give a passwords at this stage. Choose a good password for this as
it is the last resort of defense for you CA and leaking this could have
devastating consequences expanding beyond the reach of your local network. What
you answer to the subsequent Common Name question is not really important. I
suggest being a little creative as it allows you to easily recognize your CA at
later times.

You now have a certificate for your CA at `pki/ca.crt`. By the way, from an
purely technical point of view this is all you need to start your own company
and sell overpriced ssl certificates to random people. This and a way to copy
your `ca.crt` over to almost every internet connected device there is.

## Private Key and Certificate for your Service

This process you must repeat for every service you want to secure. Usually you
want to do this on the machine hosting your service, but in principle it doesn't
matter, as long as you have a secure way of transporting the private key through
the network.

```
mkdir -p /etc/easy-rsa/someservice
cd /etc/easy-rsa/someservice
```

again download or copy [openssl-easyrsa.cnf](/static/easyrsa/openssl-easyrsa.cnf) into the folder and run

```
easyrsa init-pki
easyrsa gen-req servername nopass
```

This time I suggest nopass, because you want your server to be able to start
itself without you having to type in a password each time. Also here the *Common
Name* does matter. It should be the exact server name under witch you intend to
make your service reachable. E.g *service.local.net** or something similar.

**Note** it is not possible to use top level domain names. E.g. you cant have a
server reachable under *service*. This works for your network, but every browser
will reject any such certificate and all work is for nothing.


# Quellen
https://wiki.archlinux.org/index.php/Easy-RSA
https://github.com/OpenVPN/easy-rsa
https://easy-rsa.readthedocs.io/en/latest/
