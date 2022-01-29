# ansible_pwnkit_migitation

![Linux](https://img.shields.io/badge/os-Linux-yellow)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE.md)

Ansible playbook for PwnKit temporary mitigation on Linux host.

## Table of Contents

- [About](#about)
- [Disclaimer](#disclaimer)
- [Supported Platforms](#supported-platforms)
- [Requirements](#requirements)
- [Dependencies](#dependencies)
- [Variables](#variables)
- [Usages](#usages)
- [Example](#examples)
- [Bonus](#bonus)
- [License](LICENSE.md)

## About

PwnKit vulnerability allows obtaining full root privileges from any unprivileged local user using Polkit component (with pkexec binary)b  on multiple Linux distributions.

This playbook allow apply the Polkit temporary mitigation before applying the patch.
It check if Polkit is installed and remove SUID-bit permission for pkexec binary on a Linux host.

More information about PwnKit vulnerability:

- CVE-2021-4034:
https://blog.qualys.com/vulnerabilities-threat-research/2022/01/25/pwnkit-local-privilege-escalation-vulnerability-discovered-in-polkits-pkexec-cve-2021-4034

## Disclaimer

**Use of that Ansible playbook implies acceptance of the terms of this disclaimer.**

That Ansible playbook is provided as is and without any warranty.
All express or implied conditions, representations and warranties, including any warranty of non-infringement or warranty of merchantability or fitness for a particular purpose, are hereby disclaimed and excluded to the extent allowed by applicable law.

In no event will I be liable for any direct or indirect damages, including but not limited to lost revenue, profit or data, or for direct, special, indirect, consequential, incidental or punitive damages however caused and regardless of the theory of liability arising out of the use of or inability to use the software, even if I has been advised of the possibility of such damages.

You are always responsible for the security of your server and this playbook does not replace possible security checks and measures.

## Supported Platforms

The following Linux system are supported (non-exhaustive list):

- Fedora-based:
	- **Fedora**
	- **RHEL**
	- **CentOS**
	- **Rocky**
	- **Oracle Linux**
	- **AlmaLinux**
	- **Amazon Linux**
- Debian-based:
	- **Debian**
	- **Ubuntu**
- OpenSUSE-based:
	- **OpenSUSE**
	- **SLES**

**Note: It was only tested on Rocky Linux 8.5.**

## Requirements

List of requirements to use it:

- Ansible: 1.4 or higher
- Root permissions (directly or with sudoers)

**Note: Tested on Ansible core 2.9.27 with Python 3.6.8.**

## Dependencies

None.

## Variables

List of playbook variables availables:

| Variable                    | Required | Default              | Type     | Comments                                   |
| --------------------------- | -------- | -------------------- | -------- | ------------------------------------------ |
| `polkit_package`            | no       | *depending os*       | `string` | Polkit package name.                       |
| `pkexec_path`               | no       | `/usr/bin/pkexec`    | `string` | pkexec binary path.                        |
| `pkexec_path`               | no       | `0775`               | `string` | Permission mode to apply on pkexec.        |

## Usage

1. Create playbook directory:
```shell
$ mkdir ansible_pwnkit_migitation
```

2. Clone it into `ansible_pwnkit_migitation` directory:
```shell
$ git clone https://github.com/sanchez-anthony/ansible_pwnkit_migitation.git ansible_pwnkit_migitation
```

3. Running playbook with **your own host inventory**:
```shell
$ ansible-playbook -i inventory/hosts ansible_pwnkit_migitation/tasks/pwnkit_migitation.yml
```

# Example

Running playbook on localhost:
```shell
root@localhost:~$ stat /usr/bin/pkexec
  File: /usr/bin/pkexec
  Size: 29048           Blocks: 64         IO Block: 4096   regular file
Device: fd02h/64770d    Inode: 143865      Links: 1
Access: (4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-01-28 22:07:23.591997139 +0100
Modify: 2021-10-11 08:22:38.000000000 +0200
Change: 2021-11-27 20:17:13.770135050 +0100
 Birth: 2021-11-27 20:17:13.769135050 +0100

root@localhost:~$ ansible-playbook ~/ansible_pwnkit_migitation/pwnkit_migitation.yml

PLAY [PwnKit mitigation - Check if Polkit is installed and remove SUID-bit permission on pkexec] *******************************************************

TASK [Gathering Facts] **************************************************************************************************************************************
ok: [localhost]

TASK [Gather the package facts] *****************************************************************************************************************************
ok: [localhost]

TASK [Loading the OS specific variables] ********************************************************************************************************************
ok: [localhost]

TASK [Show message when polkit is installed] ****************************************************************************************************************
ok: [localhost] => {
    "msg": "Package polkit in version 0.115 is installed."
}

TASK [Show message when polkit isn't installed] *************************************************************************************************************
skipping: [localhost]

TASK [Check if pkexec binary is present] ********************************************************************************************************************
ok: [localhost]

TASK [Remove SUID-bit on pkexec binary when present] ********************************************************************************************************
changed: [localhost]

PLAY RECAP **************************************************************************************************************************************************
localhost                  : ok=6    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

root@localhost:~$ stat /usr/bin/pkexec
  File: /usr/bin/pkexec
  Size: 29048           Blocks: 64         IO Block: 4096   regular file
Device: fd02h/64770d    Inode: 143865      Links: 1
Access: (0755/-rwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-01-28 22:07:23.591997139 +0100
Modify: 2021-10-11 08:22:38.000000000 +0200
Change: 2022-01-28 23:49:31.600863430 +0100
 Birth: 2021-11-27 20:17:13.769135050 +0100
```

Permission mode for pkexec binary successfully changed from `4755` to `0755`.

## Bonus

You can check if Polkit it's patched on host:
- Fedora-based: `rpm -q --changelog polkit | grep -B2 CVE-2021-4034`
- Debian-based: `apt changelog policykit-1=$(dpkg-query --showformat='${Version}' --show policykit-1) | grep -B3 CVE-2021-4034`
