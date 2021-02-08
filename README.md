# Waybar-Scripts

## cpu_graphical

Shows CPU usage per-core as a text bar-chart.

### Usage

`cpu_graphical [INTERVAL] [SAMPLES] [SHADING]`

INTERVAL: Number of seconds between each measurement. Default is 1. Non-integers are allowed, eg 0.1 for 10 times a second.

 SAMPLES: Number of past measurements to average. Default is 1.

 SHADING: Whether or not ['on' or 'off'] to shade each bar according to its value. If this is turned on, you'll want to change the background and foreground colours in the `states` map inside the script.


## mem_graphical

Shows a bar indicating total memory usage, and another that excludes cache and buffers.


## status

A script that runs various user-definable status checking scripts, and displays any messages found to the taskbar.

### Usage

In the same directory as the `status` script, there must be a directory named `status.d`. Inside this directory there may be executable scripts that print out any messages that should be displayed to the user. These scripts need to have the executable flag set, and the correct she-bang on the top line (or may even be binary executables).

The output of messages must be in the format:

`type|label|message`

Where:

`type` is either `info` or `error`.

`label` is a user-defined category or source. If there are multiple messages from the same source, just the number of messages for that source will be shown on the taskbar. All messages will be shown in the tooltip text.

`message` is the text to display, this may have `|` characters in it, but must all be on the same line.


## waybar_wait_for_network

A script that launches Waybar only once the network is definitely available.
