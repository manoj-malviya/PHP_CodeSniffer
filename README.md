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

```{
    "require-dev": [
        "squizlabs/php_codesniffer": "2.0.*@dev"
    ],
    "scripts": {
        "post-install-cmd": [
            "bash contrib/setup.sh"
        ]
    }
```}