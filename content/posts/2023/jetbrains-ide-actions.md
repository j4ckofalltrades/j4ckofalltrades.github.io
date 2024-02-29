---
title: "Programatically invoking JetBrains IDE actions"
description: "Performing actions via the JetBrains IDE scripting console"
date: 2023-05-20T13:22:32+08:00
draft: false
categories:
- dev
tags:
- bash
- ide
- automation
---

I've had a Stream Deck for a while now but haven't really configured it
for any coding related workflows. I use several JetBrains IDEs for work and
personal use (Intellij, PyCharm, WebStorm to name a few), so I started looking
into what the possible options were.

The simplest solution would be to just invoke keyboard shortcut for a specific
but the downside of this approach is you can only have so much keycodes assigned
(which may conflict with system shortcuts and possibly other apps), not to
mention you'd have to change these when you switched keymaps or used another
operating system.

Ideally there would be a command where you can provide an *action* to the IDE,
possibly through a plugin.

## IDE Scripting Console

While reading through the Intellij documentation, I
stumbled upon the [IDE scripting console](https://www.jetbrains.com/help/idea/ide-scripting-console.html).

> The IDE Scripting Console can be used to write simple scripts that automate IntelliJ IDEA
> features and extract various information. With access to the IntelliJ platform API, you can
> think of it as a lightweight alternative to a plugin, which adds or modifies some behavior
> of the IDE.
>
> By default, it supports scripts written in Kotlin, JavaScript, and Groovy. However, you can use any scripting language that is compliant with JSR 223, for example, Python, Ruby, Clojure, and so on.

This looks promising; I copied and modified some sample code from the docs to
display a "Hello World" message in a dialog in the IDE.

```kotlin
import com.intellij.openapi.ui.Messages

val b = bindings as Map<String, Any>
val IDE = b["IDE"] as com.intellij.ide.script.IDE

Messages.showInfoMessage("Hello World")
```

## Invoking Actions

The next step was figuring out how to invoke *actions* in the IDE.
The Intellij Platform Plugin SDK defines the requirements in its
[Action system](https://plugins.jetbrains.com/docs/intellij/basic-action-system.html) documentation.

An `ActionManager` instance is used to execute an IDE `Action` that is referred
to by its unique id -- this can either be a custom action from an installed plugin or the [standard IntelliJ Platform actions](https://github.com/JetBrains/intellij-community/tree/idea/231.8109.175/platform/ide-core/src/com/intellij/openapi/actionSystem/IdeActions.java).

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

Up to now, I've been able to play around with the scripts from within the IDEs
-- the next step is to find a way to invoke them externally.

Fortunately, this feature is already available since [version 2021.1](https://youtrack.jetbrains.com/issue/IDEA-245847);
it requires the command line scripts for the IDEs to be installed e.g.
`idea` for Intellij IDEA. This can be configured via the [JetBrains Toolbox](https://www.jetbrains.com/help/idea/working-with-the-ide-features-from-command-line.html#toolbox).

The command to run script(s) is `idea ideScript <files>`.

Caveat: The script does not work (when invoked from the command line at least) unless
the IDE action execution code is wrapped in the following snippet:

```kotlin
IDE.application.invokeLater {
  // action execution code goes here 
}
```

My guess is that it needs to be non-blocking, since it is invoked externally.
One other thing to note is that if you have multiple windows of an IDE running,
the action will be executed in the last active window.

## Parameterizing the script

The last thing I wanted to add was to parameterize the script, where I could
pass in the action name e.g. "Run tests" and the IDE where the aforementioned
action would be executed.

I tried a couple of things; my first implementation involved passing the params
as environment variables and parsing them in the script. That didn't seem to
work as when I tried to log the params they would always be `null`. The next
thing I tried was to reading the params from a text file which also did not work.

Eventually, my solution involved creating a shell script that:

1. Requires (and parses) the IDE name and action name as arguments
2. Writes out the IDE script to a file (taking params into account)
3. Executes the IDE script
4. Deletes the IDE script file

So if I wanted to perform an action on an Intellij IDEA window, the command
will look like `ide-script.sh --ide idea --action action_name_to_perform`

## Putting it all together

As a side note, Intellij IDEA ships with Kotlin so the samples will work there
-- it may work on other IDEs if the Kotlin plugin is installed but I haven't
tested that so YMMV.

I ended up using Groovy since that ships with the other IDEs I mentioned earlier
and I've been able to verify that it works on WebStorm and PyCharm. As for
operating systems, I've tested this script on both macOS and Linux. The steps
should be similar on Windows; you also might be able to reuse the shell script
if you are running WSL but I imagine you'll need to update the path of the
command-line IDE launchers.

<script src="https://gist.github.com/j4ckofalltrades/d7aac303466746e67287441e4fb9e0fe.js"></script>

You can find the full code on [GitHub](https://gist.github.com/j4ckofalltrades/d7aac303466746e67287441e4fb9e0fe).

I hope others find this useful; now on to configuring the Stream Deck.
