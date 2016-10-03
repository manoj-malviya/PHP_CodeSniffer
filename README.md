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
        "manoj-malviya/php_codesniffer": "^2.4"
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

This will copy our pre-commit script from the contrib directory to the hooks section of the special git directory and make it executable.

Create the Pre-commit Hook
--------------------------

Whenever a contributing developer attempts to commit their code, it will run our pre-commit script. Now all we need to do is run the code sniffer rules on relavent files specific to this commit:

```
#!/bin/sh

PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"`
STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`

# Determine if a file list is passed
if [ "$#" -eq 1 ]
then
    oIFS=$IFS
    IFS='
    '
    SFILES="$1"
    IFS=$oIFS
fi
SFILES=${SFILES:-$STAGED_FILES_CMD}

echo "Checking PHP Lint..."
for FILE in $SFILES
do
    php -l -d display_errors=0 $PROJECT/$FILE
    if [ $? != 0 ]
    then
        echo "Fix the error before commit."
        exit 1
    fi
    FILES="$FILES $PROJECT/$FILE"
done

if [ "$FILES" != "" ]
then
    echo "Running Code Sniffer..."
    ./vendor/bin/phpcs --standard=PSR1 --encoding=utf-8 -n -p $FILES
    if [ $? != 0 ]
    then
        echo "Fix the error before commit."
        exit 1
    fi
fi

exit $?
```

This script will get the staged files of the commit, run a php lint check (always nice), and apply the code sniffer rules to the staged files.

If there is a code standards violation, the phpcs process will return a non-zero exit status which will tell git to abort the commit.

Bringing it all together
------------------------

With all of these things in place, the workflow is as follows:

Developer runs composer install.
PHP Code Sniffer is installed via a dev dependency.
The post-install command automatically copies the pre-commit hook into the developer’s local git hooks.
When the developer commits code, the pre-commit hook fires and checks the staged files for coding standards violations and lint checks.
This is a relatively simple setup that can save pull request code reviews a significant amount of time preventing back-and-forth on simple things such as mixed tabs/spaces, bracket placement, etc.