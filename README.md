# Waybar-Scripts

A collection of scripts to be used with [Waybar](https://github.com/Alexays/Waybar).

## cpu_graphical

Shows CPU usage per-core as a text bar-chart.

### Usage

`cpu_graphical [INTERVAL] [SAMPLES] [SHADING]`

`INTERVAL`: Number of seconds between each measurement. Default is 1. Non-integers are allowed, eg 0.1 for 10 times a second.

`SAMPLES`: Number of past measurements to average. Default is 1.

`SHADING`: Whether or not ['on' or 'off'] to shade each bar according to its value. Default is off. If this is turned on, you'll want to change the background and foreground colours in the `cpu_graphical.yml` config file.


## mem_graphical

Shows a bar indicating total memory usage (on the left), and another that excludes cache and buffers (on the right).

This also uses the `cpu_graphical.yml` config file to get the colours and states, as well as the brightness level of the bar that includes cache compared to the bar that excludes it.


## status

A script that runs various user-definable status checking scripts, and displays any messages found to the taskbar.

### Usage

In the same directory as the `status` script, there must be a directory named `status.d`. Inside this directory there may be executable scripts that print out any messages that should be displayed to the user. These scripts need to have the executable flag set, and the correct she-bang on the top line (or may even be binary executables).

The output of messages must be in the format:

`level|label|message`

Where:

`level` is either `info` or `error`.

`label` is a user-defined category or source. If there are multiple messages from the same source, just the number of messages for that source will be shown on the taskbar. All messages will be shown in the tooltip text.

`message` is the text to display, this may have `|` characters in it, but must all be on the same line.

### Configuration

`status.yml` contains a list of message levels, and the background and foreground colours for them. Additional levels may be added to this file. The order of the levels determines the order of precedence.

### Status Scripts

#### status.d/internet

Simple script to check internet connectivity. Pings Google's DNS servers.

#### status.d/xsettingsd

Check that the xsettingsd service is running.

#### youtube/recent_subs

Checks if there are any new videos from any of the channels you're subscribed to.


## waybar_wait_for_network

A script that launches Waybar only once the network is definitely available.

### Usage

Run with the name of the ethernet adapter as the first and only argument, eg:

`waybar_wait_for_network eno1`
