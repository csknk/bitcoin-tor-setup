Install Tor and Setup Bitcoin Core to Use an Automatic Hidden Tor Service
=========================================================================

Install Tor and setup Tor as a hidden service for Bitcoin on Ubuntu 20.04 Focal Fossa.

Usage
-----

Make `tor-setup-ubuntu.sh` executable and run as sudo.

Log out and back in for user group changes to take effect.

What the Script Does
--------------------

The script installs and sets up Tor to run as an automatic hidden service that is initiated by Bitcoin Core.

### Install Tor

Set some global variables: setting `$bitcoin_user` is necessary as the script is run as sudo by the user that will run the Bitcoin node. We get this using `$(logname)`.

```bash
bitcoin_user=$(logname)
config=/home/"${bitcoin_user}"/.bitcoin/bitcoin.conf
```

Setup `apt` to use HTTPS for sources.
```bash
apt install apt-transport-https
```

Add Tor repository sources. Note: Ubuntu Focal dropped support for 32-bit, so you need to specify the correct architecture: `[arch=amd64]`

```bash
echo "deb [arch=amd64] https://deb.torproject.org/torproject.org focal main" >> /etc/apt/sources.list
echo "deb-src [arch=amd64] https://deb.torproject.org/torproject.org focal main" >> /etc/apt/sources.list 
```

Add GPG signing key. Current 20 Jan 2021:

```bash
wget -qO- https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -
```

Install Tor and Tor debian keyring. The keyring keeps the signing key current. Tor will start automatically after install.

```bash
apt update
apt install -y tor deb.torproject.org-keyring
```

Add the original user to the debian-tor group - this is the user that will normally run Bitcoin Core.
```bash
usermod -a -G debian-tor "${bitcoin_user}"
```

Amend `/etc/tor/torrc` to allow Bitcoin Core to initiate an automatic hidden service:

```bash
echo "# Bitcoin Core access to hidden service" >> /etc/tor/torrc
echo "ControlPort 9051" >> /etc/tor/torrc
echo "CookieAuthentication 1" >> /etc/tor/torrc
echo "CookieAuthFileGroupReadable 1" >> /etc/tor/torrc
service tor reload
```

Amend Bitcoin Core config
```bash
echo "listen=1" >> "${config}"

# Only connect out to hidden services (not recommended)
#echo "onlynet=onion" >> "${config}"

# Tor nodes that will help your node find peers
echo "seednode=nkf5e6b7pl4jfd4a.onion" >> "${config}"
echo "seednode=xqzfakpeuvrobvpj.onion" >> "${config}"
echo "seednode=tsyvzsqwa2kkf6b2.onion" >> "${config}"
echo "seednode=wxvp2d4rspn7tqyu.onion" >> "${config}"
echo "seednode=bk5ejfe56xakvtkk.onion" >> "${config}"
echo "seednode=bpdlwholl7rnkrkw.onion" >> "${config}"
echo "seednode=hhiv5pnxenvbf4am.onion" >> "${config}"
echo "seednode=4iuf2zac6aq3ndrb.onion" >> "${config}"

# Limit potential DOS attacks over Tor
#echo "banscore=10000" >> "${config}"
#echo "bantime=11" >> "${config}"
```

References
----------
* [Tor support in Bitcoin Core ][1]
* [Tor installation docs][2]
* [Bitcoin Wiki: Setting up a Tor Hidden Service][3]
* [Bitcoin SE answer][4]

[1]: https://github.com/bitcoin/bitcoin/blob/master/doc/tor.md
[2]: https://2019.www.torproject.org/docs/debian.html.en
[3]: https://en.bitcoin.it/wiki/Setting_up_a_Tor_hidden_service
[4]: https://bitcoin.stackexchange.com/a/70070/56514
