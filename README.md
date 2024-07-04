# eos_agni_radsec

## Overview

**eos_agni_radsec** is a role to configure the proper SSL profile on EOS devices to form a RadSec tunnel using SSL encryption with AGNI.

General steps done by this role to build the SSL profile:

1. Pull the AGNI RadSec CA Certificate 
2. Generate a private key and CSR on EOS
3. Send the CSR to AGNI and get a signed client certificate
4. Copy both certificates to EOS
5. Configure the SSL profile using the previous certificates/key
6. Validate the SSL profile

## Roadmap

- Add the option to configure AGNI as a RadSec server in the switch using the configured SSL profile

## Prerequisites

- Install [Python](https://www.python.org/downloads/) 3.9 or later
- Install [ansible-core](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) from **2.15.0** to **2.17.x.** Avoid the `pipx` installation method
- Install `ansible.netcommon` collection
- Install `arista.eos` collection
- Network access from the Ansible control node to both AGNI service and EOS devices
- EOS devices must have `aaa authorization exec default` configured for SCP to work

Note: This role **must** uses `network_cli` connection type and `paramiko` SSH type. These settings are pre-configured in the role's vars:
```yaml
ansible_connection: network_cli
ansible_network_os: eos
ansible_network_cli_ssh_type: paramiko
```

## Installation
To use this role, you need to download it from GitHub and place it in a location where Ansible can find it. Here are the steps:

1. Create a roles directory in your Ansible project if you haven't already:
```bash
mkdir -p roles
```
2. Clone the role repository into your roles directory:
```bash
cd roles
git clone https://github.com/carl-baillargeon/eos_agni_radsec.git
```
3. Ensure your `ansible.cfg` file includes the roles path. Add or modify the following line:
```bash
roles_path = roles
```

## Running the Playbook

To run the playbook that uses this role:

1. Ensure you have set the required environment variables as described in the [Environment Variables](#environment-variables) section.

2. Create a playbook file (e.g., `configure_agni_radsec.yml`) with the content as shown in the [Example](#example) section. **Don't forget to provide the appropriate CSR information.**

3. Run the playbook using the following command:
```bash
ansible-playbook configure_agni_radsec.yml -i your_inventory_file
```
Replace `your_inventory_file` with the path to your Ansible inventory file.

1. If you need to pass additional variables or override defaults, you can use the `-e` option:
```bash
ansible-playbook configure_agni_radsec.yml -i your_inventory_file -e "agni_base_url=https://your-agni-url.com"
```
Remember to ensure that your Ansible control node has network access to both the AGNI service and your EOS devices.

## Example

```yaml title="configure_agni_radsec.yml"
---
- name: Configure SSL profile for AGNI RadSec on EOS Switches
  hosts: GLOBAL # <-- Targeted devices
  gather_facts: no
  tasks:
    - name: Load and run the role eos_agni_radsec
      ansible.builtin.import_role:
        name: eos_agni_radsec
      vars:
        agni_base_url: "https://beta.agni.arista.io"
        eos_csr_info:
          country: "CA"
          state: "QC"
          locality: "MTL"
          organization: "Home"
          organizational_unit: "Lab"
          domain: "home.arpa"
```

## Input variables

```yaml
---
# Path on local machine to store the temporary certificates
temp_path: <str; | default="/tmp">

# AGNI base URL
agni_base_url: <str; | default="https://beta.agni.arista.io">

# AGNI RadSec CA Certificate name
radsec_ca_certificate: <str; | default="radsec_ca_certificate">
radsec_ca_certificate_format: <str; | default="pem">

# EOS SSL Profile name for AGNI RadSec
eos_ssl_profile: <str; | default="agni-server">

# EOS private key name for AGNI RadSec
eos_private_key: <str; | default="agni-private.key">

# EOS path to save the certificates
eos_cert_path: "/mnt/flash/"

# EOS CSR information. Required
eos_csr_info:
  country: <str>
  state: <str>
  locality: <str>
  organization: <str>
  organizational_unit: <str>
  domain: <str>
```
## Environment Variables

This Ansible role requires certain environment variables to be set for secure operation. These variables contain sensitive information and should be managed carefully.

### Required Environment Variables

Before running the playbook that uses this role, ensure the following environment variables are set:

1. **AGNI_KEY_ID**
   - Description: The key ID for authenticating with the AGNI service.
   - Example: `export AGNI_KEY_ID="your_agni_key_id_here"`

2. **AGNI_KEY_VALUE**
   - Description: The key value corresponding to the AGNI_KEY_ID for authentication.
   - Example: `export AGNI_KEY_VALUE="your_agni_key_value_here"`

3. **AGNI_ORG_ID**
   - Description: Your organization ID in the AGNI service.
   - Example: `export AGNI_ORG_ID="your_agni_org_id_here"`

Please refer to the AGNI API Guide found [below](#relevant-documentation) to get these values.

### Setting Environment Variables

You can set these variables in your shell before running the Ansible playbook:

```bash
export AGNI_KEY_ID="your_agni_key_id_here"
export AGNI_KEY_VALUE="your_agni_key_value_here"
export AGNI_ORG_ID="your_agni_org_id_here"
```

## Relevant documentation

- [AGNI API Guide](https://www.arista.com/assets/data/pdf/user-manual/um-books/AGNI-API-Guide.pdf)
- [Configuring RadSec profile in EOS](https://arista.my.site.com/AristaCommunity/s/article/Configuring-RadSec-profile-in-EOS)



