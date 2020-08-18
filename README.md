# Steps on a Mac

* You'll need this repo and https://github.com/OpenVPN/openvpn
* Start with openvpn
* Apply patch
  * `cd /path/to/openvpn && git apply ~/this/repo/openvpn-master.diff`
* Build `openvpn`
  * You will need some GNU build tools: `brew install autoconf automake shtool lzo`
  * `cp $(which shtool) ./`
  * `autoreconf -i` to generate a `configure` script
  * `./configure OPENSSL_LIBS="-L/usr/local/Cellar/openssl@1.1/1.1.1g/lib -lssl -lcrypto" OPENSSL_CFLAGS="-I/usr/local/Cellar/openssl@1.1/1.1.1g/include" --with-crypto-library=openssl`
    * You'll have to update this to point to a >= 1.1.1 version of OpenSSL
    * The paths above use an existing openssl from `brew`
  * `make && make install`
  * Check where `openvpn` was installed
    * For example,  `/usr/local/sbin/openvpn`
* Go back to aws-vpn-client
* Copy the VPN's `.ovpn` file to the directory
  * remove the `auth-federate` entry -- openvpn does not support it
* Update the `aws-connect.sh` script's envvars
  * `VPN_HOST` is in the `.ovpn` file
  *  `OVPN_BIN` is the patch `openvpn` from above
  * `OVPN_CONF` is the modified `.ovpn` you copied to this directory
* Continue with [step 2](#how-to-use) below

## Status

I was not able to get the `aws-connect.sh` script to work fully.

I see the Auth0 challege and complete it. The go server receives the SAML response and writes the file.
But, when I get to the `echo "Getting SAML redirect URL...` step, `openvpn` does not seem able to connect to the Client VPN in AWS.

By moving the `OVPN_OUT` command out of subshell, I see this:

```
2020-08-17 16:54:12 VERIFY OK: depth=0, CN=internal-vpn-sso.auth-account.nulogy.net
2020-08-17 16:54:12 Control Channel: TLSv1.2, cipher TLSv1.2 ECDHE-RSA-AES256-GCM-SHA384, 2048 bit RSA
2020-08-17 16:54:12 [internal-vpn-sso.auth-account.nulogy.net] Peer Connection Initiated with [AF_INET]3.221.141.16:443
2020-08-17 16:54:13 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:18 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:23 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:28 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:33 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:38 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:43 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:49 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:54 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:54:59 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:55:04 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:55:10 SENT CONTROL [internal-vpn-sso.auth-account.nulogy.net]: 'PUSH_REQUEST' (status=1)
2020-08-17 16:55:15 No reply from server after sending 12 push requests
2020-08-17 16:55:15 SIGUSR1[soft,no-push-reply] received, process restarting
```

I have not dug into it further.

**original README**

# aws-vpn-client

This is PoC to connect to the AWS Client VPN with OSS OpenVPN using SAML
authentication. Tested on macOS, should also work on FreeBSD/Linux with a minor changes.

See [my blog post](https://smallhacks.wordpress.com/2020/07/08/aws-client-vpn-internals/) for the implementation details.

## Content of the repository

- [openvpn-master.diff](openvpn-master.diff) - patch required to build AWS compatible OpenVPN
- [server.go](server.go) - Go server to listed on http://127.0.0.1:35001 and save
SAML Post data to the file
- [aws-connect.sh](aws-connect.sh) - bash wrapper to run OpenVPN. It runs OpenVPN first time to get SAML Redirect and open browser and second time with actual SAML response

## How to use

1. Build patched openvpn version and put it to the folder with a script
1. Start HTTP server with `go run server.go`
1. Set VPN_HOST in the [aws-connect.sh](aws-connect.sh)
1. Replace CA section in the sample [vpn.conf](vpn.conf) with one from your AWS configuration
1. Finally run `aws-connect.sh` to connect to the AWS.

## Todo

Better integrate SAML HTTP server with a script or rewrite everything on golang
