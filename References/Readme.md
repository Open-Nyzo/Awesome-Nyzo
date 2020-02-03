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
