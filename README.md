# r8125: Linux device driver for Realtek Ethernet controllers

This is the Linux device driver released for RealTek RTL8125 2.5Gigabit Ethernet controllers with PCI-Express interface.

## Requirements

- Kernel source tree (supported Linux kernel 2.6.x and 2.4.x)
- For linux kernel 2.4.x, this driver supports 2.4.20 and later.
- Compiler/binutils for kernel compilation

## Quick install with proper kernel settings

Unpack the tarball:

```console
$ tar vjxf r8125-8.aaa.bb.tar.bz2
```

Change to the directory:

```console
$ cd r8125-8.aaa.bb
```

If you are running the target kernel, then you should be able to do:

```console
$ ./autorun.sh	(as root or with sudo)
```

You can check whether the driver is loaded by using following commands.

```console
$ lsmod | grep r8125
$ ifconfig -a
```

If there is a device name, ethX, shown on the monitor, the linux
driver is loaded. Then, you can use the following command to activate
the ethX.

```console
$ ifconfig ethX up
```

where X=0,1,2,...

## Set the network related information

1. Set manually

	1. Set the IP address of your machine.

	   ```console
	   $ ifconfig ethX "the IP address of your machine"
	   ```

	2. Set the IP address of DNS.

	   Insert the following configuration in `/etc/resolv.conf`.

	   ```
	   nameserver "the IP address of DNS"
	   ```

	3. Set the IP address of gateway.

	   ```console
	   $ route add default gw "the IP address of gateway"
	   ```

2. Set by doing configurations in `/etc/sysconfig/network-scripts/ifcfg-ethX`
   for Redhat and Fedora, or `/etc/sysconfig/network/ifcfg-ethX`
   for SuSE. There are two examples to set network
   configurations.

	1. Fixed IP address:
	   
	   ```
	   DEVICE=eth0
	   BOOTPROTO=static
	   ONBOOT=yes
	   TYPE=ethernet
	   NETMASK=255.255.255.0
	   IPADDR=192.168.1.1
	   GATEWAY=192.168.1.254
	   BROADCAST=192.168.1.255
	   ```

	2. DHCP:
	   
	   ```
	   DEVICE=eth0
	   BOOTPROTO=dhcp
	   ONBOOT=yes
	   ```

## Modify the MAC address

There are two ways to modify the MAC address of the NIC.

1. Use `ifconfig`:

   ```console
   $ ifconfig ethX hw ether YY:YY:YY:YY:YY:YY
   ```

   where X is the device number assigned by Linux kernel, and
   `YY:YY:YY:YY:YY:YY` is the MAC address assigned by the user.

2. Use `ip`:

   ```console
   $ ip link set ethX address YY:YY:YY:YY:YY:YY
   ```

   where X is the device number assigned by Linux kernel, and
   `YY:YY:YY:YY:YY:YY` is the MAC address assigned by the user.

## Force Link Status

1. Force the link status when insert the driver.

   If the user is in the path `~/r8125`, the link status can be forced
   to one of the 5 modes as following command.

   ```console
   $ insmod ./src/r8125.ko speed=SPEED_MODE duplex=DUPLEX_MODE autoneg=NWAY_OPTION
   ```

   where

   ```
   SPEED_MODE	= 1000	for 1000Mbps
   		= 100	for 100Mbps
   		= 10	for 10Mbps
   DUPLEX_MODE	= 0	for half-duplex
   		= 1	for full-duplex
   NWAY_OPTION	= 0	for auto-negotiation off (true force)
   		= 1	for auto-negotiation on (nway force)
   ```

   For example:

   ```console
   $ insmod ./src/r8125.ko speed=100 duplex=0 autoneg=1
   ```

   will force PHY to operate in 100Mpbs Half-duplex(nway force).

2. Force the link status by using `ethtool`.

	1. Insert the driver first.
	2. Make sure that `ethtool` exists in `/sbin`.
	3. Force the link status as the following command.

	   ```console
	   $ ethtool -s ethX speed SPEED_MODE duplex DUPLEX_MODE autoneg NWAY_OPTION
	   ```

	   where

	   ```
	   SPEED_MODE	= 1000	for 1000Mbps
	   		= 100	for 100Mbps
	   		= 10	for 10Mbps
	   DUPLEX_MODE	= half	for half-duplex
	   		= full	for full-duplex
	   NWAY_OPTION	= off	for auto-negotiation off (true force)
	   		= on	for auto-negotiation on (nway force)
	   ```

	   For example:

	   ```console
	   $ ethtool -s eth0 speed 100 duplex full autoneg on
	   ```

	   will force PHY to operate in 100Mpbs Full-duplex(nway force).

## Jumbo Frame

Transmitting Jumbo Frames, whose packet size is bigger than 1500 bytes, please change mtu by the following command.

```console
$ ifconfig ethX mtu MTU
```

where X=0,1,2,..., and MTU is configured by user.

RTL8125 supports Jumbo Frame size up to 9 kBytes.
