[![Ansible Galaxy](https://img.shields.io/badge/galaxy-smartgic.mycroft-green.svg?style=flat)](https://galaxy.ansible.com/smartgic/mycroft)
[![Discord](https://img.shields.io/discord/809074036733902888)](https://discord.gg/Vu7Wmd9j) 

# Ansible Role: Mycroft

This Ansible role will install and configure Mycroft AI Core from GitHub repository.

The installation process of Mycroft is inspired by the `dev_setup.sh` script provided with the Mycroft AI Core. This playbook is not better than the script provided by the core team, it's just a different approach.

The main differences with `dev_setup.sh` are the following:

- Idempotency
- systemd integration
- Extra skills installation
- `boto3`, `py_mplayer` and `pyopenssl` librairies install
- RAMDisk support for IPC
- File configuration support
- PulseAudio optimizations
- Install and configure Mycroft AI Core on multiple hosts at the same time with the same configuration
- Secure Mycroft message bus websocket

`pairing-code.sh` script is generated during the installation process, this script helps you to retrieve the pairing code if you don't hear it.

```shell
$ pairing-code.sh
Pairing code: HRAK77
Generated: 2020-12-21 17:04:42.408
```

## Requirements

When running on a Raspberry Pi, the `prepi` Ansible role is recommended but not required.

Using the `prepi` Ansible role will ensure your Raspberry Pi to be properly configured for the best of Mycroft experience.

The `prepi` Ansible role will perform the following tasks _(depending your wish)_:

- Update Raspberry Pi OS to the latest version
- Add Debian backports repository _(customizable)_
- Update firmware using the `next` branch which provide kernel 5.10 _(customizable)_
- Update EEPROM using the `beta` version _(customizable)_
- Setup `initial_turbo` to speedup the boot process
- Overclock the Raspberry Pi to 2Ghz _(customizable)_
- Mount `/tmp` on a RAMDisk for Mycroft TTS cache files
- Optimize `/` partition mount options to improve SDcard read/write
- Enable I2C, SPI & UART interfaces _(customizable)_
- Set CPU governor to `performance` to avoid context switching between the `idle*` kernel functions _(customizable)_
- Install and configure PulseAudio _(customizable)_

## Role Variables

Available variables are listed below, along with default values _(see `defaults/main.yml`)_:

```yaml
# Mycroft AI Core Git repository
mycroft_git_repo: https://github.com/MycroftAI/mycroft-core.git

# Mycroft AI Core branch to deploy based on the Git repository
# From the official Git repository, two branches are available:
#  - dev:    unstable
#  - master: stable
mycroft_branch: master

# Mycroft AI Core is actively developed and constantly evolving
# It is recommended that you update regularly
mycroft_autoupdate: yes

# Duration in second before Mycroft stop listening if no one speaking
mycroft_recording_timeout_with_silence: 1.0

# Configure the pre-commit hook to automatically check code-style when submitting code
mycroft_precommit_hook: yes

# Install Python extra requirements
#  - extra-audiobackend.txt
#  - extra-stt.txt
#  - extra-mark1.txt
mycroft_extras_requirements: yes

# Make a sound when Mycroft if listening
mycroft_confirm_listening: yes

# Install Python test requirements
mycroft_tests_requirements: no

# Compile and install Mimic TTS
mycroft_install_mimic: no

# Interval in hour when the skills should be updated
mycroft_skills_update_interval: 6.0

# Log level verbosity of Mycroft AI Core services
#  - CRITICAL
#  - ERROR
#  - WARNING
#  - INFO
#  - DEBUG
mycroft_log_level: INFO

# Enclosure platform name
# picroft and mycroft_mark_1 are recognized which set a little icon on home.mycroft.ai
#  - picroft
#  - mycroft_mark_1
#  - smartgic
#  - choose-yours
mycroft_name: picroft

# Protect Mycroft message bus port by dropping network connection on port 8181
mycroft_bus_firewall: yes

# Bind the Mycroft websocket on a specific IP address, by default listening on 127.0.0.1
mycroft_bus_bind_address: "127.0.0.1"

# Enable SSL encryption for Mycroft websocket
mycroft_bus_ssl: yes

# Confirmation about Mycroft AI Core uninstall
mycroft_uninstall: no

# Use VLC backend for audio service
mycroft_vlc_backend: yes

# Install some extra skills from Git repositories, there is no limit
# An empty variable will not install any extra skills.
mycroft_extra_skills:
  - https://github.com/JarbasSkills/skill-ddg.git
  - https://github.com/JarbasSkills/skill-wolfie.git
  - https://github.com/JarbasSkills/skill-monkey-patcher
  - https://github.com/smartgic/mycroft-wakeword-led-gpio-skill.git
  - https://github.com/smartgic/mycroft-finished-booting-skill.git
  - https://github.com/MycroftAI/skill-homeassistant.git
```

## Dependencies

Same as for the requirements section.

## Example Playbook

Inventory file with `rpi` which has one host named `rpi4b01` with the IP address `192.168.1.97`.

```ini
[rpi]
rpi4b01 ansible_host=192.168.1.97 ansible_user=pi
```

Basic playbook running on `rpi` using the `pi` user to connect via SSH _(based on the inventory)_ with some custom variables.

```yaml
---
- hosts: rpi
  become: yes

  vars:
    mycroft_version: dev
    mycroft_voice_tts: google
    mycroft_skills_update_interval: 2.0

  tasks:
    - import_role:
        name: smartgic.mycroft
      tags:
        - mycroft
```

The next playbook, installs Python, uses `prepi` and `mycroft` Ansible roles using the same inventory file as above.

```yaml
---
- hosts: rpi
  gather_facts: no
  become: yes

  # Install Python using raw module to make sure requirements are installed
  pre_tasks:
    - name: Install Python 3.x Ansible requirement
      raw: apt-get install -y python3
      changed_when: no

- hosts: rpi
  become: yes

  tasks:
    - import_role:
        name: smartgic.prepi
      tags:
        - prepi

    - import_role:
        name: smartgic.mycroft
      tags:
        - mycroft
```

Some tags are availble to trigger only required tasks from the role, such Mycroft's configuration. The `install.yml` content is from the playbook above.

```shell
$ ansible-playbook -i inventory install.yml -t mycroft-config
```

## License

MIT

## Author Information

I'm [Gaëtan Trellu (goldyfruit)](https://smartgic.io/), let's discuss :) - 2021
