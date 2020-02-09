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
