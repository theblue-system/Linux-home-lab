# Linux-home-lab
This project documents the creation of a home networking lab using Ubuntu Server as a router and DHCP server. The objective is to understand networking fundamental such as DHCP.

# Ubuntu Home Lab Setup: From Installation to DHCP

## Step 1: Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Step 2: Identify Network Interfaces

Display available interfaces:

```bash
ip addr
```

Example:

* `enp0s3` → WAN interface
* `enp0s8` → LAN interface

---

## Step 3: Configure the Network with Netplan

Edit the configuration file:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configure:

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:

    # WAN Interface
    enp0s3:
      dhcp4: true

    # LAN Interface
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.20.1/24
```

Apply the configuration:

```bash
sudo netplan apply
```

Verify:

```bash
ip addr
```

Expected:

* WAN: Receives an IP from the home router.
* LAN: Static address `192.168.20.1/24`.

---

## Step 4: Install the DHCP Server

Install ISC DHCP:

```bash
sudo apt install isc-dhcp-server
```

Verify installation:

```bash
systemctl status isc-dhcp-server
```

---

## Step 5: Specify the Interface Used by DHCP

Edit:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Set:

```conf
INTERFACESv4="enp0s8"
```

Save and exit.

---

## Step 6: Configure DHCP

Edit:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Remove unnecessary example entries and add:

```conf
default-lease-time 600;
max-lease-time 7200;

authoritative;

subnet 192.168.20.0 netmask 255.255.255.0 {

    range 192.168.20.50 192.168.20.120;

    option subnet-mask 255.255.255.0;

    option broadcast-address 192.168.20.255;

    option routers 192.168.20.1;

    option domain-name-servers 8.8.8.8, 1.1.1.1;
}
```

Save and exit.

---

## Step 7: Restart the DHCP Service

```bash
sudo systemctl restart isc-dhcp-server
```

Enable it at boot:

```bash
sudo systemctl enable isc-dhcp-server
```

---

## Step 8: Verify Service Status

```bash
sudo systemctl status isc-dhcp-server
```

Expected:

```text
Active: active (running)
```

---

## Step 9: Check for Configuration Errors

Test the DHCP configuration:

```bash
sudo dhcpd -t
```

View logs:

```bash
sudo journalctl -u isc-dhcp-server
```

---

## Step 10: Connect a Client

On the client:

installation of the client os.remember to choose do not connect to internet option

## Step 11: Verify the Assigned Address

On the client:

```bash
ip addr
```

Expected:

```text
inet 192.168.20.50/24
```

(or another address within the range)

---

## Step 12: Verify Connectivity to the Server

From the client:

```bash
ping 192.168.20.1
```

Successful replies confirm that:

* The DHCP server is functioning.
* The client and server are on the same subnet.
* Layer 2 and Layer 3 communication are operational.

---

# VirtualBox Home Lab Topology

```text
                    Internet
                        |
                  Home Router
                  192.168.10.1
                        |
               WAN Interface (enp0s3)
                  Ubuntu Router VM
                  192.168.10.50
                        |
               LAN Interface (enp0s8)
                  192.168.20.1
                        |
              VirtualBox Internal Network
                 (acts as a switch)
             _________________________
            |                         |
        Client 1 VM               Client 2 VM
        192.168.20.50             192.168.20.51
```


