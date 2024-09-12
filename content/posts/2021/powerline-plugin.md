---
title: "Writing a custom Powerline plugin"
description: Guide for writing (and publishing) custom Powerline plugins
date: 2021-09-12T20:20:20+08:00
draft: false
categories:
- dev
tags:
- kubernetes
- powerline
- pypi
- python
featuredImage: "https://raw.githubusercontent.com/j4ckofalltrades/dotfiles/master/assets/env.png"
featuredImageAltText: "Screenshot of a terminal running tmux and powerline"
---

[Powerline](https://github.com/powerline/powerline) is a tool I use as part of my dev environment setup, and have my config backed up as part of my [dotfiles](https://github.com/j4ckofalltrades/dotfiles).

While it comes with a lot of integrations out of the box i.e. `bash`, `zsh`, `tmux`, `vim`, etc, it also provides
a way for you to write your own "segments". This should serve as a quick guide for rolling your own custom Powerline plugin.

{{< toc >}}

## Basic structure and configuration

Each powerline segment is a callable object. It is supposed to be either a Python function or
a subclass of the `powerline.segments.Segment` class.

I recently wrote my own custom one that displays the current Kubernetes context and namespace,
which uses a `Segment` class. Here is a shortened version which shows the basic structure.

```python
class KubernetesSegment(Segment):
    """Constructs the segment's sections with the configured colorscheme and
    visibility options applied."""

    @staticmethod
    def kube_ctx_info(pl):
        """Resolves the current active Kubernetes context (and namespace)
        from `$KUBECONFIG`."""
        try:
            current_context = config.list_kube_config_contexts()[1]
            return current_context['name'] or 'N/A', \
                   current_context['context']['namespace'] or 'default'
        except Exception as e:
            pl.error(e)

    def __call__(self, pl):
        pl.debug('Running powerline-k8s...')

        sections = []

        # additional logic to determine segment contents

        sections.append({
            'contents': f'u'\U00002638' ',
            'highlight_groups': 'k8s',
            'divider_highlight_group': 'k8s:divider',
        })

        return sections

k8s = with_docstring(KubernetesSegment(),
"""Return the current Kubernetes context and namespace.

It will show the current context and namespace from `$KUBECONFIG`.

Divider highlight group used: ``k8s:divider``.

Highlight groups used: ``k8s``, ``k8s_context``, ``k8s_namespace``.
""")
"""Custom segment entry point."""
```

*Make sure to add the [powerline-status](https://pypi.org/project/powerline-status) package as a dependency.*

Basically the class contains a function that returns a [Segment dictionary](https://powerline.readthedocs.io/en/latest/develop/segments.html#segment-dictionary) which tells `Powerline` what to display.

In this particular example, the following keys were used:

- `contents`: Actual segment contents, excluding dividers and before/after.
May be `None`.

- `highlight_groups`, `divider_highlight_group`: Used highlight groups.  May be `None`.

Highlight groups determine the 'style' that is used for a particular segment e.g. background and foreground,
divider color to clearly distinguish one segment from another.

The colors that are available to you will depend on the current colorscheme that you are using. In general
you'll need to add the "groups" definition to the colorscheme config file, which in this case was
`<powerline_dir>/colorschemes/solarized.json`.

```json
{
  "k8s":           { "fg": "solarized:blue", "bg": "solarized:base02", "attrs": [] },
  "k8s:divider":   { "fg": "gray4",          "bg": "solarized:base02", "attrs": [] }
}
```

Check out the Powerline docs for a more detailed view into [configuration and customization](https://powerline.readthedocs.io/en/latest/configuration.html).

The next step is to let Powerline know of the new segment by adding it to the segment's config file.
To add the new segment to the current shell prompt, add the following entry to the `<powerline_dir>/themes/shell/default.json`
config file.

```json
{
  "function": "k8s",
  "priority": 30
}
```

## Installation

Once all configuration has been done it's time to "install" the segment and try it out, you can do so by executing:

```python
python3 -m pip install --editable .
```

Installing the package in editable mode saves you from having to "re-install" to see the latest changes.
If everything went well you should be able to see your new segment.

![powerline-k8s](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1623588713/foss/powerline-k8s_uc0cxj.png)

## Troubleshooting

If you are having issues with your plugin, try out the following:

- Run `powerline-lint` to check for errors in the configuration files i.e.  colorscheme and/or segment config.

- Restart powerline by running `powerline-daemon --replace`.

- Configure logging with [PowerlineLogger](https://powerline.readthedocs.io/en/master/develop/segments.html#powerlinelogger-class) and [where you want the logs to be written](https://powerline.readthedocs.io/en/master/configuration/reference.html#config-common-log) for easier debugging.

It is also worth checking out the [Powerline docs](https://powerline.readthedocs.io/en/master/troubleshooting.html) for other common issues that you may encounter.

## Misc

- Check out the full source on GitHub for the [powerline-k8s](https://github.com/j4ckofalltrades/powerline-k8s) plugin.
- If you want to share your custom Powerline segment, you might want to check out this guide about [publishing packages to PyPI](/posts/pypi-publish/).
