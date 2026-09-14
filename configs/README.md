# Configuration Output

Raw command output captured from the lab, kept as plain text so the values in the
main [README](../README.md) can be verified rather than taken on trust.

## Files to capture

Run these inside the Kali VM and save the output here.

### `ip-config.txt`

```bash
{
  echo "=== ip addr show ==="
  ip addr show
  echo
  echo "=== ip route show ==="
  ip route show
  echo
  echo "=== /etc/resolv.conf ==="
  cat /etc/resolv.conf
} > ip-config.txt
```

### `connectivity-tests.txt`

```bash
{
  echo "=== ping gateway ==="
  ping -c 4 10.0.0.1
  echo
  echo "=== ping 8.8.8.8 (routing) ==="
  ping -c 4 8.8.8.8
  echo
  echo "=== ping google.com (DNS) ==="
  ping -c 4 google.com
} > connectivity-tests.txt
```

### `vm-specs.txt`

Run on the **Windows host**, from the VirtualBox install directory:

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" showvminfo KALI > vm-specs.txt
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" natnetwork list >> vm-specs.txt
```

> Review these files before committing. Redact anything you would not want public —
> the virtual MAC addresses and `10.0.0.x` lab addresses are harmless, but your
> host's public IP or home LAN range is not worth publishing.
