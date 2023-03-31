---
title: "How to bump version number on Git push for projects using npm"
date: 2023-04-01
series: ["Coding"]
weight: 1
tags: ["Coding", "Git", "Husky"]
author: "Arto Salminen"
draft: false
---

Note: **This works only from command line** as user feedback is needed from keyboard to know which part of the version number to bump. It should not break pushing from a GUI tool (e.g. TortoiseGit), but you will not be prompted to update the version number. The positive thing is that the version change is done neatly as a separate commit.

1. Install `husky` as dev-dependency

   See [instructions](https://typicode.github.io/husky/#/?id=install).

2. Add the following script into new file _.husky/pre-push_

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

red=`tput setaf 1`
green=`tput setaf 2`
yellow=`tput setaf 3`
reset=`tput sgr0`

if sh -c ": >/dev/tty" >/dev/null 2>/dev/null; then
    # /dev/tty is available and usable
    # Allows us to read user input below, assigns stdin to keyboard
    exec < /dev/tty

    echo "${yellow}Enter version update you want to do:"
    echo "  [p]atch (e.g. 0.1.3 -> 0.1.4)"
    echo "  m[i]nor (e.g. 0.1.4 -> 0.2.0)"
    echo "  m[a]jor (e.g. 0.2.0 -> 1.0.0)"
    echo "  [n]one  (version number not changed)${reset}"
    read update_type

    case "$update_type" in
        patch | [pP] )
            yarn version --patch --no-commit-hooks
            ;;
        minor | [iI] )
            yarn version --minor --no-commit-hooks
            ;;
        major | [aA])
            yarn version --major --no-commit-hooks
            ;;
        none | [nN])
            echo "No version change"
            ;;
        *)
            echo "${red}Invalid update type: $update_type ${reset}"
            ;;
    esac

else
    # /dev/tty is not available
    echo "${red}[pre-push hook] Use command line to set the version number on push${reset}"
fi
```

3. Do the Git push from Terminal

`git push origin <your branch here>:<your remote branch here>`

## Optionally, if you need to update Helm chart versions as well

Change the script to include Helm Chart.xml update part.

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

red=`tput setaf 1`
green=`tput setaf 2`
yellow=`tput setaf 3`
reset=`tput sgr0`

version_updated=false

if sh -c ": >/dev/tty" >/dev/null 2>/dev/null; then
    # /dev/tty is available and usable
    # Allows us to read user input below, assigns stdin to keyboard
    exec < /dev/tty

    echo "${yellow}Enter version update you want to do:"
    echo "  [p]atch (e.g. 0.1.3 -> 0.1.4)"
    echo "  m[i]nor (e.g. 0.1.4 -> 0.2.0)"
    echo "  m[a]jor (e.g. 0.2.0 -> 1.0.0)"
    echo "  [n]one  (version number not changed)${reset}"
    read update_type

    case "$update_type" in
        patch | [pP] )
            yarn version --patch --no-commit-hooks
            version_updated=true
            ;;
        minor | [iI] )
            yarn version --minor --no-commit-hooks
            version_updated=true
            ;;
        major | [aA])
            yarn version --major --no-commit-hooks
            version_updated=true
            ;;
        none | [nN])
            echo "No version change"
            ;;
        *)
            echo "${red}Invalid update type: $update_type ${reset}"
            ;;
    esac

else
    # /dev/tty is not available
    echo "${red}[pre-push hook] Use command line to set the version number on push${reset}"
fi

if [ "$version_updated" = true ] ; then
    the_version=$(git describe --abbrev=0)
    perl -pi -e "s/^appVersion:.*$/appVersion: ${the_version:1}/" helm/Chart.yaml
    git tag -d ${the_version}
    git add helm/Chart.yaml
    git commit -m "Set appVersion to ${the_version} in Helm Chart" -n
    git tag ${the_version}
fi
```

