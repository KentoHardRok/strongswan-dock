# strongSwan IKEv2 VPN on Docker

Base docker image to run a strongSwan IKEv2 VPN (PSK and certs) server.

## Server Setup

### Gather necessary files

Prepare configuration file according to [Quericy Eden*](https://quericy.me/blog/512/),
then place the files in a folder like the following directory structure.

If you do not need to verify by certificate,
you can ignore the three certificate directories.

```
.
├── ipsec.conf
│
├── ipsec.secrets
│
├── strongswan.conf
│
├── cacerts
│   └── ca.cert.pem
│
├── certs
│   ├── client.cert.pem
│   └── server.cert.pem
│
└── private
    ├── client.pem
    └── server.pem
```

### Start Docker container

Running this particular Docker container typically requires
running with elevated privileges `--privileged`. It will have permission to
modify your Docker host's sysctl and iptables configuration.

Ensure the config folder is in your current directory ($PWD) and run:

    docker build -t strongswan https://github.com/Zimong/docker-strongswan

    docker run -d \
      --restart always \
      --privileged \
      -p 500:500/udp \
      -p 4500:4500/udp \
      -v $PWD:/etc/ipsec.d \
      --name=strongswan \
      strongswan

<!--
### Check status

There are various ways to check on StrongSwan, including tailing
the Docker logging output (stdout/stderr), the `ipsec` command,
and the `swanctl` command:

    docker logs -f --tail 100 strongswan
    docker exec -it strongswan ipsec statusall
    docker exec -it strongswan swanctl --list-sas

## Client Setup

### OSX 10.12 Sierra

Crypto: IKEv2 AES256-SHA256-MODP2048

On OSX, you'll need to import and trust the client certificate
and the CA certificate from `config/ipsec.d/cacerts/caCert.pem`
and possibly the exported .p12 file if you are using
certificate-based authentication. For iOS, you can email yourself
the pem and .p12 and import them as a new profile onto the device.

* For eap password, select Username for the authentication type
* For eap cert, select Certificate for the authentication type
* For pubkey cert, select None for the authentication type and select cert

### Windows 10

Crypto: IKEv2 AES256-SHA256-MODP1024

For Windows, you'll need to import the certificates into your
trusted root CA store (Machine) from the exported .p12 file.

After creating the VPN connection, go to the properties for the
network connection, click on the Networking tab, and go to the
IPv4 connection properties. Click Advanced and check the box,
"Use default gateway on remote network" to allow tunneling.

To enable IPv6 gateway you'll need aministrator access. Run
the first command to find your interface number (`Idx`) which
should be named the same as the name of your VPN connection.

    netsh int ipv6 show interfaces
    netsh interface ipv6 add route ::/0 interface=INTERFACE_NUMBER

In Windows, modp2048 Diffie Hellman is disabled by default, you
can change this behavior by creating or setting the following
registry REG_DWORD to a (Hex) value of 0, 1 or 2. No reboot should
be required.

    HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters\NegotiateDH2048_AES256

* 0 = disable (default)
* 1 = enable modp2048
* 2 = enforce modp2048 and aes-256-cbc

### iOS 10.x

Crypto: IKEv2 AES256-SHA256-MODP2048

To setup, go to Settings -> General -> VPN. Add a new VPN configuration
with type "IKEv2". Enter a description, server, remote ID, and local
ID. Local ID should typically be your username. For authentication,
you can select "Username" for EAP+mschapv2, "Certificate" for EAP+tls, or
"None" for pubkey or PSK-based authentication.

### Android

Crypto: IKEv2 CHACHA20POLY1305-PRFSHA256-ECP256 (via strongSwan VPN Client)

Native Android VPN on Android 5 Lollipop and Andorid 6 Marshmallow is
limited to IKEv1 which is not supported in this configuration.

Most users should consider using the excellent [strongSwan VPN Client](https://play.google.com/store/apps/details?id=org.strongswan.android&hl=en).

### Linux

Crypto: IKEv2 CHACHA20POLY1305-PRFSHA256-NEWHOPE128

In addition to serving VPN connections, StrongSwan will act as a client.
You can use this Docker image on Linux to act as a client (and it can act
as a client and server simultaneously). (Note: You should only run one
instance of the strongswan Docker container per host.)

To configure a StrongSwant client to be used with this Docker image,
you can use same configuration for the server (above), namely:
`ipsec.conf`, `ipsec.secrets`, `strongswan.conf`, and any generated
cacerts and client certificates in `ipsec.d/cacerts` and `ipsec.d/certs`.

The `ipsec.conf` file has rules for a client connection called
"home". If you haven't already, start up the Docker container the same
way you would when starting it for the server (above) and issue
commands such as the ones below to connect, check status, and
disconnect:

    docker exec -it strongswan ipsec up home
    docker exec -it strongswan ipsec status
    docker exec -it strongswan ipsec down home

You can also choose to install the StrongSwan package from your
Linux distribution and use the files in `config/` as a reference
for setting it up. If you're looking for a GUI, there's a
NetworkManager plugin (in Ubuntu, the package is called
`network-manager-strongswan`).

## Other Info

### Cipher Suites

This configuration attemps to balance higher grade ciphers with
performance and compatibility with the lastest versions of OSX,
Windows, Linux, and mobile devices. The list of ciphers is intentionally
kept short, users can always modify the list if higher grade
encryption is desired.

This build uses OpenSSL rather than LibreSSL in order to support
the NIST Elliptic Curves (ecp256, etc). In addition, OpenSSL provides
the same functionality provided by gmp.

I would love hearing any feedback regarding the ciphers or overall
ways we can improve the current settings.
-->

### References

Useful resources:

* [Ubuntu、CentOS搭建IPSec/IKEv2 VPN服务器全攻略](https://quericy.me/blog/512/)
* [strongSwan IKEv2 VPN setup](https://github.com/truemetal/ikev2_vpn)
<!--
* [Conn Reference](https://wiki.strongswan.org/projects/strongswan/wiki/ConnSection)
* [FARP for IPv6](https://wiki.strongswan.org/issues/1008)
* [IKEv2 Cipher Suites](https://wiki.strongswan.org/projects/strongswan/wiki/IKEv2CipherSuites)
* [StrongSwan Autoconf](https://wiki.strongswan.org/projects/strongswan/wiki/Autoconf)
* [StrongSwan ikev2 Tests](https://www.strongswan.org/testresults.html)
* [StrongSwan on Windows 7](https://wiki.strongswan.org/projects/strongswan/wiki/Windows7)
-->
