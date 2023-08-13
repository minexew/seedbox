## Architecture

- host OS: RHEL 9 clone
- guest OS: RHEL 9 clone
- networking: NAT between host & guest; all guest traffic routed through WireGuard VPN
- client: Transmission 4.x (Geekery repo)

## Setup

1. Create Guest VM via host's Cockpit VM: 2 GB RAM, 10 GB disk. Install guest OS
2. Via Cockpit add also shared folder for NAS
3. (In the following steps, all commands/files refer to the guest VM unless stated otherwise)
4. Disable SELinux because it's a pain in the ass (`/etc/selinux/config`)
5. In `/etc/ssh/sshd_config` set `PermitRootLogin=yes`
6. Add to `/etc/fstab`:
```
nas     /mnt/nas        virtiofs        defaults        0 0
```
6. Download WireGuard config from VPN provider, save as `/etc/wireguard/wg-vpn.conf`
7. Install EPEL repo and necessary (transmission, wireguard) + useful (mc, tmux) packages:
```
dnf install epel-release
dnf update -y
dnf install -y mc tmux wireguard-tools
```
8. Install Geekery repo & Transmission 4.x
```
curl -LO http://geekery.altervista.org/geekery/el9/x86_64/geekery-release-9-0.noarch.rpm
rpm -i geekery-release-9-0.noarch.rpm
dnf install -y transmission-daemon
```
9. In `/var/lib/transmission/.config/transmission-daemon/settings.json` set `"rpc-whitelist-enabled": false`
  - necessary to run Transmission once to create the file? or can do it manually?
10. Configure firewall and systemd:
```
systemctl enable --now systemd-resolved
systemctl enable --now wg-quick@wg-vpn
systemctl enable --now transmission-daemon
firewall-cmd --add-port=9091/tcp --permanent
firewall-cmd --add-port=51413/tcp --permanent
```

## Usage

To access the Transmission Web UI, navigate to http://GUEST.VM.IP:9091/
