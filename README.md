### Requirements

Lithium QA can be used as standalone application _or_ a plugin. The first requiring just lithium itself, the latter requiring an application and lithium. We take care that Lithium QA master is compatible with most recent lithium core. However sometimes changes have to be made rendering QA incompatible with older lithium cores. In that case please choose a tag version of Lithium QA.

### Installation

This project makes use of git submodules. In order to install, execute the following commands from the console. Please note that components of Lithium QA interact with the VCS of your project. Currently only GIT is supported a such.

```
cd ~
git clone code@rad-dev.org:lithium_qa.git 
cd lithium_qa
git submodule init
git submodule update
```

### Syntax Command

Files can be syntax checked using the Syntax command which comes with the application. The command utilizes [PHPca](http://github.com/UnionOfRAD/phpca/) to check the syntax of PHP files against a set of rules. These rules are based upon the [Lithium Coding Standards](http://dev.lithify.me/lithium/wiki/standards/coding) and the [Lithium Code Documentation](http://dev.lithify.me/lithium/wiki/standards/documenting) Standards.

The basic usage is: 

```
li3 syntax [--metrics] [--blame] PATH
```

Here we are checking the whole file tree of our pet project:

```
cd path/to/lithium_qa
li3 syntax /path/to/pet
```

If you want to log the output to a file (and your on *nix) this might come in handy:

```
li3 syntax /path/to/pet 2>&1 | tee verify_pet.log
```

Blame each failure and show metrics:

```
li3 syntax --metrics --blame /path/to/pet
```

### GIT Pre Commit Hook

This pre commit hook is based upon the example found in `.git/hooks/pre-commit.sample`. Copy the sample script to `/path/to/project/.git/hooks/pre-commit` and make it executable. Then, replace the code in the script with the code shown below and adjust the paths to Lithium QA and the li3 command.

```
cd /path/to/project
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit
chmod a+x .git/hooks/pre-commit
```
   
Now add the following code to .git/hooks/pre-commit and adjust the `LITHIUM_QA` and `LI3` values.

```sh
#!/bin/sh

LITHIUM_QA=/path/to/lithium_qa
LI3=/path/to/lithium/libraries/lithium/console/li3

if git-rev-parse --verify HEAD >/dev/null 2>&1
then
    AGAINST=HEAD
else
    # Initial commit: diff against an empty tree object
    AGAINST=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

EXIT_STATUS=0
PROJECT=`pwd`

for FILE in `git diff-index --cached --name-only --diff-filter=AM ${AGAINST}`
do
    cd ${LITHIUM_QA} && ${LI3} syntax ${PROJECT}/${FILE}
    test $? != 0 && EXIT_STATUS=1
done

exit ${EXIT_STATUS}
```

Now when committing each file the syntax is checked. The commit is aborted if a check failed. If you don't want to have the hook run on commit pass the `--no-verify` option to git commit.

### Covered Command

Allows for checking if classes in a given library all have corresponding tests according to the Lithium standard. Also prints out coverage percentage.
```
li3 covered /path/to/library
```

Will output something similar to the following.

```sh
has test |  95.00% | lithium\util\Collection
has test |  97.98% | lithium\util\Inflector
has test | 100.00% | lithium\util\collection\Filters
has test |     n/a | lithium\test\Controller
has test |  95.24% | lithium\test\Dispatcher
 no test |     n/a | lithium\test\Filter
```

