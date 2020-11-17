# Ansible Role: Cobalt Strike
This Ansible Role is intended to install Cobalt Strike >= 4.0 and to configure it either as Operator (client) or Teamserver (server). It has been tested on Debian Stretch Amazon AMIs.

It is based on [chryzsh's CS Ansible Role](https://github.com/chryzsh/ansible-role-cobalt-strike), with bug fixes and the implementation of the following features:
- BYO TLS key pair
- Runtime generated TLS JKS w/random password 
- Inclusion of C2 profiles locally
- Oracle Java 8 support (the recommended CS Java runtime)

## Modes of Operation

### Teamserver
- If configured as Teamserver, it will start automatically with the supplied password and C2 profile.
 - If the `domain` variable is not set, TLS logic & variables will be ignored and CS will automatically create a self-signed (and easily detected) certificate.

### Operator
- If configured as Operator, it will download the artifact and resource kit automatically.
  - Kits are placed in `~/cobaltstrike/kits`

### Global
- The role installs Cobalt Strike into a cobalstrike folder in your user's home directory `~/cobaltstrike`. 
  - Profiles are placed in `~/cobaltstrike/profiles`

## Installation
```bash
ansible-galaxy install joeminicucci.cobalt_strike
```

This repo is typically utilized within a devops pipeline that has C2 profiles, TLS certficates, and Oracle JDK tarballs dynamically generated & placed before the playbook is executed. For reference, here are the manual steps which would need to be taken  assuming the Role is invoked with default variables, on a fresh machine:

```bash
#global
mkdir ../oracle/
#teamserver
mkdir ../certificates/
mkdir ../c2_profiles/

#global
cp <location of JDK 8 tarball> ../oracle/
#teamserver
cp <location of TLS pub/priv cert> ../certificates
cp <location of c2 profile files> ../c2_profiles
```

## Role Variables

### Globals

| Variable                          | Default                 | Comments                                                                                                                                                                       |
| :-------------------------------- | :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| cobalt_strike_role            | teamserver                   | 
| license_key            | aaaa-bbbb-cccc-dddd                   | Valid CS License key
| oracle_directory            | {{ playbook_dir }}/../oracle/                   |  Location of the Oracle JDK 8 file
| oracle_jdk            |  jdk-8u261-linux-x64.tar.gz                   |  The name of the Oracle JDK 8 file

### Teamserver
| Variable                          | Default                 | Comments                                                                                                                                                                       |
| :-------------------------------- | :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| bind_address            | 127.0.0.1                   | IP that the CS teamserver  
| c2_profile            | None                   |  Filename of the C2_profile. Defaults to Mudge's Reference.
| cert_directory            | {{ playbook_dir }}/../certificates                   | Location of the TLS certificates 
| cert_filename            | {{ domain }}_cert.pem                   | Filename of the TLS public certificate
| domain            | None                   | Registered domain name of C2 
| priv_cert_filename            | {{ domain }}_privkey.pem                   | Filename of the TLS private certificate 
| profile_directory            | {{ playbook_dir }}/../c2_profiles/                   | Location of the C2 profiles 
| teamserver_password            | passypass                   | Password for the Teamserver
| tls_pass            | badPassw0rd                    | Password used for TLS JKS

## Examples
### Playbook Execution
```yaml
- name: Cobalt Strike Deployment
  hosts: all
  tasks:
    - set_fact:
        tls_pass: "{{ lookup('password', '/dev/null length=16 chars=ascii_letters') }}"
    - name: Cobalt Strike Deployment
      include_role:
        name: jm_cobalt_strike
      tags: always
```

### Teamserver Execution

Execute the playbook with `--extra-vars` to avoid placing your license key or password in any code.

```bash
ansible-playbook --extra-vars 'license_key=aaaa-bbbb-cccc-dddd bind_address=1.1.1.1 teamserver_password=goodpass domain=megacorp.com c2_profile=jquery-3.3.1.profile' --user=admin --private-key=./data/ssh_keys/172.16.139.216 -i 172.16.139.216, ./cobalt_strike.yml

```

Installing as `operator` only requires a license key variable. The interpreter directive may not be necessary depending on the environment.

```bash
ansible-playbook --extra-vars "license_key=1234-5678-1234-5678 cobalt_strike_role=operator ansible_python_interpreter=/usr/bin/python3" --user=$USER --connection=local -i localhost, ./cobalt_strike.yml
```

## Contributions

- The original base role was created by [@chryzsh](https://github.com/chryzsh/).
- This role created by [@joeminicucci](https://joeminicucci.com/)

## TODO 
- OpenJDK 11 support - recommended sources:
  - https://adoptopenjdk.net/
  - jdk.java.net
- Dynamic C2 generation using either [C2 Concealer](https://github.com/FortyNorthSecurity/C2concealer) or [C2 Randomizer](https://github.com/bluscreenofjeff/Malleable-C2-Randomizer) or simply templating Jinja in native Ansible.
- Aggressor script inclusion. Some sources:
  - [CS Toolkit](https://github.com/killswitch-GUI/CobaltStrike-ToolKit)
