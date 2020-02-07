# srx210-centurylink-fiber
Juniper SRX210 configuration for CenturyLink fiber.

```
version 12.1X46-D50.4;
system {
    host-name junos-gateway;
    time-zone UTC;
    root-authentication {
        encrypted-password "SECRET-DATA"; ## SECRET-DATA
    }
    name-server {
        205.171.3.66;
        205.171.202.166;
    }
    services {
        ssh;
        web-management {
            http {
                interface vlan.0;
            }
            https {
                system-generated-certificate;
                interface vlan.0;
            }
        }
        dhcp {
            router {
                192.168.20.1;
            }
            pool 192.168.20.0/24 {
                address-range low 192.168.20.100 high 192.168.20.254;
            }
            propagate-settings ge-0/0/0.0;
        }
    }
    syslog {
        archive size 100k files 3;
        user * {
            any emergency;
        }
        file messages {
            any critical;
            authorization info;
        }
        file interactive-commands {
            interactive-commands error;
        }
    }
    max-configurations-on-flash 5;
    max-configuration-rollbacks 5;
    license {
        autoupdate {
            url https://ae1.juniper.net/junos/key_retrieval;
        }
    }
    ntp {
        server 67.18.187.111;
        server 69.28.67.44;
        server 209.114.111.1;
        server 74.117.238.11;
    }
}
interfaces {
    interface-range interfaces-trust {
        member ge-0/0/1;
        member fe-0/0/3;
        member fe-0/0/4;
        member fe-0/0/5;
        member fe-0/0/6;
        member fe-0/0/7;
        member fe-0/0/2;
        unit 0 {
            family ethernet-switching {
                vlan {
                    members vlan-trust;
                }
            }
        }
    }
    ge-0/0/0 {
        description "CenturyLink uplink";
        vlan-tagging;
        unit 0 {
            encapsulation ppp-over-ether;
            vlan-id 201;
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 127.0.0.1/32;
            }
        }
    }
    pp0 {
        unit 0 {
            point-to-point;
            ppp-options {
                pap {
                    local-name "USERNAME@centurylink.net";
                    local-password "SECRET-DATA"; ## SECRET-DATA
                    passive;
                }
            }
            pppoe-options {
                underlying-interface ge-0/0/0.0;
                idle-timeout 0;
                auto-reconnect 10;
                client;
            }
            no-keepalives;
            family inet {
                mtu 1492;
                primary;
                negotiate-address;
            }
        }
    }
    vlan {
        unit 0 {
            family inet {
                address 192.168.20.1/24;
            }
        }
    }
}
routing-options {
    static {
        route 0.0.0.0/0 next-hop pp0.0;
    }
}
security {
    screen {
        ids-option untrust-screen {
            icmp {
                ping-death;
            }
            ip {
                source-route-option;
                tear-drop;
            }
            tcp {
                syn-flood {
                    alarm-threshold 1024;
                    attack-threshold 200;
                    source-threshold 1024;
                    destination-threshold 2048;
                    timeout 20;
                }
                land;
            }
        }
    }
    nat {
        source {
            rule-set trust-to-untrust {
                from zone trust;
                to zone untrust;
                rule source-nat-rule {
                    match {
                        source-address 0.0.0.0/0;
                    }
                    then {
                        source-nat {
                            interface;
                        }
                    }
                }
            }
        }
    }
    policies {
        from-zone trust to-zone untrust {
            policy trust-to-untrust {
                match {
                    source-address any;
                    destination-address any;
                    application any;
                }
                then {
                    permit;
                }
            }
        }
    }
    zones {
        security-zone trust {
            host-inbound-traffic {
                system-services {
                    all;
                }
                protocols {
                    all;
                }
            }
            interfaces {
                vlan.0;
            }
        }
        security-zone untrust {
            screen untrust-screen;
            host-inbound-traffic {
                system-services {
                    all;
                }
            }
            interfaces {
                ge-0/0/0.0;
                pp0.0;
            }
        }
    }
}
vlans {
    vlan-trust {
        vlan-id 3;
        l3-interface vlan.0;
    }
}
```
