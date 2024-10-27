---
title: "Programmatically invoking JetBrains IDE actions"
description: "Performing actions via the JetBrains IDE scripting console"
date: 2023-05-20T13:22:32+08:00
draft: false
categories:
- dev
tags:
- bash
- ide
- automation
featured_image:
  url: "https://res.cloudinary.com/j4ckofalltrades/image/upload/v1709917134/blog/ide-scripting_w6bswp.png"
  alt_text: "Intellij IDEA with a 'Hello world!' popup message"
---

_Update (2024-03-08):_

- _Removed embedded gist and linked to GitHub instead._
- _Add banner image and demo gif to this post._
- _JetBrains has since released an [official plugin](https://github.com/JetBrains/intellij-streamdeck-plugin) on August 2023 but if you're running Linux (Stream Deck SDK only supports Windows and Mac currently) you may still find this article guide useful._

I've had a Stream Deck for a while now but haven't really configured it for any coding related workflows.
I use several JetBrains IDEs for work and personal use (Intellij, PyCharm, WebStorm to name a few), so I
started looking into what the possible options were.

The simplest solution would be to just invoke keyboard shortcut for a specific but the downside of this
approach is you can only have so much keycodes assigned (which may conflict with system shortcuts and
possibly other apps), not to mention you'd have to change these when you switch keymaps or use
another operating system.

Ideally there would be a command where you can provide an *action* to the IDE, possibly through a plugin.

{{< toc >}}

## IDE Scripting Console

While reading through the Intellij documentation, I stumbled upon the [IDE scripting console](https://www.jetbrains.com/help/idea/ide-scripting-console.html).

> The IDE Scripting Console can be used to write simple scripts that automate IntelliJ IDEA
> features and extract various information. With access to the IntelliJ platform API, you can
> think of it as a lightweight alternative to a plugin, which adds or modifies some behavior
> of the IDE.
>
> By default, it supports scripts written in Kotlin, JavaScript, and Groovy. However, you can use any scripting language that is compliant with JSR 223, for example, Python, Ruby, Clojure, and so on.

This looks promising; I copied and modified some sample code from the docs to display a "Hello World"
message in a dialog in the IDE.

```kotlin
import com.intellij.openapi.ui.Messages

val b = bindings as Map<String, Any>
val IDE = b["IDE"] as com.intellij.ide.script.IDE

Messages.showInfoMessage("Hello World")
```

## Invoking Actions

The next step was figuring out how to invoke *actions* in the IDE.
The Intellij Platform Plugin SDK defines the requirements in its [Action system](https://plugins.jetbrains.com/docs/intellij/basic-action-system.html) documentation.

An `ActionManager` instance is used to execute an IDE `Action` that is referred to by its unique id
-- this can either be a custom action from an installed plugin or the [standard IntelliJ Platform actions](https://github.com/JetBrains/intellij-community/tree/idea/231.8109.175/platform/ide-core/src/com/intellij/openapi/actionSystem/IdeActions.java).

The following snippet executes the standard *NextTab* action:

```kotlin
import com.intellij.openapi.actionSystem.ActionManager
import com.intellij.openapi.actionSystem.AnAction

val b = bindings as Map<String, Any>
val IDE = b["IDE"] as com.intellij.ide.script.IDE

val actionManager: ActionManager = ActionManager.getInstance()
// move focus to the next editor tab
val action: AnAction = actionManager.getAction("NextTab")
actionManager.tryToExecute(action, null, null, null, false)
```

## Invoking the script from the command line

Up to now, I've been able to play around with the scripts from within the IDEs -- the next step is
to find a way to invoke them externally.

Fortunately, this feature is already available since [version 2021.1](https://youtrack.jetbrains.com/issue/IDEA-245847).
It requires the command line scripts for the IDEs to be installed e.g. `idea` for Intellij, this can be configured via the [JetBrains Toolbox](https://www.jetbrains.com/help/idea/working-with-the-ide-features-from-command-line.html#toolbox).

The command to run script(s) is `idea ideScript <files>`.

Caveat: The script does not work (when invoked from the command line at least) unless the IDE action
execution code is wrapped in the following snippet:

```kotlin
IDE.application.invokeLater {
  // action execution code goes here 
}
```

My guess is that it needs to be non-blocking, since it is invoked externally. One other thing to note
is that if you have multiple windows of an IDE running, the action will be executed in the last
active window.

## Parameterizing the script

The last thing I wanted to add was to parameterize the script, where I can pass in the action name e.g.
"Run tests" and the IDE where the aforementioned action would be executed.

I tried a couple of things; my first implementation involved passing the params as environment
variables and parsing them in the script. That didn't seem to work as when I tried to log the params
they would always be `null`. The next thing I tried was to reading the params from a text file which
also did not work.

Eventually, my solution involved creating a shell script that:

1. Requires (and parses) the IDE name and action name as arguments
2. Writes out the IDE script to a file (taking params into account)
3. Executes the IDE script
4. Deletes the IDE script file

So if I wanted to perform an action on an Intellij IDEA window, the command will look like
`ide-script.sh --ide idea --action action_name_to_perform`

## Putting it all together

I've tested this script on both macOS and Linux. The steps should be similar on Windows; you also might
be able to reuse the shell script if you are running WSL but I imagine you'll need to update the path of the command-line IDE launchers.

```sh
#!/usr/bin/env bash

set -Eeuo pipefail
trap cleanup SIGINT SIGTERM ERR EXIT

script_dir=$(cd "$(dirname "${BASH_SOURCE[0]}")" &>/dev/null && pwd -P)
ide_script_name="ide_script.groovy"

usage() {
  cat <<EOF
Usage: $(basename "${BASH_SOURCE[0]}") [-h] [-f] -i ide -a action
Shell script wrapper that executes actions on JetBrains IDEs via the scripting console.
Available options:
-h, --help    Print this help and exit
-i, --ide     JetBrains IDE command-line launcher e.g. idea, webstorm, pycharm
-a, --action  IDE action to execute
EOF
  exit
}

cleanup() {
  trap - SIGINT SIGTERM ERR EXIT
  rm -f "$script_dir/$ide_script_name"
}

setup_colors() {
  if [[ -t 2 ]] && [[ -z "${NO_COLOR-}" ]] && [[ "${TERM-}" != "dumb" ]]; then
    NOFORMAT='\033[0m' RED='\033[0;31m' GREEN='\033[0;32m' ORANGE='\033[0;33m' BLUE='\033[0;34m' PURPLE='\033[0;35m' CYAN='\033[0;36m' YELLOW='\033[1;33m'
  else
    NOFORMAT='' RED='' GREEN='' ORANGE='' BLUE='' PURPLE='' CYAN='' YELLOW=''
  fi
}

msg() {
  echo >&2 -e "${1-}"
}

die() {
  local msg=$1
  local code=${2-1} # default exit status 1
  msg "$msg"
  exit "$code"
}

parse_params() {
  ide=''
  action=''

  while :; do
    case "${1-}" in
    -h | --help) usage ;;
    -v | --verbose) set -x ;;
    --no-color) NO_COLOR=1 ;;
    -f | --flag) flag=1 ;;
    -i | --ide)
        ide="${2-}"
        shift
        ;;
    -a | --action)
        action="${2-}"
        shift
        ;;
    -?*) die "Unknown option: $1" ;;
    *) break ;;
    esac
    shift
  done

  args=("$@")

  # check required params and arguments
  [[ -z "${ide-}" ]] && die "Missing required parameter: ide"
  [[ -z "${action-}" ]] && die "Missing required parameter: action"

  return 0
}

# Invoke supported IDE actions using the IDE scripting console. See links for more details.
# 
# Supported IDE actions: https://github.com/JetBrains/intellij-community/blob/idea/231.8109.175/platform/ide-core/src/com/intellij/openapi/actionSystem/IdeActions.java
# IDE scripting console: https://www.jetbrains.com/help/idea/ide-scripting-console.html
generate_ide_script() {
  cat <<EOF
import com.intellij.openapi.actionSystem.ActionManager
import com.intellij.openapi.actionSystem.AnAction
import com.intellij.openapi.ui.Messages
var actionManager = ActionManager.getInstance()
IDE.application.invokeLater {
  try {
    var action = actionManager.getAction("$action")
    var result = actionManager.tryToExecute(action, null, null, null, false)
    if (result.rejected) {
      Messages.showErrorDialog(result.error, "IDE action error")
    }
  } catch (ex) {
    Messages.showErrorDialog(ex.message, "IDE action error")
  }
}
EOF
}

parse_params "$@"
setup_colors

# Generate IDE script and execute the given action
ide_script="$script_dir/$ide_script_name"
generate_ide_script > "$ide_script"
$ide ideScript "$ide_script"
cleanup
```

The code is also available on GitHub as a [gist](https://gist.github.com/j4ckofalltrades/d7aac303466746e67287441e4fb9e0fe).

## Demo: Launching the Help Menu programmatically

The `Help Menu` action is invoked programmatically by clicking on a configured button on the Stream Deck.

![Stream deck UI and an Intellij IDEA instance side-by-side running IDE actions launching the Help Menu in the IDE programmatically](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1709915929/blog/jetbrains-ide-actions_qyighl.gif)

View full size image [here](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1709915929/blog/jetbrains-ide-actions_qyighl.gif).

I hope others find this useful; now on to configuring the Stream Deck.
