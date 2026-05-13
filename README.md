# Ansible Playbooks

A collection of Ansible playbooks for infrastructure automation in multi-site environments. Built for real production use across Windows Server, Linux, and network infrastructure.

## Requirements

- Ansible 2.10+
- Python 3.x
- Collections installed per playbook category (see below)

## Repository Structure

```
Ansible/
├── ansible.cfg                          # Ansible configuration
├── inventories/
│   ├── hosts.yml                        # Inventory (see example_inventory.yml)
│   ├── group_vars/
│   │   ├── example_smartshield.yml      # SmartShield connection variables
│   │   ├── example_windows.yml         # Windows connection variables
|   |   └── examble_netbox_servers_vars.yml  # Netbox connection variables
│   └── host_vars/
│       └── example_firewall-site1.yml  # Per-host FortiGate API token example
├── playbooks/
│   ├── network/
│   │   └── fortigate_backup.yml
│   ├── smartshield/
│   │   ├── smartshield_restart_services.yml
│   │   ├── smartshield_diagnostics.yml
│   │   └── smartshield_upgrade.yml
│   └── windows/
│       ├── windows_update_audit.yml
│       ├── windows_patch_after_hours.yml
│       └── windows_dhcp_stats.yml
└── roles/
```

---

## Configuration

Copy `ansible.cfg` to your project root. Key settings:

- Inventory defaults to `./inventories/hosts.yml`
- Vault password is prompted automatically on every run
- Diff mode is enabled to show changes
- YAML output format for readability

---

## Playbooks

### Network

#### FortiGate Configuration Backup
**File:** `playbooks/network/fortigate_backup.yml`

Automated backup of FortiGate firewall configurations using the Fortinet REST API. Saves timestamped config files locally for each device in the inventory.

**Requirements:**
```
ansible-galaxy collection install fortinet.fortios
```

**Usage:**
```
ansible-playbook playbooks/network/fortigate_backup.yml
```

**Certificate Validation:**
`ansible_httpapi_validate_certs` is set to `false` by default for compatibility with self-signed certificates. Set to `true` and use FQDNs in your inventory if your FortiGates have valid certificates.

---

### SmartShield (Lightspeed)

#### Restart SmartShield Services
**File:** `playbooks/smartshield/smartshield_restart_services.yml`

Restarts the three core SmartShield filtering services (`lsfilterd`, `lsrelayd`, `lantern`) and verifies each is running after restart. Fails with a descriptive message if any service does not come back up.

**Usage:**
```
ansible-playbook playbooks/smartshield/smartshield_restart_services.yml
```

---

#### SmartShield Diagnostics
**File:** `playbooks/smartshield/smartshield_diagnostics.yml`

Runs a quick diagnostic check against SmartShield appliances including DNS filtering validation, Relay Rocket version check, and OS version reporting.

**Usage:**
```
ansible-playbook playbooks/smartshield/smartshield_diagnostics.yml
```

---

#### SmartShield Upgrade
**File:** `playbooks/smartshield/smartshield_upgrade.yml`

Performs a full APT dist-upgrade on SmartShield appliances and reboots if required. Safe to run against multiple appliances simultaneously.

**Usage:**
```
ansible-playbook playbooks/smartshield/smartshield_upgrade.yml
```

---

### Windows

**Requirements:**
```
ansible-galaxy collection install ansible.windows
```

#### Windows Update Audit
**File:** `playbooks/windows/windows_update_audit.yml`

Scans Windows servers for available updates without installing anything. Also checks for pending reboot state via registry. Useful for pre-maintenance assessment.

**Usage:**
```
ansible-playbook playbooks/windows/windows_update_audit.yml
```

---

#### Windows After-Hours Patching
**File:** `playbooks/windows/windows_patch_after_hours.yml`

Patches Windows servers with DC-first sequencing. Patches the primary DC first, verifies AD health with dcdiag, then patches remaining servers one at a time using `serial: 1`. Handles pre and post-patch reboot detection and WinRM reconnection automatically.

**Inventory requirements:**
- `primary_dc` group must contain exactly one host (your primary domain controller)
- `windows_servers` group contains all Windows servers including the primary DC

**Usage:**
```
ansible-playbook playbooks/windows/windows_patch_after_hours.yml
```

---

#### Windows DHCP Scope Statistics
**File:** `playbooks/windows/windows_dhcp_stats.yml`

Queries DHCP scope statistics from Windows DHCP servers. Reports usage percentage and free IP count per scope. Generates a warning for any scope exceeding 80% utilization.

**Usage:**
```
ansible-playbook playbooks/windows/windows_dhcp_stats.yml
```

---

## Inventory Setup

See `inventories/example_inventory.yml` for the expected host group structure. Key groups:

| Group | Used By |
|-------|---------|
| `fortigates` | FortiGate backup |
| `smartshield` | All SmartShield playbooks |
| `primary_dc` | Windows patching (DC-first sequencing) |
| `windows_servers` | Windows patching, update audit |
| `windows_dhcp_servers` | DHCP statistics |

## Variable Setup

Connection variables live in `inventories/group_vars/`. See the example files for the expected structure:

- `example_smartshield.yml` -- SSH connection and sudo escalation for SmartShield appliances
- `example_windows.yml` -- WinRM connection variables for Windows hosts

Per-host FortiGate API tokens live in `inventories/host_vars/`. See `example_firewall-site1.yml` for the expected structure.

All sensitive values should be encrypted with Ansible Vault before committing:
```
ansible-vault encrypt_string 'your_value' --name 'variable_name'
```

## Security Notes

- Never commit real credentials, API tokens, or internal hostnames to source control
- All sensitive values in this repo are Ansible Vault encrypted
- Example files use placeholder values only

## License

MIT License
