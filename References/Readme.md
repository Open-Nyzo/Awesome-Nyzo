# Various Reference docs, snipets and links

To be organized better

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

### Seed tx amount / Block reward plotting by Snipe

Run the fetch script above before using this.


```import matplotlib.pyplot as plt
import ast

display_determinant = input('Please choose the plotting you would like to see\n[0]: Seed tx amount\n[1]: Block reward\nResponse: ')
try:
    display_determinant=int(display_determinant)
except:
    display_determinant=2

if display_determinant == 0:
    print('You have selected SEED TX AMOUNT')
elif display_determinant == 1:
    print('You have selected BLOCK REWARD')
else:
    print(display_determinant)
    print('Wrong input, try again.')
    exit()

with open('seed_res','r') as f:
    seed_dict = ast.literal_eval(f.readlines()[0])

x_l = []
y_l = []
y2_l = []

omitted_blocks = [1295912, 2233904, 4726988]

for block in seed_dict:
    if block in omitted_blocks:
        continue

    x_l.append(block)
    y_l.append(seed_dict[block]['seed_tx_amt'])
    y2_l.append(seed_dict[block]['0.25%'])

plt.xlabel('Block height')

if display_determinant == 0:
    plt.ylabel('Seed tx amount')
    plt.plot(x_l, y_l)
else:
    plt.ylabel('Block reward')
    plt.plot(x_l, y2_l)

plt.show()
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