lxc-gentoo: Linux Containers Gentoo Guest Template Script
=========================================================

[![Build Status](https://travis-ci.org/globalcitizen/lxc-gentoo.png)](https://travis-ci.org/globalcitizen/lxc-gentoo)

https://github.com/globalcitizen/lxc-gentoo

The script creates a root filesystem and config
file suitable for initializing a Gentoo guest
within an LXC (Linux Containers) environment.

<img alt="LXC Boot Time Screenshot" src="https://github.com/globalcitizen/lxc-gentoo/raw/master/screenshot.jpg">

Typical startup time on modern hardware (even
without an SSD) is under half a second, and 
as hardware detection and kernel bootstrapping
is not required, the init process is largely 
IO bound (however adding more network interfaces
does increase startup latency).


Security Notes
--------------
 - Don't treat guests as root safe
 - Best practice is to be paranoid:
     - Drop most capabilities
     - Give each guest a dedicated filesystem (eg. separate LVM2 logical block device, ZFS dataset, or loopback-mounted file)
     - Do not use UIDs on the guest that intsersect with the host system
 - Make sure you never both (1) mount ```proc``` in a guest that you don't trust, and (2) have ```CONFIG_MAGIC_SYSRQ``` 'Magic SysRq Key' enabled in your kernel (which creates ```/proc/sysrq-trigger```) ... as this can be abused for denial of service
 - If you use DHCP be sure to use the default busybox DHCP daemon as your client (to avoid the bash shellshock issues)
 - If applicable to your kernel, ensure `sysctl -w kernel.core_pattern=core`. (see http://www.openwall.com/lists/oss-security/2015/04/14/4 for details)

Requirements
------------
 - Recent Linux kernel (>=3.2.x recommended, >=3.7.x actively tested)
    http://www.kernel.org/ (Gentoo: `emerge hardened-sources` / `emerge gentoo-sources` / `emerge vanilla-sources`)
     - Relevant kernel options enabled (try `lxc-checkconfig` or review the documentation at http://wiki.gentoo.org/wiki/Lxc)
 - Recent lxc userspace utilities
    (Gentoo: `emerge lxc`)
 - fuidshift
   ```
   go get github.com/lxc/lxd && go get github.com/gorilla/websocket
   git clone https://github.com/lxc/lxd
   cd lxd/fuidshift
   make && cp fuidshift /usr/local/bin
   ```

Usage
-----
While normally run interactively, the script also
accepts input from various environment variables.

 - interactive: `lxc-gentoo create`
 - interactive (with environment): `CACHE=/cache lxc-gentoo create`
 - automated: `lxc-gentoo create -q`
 - automated (with environment): `CACHE=/cache lxc-gentoo create -q`
 - automated: `lxc-gentoo create -n test-lvm -u test-lvm.local
  -a amd64 -q
  -s 10000000:10000000
  -i 192.168.3.99/24 -g 192.168.3.254
  -P /usr/portage/tree -B lvm`

Available environment variables are as follows.

<table>
 <thead>
  <tr>
   <td><b>Property</b></td>
   <td><b>Environment<br>Variable</b></td>
   <td><b>Default<br>Value</b></td>
   <td><b>Notes</b></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td><b>Cache Path</b></td>
   <td><pre>$CACHE</pre></td>
   <td><i>/var/cache/lxc/gentoo</i></td>
   <td>Stores arch/subarch/variant combo specific stage3 tarballs and the portage snapshot.</td>
  </tr>
  <tr>
   <td><b>Mirror</b></td>
   <td><pre>$MIRROR</pre></td>
   <td><i>http://distfiles.gentoo.org</i>
   <td>Specifies the location from which the stage3 tarball, portage snapshot and metadata should be fetched.</td>
  </tr>
  <tr>
   <td><b>Stage 3 tarball</b></td>
   <td><pre>$STAGE3_TARBALL</pre></td>
   <td><i></i>
   <td>Specifies the location of a custom stage3 tarball. When this option is present, fetching will be skipped</td>
  </tr>
  <tr>
   <td><b>Portage source</b></td>
   <td><pre>$PORTAGE_SOURCE</pre></td>
   <td><i></i>
   <td>path to a custom portage to use. Can be one of:
    <li> a tarball -- will be extracted</li>
    <li> a directory -- will be bind-mounted read-only</li>
    <li> "none" -- do not set up a portage tree</li>
    <li> undefined -- a portage snapshot will be downloaded and extracted into the rootfs</li>
   </td>
  </tr>
  <tr>
   <td><b>LXC Container Name</b></td>
   <td><pre>$NAME</pre></td>
   <td><i>gentoo</i></td>
   <td>Used by <i>lxc-start</i>, <i>lxc-stop</i>, etc.</td>
  </tr>
  <tr>
   <td><b>Hostname</b></td>
   <td><pre>$UTSNAME</pre></td>
   <td><i>gentoo</i></td>
   <td>May be altered by DHCP.</td>
  </tr>
  <tr>
   <td><b>IPv4 Address</b></td>
   <td><pre>$IPV4</pre></td>
   <td><i>dhcp</i></td>
   <td><i>dhcp</i> is a special value; normally format is <i>x.x.x.x/y</i>.</td>
  </tr>
  <tr>
   <td><b>IPv4 Gateway</b></td>
   <td><pre>$IPV4_GATEWAY</pre></td>
   <td><i>auto</i></td>
   <td><i>auto</i> is a special value; normally format is <i>x.x.x.x/y</i>. Ignored if <i>$IPV4</i> is <i>dhcp</i>.</td>
  </tr>
  <tr>
   <td><b>IPv6 Address</b></td>
   <td><pre>$IPV6</pre></td>
   <td><i>dhcp</i></td>
   <td><i>dhcp</i> is a special value; normally format is <i>x/y</i>.</td>
  </tr>
  <tr>
   <td><b>IPv6 Gateway</b></td>
   <td><pre>$IPV6_GATEWAY</pre></td>
   <td><i>auto</i></td>
   <td><i>auto</i> is a special value; normally format is <i>x/y</i>. Ignored if <i>$IPV6</i> is <i>dhcp</i>.</td>
  </tr>
  <tr>
   <td><b>Guest Root Password</b></td>
   <td><pre>$GUESTROOTPASS</pre></td>
   <td><i></i></td>
   <td>Will be phased out soon.</td>
  </tr>
  <tr>
   <td><b>Gentoo Architecture</b></td>
   <td><pre>$ARCH</pre></td>
   <td><i>amd64</i></td>
   <td>Gentoo architecture code: alpha, amd64, arm, hppa, ia64, ppc, s390, sh, sparc, x86</td>
  </tr>
  <tr>
   <td><b>Gentoo Architecture Variant</b></td>
   <td><pre>$ARCHVARIANT</pre></td>
   <td>(none)</td>
   <td>Usually none, <i>hardened</i> or <i>hardened+nomultilib</i></td>
  </tr>
  <tr>
   <td><b>PGP (GPG/GNUPG) directory</b></td>
   <td><pre>$PGP_DIR</pre></td>
   <td><i>$HOME/.gnupg</i></td>
   <td>Preferred key directory, or empty string (disable). See notes below.</td>
  </tr>
  <tr>
   <td><b>lxc.conf Location</b></td>
   <td><pre>$CONFFILE</pre></td>
   <td><i>${NAME}.conf</i></td>
   <td>Path at which to generate the <i>lxc.conf</i> file, one of:
    <li> undefined -- config will be placed into ./${NAME}.conf</li>
    <li> a directory -- config will be placed into dir/${NAME}.conf</li>
    <li> file path -- config will be placed into it</li>
   </td>
  </tr>
 </tbody>
</table>

PGP setup
---------

There are 3 possible setups for PGP/GPG (GNU Privacy Guard) signature checking:
 - ___off___
  - `PGP_DIR="" lxc-gentoo ...`
 - ___on with $HOME/.gnupg as the keys directory___
  - `lxc-gentoo ...`
  - `PGP_DIR="$HOME/.gnupg" lxc-gentoo ...`
 - ___on with random directory___
  - `PGP_DIR="/path/to/random/dir" lxc-gentoo ...`

__GNUPG key setup:__

You need the 'Gentoo Linux Release Engineering (Automated Weekly Release Key)'
that can be found at https://wwwold.gentoo.org/proj/en/releng/index.xml

If you wish to use a custom key storage directory, first create it as follows.
Otherwise, GPG will use your default (`$HOME/.gnupg`) directory.

```
# create and use a temporary GPG key storage dir (optional)
PGP_DIR=/path/to/random/dir
mkdir -p ${PGP_DIR}
chmod 0700 ${PGP_DIR}
alias gpg="gpg --homedir ${PGP_DIR}"
```

Now continue with the key imports.

```
# Import stage3 signing key (subkeys.pgp.net is flakey)
gpg --keyserver pool.sks-keyservers.net --recv-keys 0xBB572E0E2D182910
# Check fingerprint (2015/04 = 13EB BDBE DE7A 1277 5DFD  B1BA BB57 2E0E 2D18 2910)
gpg --fingerprint 0xBB572E0E2D182910
# Trust it
gpg --edit-key 0xBB572E0E2D182910 trust

# If you do not have the portage tree yet: import portage signing key
gpg --keyserver pool.sks-keyservers.net --recv-keys 0xDB6B8C1F96D8BF6D
# Check fingerprint (2015/04 = DCD0 5B71 EAB9 4199 527F  44AC DB6B 8C1F 96D8 BF6D)
gpg --fingerprint 0xDB6B8C1F96D8BF6D
# Trust it
gpg --edit-key 0xDB6B8C1F96D8BF6D trust
```

Be sure to verify that the keys are actually the right ones (check fingerprint with
friends, ask on IRC `#gentoo-releng`, `#gentoo`, `#gentoo-containers`, `#lxccontainers`,
visit https://wwwold.gentoo.org/proj/en/releng/index.xml )

When you're done running `lxc-gentoo`, if you used a custom key storage dir, you may 
want to reset your GPG alias and environment.

```
unalias gpg
unset PGP_DIR
```

Network Configuration Notes
---------------------------

LXC guests can have zero or more network interfaces, which can be of various
types, and each of which may be configured with zero or more addresses. They
may, regardless of the above, be granted access to zero or more external
networks, real or virtual.

As is typical of Unix (and Linux networking in particular), this basically 
means "you can probably achieve anything you set your mind to, but it's not 
going to be easy".

The `lxc-gentoo` script therefore tries to provide a reasonable default for
normal use cases, ie. by configuring guests to use one `veth`-type interface
that can be connected to the outside world via `iptables`.

Basic connectivity can be established with the following host-side commands:
 - `lxc-start -n guest -f guest.conf`
 - `ifconfig guest x.x.x.x`
   (You should now be able to ping the guest. If not, check your `guest.conf`
    network configuration versus the host-side configuration. Make sure that
    both addresses are in the same range and differ, for example the host
    may be `10.10.10.1` and the guest may be `10.10.10.2`)

Once you have established basic connectivity, external network connectivity
can be established as follows:
 - `sysctl net.ipv4.ip_forward=1` and/or `sysctl net.ipv6.ip_forward=1`
   (Optionally also set these in `/etc/systctl.conf` to persist after reboot)
 - `iptables -t nat -A POSTROUTING -o outward-interface -j MASQUERADE`
   (Where `outward-interface` is the name of the interface that carries
    traffic to/from the host and the internet, or other destination that
    you wish to allow the guest to connect to. Different distributions have
    different ways to persist these `iptables` rules, but you can use
    `iptables-save >some-ruleset` and `iptables-restore <some-ruleset` on
    any distribution)

Alternatively to a pure `iptables`-based approach, you may consider
interface bridging. A bridge interface is a layer 2 software-based bridge 
on the host to which guest interfaces may be linked. It requires some 
configuration. Here is an IPv4 example:
 - install the `bridge-utils` package (gentoo: `emerge bridge-utils`)
 - `brctl addbr br0` (create a bridge called br0)
 - `brctl setfd br0 0` (set forward delay of zero for optimisation)
 - `ifconfig br0 172.20.0.1 255.255.255.0` (select an address range)
 - `brctl addif br0 <guest-interface>` (add guest to bridge)

For further reading, the following resources are recommended:
 - `lxc.conf` man page, ie. `man 5 lxc.conf`.
 - `man 8 iptables`
 - `man 8 brctl`
 - `man 8 ifconfig`; or the modern alternative `man 8 ip`

The following notes describe LXC-specific network topology design
considerations:
 - Guest startup times will be higher if DHCP is used. In addition, DHCP use
   creates a dependency on a valid guest-external DHCP configuration which
   can compromise portability or reliability when executing in different
   environments. As such, if you are planning to use your LXC guests for
   executing what should be reliable, repeated jobs, consider avoiding DHCP.
   Basically it is nothing but a potential source of failure (eg. address
   pool exhaustion, DHCP server configuration expectation differences between
   multiple guests, etc.) and should be removed if your infrastructure can
   be configured to facilitate it. (KISS principal)
 - VLANs have been observed to sometimes come up with unavoidable delays
   (depending upon various factors such as spanning tree configuration and
   intermediate hardware/software). For this reason they are perhaps best
   left for the host system to establish connectivity with and for any LXC
   guest access to be provided via the host `iptables` configuration. The
   ideal setup will depend upon your particular use case. KISS.
 - There have been bugs regarding relative MAC addresses in other LXC 
   interface types in the past, which initially caused us to move toward 
   `veth`-style interface configuration. Bear this in mind if moving back!


Emulating other architectures with QEMU...
------------------------------------------
 - Enable `BINFMT_MISC` support in your kernel.

 - `emerge` *static* `qemu` with the relevant architecture enabled in
   `QEMU_USER_TARGETS=""`.
   Hint: Do this in a native container so you don't have left over
   static binaries on your system :)

 - Use either the `qemu`-provided `/etc/init.d/qemu-binfmt` script to set
   up the binfmt handlers or something of your own.
   Note that the ARM handler @ `qemu-binfmt` is broken
   and you will probably have to replace it with the line found here:
   https://bugs.gentoo.org/show_bug.cgi?id=407099

 - Copy the statically-linked `qemu-$ARCH` executable into the rootfs
   (do `cat /proc/sys/fs/binfmt_misc/$ARCH` to see where).

Updates
-------

___Feburary 2017___
 - Fix comment typo

___January 2016___
 - Add basic versionless ebuild (thanks to @josch09)
 - Add Travis CI automated build testing

___October 2015___
 - Add IPv6 support (largely untested)
 - Adopt new defaults ('dhcp' for IPv4/IPv6 addresses, and 'auto' for gateways)

___April 2015___
 - Add GPG signature and checksum validation
 - Add major kernel bug to security notes
 - Improved documentation

___February 2015___
 - Minor update to mirror information parsing to suit new format
 - Additional notes on loopback setup (see below)

___September 2014___
 - Cleanup old init system workarounds
 - Remove hushlogin default

___July 2014___
 - Hacky FQDN support within `UTSNAME`

___June 2014___
 - Fix to `wget` argument handling to improve reliability and performance
   on developing world or other real / half-broken internet connections

___May 2014___
 - Remove `kmod-static-nodes` from `sysinit` runlevel to avoid `openrc`
   related error messages during guest startup.

___April 2014___
 - No longer drop the `sys_boot` capability in containers, as this prevents
   `shutdown` or `poweroff` command within the container from properly
   closing the container, resulting in a hung `init` process and failure
   to recognize the container state on the host side.

___February 2014___
 - External networking documentation
 - Discourage intra-guest dynamic network configuration for portability
 - Add `/etc/rc.conf` line: `rc_provide="net"` to fix service start issues

___January 2014___
 - Resolve issues downloading stage3 archives
 - Set a default, unicode-enabled locale to silence `perl` whinging
 - Fallback to local cache when offline
 - Silence errors for antique OpenRC fixes
 - Minor fixes for recent OpenRC releases
 - Working defaults for quiet mode (`lxc-gentoo create -q`)
 - Minor typo fixes

___June 2013___
 - Bashisms
 - Cleaner syntax
 - Improved error handling
 - Speedups
 - Better/improved locking
 - More full-featured prompting
 - Portage tree bind mount support
 - Portage workspace now `tmpfs` mounted
 - More verbose download
 - Compress cache at `/var/cache/lxc/gentoo`

___January 2013___
 - Deployment of whizz-bang screenshot eyecandy.
 - Up to date OpenRC fixes for fast and minimalist
   boot (eg. newer OpenRC `net` dependency handling)
 - Additional boot verbosity with `agetty --noclear`
 - Fairly significant updates to error handling,
   which should now be relatively reliable.
 - Improved internal and external documentation.
 - Explicit inclusion of GPLv3 license text
 - Cancel the following point; we have a fairly
   large stylistic mismatch in addition to our
   use of GPLv3 and their use of LGPL.  I guess
   that's a good thing in some ways because we
   can continue to hack freely without being
   stylistically constrained ;)
 - Development of this script will soon be moving
   to the main lxc utils repository, which has
   recently moved to github.  While it has not
   yet been committed (a style review is pending
   vs. other scripts), you can find that repo
   over here: https://github.com/lxc/lxc

___November 2012___
 - Comments regarding recent kernel JIT spraying
   vulnerability: http://bit.ly/T9CkqJ
 - Various contributed minor improvements around
   locking, indentation, shell syntax, etc.
 - Don't drop `CAP_FOWNER` capability, as it breaks
   portage's ability to chown.
 - Don't create `/etc/init.d/net.eth0` unless DHCP
   is specified.

___October 2012___
 - Migrate stage3 URL from `ARCH` to `SUBARCH`
   basis, as per Gentoo Release Guidelines.

___September 2012___
 - Default network config has changed. Instead
   of assuming a bridge setup, we use simpler 
   `veth` based tunnels direct to the host,
   which now appear as ''guestname'' in the
   host's interface list.  (Also resolves an
   apparently outstanding bug related to random
   MAC assignment, see http://bit.ly/QWAkOy )
 - Generated guests now attempt to aggressively 
   drop capabilities (`man 7 capabilities`) in
   a bid to plug known security issues, also to
   pre-mount `/proc` and remove `/sys` for the same
   purpose.  (See also: http://bit.ly/SSDbY0 )
 - Add DHCP support
 - `sshd` setup code dropped as out of scope
 - More OpenRC related fixes for faster startup.
 - Various minor updates


Troubleshooting
---------------

If you do not generate your guest on a dedicated
filesystem and/or block device then you are very
likely to encounter inode exhaustion on many
default `extN`-class filesystems. Therefore, do
create a new device. A good modern solution is
to use a new ZFS dataset (`man zfs` and/or see
http://zfsonlinux.org) or an LVM2 logical volume
(`man lvcreate` and/or see http://sourceware.org/lvm2/).

However, ZFS or LVM2 are not always available. 
You can achieve a similar, more portable but lower
performance and storage efficiency result with the
nearly always available Linux loopback device driver.

First, create a 1024MB (for example) virtual
block device image file.

```
 # dd if=/dev/zero of=myguest.image bs=1MiB count=1024
```

Second, manually request the generation of a 
loopback device in order to facilitate the initial
process of filesystem creation.

```
 # losetup --show -f myguest.image
 /dev/loop0
```

Third, create an appropriate filesystem.

```
 # mkfs -t ext4 /dev/loop0
```

Finally, detach the device.
```
 # losetup --detach /dev/loop0
```

You can now mount the image in which to store your
guest as follows:
```
 # mkdir /mnt/myguest
 # mount -o loop myguest.image /mnt/myguest
 # cd /mnt/myguest
 # /path/to/lxc-gentoo create
```


History
-------

 The project was originally hosted at...
  http://sourceforge.net/projects/lxc-gentoo/
 
 It was moved to github by Julien Sanchez at:
  https://github.com/gentooboontoo/lxc-gentoo

 This was then forked again by the original 
 author in order to move project hosting to 
 github.
  https://github.com/globalcitizen/lxc-gentoo

 Since then it has seen contributions from
 many parties.
