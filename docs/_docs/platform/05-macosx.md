---
title: Mac OS X
head_inline: "<style> .blue { color: blue; } </style>"
---

This guide is based on **macOS Catalina 10.15**.
{: .blue}

### Installing Homebrew
---

```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Getting MongoDB
---

Install MongoDB with Package Manager.
```bash
$ brew tap mongodb/brew
$ brew install mongodb-community
```

Run MongoDB server.
```bash
$ mongod --config /usr/local/etc/mongod.conf
```

**Tip:** MongoDB is persistent after rebooting with the following commands:
`$ brew services start mongodb-community`
{: .notice--info}


### Setting up TUN device (No persistent after rebooting)
---

Install TUN/TAP driver
- You can download it from [http://tuntaposx.sourceforge.net/](http://tuntaposx.sourceforge.net/)
- And then, run tuntap_20150118.pkg to install TUN/TAP driver.

Configure the TUN device.
```bash
$ sudo ifconfig lo0 alias 127.0.0.2 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.3 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.4 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.5 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.5 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.6 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.7 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.8 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.9 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.10 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.11 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.12 netmask 255.255.255.255
$ sudo ifconfig lo0 alias 127.0.0.13 netmask 255.255.255.255
```

Enable IP forwarding & Masquerading
```bash
$ sudo sysctl -w net.inet.ip.forwarding=1
$ sudo sh -c "echo 'nat on {en0} from 10.45.0.0/16 to any -> {en0}' > /etc/pf.anchors/org.open5gs"
$ sudo pfctl -e -f /etc/pf.anchors/org.open5gs
```

**Tip:** The script provided in [$GIT_REPO/misc/netconf.sh](https://github.com/{{ site.github_username }}/open5gs/blob/master/misc/netconf.sh) makes it easy to configure the TUN device as follows:  
`$ sudo ./misc/netconf.sh`
{: .notice--info}

### Building Open5GS
---

Install the depedencies for building the source code.
```bash
$ brew install mongo-c-driver gnutls libgcrypt libidn libyaml libmicrohttpd curl pkg-config
```

Install Meson using Homebrew.
```bash
$ brew install meson
```

Git clone.

```bash
$ git clone https://github.com/{{ site.github_username }}/open5gs
```

To compile with meson:

```bash
$ cd open5gs
$ meson build --prefix=`pwd`/install -D c_std=c99
$ ninja -C build
```

Check whether the compilation is correct.

**Note:** This should require *sudo* due to access `/dev/tun0`.
{: .notice--danger}

```bash
$ sudo ./build/tests/attach/attach ## EPC Only
$ sudo ./build/tests/registration/registration ## 5G Core Only
```

Run all test programs as below.

**Note:** This should require *sudo* due to access `/dev/tun0`.
{: .notice--danger}

```bash
$ cd build
$ sudo meson test -v
```

**Tip:** You can also check the result of `ninja -C build test` with a tool that captures packets. If you are running `wireshark`, select the `loopback` interface and set FILTER to `s1ap || gtpv2 || pfcp || diameter || gtp || ngap || http`.  You can see the virtually created packets. [testattach.pcapng]({{ site.url }}{{ site.baseurl }}/assets/pcapng/testattach.pcapng)/[testregistration.pcapng]({{ site.url }}{{ site.baseurl }}/assets/pcapng/testregistration.pcapng)
{: .notice--info}

You need to perform the **installation process**.
```bash
$ cd build
$ ninja install
$ cd ../
```


### Building WebUI of Open5GS
---

[Node.js](https://nodejs.org/) is required to build WebUI of Open5GS

```bash
$ brew install node
```

Install the dependencies to run WebUI

```bash
$ cd webui
$ npm install
```

The WebUI runs as an [npm](https://www.npmjs.com/) script.

```bash
$ npm run dev
```

