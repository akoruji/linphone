# Asterisk + Linphone with Docker (Ubuntu/Linux)

This project provides a basic Asterisk SIP PBX for two Linphone softphone accounts:
- Extension 1001
- Extension 1002
- SIP over UDP port 5060
- RTP audio ports UDP 10000-10100
- Optional audio loopback test extension *43

## 1. Requirements

- Ubuntu/Linux host with Docker Engine and Docker Compose plugin installed.
- Asterisk image `andrius/asterisk:22` available from Docker Hub.
- For the first test, put the Linphone devices and server on the same LAN.

## 2. Set strong passwords

Edit `asterisk/pjsip.conf` and replace:
- `CHANGE_THIS_1001_PASSWORD`
- `CHANGE_THIS_1002_PASSWORD`

Use separate, long, unique passwords. Do not expose the service to the public internet with example credentials.

## 3. Check the image and start

From this directory:

```bash
docker compose pull
docker compose config
docker compose up -d
docker compose logs -f asterisk
```

If the image tag is unavailable or the container exits, check the image's current Docker Hub documentation and supported tags before proceeding.

## 4. Verify Asterisk

```bash
docker exec -it sip-asterisk asterisk -rvvv
```

At the Asterisk CLI:

```text
core show version
pjsip show transports
pjsip show endpoints
dialplan show internal
```

Expected endpoints include `1001` and `1002`. Type `exit` to leave the CLI.

## 5. Configure Linphone

Find the server's LAN address:

```bash
hostname -I
```

On Linphone, add a SIP account manually.

For phone 1:
- Username / SIP identity user: `1001`
- Password: the password set for 1001
- Domain / SIP server: the Linux host's LAN IP, e.g. `192.168.1.50`
- Transport: UDP
- SIP port: `5060`, if the app asks for it

For phone 2, use username `1002` and its password. Use the same server IP.

Once both accounts register, dial `1002` from 1001. For an audio loopback test, dial `*43` if the dialplan loaded successfully.

## 6. Firewall

If UFW is enabled, allow only the trusted LAN where possible. Example for a LAN using `192.168.1.0/24` (replace with your actual subnet):

```bash
sudo ufw allow from 192.168.1.0/24 to any port 5060 proto udp
sudo ufw allow from 192.168.1.0/24 to any port 10000:10100 proto udp
sudo ufw status
```

Do not blindly use this example subnet if your LAN is different. Confirm your network range first.

## 7. Troubleshooting

```bash
docker compose ps
docker compose logs --tail=200 asterisk
docker exec -it sip-asterisk asterisk -rx "pjsip show endpoints"
docker exec -it sip-asterisk asterisk -rx "pjsip show contacts"
docker exec -it sip-asterisk asterisk -rx "dialplan show internal"
```

If calls connect but have no audio, check the RTP UDP range, firewall, NAT, and whether Asterisk advertises an address reachable by the clients. For remote clients, use a VPN or configure NAT/TLS/SRTP and appropriate external/local addresses; do not expose this starter configuration directly to the public internet.

## Notes

- This is a starter configuration, not a hardened internet-facing PBX.
- The example uses UDP and unencrypted SIP/RTP on a trusted LAN.
- Configuration files are mounted read-only. To apply changes, restart the container:

```bash
docker compose restart asterisk
```
