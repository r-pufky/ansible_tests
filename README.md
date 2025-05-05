# Extended testing framework
Common testing patterns for ansible roles.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_tests/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_tests/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_tests/tree/main/defaults/main/)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
Only use this role during Molecule testing and never for live roles. See each
task for detailed argument list. Tasks must be explicitly called.

### assertions (assert logic)
Verify assertion logic for roles are trigger correctly.

Assertion tests must be called during the **coverage** step; otherwise failures
will never allow Molecule to reach the verification step.

converge.yml
``` yaml
- name: 'role my_role fails when mutually exclusive assertion logic fails'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.tests'
    tasks_from: 'assertions.yml'
  vars:
    test_name: 'test mutually-exclusive gate'
    test_role: 'r_pufky.srv.my_role'
    test_fail_message: 'my_var, my_var2 did not trigger mutually exclusive assertion.'
    my_var: true
    my_other_var: 'alternative'
```

### copy (static file testing)
Static test file contents and permissions.

Copies a known good testing file to remote system and validates target file is
the same.

verify.yml
``` yaml
- name: 'Verify cert.pem'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.tests'
    tasks_from: 'copy.yml'
  vars:
    test_name: 'Verify cert.pem'
    test_src: '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") }}/molecule/cache/cert.pem'
    test_file: '/etc/mail/cert.pem'
    test_owner: 'mail'
    test_group: 'mail'
    test_mode: '0400'
    test_diff: false
```

### file (existence and permissions)
Test file existence and permissions. May be applied to file **or** directories.

verify.yml
``` yaml
- name: 'Verify permissions'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.tests'
    tasks_from: 'file.yml'
  vars:
    test_name: 'Verify permissions | {{ item }}'
    test_file: '{{ item }}'
    test_owner: '305'
    test_group: '305'
    test_mode: '0755'
    test_state: 'directory'
  loop:
    - '/var/lib/forgejo/gitea-repositories'
    - '/var/lib/forgejo/data/tmp/uploads'
    - '/var/lib/forgejo/tmp/local-repo'
```

### lineinfile (existence of line in file)
Test existence of line in file.

verify.yml
``` yaml
- name: 'Verify | assert postgres settings'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.tests'
    tasks_from: 'lineinfile.yml'
  vars:
    test_name: 'Verify | assert postgres settings | {{ item }}'
    test_file: '/etc/forgejo/app.ini'
    test_line: '{{ item }}'
    test_state: 'present'
  loop:
    - 'DB_TYPE = postgres'
    - 'HOST = 127.0.0.1'
    - 'NAME = forgejo'
    - 'USER = root'
    - 'PASSWD = `test`'
```

### remote_file_diff (dynamic runtime file testing)
Remote test file contents and permissions.

Tests requiring run-time options (e.g. injected hashes that must exist but
are random per-run) should use this instead of copy test to verify file
contents are the same. Both files must exist on the remote system (generally
requiring the source of truth file to be rendered on the remote host in
converge).

verify.yml
``` yaml
- name: 'Verify | app.ini'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.tests'
    tasks_from: 'remote_file_diff.yml'
  vars:
    test_name: 'Verify | app.ini'
    test_remote_src: '/tmp/app.ini'
    test_file: '/etc/forgejo/app.ini'
    test_owner: 'git'
    test_group: 'git'
    test_mode: '0640'
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_tests/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
