# Vyos-Config
My quick VyOS firewall configuration using command line
Files should be place into /opt/vyatta/etc/config/

This is my working config which is complete with port forwarding, cake qos, custom dns server, and IPV6 support using the command line firewall OS called VyOS.

## config.boot
```
firewall {
    interface eth0 {
        in {
            ipv6-name "OUTSIDE-IN6"
            name "OUTSIDE-IN"
        }
        local {
            ipv6-name "OUTSIDE-LOCAL6"
            name "OUTSIDE-LOCAL"
        }
    }
    ipv6-name OUTSIDE-IN6 {
        default-action "drop"
        description "WAN inbound traffic forwarded to LAN"
        enable-default-log { }
        rule 10 {
            action "accept"
            description "Allow established/related sessions"
            state {
                established "enable"
                related "enable"
            }
        }
        rule 20 {
            action "drop"
            description "Drop invalid state"
            state {
                invalid "enable"
            }
        }
    }
    ipv6-name OUTSIDE-LOCAL6 {
        default-action "drop"
        description "WAN inbound traffic to the router"
        enable-default-log { }
        rule 10 {
            action "accept"
            description "Allow established/related sessions"
            state {
                established "enable"
                related "enable"
            }
        }
        rule 20 {
            action "drop"
            description "Drop invalid state"
            state {
                invalid "enable"
            }
        }
        rule 30 {
            action "accept"
            description "Allow IPv6 icmp"
            protocol "ipv6-icmp"
        }
        rule 40 {
            action "accept"
            description "allow dhcpv6"
            destination {
                port "546"
            }
            protocol "udp"
            source {
                port "547"
            }
        }
    }
    name OUTSIDE-IN {
        default-action "drop"
        rule 10 {
            action "accept"
            state {
                established "enable"
                related "enable"
            }
        }
        rule 20 {
            action "drop"
            description "Drop invalid state"
            state {
                invalid "enable"
            }
        }
        rule 30 {
            action "accept"
            description "Firewall: PiVPN"
            destination {
                address "192.168.1.210"
                port "1194"
            }
            protocol "udp"
            state {
                new "enable"
            }
        }
        rule 40 {
            action "accept"
            description "Firewall: Plex"
            destination {
                address "192.168.1.250"
                port "32400"
            }
            disable { }
            protocol "tcp"
            state {
                new "enable"
            }
        }
    }
    name OUTSIDE-LOCAL {
        default-action "drop"
        rule 10 {
            action "accept"
            description "Allow established/related"
            state {
                established "enable"
                related "enable"
            }
        }
        rule 20 {
            action "drop"
            description "Drop invalid state"
            state {
                invalid "enable"
            }
        }
    }
}
interfaces {
    ethernet eth0 {
        address "dhcp"
        address "dhcpv6"
        description "OUTSIDE"
        dhcpv6-options {
            pd 0 {
                interface eth1 {
                    address "1"
                    sla-id "0"
                }
                length "64"
            }
            rapid-commit { }
        }
        ipv6 {
            address {
                autoconf { }
            }
        }
        offload {
            gro { }
            gso { }
            sg { }
            tso { }
        }
    }
    ethernet eth1 {
        address "192.168.1.1/24"
        description "INSIDE"
        offload {
            gro { }
            gso { }
            sg { }
            tso { }
        }
    }
    loopback     lo { }
}
nat {
    destination {
        rule 11 {
            description "Port Forward: xxxx"
            destination {
                port "xxxxx"
            }
            inbound-interface "eth0"
            protocol "tcp"
            translation {
                address "192.168.1.xxx"
            }
        }
        rule 12 {
            description "Port Forward: xxxxx"
            destination {
                port "xxx"
            }
            inbound-interface "eth0"
            protocol "udp"
            translation {
                address "192.168.1.xxx"
            }
        }
    }
    source {
        rule 100 {
            outbound-interface "eth0"
            source {
                address "192.168.1.0/24"
            }
            translation {
                address "masquerade"
            }
        }
    }
}
qos {
    interface eth0 {
        egress "CAKE_UP"
    }
    interface eth1 {
        egress "CAKE_DOWN"
    }
    policy {
        cake CAKE_DOWN {
            bandwidth "900mbit"
        }
        cake CAKE_UP {
            bandwidth "22mbit"
        }
    }
}
service {
    dhcp-server {
        shared-network-name LAN {
            subnet 192.168.1.0/24 {
                default-router "192.168.1.1"
                domain-name "vyos.net"
                lease "86400"
                name-server "192.168.1.xxx"
                range 0 {
                    start "192.168.1.9"
                    stop "192.168.1.254"
                }
                static-mapping OmadaController {
                    ip-address "192.168.1.xxx"
                    mac-address "xx:xx:xx:xx:xx:xx"
                }
                static-mapping PiHole {
                    ip-address "192.168.1.xxx"
                    mac-address "xx:xx:xx:xx:xx:xx"
                }
                static-mapping PiVPN {
                    ip-address "192.168.1.xxx"
                    mac-address "xx:xx:xx:xx:xx:xx"
                }
                static-mapping TrueNAS {
                    ip-address "192.168.1.xxx"
                    mac-address "xx:xx:xx:xx:xx:xx"
                }
            }
        }
    }
    dns {
        forwarding {
            allow-from "192.168.1.0/24"
            cache-size "0"
            listen-address "192.168.1.1"
        }
    }
    ntp {
        allow-client {
            address "0.0.0.0/0"
            address "::/0"
        }
        server         time1.vyos.net { }
        server         time2.vyos.net { }
        server         time3.vyos.net { }
    }
    router-advert {
        interface eth1 {
            default-lifetime "300"
            default-preference "high"
            hop-limit "64"
            interval {
                max "30"
            }
            link-mtu "1500"
            name-server "xxxx::xxxx:xxxx:xxxx:xxxx"
            other-config-flag { }
            prefix ::/64 {
                preferred-lifetime "300"
                valid-lifetime "900"
            }
            reachable-time "900000"
            retrans-timer "0"
        }
    }
    ssh {
        port "22"
    }
}
system {
    config-management {
        commit-revisions "100"
    }
    conntrack {
        modules {
            ftp { }
            h323 { }
            nfs { }
            pptp { }
            sip { }
            sqlnet { }
            tftp { }
        }
    }
    console {
        device ttyS0 {
            speed "115200"
        }
    }
    host-name "vyos"
    login {
        user {Your name Here} {
            authentication {
                encrypted-password ""
            }
        }
    }
    name-server "eth0"
    syslog {
        global {
            facility all {
                level "info"
            }
            facility local7 {
                level "debug"
            }
        }
    }
}
```
