# snmp-lab
Experiments extending the Net-SNMP agent

## Introduction
This project is to share my experience in extending the [Net-SNMP](https://en.wikipedia.org/wiki/Net-SNMP) agent by creating a custom [MIB](https://en.wikipedia.org/wiki/Management_information_base) and then populating the generated stubs from mib2c(1).

I am writing this in November, 2024 and the current source version of Net-SNMP is 5.9.4, my development box is a ThinkPad on Ubuntu 22 LTS (Jammy Jellyfish).

## Building Net-SNMP from scratch
1. Obtain and unpack the source tar ball.
1. ./configure (there will be interactive questions)
    1. I use SNMPv2c for development and use SNMPv3 for production.

```
---------------------------------------------------------
            Net-SNMP configuration summary:
---------------------------------------------------------

  SNMP Versions Supported:    1 2c 3
  Building for:               linux
  Net-SNMP Version:           5.9.4.pre2
  Network transport support:  Callback Unix Alias TCP UDP TCPIPv6 UDPIPv6 IPv4Base SocketBase TCPBase UDPIPv4Base UDPBase IPBase IPv6Base
  SNMPv3 Security Modules:     usm
  Agent MIB code:            default_modules =>  snmpv3mibs mibII ucd_snmp notification notification-log-mib target agent_mibs agentx disman/event disman/schedule utilities host
  MYSQL Trap Logging:         unavailable
  Embedded Perl support:      enabled
  SNMP Perl modules:          building -- embeddable
  SNMP Python modules:        disabled
  Crypto support from:        use_pkg_config_for_openssl
  Authentication support:     MD5
  Encryption support:         
  Local DNSSEC validation:    disabled

---------------------------------------------------------
```

1. make;sudo make install
binaries to /usr/local/sbin (and /usr/local/bin)
headers to /usr/local/include/net-snmp
libraries to /usr/local/lib
basic mib to /usr/local/share/snmp/mibs
snmpd configuration to /usr/local/share

does not install as a systemd service
logs to /var/log/snmpd.log

cp snmp.conf /usr/local/share/snmp


IANA has assigned enterprise 5088 to me, so the parent OID will be 1.3.6.1.4.1.5088



1. logs /var/log/snmpd.log 
1. storage /var/net-snmp
