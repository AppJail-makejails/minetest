# Minetest

Minetest is an open source voxel game-creation platform with easy modding and game creation.

wikipedia.org/wiki/Minetest

![minetest logo](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Minetest_logo.svg/512px-Minetest_logo.svg.png)

## How to use this Makejail

### Standalone

```sh
appjail makejail \
    -j minetest \
    -f gh+AppJail-makejails/minetest \
    -o virtualnet=":<random> default" \
    -o nat
```

### Deploy using appjail-director

**appjail-director.yml**:

```yaml
options:
  - virtualnet: ':<random> default'
  - nat:

services:
  minetest:
    name: minetest
    makejail: gh+AppJail-makejails/minetest
    volumes:
      - data: minetest-data
      - log: minetest-log

default_volume_type: '<volumefs>'

volumes:
  data:
    device: .volumes/data
  log:
    device: .volumes/log
```

**.env**:

```
DIRECTOR_PROJECT=minetest
```

### Customize

If you want to customize your minetest instance, it's very simple. For example, let's add a new mod:

1. Create the `mods/` directory:

```sh
mkdir -p .volumes/data/.minetest/mods
```

2. Download the mod (and its dependencies):

```sh
cd .volumes/data/.minetest/mods
fetch -o animalia.zip https://content.minetest.net/packages/ElCeejo/animalia/releases/23715/download/
fetch -o creature.zip https://content.minetest.net/packages/ElCeejo/creatura/releases/22754/download/
```

3. Unzip the mod (and its dependencies):

```sh
unzip animalia.zip
unzip creature.zip
```

4. Remove the ZIP files:

```sh
rm *.zip
```

5. Fix the owner and group:

```sh
cd -
chown -Rf 976:976 .volumes/data/.minetest/mods
```

6. Restart the `rc(8)` script:

```sh
appjail service jail minetest minetest restart
```

7. Change `load_mod_<mod> = false` to `load_mod_<mod> = true` for each mod you want to enable:

```sh
$EDITOR .volumes/data/world/world.mt
```

8. Restart the `rc(8)` script again:

```sh
appjail service jail minetest minetest restart
```

**Recommendation**: Visit the wiki for more details: https://wiki.minetest.net

### Configuration

To preserve the configuration settings we have made in our `minetest.conf` configuration file, we must copy it for each jail recreation. Here the step by step:

1. Create the `files/usr/local/etc` directory:

```sh
mkdir -p files/usr/local/etc
```

2. Copy the `minetest.conf` configuration file from the jail to the host:

```sh
appjail cmd local minetest \
    cp -a usr/local/etc/minetest.conf $PWD/files/usr/local/etc/minetest.conf
```

3. (optional): Edit the configuration file:

```sh
$EDITOR files/usr/local/etc/minetest.conf
```

4. Edit the Director file:

```yaml
options:
  - virtualnet: ':<random> default'
  - nat:
  - copydir: !ENV '${PWD}/files'

services:
  minetest:
    name: minetest
    makejail: gh+AppJail-makejails/minetest
    options:
      - file: /usr/local/etc/minetest.conf
    volumes:
      - data: minetest-data
      - log: minetest-log

default_volume_type: '<volumefs>'

volumes:
  data:
    device: .volumes/data
  log:
    device: .volumes/log
```

5. Recreate the project:

```
appjail-director up
```

### Volumes

| Name          | Owner | Group | Perm | Type | Mountpoint        |
| ------------- | ----- | ----- | ---- | ---- | ----------------- |
| minetest-data | 976   | 976   |  -   |  -   | /var/db/minetest  |
| minetest-log  | 976   | 976   |  -   |  -   | /var/log/minetest |

## Tags

| Tag    | Arch    | Version        | Type   |
| ------ | ------- | -------------- | ------ |
| `13.3` | `amd64` | `13.3-RELEASE` | `thin` |
| `14.0` | `amd64` | `14.0-RELEASE` | `thin` |
