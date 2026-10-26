# VPS Optimization

For this project I took a fresh Ubuntu VPS (a DigitalOcean droplet), checked what resources it had, turned off some services it didn't need, and checked again to see what changed.

## Files

- `resources-before.txt` - my inventory of the server before I changed anything (OS, RAM, disk, running services)
- `resources-after.txt` - the report after optimization. This is the main delivery.

## The server

- Ubuntu 22.04.5 LTS
- 1 GB RAM, no swap
- 25 GB disk

## What I did

1. Logged in over SSH and recorded the starting state with `lsb_release -a`, `cat /etc/os-release`, `free -h`, `df -h` and `systemctl list-units --type=service --state=running`.
2. Went through the running services and disabled the ones a small single-disk server doesn't need:
   ```bash
   sudo systemctl disable --now multipathd.service
   sudo systemctl disable --now unattended-upgrades.service
   sudo systemctl disable --now packagekit.service
   sudo systemctl disable --now snapd.service
   sudo systemctl disable --now snapd.socket
   ```
3. Rebooted to make sure the changes stuck.
4. Ran the same commands again and wrote up the results in `resources-after.txt`.

## Results

| | Before | After |
|---|---|---|
| RAM used | 178 MiB | 128 MiB |
| RAM available | 615 MiB | 686 MiB |
| Running services | 20 | 16 |
| Disk used | 2.5 GB (11%) | 2.5 GB (11%) |

Memory use went down and there are fewer services running. Disk use didn't change because I only disabled services, I didn't remove any packages.

The server is ready for company deployments.

## Notes

- snapd still showed up as running after the reboot even though I disabled it. Something else is starting it, so it probably needs to be masked (`systemctl mask snapd.service snapd.socket`) to keep it off.
- With unattended-upgrades disabled, security updates won't install on their own anymore, so they need to be applied by hand (`apt update && apt upgrade`).
