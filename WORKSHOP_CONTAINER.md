Workshop Container Setup (Lab-Only)

This setup is intended for authorized, closed lab use only.

Workshop split:
- student-facing setup materials live in [student-kit/README.md]
- hosted game deployment/admin materials live in [server-side/README.md]

Goals
- Run in Docker on Windows/macOS/Linux with the same image.
- Avoid external DNS by using local hosts file entries.
- Preserve TLS behavior with a lab CA you can trust on client machines.

Quick Start
1) Build and start the container:
   - `docker compose up --build`
2) Keep the data directory in the repo:
   - `./workshop-data` will hold config and certificates.

TLS In a Lab (No Public DNS)
The simplest way to keep TLS working without DNS is to run in developer mode
and trust the lab CA on all client machines.

- The CA file is created in `./workshop-data/crt/ca.crt` after first run.
- Install that CA into the OS trust store on each client.

Hosts File Overrides
Map your lab hostnames to the machine running Docker. For single-user laptops,
`127.0.0.1` works with the port mappings in `docker-compose.yml`.

For the DC256 victim game, add the Evilginx lure hostnames below. These are
the fake `dc256.lab` domains participants paste into the hosted game. Hosts
files do not support wildcards, so add each hostname explicitly.

If each participant runs Evilginx locally:

```text
127.0.0.1   payroll-login.dc256.lab
127.0.0.1   ledger-login.dc256.lab
127.0.0.1   travel-login.dc256.lab
127.0.0.1   git-login.dc256.lab
127.0.0.1   build-login.dc256.lab
127.0.0.1   tickets-login.dc256.lab
127.0.0.1   people-login.dc256.lab
127.0.0.1   benefits-login.dc256.lab
127.0.0.1   learn-login.dc256.lab
127.0.0.1   helpdesk-login.dc256.lab
127.0.0.1   devices-login.dc256.lab
127.0.0.1   cloud-login.dc256.lab
```

If Evilginx runs on a shared lab VM, replace `127.0.0.1` with that VM's IP:

```text
10.10.10.50 payroll-login.dc256.lab
10.10.10.50 ledger-login.dc256.lab
10.10.10.50 travel-login.dc256.lab
10.10.10.50 git-login.dc256.lab
10.10.10.50 build-login.dc256.lab
10.10.10.50 tickets-login.dc256.lab
10.10.10.50 people-login.dc256.lab
10.10.10.50 benefits-login.dc256.lab
10.10.10.50 learn-login.dc256.lab
10.10.10.50 helpdesk-login.dc256.lab
10.10.10.50 devices-login.dc256.lab
10.10.10.50 cloud-login.dc256.lab
```

macOS/Linux:
- Edit `/etc/hosts` and add entries like:
  `127.0.0.1   lab.example.test`
  `127.0.0.1   login.lab.example.test`

Windows:
- Edit `C:\Windows\System32\drivers\etc\hosts` with entries like:
  `127.0.0.1   lab.example.test`
  `127.0.0.1   login.lab.example.test`
- Then flush DNS cache:
  `ipconfig /flushdns`

Notes
- If you want ACME/Let's Encrypt instead of developer mode, you must use real
  DNS and make ports 80/443 reachable from the public internet.
- The built-in CLI has a "get-hosts" helper that can output host entries for
  configured lab hostnames.

Firefox Certificate Import (macOS/Linux/Windows)
Use this if Firefox is the browser for the workshop. It imports the lab CA
directly into Firefox's trust store.

Steps (all platforms):
1) Open Firefox.
2) Go to Settings/Preferences -> Privacy & Security.
3) Scroll to "Certificates" and click "View Certificates...".
4) In the "Authorities" tab, click "Import...".
5) Select `./workshop-data/crt/ca.crt`.
6) Check "Trust this CA to identify websites" and confirm.

Notes:
- If Firefox is set to use the OS trust store on your platform, you can also
  install the CA at the OS level instead of importing into Firefox.

Initial CLI Setup (Lab Configuration)
Run the container with an interactive terminal when you need to set initial
values. If you are already running detached, use:
- `docker compose exec evilginx sh`
- Then run: `evilginx -p /usr/share/evilginx/phishlets -c /data -developer`

Suggested base config for a lab:
1) Set the base domain used by lab hostnames:
   - `config domain lab.example.test`
2) Set the external IPv4 address of the Docker host:
   - `config ipv4 external <host-ip>`
   - Single-machine lab option: `config ipv4 external 127.0.0.1`
3) Set the bind address for the service (usually 0.0.0.0):
   - `config ipv4 bind 0.0.0.0`
4) Verify values:
   - `config`

Notes:
- Use a non-public lab domain like `lab.example.test`.
- With `-developer`, certificates are generated locally and use the lab CA from
  `./workshop-data/crt/ca.crt`.
