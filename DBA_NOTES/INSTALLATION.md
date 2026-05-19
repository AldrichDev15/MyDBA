# ORACLE CORE
## INSTALLATION AND ARCHITECTURE

First we will configure an ORACLE 19c installation.
We use the "oracle-database-preinstall-19c" package to perform all the prerequisite setup:

`dnf install -y oracle-database-preinstall-19c`

Dependencies installed using this method:

`ksh-20120801-271.0.1.el8_10.x86_64`
KornShell (ksh) interpreter used by legacy ORACLE scripts.

`libXxf86dga-1.1.5-1.el8.x86_64`
Legacy X11 Direct Graphics Access library (mostly kept for compatibility).

`libaio-devel-0.3.112-1.el8.x86_64`
Development package for asynchronous I/O operations (used for redo logs, datafiles, high-performance disk access, etc.).

`libdmx-1.1.4-4.el8_10.x86_64`
Legacy X11 distributed multi-head extension library.

`libnsl-2.28-251.0.2.el8.x86_64`
Compatibility library for legacy network services APIs (NIS/YP).

`oracle-database-preinstall-19c-1.0-2.el8.x86_64`
Automatically configures users, groups, kernel parameters, limits.conf, required packages, and recommended system settings for Oracle 19c.

`xorg-x11-utils-7.5-28.el8.x86_64`
X11 utility tools required by some graphical installers and legacy applications.

Update the system:

`dnf update -y`

### `/etc/hosts` configuration

We add the host's FQDN (Fully Qualified Domain Name) and short hostname to the `/etc/hosts` file associated with an IP address:

`127.0.1.1    MAJULA.dba MAJULA`

This allows the local system to resolve the hostname locally without relying on external DNS services.
We use `127.0.1.1` instead of the standard loopback address (`127.0.0.1`) because some applications distinguish between `localhost` and the machine's actual hostname during hostname resolution.

After adding this entry, operations such as:

`ping MAJULA.dba`

will resolve correctly from the local machine.

Internally, many applications — including Oracle Database — rely on standard system hostname resolution mechanisms.
When the hostname is correctly defined in `/etc/hosts`, these lookups succeed immediately and return the associated IP address.

ORACLE DB depends heavily on hostname resolution for components such as ORACLE Net, listeners, TCP/IP connections, communication between instances...
If the hostname cannot be resolved correctly, some ORACLE tools and services may fail with networking or listener-related errors.

In a real network environment with a static IP address, we could replace `127.0.1.1` with the machine's actual static LAN IP address.
This would allow other machines on the network to resolve and connect to the ORACLE server as well.

