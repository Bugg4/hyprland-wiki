---
weight: 12
title: Systemd startup
---

## UWSM

- [Universal Wayland Session Manager](https://github.com/Vladimir-csp/uwsm) wraps standalone Wayland compositors into a set of Systemd units and provides robust session management including environment, XDG autostart support, bi-directional binding with login session, and clean shutdown.

Please note uwsm is for advanced users and has its issues and additional quirks.

## Installation

{{% details title="Arch" closed="true" %}}

uwsm is available in Arch official repositories.

```sh
pacman -S uwsm libnewt
```

{{% /details %}}

{{% details title="Nix/NixOS" closed="true" %}}

```nix
{
  programs.hyprland.withUWSM  = true;
}
```

The above option generates a new desktop entry, `hyprland-uwsm.desktop`, which will be available in display managers.

For more info, read the [option](https://search.nixos.org/options?channel=unstable&show=programs.uwsm.enable&from=0&size=50&sort=relevance&type=packages&query=uwsm).

{{< callout >}}

If you use the [Home Manager module](../../Nix/Hyprland-on-Home-Manager), make sure to disable the systemd integration, as it conflicts with uwsm.

```nix
{
  wayland.windowManager.hyprland.systemd.enable = false;
}
```

{{< /callout >}}

{{% /details %}}

{{< callout type=info >}}

For instructions for other distros and manual building, see [building and installing](https://github.com/Vladimir-csp/uwsm?tab=readme-ov-file#installation) section on the project's page.

{{< /callout >}}

## Launching Hyprland with uwsm

{{< callout type=info >}}

Pay attention to the warnings in [Environment variables](../../Configuring/Environment-variables), [Multi-GPU](../../Configuring/Multi-GPU) and [Dispatchers](../../Configuring/Dispatchers) sections.

{{< /callout >}}

### In tty

To launch Hyprland with uwsm, add this code in your shell profile.

```
if uwsm check may-start && uwsm select; then
	exec uwsm start default
fi
```

This will bring uwsm compositor selection menu after you log in tty1. Choose `Hyprland` entry and you're good to go.

If you want to bypass compositor selection menu and launch Hyprland directly, use this code in your shell profile, instead.

```
if uwsm check may-start; then
    exec uwsm start hyprland-uwsm.desktop
fi
```

### Using a display manager

If you use a display manager, choose `hyprland (uwsm-managed)` entry in a display manager selection menu.

## Launching applications inside session

The concept of a session managed by Systemd implies also running applications as units. Uwsm [provides](https://github.com/Vladimir-csp/uwsm#3-applications-and-slices) a helper to do it. Running applications as child processes inside compositor's unit is discouraged.

Prefix application startup commands with `uwsm app --`. It also supports launching Desktop Entries by IDs or paths. See `man uwsm` or `uwsm app --help`.

Examples for autostart and bind entries:

```
exec-once = uwsm app -- mycommand --arg1 --arg2
bind = SUPER, E, exec, uwsm app -- pcmanfm-qt.desktop
```

## Autostart

XDG Autostart is handled by systemd, and its target is activated in uwsm-managed session automatically.

Some applications provide native systemd user units to be autostarted with. Those might need to be enabled explicitly via `systemctl --user enable [some-app.service]`. Or, in case the unit is missing `[Install]` section, enabled more directly: `systemctl --user add-wants graphical-session.target [some-app.service]`. Also ensure the unit has `After=graphical-session.target` ordering (it can be added via drop-in).

More autostart-related examples and tricks can be found [here](https://github.com/Vladimir-csp/uwsm/tree/master/example-units).
