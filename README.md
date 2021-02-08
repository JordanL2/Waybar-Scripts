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


