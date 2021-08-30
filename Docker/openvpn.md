Pick a name for the `$OVPN_DATA` data volume container. It's recommended to use the `ovpn-data-` prefix to operate seamlessly with the reference systemd service.  Users are encourage to replace `example` with a descriptive name of their choosing.

```
～]# OVPN_DATA="ovpn-data"
```

Initialize the `$OVPN_DATA` container that will hold the configuration files and certificates.  The container will prompt for a passphrase to protect the private key used by the newly generated certificate authority.

```
～]# docker volume create --name $OVPN_DATA
～]# docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://vpn.sjhz.ga
～]# docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/pki


Using SSL: openssl OpenSSL 1.1.1g  21 Apr 2020

Enter New CA Key Passphrase:VKQubexXiNvZKyY4

密码设置：caobo / VKQubexXiNvZKyY4

后续测试密码
wegl9Tcm0GSb4Wsh

```

Start OpenVPN server process

```
～]# docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn
```

Generate a client certificate without a passphrase

```
～]# docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full clsovpn nopass
```

Retrieve the client configuration with embedded certificates

```
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient clsovpn > cls.ovpn
```

Client download

```
https://github.com/Tunnelblick/Tunnelblick/releases
```

