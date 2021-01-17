#!/usr/bin/python3

import json
import math
import re
import statistics
import subprocess
import sys
import time


# ARGUMENTS

# 1. Interval - default 1
t = 1
if len(sys.argv) > 1:
    t = float(sys.argv[1])

# 2. Number of timings to measure and average over - default 1
max_times = 1
if len(sys.argv) > 2:
    max_times = int(sys.argv[2])

# 3. Shading mode - default off, 'on' to enable
colour_bars = 'off'
if len(sys.argv) > 3:
    colour_bars = sys.argv[3]


# Tooltips will update every this many seconds
tooltip_update_time = 1


# Appearance

chars = ' ▁▂▃▄▅▆▇█'

states = {
    0: {
        'status': '',
        'background': '#31363b',
        'foreground': '#ffffff',
    },
    70: {
        'status': 'warning',
        'background': '#442b11',
        'foreground': '#ffa500',
    },
    90: {
        'status': 'critical',
        'background': '#441111',
        'foreground': '#ff0000',
    },
}
state_keys = sorted(list(states.keys()), reverse=True)


def show_bar(perc, status):
    bar = math.ceil(perc / (100 / (len(chars) - 1)))
    char = chars[bar]

    if colour_bars == 'on':
        colour = colour_mix(status['background'], status['foreground'], perc / 100)
        return "<span foreground='{1}'>{0}</span>".format(char, colour)
    else:
        return char

def call(command):
    result = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    return result.stdout.decode('utf-8').rstrip("\n")

def cpu_times():
    with open('/proc/stat') as f:
        raw = f.read()
        line_regex = re.compile('^cpu\d+\s+(.+)')
        cpus = []
        for line in raw.split("\n"):
            line_match = line_regex.match(line)
            if line_match:
                fields = line_match.group(1).split(' ')
                user = float(fields[0])
                nice = float(fields[1])
                system = float(fields[2])
                idle = float(fields[3])
                iowait = float(fields[4])
                irq = float(fields[5])
                softirq = float(fields[6])
                steal = 0
                if len(fields) > 7:
                    steal = float(fields[7])

                cpus.append(user + nice + system + iowait + irq + softirq + steal)
            elif len(cpus) > 0:
                break
    return cpus

def colour_parse(colour):
    c1 = int(colour[1:3], base=16)
    c2 = int(colour[3:5], base=16)
    c3 = int(colour[5:7], base=16)
    return [c1, c2, c3]

def colour_mix(colour1, colour2, mix):
    colour1 = colour_parse(colour1)
    colour2 = colour_parse(colour2)
    new_colour = []
    for i, c1 in enumerate(colour1):
        c2 = colour2[i]
        new_colour.append(c1 * (1 - mix) + c2 * mix)
    return colour_hex(new_colour)

def colour_hex(colour):
    return '#' + ''.join(["{:02x}".format(int(c)) for c in colour])


clock_ticks_per_second = float(call('getconf CLK_TCK'))

cpu_timings = []
cpu_timings.append(cpu_times())
number_of_cpus = len(cpu_timings[0])
last_time = time.perf_counter()

tooltip = ''
tooltip_time = 0
tooltip_max_times = max(round(1 / t), 1)

while True:
    # Sleep the correct length of time
    new_time = time.perf_counter()
    delta_t = new_time - last_time
    time.sleep(max(t - delta_t, 0))
    new_time = time.perf_counter()
    delta_t = new_time - last_time
    last_time = new_time
    
    # Take a CPU reading
    cpu_timings.append(cpu_times())
    while len(cpu_timings) > max(max_times, tooltip_max_times) + 1:
        cpu_timings.pop(0)

    # Calculate average usage per core between first and last reading
    start_i = max(len(cpu_timings) - 1 - max_times, 0)
    cpus = [max(min(round(
                (cpu_timings[-1][i] - cpu_timings[start_i][i]) / clock_ticks_per_second * 100 / delta_t / (len(cpu_timings) - 1 - start_i)
               ), 100), 0)
            for i in range(0, number_of_cpus)]
    tooltip_start_i = max(len(cpu_timings) - 1 - tooltip_max_times, 0)
    tooltip_cpus = [max(min(round(
                (cpu_timings[-1][i] - cpu_timings[tooltip_start_i][i]) / clock_ticks_per_second * 100 / delta_t / (len(cpu_timings) - 1 - tooltip_start_i)
               ), 100), 0)
            for i in range(0, number_of_cpus)]

    # Average usage across all cores
    average = statistics.mean(cpus)

    # Status calculated from average usage
    status = None
    for status_level in state_keys:
        if average >= status_level:
            status = states[status_level]
            break

    # Make bar graphical output
    bars = ''.join([show_bar(c, status) for c in cpus])

    # Tooltip text - right aligned
    tooltip_time += delta_t
    if tooltip_time >= tooltip_update_time:
        tooltip_time = 0
        max_width = max([len("{}".format(c)) for c in tooltip_cpus])
        tooltip_format = '{:' + str(max_width) + '}%'
        tooltip = "\n".join([tooltip_format.format(c) for c in tooltip_cpus])

    # Output JSON row
    print(json.dumps({
        'text': bars,
        'class': status['status'],
        'percentage': int(average),
        'tooltip': tooltip
        }), flush=True)
