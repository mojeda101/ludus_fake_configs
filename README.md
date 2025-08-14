# Ansible Role: ludus_fake_configs ([Ludus](https://ludus.cloud))

Creates random, fake configuration files on **Windows** across one or more paths. Optionally embeds a provided username(s)/password(s) into some or all new files. Useful for lab noise or to play with tools like manspider.

> [!WARNING]
> If you set `fake_cfg_force_mode: "all"` its deletes EVERYTHING under each path. (Default=matching)
> If you set `fake_cfg_force: true ` true = purge first, then recreate files (Default=true).
> The repo was basically fast vibe coded so check if its works for you.

> [!NOTE]
> The current default path are:
> fake_cfg_paths:
>  - 'C:\shares\all'
>  - 'C:\shares\public'

The orginaly created to use with [ludus-ad-vulns](https://github.com/Primusinterp/ludus-ad-vulns/tree/main). I found the created open share nice but empty.


## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

~~~yaml
# === Targets ===
# One or more paths to populate. Each path can be created automatically.
fake_cfg_paths:
  - 'C:\shares\all'
  - 'C:\shares\public'
# Ensure target paths exist
fake_cfg_create_path: true

# Prefer PowerShell 7 if installed (falls back otherwise)
fake_cfg_use_pwsh: false

# === Generation knobs ===
fake_cfg_count: 50                # total files to end up with (per path; idempotent unless forced)
fake_cfg_extensions:              # pool of extensions to generate
  - ini
  - yaml
  - yml
  - json
  - xml
  - conf
  - properties
fake_cfg_min_kib: 1               # approximate size range (KiB)
fake_cfg_max_kib: 32
fake_cfg_subdirs: ["app", "services", "agents", "drivers", "development","configs"]   # optional subfolders; [] for none
fake_cfg_prefix: "config"
# File name patterns (tokens are literal text, handled inside PowerShell)
# tokens: {{ prefix }}, {{ rand }}, {{ ext }}
fake_cfg_name_patterns:
  - !unsafe "{{ prefix }}-{{ rand }}.{{ ext }}"
  - !unsafe "{{ prefix }}_{{ rand }}.{{ ext }}"
  - !unsafe "{{ rand }}-{{ prefix }}.{{ ext }}"
fake_cfg_date_back_days_max: 120  # random backdating (0 disables)

# === Force/purge behavior ===
fake_cfg_force: true             # true = purge first, then recreate
# matching: delete only files that look like our fake configs
# all     : delete EVERYTHING under each path before recreating
fake_cfg_force_mode: "matching"

# === Credentials embedding ===
fake_cfg_credentials:
  username: "svc_labuser"
  password: "P@ssw0rd!123"

# How to embed creds:
#   none  : never embed
#   all   : embed in every new file (if ext eligible)
#   count : embed in exactly N new files per path
#   ratio : embed in about ratio*toCreate files per path
fake_cfg_creds_embed_mode: "ratio"
fake_cfg_creds_embed_count: 50
fake_cfg_creds_embed_ratio: 0.1
fake_cfg_creds_embed_exts: ['ini','yaml','yml','json','xml','conf','properties']
# Filenames matching any of these regexes (case-insensitive suggested) will ALWAYS get creds (if ext eligible)
fake_cfg_creds_embed_names: ['(?i)credential', '(?i)auth', '(?i)login', '(?i)secrets?']
~~~


## Dependencies

None.

## Example Playbook

~~~yaml
- hosts: winfileserver
  gather_facts: no
  roles:
    - role: ludus_fake_configs
      vars:
        fake_cfg_paths:
          - 'C:\shares\all'
          - 'C:\shares\public'
        fake_cfg_force: true           # recreate all files
        fake_cfg_force_mode: all       # wipe everything under paths before recreating
        fake_cfg_count: 100
        fake_cfg_credentials:
          username: 'local_svc'
          password: 'LocalP@ss123!'
        fake_cfg_creds_embed_mode: ratio
        fake_cfg_creds_embed_ratio: 0.1
        fake_cfg_use_pwsh: false
~~~

## Example Ludus Range Config

~~~yaml
ludus:
  - vm_name: "{{ range_id }}-winfileserver-2022"
    hostname: "{{ range_id }}-FS01-2022"
    template: win2022-server-x64-template
    vlan: 10
    ip_last_octet: 40
    ram_gb: 4
    cpus: 4
    windows:
      sysprep: false
    domain:
      fqdn: ludus.domain
      role: member
    testing:
      snapshot: true
      block_internet: true
    # Attach your local role (added via `ludus ansible role add -d ...`)
    roles:
      - ludus_fake_configs
    # Variables passed to ALL roles on this VM (we’re only using our role here)
    role_vars:
      # Where to place the fake config files (multi-path supported)
      fake_cfg_paths:
        - 'C:\shares\all'
        - 'C:\shares\public'

      # Create the fake directories if missing
      fake_cfg_create_path: true
      # Recreate each deploy (set to false later for idempotence)
      fake_cfg_force: true
      fake_cfg_force_mode: matching   # or 'all' to wipe everything under those paths

      # How many files to end up with (per path)
      fake_cfg_count: 120

      fake_cfg_prefix: "config"
      # Backdate timestamps to look realistic
      fake_cfg_date_back_days_max: 120

      # Embed creds in some files (for deception)
      fake_cfg_credentials:
        username: "adminuser"
        password: "P@ssw0rd123"
      fake_cfg_creds_embed_mode: ratio   # none | all | count | ratio
      fake_cfg_creds_embed_ratio: 0.10   # ~10% of newly created files


~~~

## License

[//]: # (If you change the License type, be sure to change the actual LICENSE file as well)
GPLv3

## Author Information

This role was created by [mojeda101](https://github.com/mojeda101), for [Ludus](https://ludus.cloud/).
