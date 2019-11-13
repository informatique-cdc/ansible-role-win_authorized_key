# win_authorized_key - Adds or removes an SSH authorized key

## Synopsis

* Ansible module to add or to remove SSH authorized keys for particular user accounts on Windows-based systems.

## Requirements

* [Win32 OpenSSH]

## Parameters

| Parameter      | Choices/Defaults                              | Comments                                                                                                                                                                                                                                                                                                                                                                                        |
| -------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| comment        |                                           | Change the comment on the public key.<br/>Rewriting the comment is useful in cases such as fetching it from GitHub or GitLab.<br/>If no comment is specified, the existing comment will be kept.                                                                                                                                                                                                |
| exclusive      | Choices:<ol><li>`no` <-<li>`yes`</ol>         | Whether to remove all other non-specified keys from the authorized_keys file.<br/>Multiple keys can be specified in a single key string value by separating them by newlines. This option is not loop aware, so if you use with_ , it will be exclusive per iteration of the loop, if you want multiple keys in the file you need to pass them all to key in a single batch as mentioned above. |
| key            |                                               | The SSH public key(s), as a string or url (https://github.com/username.keys)                                                                                                                                                                                                                                                                                                                    |
| key_options    |                                               | A string of ssh key options to be prepended to the key in the authorized_keys file                                                                                                                                                                                                                                                                                                              |
| manage_dir     | Choices:<ol><li>`no`<li>`yes` <-</ol>         | Whether this module should manage the directory of the authorized key file.<br/>If set to `yes`, the module will create the directory. Be sure to set `manage_dir=no` if you are using an alternate directory for authorized_keys, as set with `path`, since you could lock yourself out of SSH access.                                                                                         |
| path           |                                               | Alternate path to the authorized_keys file.<br/>When unset, this values defaults to `($home)+/.ssh/authorized_keys`                                                                                                                                                                                                                                                                             |
| state          | Choices:<ol><li>`absent`<li>`present` <-</ol> | Whether the given key (with the given key_options) should or should not be in the file.                                                                                                                                                                                                                                                                                                         |
| user           |                                               | The username on the remote host whose authorized_keys file will be modified.                                                                                                                                                                                                                                                                                                                    |
| validate_certs | Choices:<ol><li>`no`<li>`yes` <-</ol>         | This only applies if using a https url as the source of the keys.<br/>If set to `no`, the SSL certificates will not be validated.<br/>This should only set to no used on personally controlled sites using self-signed certificates as it avoids verifying the source site.                                                                                                                     |

## Examples

```yaml
---
  roles:
    - win_authorized_key

  tasks:

    - name: Set authorized key taken from file
      win_authorized_key:
        user: charlie
        state: present
        key: "{{ lookup('file', 'c:/users/charlie/.ssh/id_rsa.pub') }}"

    - name: Set authorized keys taken from url
      win_authorized_key:
        user: charlie
        state: present
        key: https://github.com/charlie.keys

    - name: Set authorized key in alternate location
      win_authorized_key:
        user: charlie
        state: present
        key: "{{ lookup('file', 'c:/users/charlie/.ssh/id_rsa.pub') }}"
        path: c:/ProgramData/ssh/administrators_authorized_key
        manage_dir: False

    - name: Set up multiple authorized keys
      win_authorized_key:
        user: deploy
        state: present
        key: '{{ item }}'
      with_file:
        - public_keys/doe-jane
        - public_keys/doe-john

    - name: Set authorized key defining key options
      win_authorized_key:
        user: charlie
        state: present
        key: "{{ lookup('file', 'c:/users/charlie/.ssh/id_rsa.pub') }}"
        key_options: 'no-port-forwarding,from="10.0.1.1"'

    - name: Set authorized key without validating the TLS/SSL certificates
      win_authorized_key:
        user: charlie
        state: present
        key: https://github.com/user.keys
        validate_certs: False

    - name: Set authorized key, removing all the authorized keys already set
      win_authorized_key:
        user: root
        key: '{{ item }}'
        state: present
        exclusive: True
      with_file:
        - public_keys/doe-jane
```

## Return Values

The following are the fields unique to this module:

| Key            | Description                                                                                                                   | Returned | Type    | Example                                |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- | -------: | ------- | -------------------------------------- |
| exclusive      | If the key has been forced to be exclusive or not.                                                                            |  success | boolean |                                        |
| key            | The key that the module was running against.                                                                                  |  success | string  | `https://github.com/user.keys`         |
| key_option     | Key options related to the key.                                                                                               |  success | string  |                                        |
| keyfile        | Path for authorized key file.                                                                                                 |  success | string  | `c:\users\charlie\ssh\authorized_keys` |
| manage_dir     | Whether this module managed the directory of the authorized key file.                                                         |  success | boolean |                                        |
| path           | Alternate path to the authorized_keys file.                                                                                   |  success | string  |                                        |
| state          | Whether the given key (with the given key_options) should or should not be in the file.                                       |  success | string  |                                        |
| user           | The username on the remote host whose authorized_keys file will be modified.                                                  |   string | boolean |                                        |
| validate_certs | This only applies if using a https url as the source of the keys. If set to `no`, the SSL certificates will not be validated. |  success | boolean |                                        |

## License

GNU General Public License v3.0

See [LICENSE](LICENSE) to see the full text.

## Author Information

* [Brad Olson (brado@movedbylight.com)](mailto:brado@movedbylight.com) - *Initial work in Python*
* [Stéphane Bilqué](https://github.com/sbilque)  - *Translation in PowerShell*

[Win32 OpenSSH]: https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH
