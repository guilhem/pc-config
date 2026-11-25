# PC Config

Install and update hardware settings for a Linux workstation as a Debian package. The first setting loads the AMD `sp5100_tco` driver and configures a five-minute hardware watchdog on Debian or Ubuntu with systemd 252 or newer.

Use this package on an amd64 machine where `sudo modprobe sp5100_tco` creates a watchdog named `SP5100 TCO timer` in `/sys/class/watchdog/watchdog*/identity`. The package sets the system-wide watchdog device to a udev link specific to that driver.

## Install from a release

Download the amd64 package from the [latest release](https://github.com/guilhem/pc-config/releases/latest), then run this in the directory containing the download.

```bash
sudo apt install ./pc-config_*_amd64.deb
```

## Install and update with APT

Add the [signed APT repository](https://guilhem.github.io/pc-config/) and scope its signing key to this source.

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSLo pc-config.asc https://guilhem.github.io/pc-config/pc-config.asc
sudo install -m 0644 pc-config.asc /etc/apt/keyrings/pc-config.asc

echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/pc-config.asc] https://guilhem.github.io/pc-config/stable/ ./' |
  sudo tee /etc/apt/sources.list.d/pc-config.list
sudo apt update
sudo apt install pc-config
```

Future versions are available through `sudo apt upgrade`.

## Check the watchdog

After the next normal reboot, check the loader, the selected device and the timer.

```bash
systemctl status pc-config.service
systemctl show -p WatchdogDevice -p RuntimeWatchdogUSec
watchdog="$(basename "$(readlink -f /dev/pc-config-watchdog)")"
cat "/sys/class/watchdog/$watchdog/state" "/sys/class/watchdog/$watchdog/timeout"
```

The expected device is `/dev/pc-config-watchdog`. The hardware state and timeout should read `active` and `300`. The service sets the timeout through systemd's D-Bus API and restores the configured value when stopped.

When upgrading from 0.1.0, the current boot keeps the existing watchdog configuration. Reboot to activate the D-Bus service. If you run `sudo apt purge pc-config` before rebooting, the package stops the service and removes the old runtime drop-in. Systemd picks up its previous watchdog device at the next boot.

## Build from source

Install `build-essential` and `debhelper`, then run `dpkg-buildpackage -us -uc -b` from the repository root. A tag matching the version in `debian/changelog` triggers the release and signed APT publication through GitHub Actions.
