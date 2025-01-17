python-config-parser
===
---
[![Tests](https://github.com/BrunoSilvaAndrade/python-config-parser/actions/workflows/tests.yml/badge.svg)](https://github.com/BrunoSilvaAndrade/python-config-parser/actions/workflows/tests.yml)
[![PyPI version](https://badge.fury.io/py/python-config-parser.svg)](https://badge.fury.io/py/python-config-parser)
[![Coverage Status](https://coveralls.io/repos/github/BrunoSilvaAndrade/python-config-parser/badge.svg)](https://coveralls.io/github/BrunoSilvaAndrade/python-config-parser)

python-config-parser lets you create runtime configuration objects using json or yaml files.

MAIN FEATURES
===
---
* Declarative configurations without using .ini files
* Access using OOP or subscriptable, which means that you can iterate the config object items
* Runtime validation using [schema](https://github.com/keleshev/schema)
* Automatic environment variables interpolation
* Automatic parser selecting by config file extension

HOW TO INSTALL
===
---
Use `pip` to install it.

```shell
pip install python-config-parser
```

HOW TO USE
===
---
By default, the config file will look for the following config files in the `.config` directory: `config.json`, `config.yaml`, `config.yml`.

You can also pass a config directory and or config file of your preference (assuming your current directory).

```python
from pyconfigparser import configparser

configparser.get_config(CONFIG_SCHEMA, config_dir='your_config_dir_path', file_name='your_config_file_name')
```

Schema validation
---

You may or may not use schema validation. If you want to use it, it will validate the whole config object before returning it.

If you choose not to use it, it won't validate the config object before returning it, and it may generate runtime access inconsistencies.

How to use schema

```python
from schema import Use, And

SCHEMA_CONFIG = {
    'core': {
        'logging': {
            'format': And(Use(str), lambda string: len(string) > 0),
            'date_fmt': And(Use(str), lambda string: len(string) > 0),
            'random_env_variable': str
        },
        'allowed_clients': [{
                'ip': str, # <- Here you can use regex to validate the ip format
                'timeout': int
            }
        ]
    }
}

```

The `config.yml` file
```yaml
core:
  random_env_variable: ${RANDOM_ENV_VARIABLE}
  logging:
    format: "[%(asctime)s][%(levelname)s]: %(message)s"
    date_fmt: "%d-%b-%y %H:%M:%S"
  allowed_clients:
  - ip: 192.168.0.10
    timeout: 60
  - ip: 192.168.0.11
    timeout: 100
```
A json config file would be something like:

```json
{
  "core": {
    "random_env_variable": "${RANDOM_ENV_VARIABLE}",
    "logging": {
      "format": "[%(asctime)s][%(levelname)s]: %(message)s",
      "date_fmt": "%d-%b-%y %H:%M:%S"
    },
    "allowed_clients": [
      {
        "ip": "192.168.0.10",
        "timeout": 60
      },
      {
        "ip": "192.168.0.11",
        "timeout": 100
      }
    ]
  }
}
```

The config instance
```python
from pyconfigparser import configparser, ConfigError
import logging

try:
    config = configparser.get_config(SCHEMA_CONFIG)  # <- Here I'm using that SCHEMA_CONFIG we had declared, and the dir file default value is being used
except ConfigError as e:
    print(e)
    exit()

# to access your config you need just:

fmt = config.core.logging.format # look this, at this point I'm already using the config variable
date_fmt = config['core']['logging']['date_fmt'] # here subscriptable access

logging.getLogger(__name__)

logging.basicConfig(
    format=fmt,
    datefmt=date_fmt,
    level=logging.INFO
)

# the list of object example:

for client in config.core.allowed_clients:
    print(client.ip)
    print(client.timeout)
    
# The config object's parts which is not a list can also be itered but, it'll give you the attribute's names
# So you can access the values by subscriptale access
for logging_section_attr_key in config.core.logging:
    print(config.core.logging[logging_section_attr_key])

# Accessing the environment variable already resolved
print(config.random_env_variable)

```
Since you've already created the first Config's instance this instance will be cached inside Config class,
so after this first creation you can just invoke Config.get_config()

```python
from pyconfigparser import configparser

config = configparser.get_config()  # At this point you already have the configuration properties in your config object
```

You can also disable the action to cache the instance config


```python
from pyconfigparser import configparser

configparser.hold_an_instance = False
```

Environment Variables Interpolation
---
If the process does not find a value already set to your env variables
It will raise a ConfigError. But you can disable this behavior, and the parser will set `None` to these unresolved env vars

```python
from pyconfigparser import configparser

configparser.ignore_unset_env_vars = True
config = configparser.get_config()
```

CONTRIBUTE
---
---

Fork https://github.com/BrunoSilvaAndrade/python-config-parser/ , create commit and pull request to ``develop``.
