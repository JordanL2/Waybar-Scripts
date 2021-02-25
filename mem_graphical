#!/usr/bin/python3

import json
import math
import os.path
import re
import subprocess
import sys
import yaml


scriptpath = sys.argv.pop(0)

chars = ' ▁▂▃▄▅▆▇█'

states = {}
config_file = os.path.abspath(os.path.dirname(scriptpath)) + "/cpu_graphical.yml"
if os.path.exists(config_file):
    with open(config_file, 'r') as fh:
        config = yaml.load(fh, Loader=yaml.CLoader)
        for state in config['states']:
            states[config['states'][state]['threshold']] = {
                'status': state,
                'background': config['states'][state]['background'],
                'foreground': config['states'][state]['foreground'],
            }
        cache_mix = config['memory']['cache_mix']
else:
    raise Exception("No such config file: {}".format(config_file))

state_keys = sorted(list(states.keys()), reverse=True)



def show_bar(perc):
    bar = math.ceil(perc / (100 / len(chars))) - 1
    if bar == -1:
        bar = 0
    if bar >= len(chars):
        bar = len(chars) - 1
    return chars[bar]

def call(command):
    result = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    return result.stdout.decode('utf-8').rstrip("\n")

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


result = call('free -wb | head -n2 | tail -n+2')
line_regex = re.compile('^Mem:\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s*$')
line_match = line_regex.match(result)
if line_match:
    total =     int(line_match.group(1))
    used =      int(line_match.group(2))
    free =      int(line_match.group(3))
    shared =    int(line_match.group(4))
    buffers =   int(line_match.group(5))
    cache =     int(line_match.group(6))
    available = int(line_match.group(7))

    real_total = used + available
    using_nocache = used + shared
    using_cache = used + shared + buffers + cache

    using_nocache_perc = using_nocache / real_total * 100
    using_cache_perc = using_cache / real_total * 100

    status = None
    for status_level in state_keys:
        if using_nocache_perc >= status_level:
            status = states[status_level]
            break

    cache_colour = colour_mix(status['background'], status['foreground'], cache_mix)

    bars = "<span foreground='{0}'>{1}</span><span foreground='{2}'>{3}</span>".format(
        cache_colour, show_bar(using_cache_perc), 
        status['foreground'], show_bar(using_nocache_perc))

    tooltips =  "    Total: {} MiB\n".format(int(total / 1024**2))
    tooltips += "Available: {} MiB\n".format(int(available / 1024**2))
    tooltips += "     Free: {} MiB\n".format(int(free / 1024**2))
    tooltips += "     Used: {} MiB\n".format(int(used / 1024**2))
    tooltips += "   Shared: {} MiB\n".format(int(shared / 1024**2))
    tooltips += "  Buffers: {} MiB\n".format(int(buffers / 1024**2))
    tooltips += "    Cache: {} MiB".format(int(cache / 1024**2))

    print(json.dumps({
        'text': bars,
        'class': status['status'],
        'percentage': int(using_nocache_perc),
        'tooltip': tooltips
        }))

else:
    print("ERROR didn't match regex: '{}'".format(result))
