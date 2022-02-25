Setup your own MIB with net-snmp
--------------------------------

Here follows a description of how I setup my own MIB to be queried by net-snmp.
The main reason for this was to study how net-snmp handles the max value
of Counter64 data, which according to the RFC 2578 ch.7.1.10 should
have a max value of 2^64-1 (18446744073709551615 decimal).

Note that the repo already holds a generated and patched C code for
returning a Counter64 variable, so unless something else should be
instrumented, step 2-4 can be skipped.


0. Locate the this repo (or at least the MIBS) at $HOME/.snmp/mibs

1. Installed net-snmp-5.9.1 from source into /usr/local

2. In the $HOME/.snmp/mibs directory, generate the MIB C stub-code as:

     mib2c TAILF-TOBBE-MIB::tfTobbeNumber

3. Patch the "holes" in the generated C stub code in order to return
   whatever data you have defined in your MIB.

4. net-snmp-config --compile-subagent mysubagent tfTobbeNumber.c

5. If it compiles correctly, you are ready to start the subagent,
   but first you have to configure the SNMP daemon:

   In the /etc/snmp/snmpd.conf , make sure the agentX stufff is enabled;
   like this:

```
     #
     #  AgentX Sub-agents
     #
     # Run as an AgentX master agent
     master          agentx
     # Listen for network connections (from localhost)
     # rather than the default named socket /var/agentx/master
     agentXSocket    tcp:localhost:1705
```

6. Restart the snmpd server: sudo systemctl restart snmpd

7. Now start your subagent as: ./mysubagent -f -Lo -x  tcp:localhost:1705

8. You should now be able to retrieve data from your MIB, like in:

```
    sudo snmpwalk -v2c -c testing -m +TAILF-TOP-MIB:TAILF-TOBBE-MIB 127.0.0.1 TAILF-TOBBE-MIB::tfTobbeMIB
    TAILF-TOBBE-MIB::tfTobbeNumber.0 = Counter64: 18446744073709551615
```
