#!/usr/bin/python3
import easysnmp
import time
import sys
import math
from easysnmp import Session
from easysnmp.exceptions import EasySNMPTimeoutError

agent_input = sys.argv
sample_time = 1 / float(agent_input[2])
ip, port, community = agent_input[1].split(':')
sample = int(sys.argv[3])
sample_freq = float(agent_input[2])

oids = []
init_oid = []
next_oid = []
unixclock = ""

for x in range(4, len(agent_input)):
    oids.append(sys.argv[x])
oids.insert(0, '1.3.6.1.2.1.1.3.0')

sesh = Session(hostname=ip, remote_port=port, community='public', version=2, timeout=1, retries=3)

def rate_calculator():
    global init_oid, init_time
    while True:
        try:
            retort = sesh.get(oids)
            break
        except EasySNMPTimeoutError:
            #print("Connection timed out. Retrying in 5 seconds...")
            time.sleep(5)
    next_oid = []

    for y in range(1, len(retort)):
        if retort[y].value != 'NOSUCHINSTANCE' and retort[y].value != 'NOSUCHOBJECT':
            if retort[y].snmp_type == 'COUNTER64' or retort[y].snmp_type == 'COUNTER32' or retort[y].snmp_type == 'COUNTER' or retort[y].snmp_type == 'GAUGE':
                next_oid.append(int(retort[y].value))
            else:
                next_oid.append(retort[y].value)
            if sample != 0 and len(init_oid) > 0:
                if retort[y].snmp_type in ['COUNTER32', 'COUNTER64','COUNTER']:
                    numerator = int(next_oid[y - 1]) - int(init_oid[y - 1])
                    denominator = round(next_time - init_time, 1)
                    rate = (numerator / denominator)
                    
                    if rate < 0:
                        if retort[y].snmp_type == 'COUNTER64':
                            numerator = numerator + 2 ** 64
                        elif retort[y].snmp_type == 'COUNTER32' or retort[y].snmp_type == 'COUNTER':
                            numerator = numerator + 2 ** 32
                    try:
                        if unixclock == str(next_time):
                            print(str(round(numerator / denominator)), end="|")
                        else:
                            print(str(next_time) + "|" + str(round(numerator / denominator)), end="|")
                            unixclock = str(next_time)
                    except:
                        print(str(next_time) + "|" + str(round(numerator / denominator)), end="|")
                        unixclock = str(next_time)

                elif retort[y].snmp_type == 'GAUGE':
                    numerator = int(next_oid[y - 1]) - int(init_oid[y - 1])
                    try:
                        if unixclock == str(next_time):
                            print(next_oid[len(next_oid) - 1], "(" + str(numerator) , ")", end="|")
                        else:
                            print(str(next_time) + "|" + str(next_oid[len(next_oid) - 1]) + "(" + str(numerator) + ")", end="|")
                            unixclock = str(next_time)
                    except:
                        print(str(next_time) + "|" + str(next_oid[len(next_oid) - 1]) + "(" + str(numerator) + ")", end="|")
                        unixclock = str(next_time)

                elif retort[y].snmp_type == 'OCTETSTR':
                    try:
                        if unixclock == str(next_time):
                            print(str(next_oid[len(next_oid) - 1]),end= "|")
                        else:
                            print(next_time, "|", str(next_oid[len(next_oid) - 1]), end="|")
                            unixclock = str(next_time)
                    except:
                        print(str(next_time) + "|" + str(next_oid[len(next_oid) - 1]),end="|")
                        unixclock = str(next_time)
    init_oid[:] = next_oid    
    init_time = next_time

if sample == -1:
    count = 0
    init_oid = []
    while True:
        next_time = time.time()
        rate_calculator()
        if count != 0:
            print(end="\n")
        retort_time = time.time()
        count = count + 1
        if  sample_time >= retort_time - next_time:
            time.sleep(abs(sample_time - retort_time + next_time))
        else:
            m = math.ceil((retort_time - next_time) / sample_time)
            time.sleep(((m * sample_time) - retort_time + next_time))
else:
    init_oid = []
    for count in range(0, sample + 1):
        next_time = time.time()
        rate_calculator()
        if count != 0:
            print(end="\n")
        retort_time = time.time()
        if  sample_time >= retort_time - next_time:
            time.sleep(abs(sample_time - retort_time + next_time))
        else:
            m = math.ceil((retort_time - next_time) / sample_time)
            time.sleep(((m * sample_time) - retort_time + next_time))
