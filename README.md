# ZeroTier for Steam Deck

Getting ZeroTier running on SteamOS can be a grind: read-only root, broken pacman keys, failed installs, services that stop the moment you switch to Game Mode… this little script automates every workaround so you can just join your network and play.

## Highlights

- Makes `/usr` writable temporarily and repairs pacman keyrings automatically.
- Installs or repairs `zerotier-one`, adds reliable systemd dependencies, and seeds `/var/lib/zerotier-one`.
- Drops in a watchdog service so ZeroTier stays online through Game Mode, sleep, and reboots.
- Prompts you to join a network and prints status reports right at the end.

---

## Quick Start

1. **Desktop Mode**  
   Switch your Steam Deck to Desktop Mode and open **Konsole**.

2. **Download & Prep**  
   ```bash
   cd ~/Downloads/ZeroTeir
   chmod +x zerotier.sh
   ```

3. **Run It**  
   ```bash
   ./zerotier.sh            # normal output
   ./zerotier.sh quiet      # minimal chatter
   ./zerotier.sh verbose    # show every command
   ```

4. **Join Your Network**  
   At the end the script shows `zerotier-cli info`, then asks for a Network ID. Paste one from [my.zerotier.com](https://my.zerotier.com/), approve the node in the web console, and you’re good.

The script is idempotent, so you can re-run it any time to repair a Deck after an update.

---

## What the Script Does

1. Ensures ZeroTier isn’t already online (skips everything if it is).  
2. Remounts the SteamOS root as writable and rebuilds `/etc/pacman.d/gnupg`.  
3. Forces pacman to sync and install `zerotier-one` (falling back to the official curl installer if needed).  
4. Adds a systemd drop-in so `zerotier-one` waits for real network connectivity, then starts/enables the service.  
5. Installs a watchdog script + unit that nudges ZeroTier back online every five minutes.  
6. Shows `zerotier-cli info` and optionally joins the network you provide, printing `listnetworks` for quick confirmation.

---

## Verifying Everything

```bash
sudo zerotier-cli info          # should end with ONLINE
sudo zerotier-cli listnetworks  # shows joined networks + IPs
sudo systemctl status zerotier-one
sudo systemctl status zerotier-watchdog.service
```

If you only see `OFFLINE`, double-check that the node is authorized in the ZeroTier web console.

---

## Troubleshooting

- **Installer fails with pacman key errors** – rerun the script; it now wipes and regenerates the keyring automatically.  
- **ZeroTier stuck offline** – run `sudo zerotier-cli leave <net>` then `sudo zerotier-cli join <net>`; the watchdog will restart the service if needed.  
- **Need to undo everything** – disable both systemd units (`zerotier-one` and `zerotier-watchdog`) and remove `/var/lib/zerotier-one`.

---

## License

Released under the MIT License (see `LICENSE`). Attribution back to this repo or any derivative fork is appreciated when redistributing.
