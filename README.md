First, we need a development dependency specified to install phpcs. It looks something like this:

```
{
    "require-dev": {
        "manoj-malviya/php_codesniffer": "^2.4"
    }
}
```

Install Scripts
---------------

Composer has a handy schema entry called scripts. It supports a script hook post-install-cmd. We will use this to install a git pre-commit hook. Adding to our example above:

```
{
    "require-dev": [
        "squizlabs/php_codesniffer": "2.0.*@dev"
    ],
    "scripts": {
        "post-install-cmd": [
            "bash contrib/setup.sh"
        ]
    }
}
```

This will run a bash script called setup.sh when the command composer install is run.

Setup the Git Pre-commit Hook
-----------------------------

In our setup.sh, we will need to copy a pre-commit script into the .git/hooks directory:

```
#!/bin/sh

cp contrib/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```