# snmp-lab
Experiments extending the Net-SNMP agent

## Introduction
This project is to share my experience in extending the [Net-SNMP](https://en.wikipedia.org/wiki/Net-SNMP) agent by creating a custom [MIB](https://en.wikipedia.org/wiki/Management_information_base) and then populating the generated stubs from mib2c(1).

I am writing this in November, 2024 and the current source version of Net-SNMP is 5.9.4, my development box is a ThinkPad on Ubuntu 22 LTS (Jammy Jellyfish).

## Building Net-SNMP from scratch
1. Obtain and unpack the source tar ball.
1. ./configure (there will be interactive questions)
    1. I use SNMPv2c for development and use SNMPv3 for production.
    1. My configuration summary:

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
    1. binaries to /usr/local/bin and /usr/local/sbin
    1. headers to /usr/local/include/net-snmp
    1. libraries to /usr/local/lib
    1. mib to /usr/local/share/snmp/mibs

1. configuration files for snmp utilities and snmpd
    1. configuration files reside in /usr/local/share/snmp
    1. examples:
        snmp.conf (utilities)
        snmpd.conf (snmpd)
    1. copy these or use snmpconf(1) to generate your own

1. almost there!
    1. does not install as a systemd service
    1. logs to /var/log/snmpd.log

## Test run vanilla agent
1. (optional) I like to run tcpdump(8) to see the agent traps.
    1. Ensure snmpd.conf has the correct IP address for your "manager" (i.e. development machine)
    1. ```tcpdump -v port 162```

1. Run snmpd
    1. Must run as root for port 161
    1. I prefer to run snmpd(8) in the foreground while developing
    1. Start the agent like ```./snmpd -f -V``
    1. Should have seen the start trap (via tcpdump)

1. Tail the log
    1. ```tail -f /var/log/snmpd.log```
    1. Should a version message waiting for you

1. Exercise the agent
    1. ```snmpwalk -v 2c -c public localhost system```
    1. You should see something like this:
    ```
    gsc@waifu:138>snmpwalk -v 2c -c public localhost system
    MODULE-IDENTITY MACRO (lines 55..79 parsed and ignored).
    OBJECT-IDENTITY MACRO (lines 81..103 parsed and ignored).
    OBJECT-TYPE MACRO (lines 212..298 parsed and ignored).
    NOTIFICATION-TYPE MACRO (lines 302..334 parsed and ignored).
    TEXTUAL-CONVENTION MACRO (lines 8..48 parsed and ignored).
    SNMPv2-MIB::sysDescr.0 = STRING: Linux waifu 6.8.0-49-generic #49~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Nov  6 17:42:15 UTC 2 x86_64
    SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
    DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (14624) 0:02:26.24
    SNMPv2-MIB::sysContact.0 = STRING: mongohax
    SNMPv2-MIB::sysName.0 = STRING: waifu
    SNMPv2-MIB::sysLocation.0 = STRING: margaritaville
    SNMPv2-MIB::sysServices.0 = INTEGER: 12
    SNMPv2-MIB::sysORLastChange.0 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORID.1 = OID: SNMP-FRAMEWORK-MIB::snmpFrameworkMIBCompliance
    SNMPv2-MIB::sysORID.2 = OID: SNMP-MPD-MIB::snmpMPDCompliance
    SNMPv2-MIB::sysORID.3 = OID: SNMP-USER-BASED-SM-MIB::usmMIBCompliance
    SNMPv2-MIB::sysORID.4 = OID: SNMPv2-MIB::snmpMIB
    SNMPv2-MIB::sysORID.5 = OID: SNMP-VIEW-BASED-ACM-MIB::vacmBasicGroup
    SNMPv2-MIB::sysORID.6 = OID: TCP-MIB::tcpMIB
    SNMPv2-MIB::sysORID.7 = OID: UDP-MIB::udpMIB
    SNMPv2-MIB::sysORID.8 = OID: IP-MIB::ip
    SNMPv2-MIB::sysORID.9 = OID: SNMP-NOTIFICATION-MIB::snmpNotifyFullCompliance
    SNMPv2-MIB::sysORID.10 = OID: NOTIFICATION-LOG-MIB::notificationLogMIB
    SNMPv2-MIB::sysORDescr.1 = STRING: The SNMP Management Architecture MIB.
    SNMPv2-MIB::sysORDescr.2 = STRING: The MIB for Message Processing and Dispatching.
    SNMPv2-MIB::sysORDescr.3 = STRING: The management information definitions for the SNMP User-based Security Model.
    SNMPv2-MIB::sysORDescr.4 = STRING: The MIB module for SNMPv2 entities
    SNMPv2-MIB::sysORDescr.5 = STRING: View-based Access Control Model for SNMP.
    SNMPv2-MIB::sysORDescr.6 = STRING: The MIB module for managing TCP implementations
    SNMPv2-MIB::sysORDescr.7 = STRING: The MIB module for managing UDP implementations
    SNMPv2-MIB::sysORDescr.8 = STRING: The MIB module for managing IP and ICMP implementations
    SNMPv2-MIB::sysORDescr.9 = STRING: The MIB modules for managing SNMP Notification, plus filtering.
    SNMPv2-MIB::sysORDescr.10 = STRING: The MIB module for logging SNMP Notifications.
    SNMPv2-MIB::sysORUpTime.1 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.2 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.3 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.4 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.5 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.6 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.7 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.8 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.9 = Timeticks: (0) 0:00:00.00
    SNMPv2-MIB::sysORUpTime.10 = Timeticks: (0) 0:00:00.00
    ```
    1. If all this works out, congratulations you have built a working snmpd (agent).

    