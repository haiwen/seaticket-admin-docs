# seaqa_web_settings.py

`seaqa_web_settings.py` is used to customize the SeaTicket web (seaqa-web) runtime settings. The file is loaded when SeaTicket starts, and values defined here override the defaults inside the SeaTicket codebase.

!!! tip
    Place `seaqa_web_settings.py` in the SeaTicket configuration directory that is mounted into the `seaqa-web` container (the directory referenced by `CONF_DIR`). Restart SeaTicket after modifying this file.

## Team role permissions

SeaTicket provides built-in **team roles**, and each role has a set of permissions. You can customize these permissions with the `ENABLED_ROLE_PERMISSIONS` setting.

Built-in team roles:

- `free`
- `start`
- `pro`
- `business`
- `enterprise`

### Available permissions

The following options can be configured per team role:

| Option | Description | Default |
| --- | --- | --- |
| `can_add_project` | Allow users in the role to create projects. | `True` |
| `can_add_group` | Allow users in the role to create groups. | `True` |
| `can_use_saml` | Allow SAML SSO for this role. | `False` for free/start/pro, `True` for business/enterprise |
| `ai_credit_per_user` | AI credits per user. Set `-1` for unlimited. | `-1` |

### Configuration example

```python
# seaqa_web_settings.py
ENABLED_ROLE_PERMISSIONS = {
    'free': {
        'can_add_project': True,
        'can_add_group': True,
        'can_use_saml': False,
        'ai_credit_per_user': -1,
    },
    'start': {
        'can_use_saml': False,
        'ai_credit_per_user': 2000,
    },
    'pro': {
        'ai_credit_per_user': 5000,
    },
    'business': {
        'can_use_saml': True,
    },
    'enterprise': {
        'can_use_saml': True,
    },
}
```

### Merge behavior

SeaTicket merges your custom settings with the defaults:

- If you only override part of a role’s permissions, the missing fields inherit from the default settings.
- If you omit a role entirely, it inherits from `free` defaults.

For example, the above configuration only changes the keys you specify and keeps the rest consistent with the built-in defaults.
