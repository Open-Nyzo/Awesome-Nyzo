# Various Reference docs, snipets and links

To be organized better

## Download all consolidated blockFiles from nyzo.co
```
import requests
from time import sleep
import os.path
from os import path

start = 0
done = 0
end = 999
end_incr=1000

def conv(n):
    return "{:06d}".format(n)

def convdir(n):
    return "{:03d}".format(n)

def dlfile(u,s):
    try:
        with requests.get(u,stream=True) as r:
            r.raise_for_status()
            with open(s,'wb') as f:
                for chunk in r.iter_content(chunk_size=8192):
                    f.write(chunk)
            print('done {}'.format(s))
    except Exception as e:
        print(e)
        print('failed to download\n{}\n{}'.format(u,s))
        print('retrying dl after 5s sleep')
        sleep(5)
        dlfile(u,s)

for i in range(1,100000):
    this_start = start+((i-1)*end_incr)
    this_end = end+((i-1)*end_incr)
    this_dir = '/var/lib/nyzo/production/blocks/'+convdir(i)
    for k in range(this_start,this_end+1):
        blockno=conv(k)
        this_block = blockno+'.nyzoblock'

        if path.exists(this_dir+'/'+this_block):
            print('already exists: {}'.format(this_block))
            continue

        dlfile('https://blocks.nyzo.co/blockFiles/'+this_block, this_dir+'/'+this_block)
 ```
 
## Seed transactions

### Official link

'https://nyzo-transactions.nyc3.digitaloceanspaces.com/' + file_name

### Mega.nz hosted backup by Snipe

https://mega.nz/#!eiozTI6T!ntHjptSdFl7GadgGhYVBt015iB2ki9BxnKYbxGEoeHo

### Fetch script by Snipe

```
import requests
from time import sleep

file = 1

def save_to_dir(res, file_name):
    with open('seed_files/'+file_name, 'wb') as f:
        f.write(res)

while True:
    file += 1
    file_name = '{:>06}.nyzotransaction'.format(file)
    res = requests.get('https://nyzo-transactions.nyc3.digitaloceanspaces.com/' + file_name)
    if res.status_code == 200:
        print('success')
        save_to_dir(res.content, file_name)
    else:
        print('error')
        sleep(2)

    print(file_name)

    sleep(0.01)
```


### Emission from seed accounts by Snipe

Emission, historical, daily interval of 12342 blocks

```
import requests
from time import sleep

block = 2
end_block = 6344524
tx_fee = 0.25
prev_rew = 598.997856
block_interval = 12342
block_dict = {}

def wtf():
    with open('seed_res', 'w') as f:
        f.write(str(block_dict))

while True:
    if block+block_interval > end_block:
        # wtf()
        break

    url = 'https://nyzo.co/blockPlain/{}'.format(block)
    block += block_interval

    res = requests.get(url)
    if res.status_code == 200:
        print('success')
        res = res.content.decode('utf-8')
        r_pos = res.find('</p><p>amount: ')
        rel_pc = res[r_pos:r_pos+50].split('∩')[1].split('<')
        cur_rew = float(rel_pc[0])
        diff_cur_prev = '{0:.10f}'.format(100 * float(prev_rew-cur_rew)/float(cur_rew))

        prev_rew = cur_rew

        print(cur_rew)
        print('Days checked:', int(block/block_interval))

        block_dict[block] = {'seed_tx_amt': cur_rew, '0.25%': cur_rew * 0.0025}
        print(block_dict)

        wtf()
    else:
        print('error')
        sleep(2)

    sleep(0.01)
```

### Cycle events gathering

```
import requests

res = requests.get('https://nyzo.co/cycleEvents').content.decode('utf-8')
loc = res.find('<p class="event">')

res_list = res.split('<p class="event">')
events = {}
c=0

def wtf():
    with open('cycle_events','w') as f:
        f.write(str(events))

for i in res_list:
    c+=1
    if c > 1:
        if 'joined at' in i:
            struct_type = 'join'
        elif 'left at' in i:
            struct_type = 'leave'
        else:
            struct_type = 'invalid'

        struct_identifier = i.split('<a href="/status?id=')[1].split('"')[0]
        struct_block_height = int(i.split('at block ')[1].split(',')[0])
        struct_cycle_size = i.split('cycle length: ')[1].split('<')[0]

        events[struct_block_height] = {'type': struct_type, 'identifier': struct_identifier, 'cycle_size': struct_cycle_size}

print(len(events), 'cycle events have been written to cycle_events')
wtf()
```

### Reward plotting

For this script to work you will need to use the two scripts above (seed_res and cycle_events required)

```
import matplotlib.pyplot as plt
import ast

blocks_per_day = 12342

reference_url = 'https://github.com/Open-Nyzo/Awesome-Nyzo/blob/master/References/Readme.md'

display_determinant = input('Please choose the plotting you would like to see\n[0]: Seed tx amount (x: block, y: tx amt)\n[1]: Block reward (x: block, y: block reward)\n[2]: Reward per day (x: block, y: blocks_per_day / cycle size * block_reward)\n[3]: In-cycle verifiers (x: block, y: in-cycle verifiers)\nResponse: ')
try:
    display_determinant=int(display_determinant)
except:
    display_determinant=4

if display_determinant == 0:
    print('You have selected SEED TX AMOUNT')
elif display_determinant == 1:
    print('You have selected BLOCK REWARD')
elif display_determinant == 2:
    print('You have selected REWARD PER DAY')
elif display_determinant == 3:
    print('You have selected IN-CYCLE VERIFIERS')
else:
    print(display_determinant)
    print('Wrong input, try again.')
    exit()

start_block = input('############################\nPlease enter a start block (default=0):')
try:
    start_block = int(start_block)
except:
    print('Using default start block (0)')
    start_block = 0

with open('seed_res','r') as f:
    seed_dict = ast.literal_eval(f.readlines()[0])
    if len(seed_dict)< 10: exit('Please fetch the seed_res file first\n'+reference_url)

with open('cycle_events', 'r') as f:
    cycle_events_dict = ast.literal_eval(f.readlines()[0])
    if len(cycle_events_dict)<10: exit('Please fetch the cycle_events file first\n'+reference_url)

x_l = []
y_l = []
y2_l = []
y3_l = []
y4_l = []

omitted_blocks = [1295912, 2233904, 4726988]
seed_blocks = []
cycle_event_blocks = []

for block in seed_dict:
    seed_blocks.append(block)

for cycle_event in cycle_events_dict:
    cycle_event_blocks.append(cycle_event)

for block in seed_dict:
    if block < start_block:
        continue

    if block in omitted_blocks:
        continue

    x_l.append(block)
    y_l.append(seed_dict[block]['seed_tx_amt'])
    y2_l.append(seed_dict[block]['0.25%'])

    nearest_cycle_event_block = min(cycle_event_blocks, key=lambda x: abs(x - block))
    relevant_cycle_event = cycle_events_dict[nearest_cycle_event_block]

    y3_l.append(blocks_per_day / int(relevant_cycle_event['cycle_size']) * seed_dict[block]['0.25%'])
    y4_l.append(int(relevant_cycle_event['cycle_size']))

plt.xlabel('Block height')

if display_determinant == 0:
    plt.ylabel('Seed tx amount')
    plt.plot(x_l, y_l)
elif display_determinant == 1:
    plt.ylabel('Block reward')
    plt.plot(x_l, y2_l)
elif display_determinant == 2:
    plt.ylabel('Reward per 24h, per verifier')
    plt.plot(x_l, y3_l)
else:
    plt.ylabel('Verifiers in-cycle')
    plt.plot(x_l, y4_l)


plt.show()
```
