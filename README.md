# android_vendor_xiaomi_gold

Proprietary files for building LineageOS 24.0 for the Redmi Note 13 5G (`gold`). They are checked out at `vendor/xiaomi/gold`.

| | |
| --- | --- |
| Device tree | [android_device_xiaomi_gold](https://github.com/Star-caorui/android_device_xiaomi_gold) |
| How to build | [local_manifests](https://github.com/Star-caorui/local_manifests) |

## Source

Every file comes from Xiaomi's full OTA for the global model:

```
gold_global-ota_full-OS3.0.5.0.VNQMIXM-user-15.0-df9ff3aa93.zip
SHA-256 35c9f1d98b28538ac10c319ad4cba993cd5a632960d9111162e3efa494dae5c9
```

| Here | In the package |
| --- | --- |
| `proprietary/vendor/` | the `vendor` partition |
| `proprietary/odm/` | `odm/` inside the `vendor` partition |
| `proprietary/system_ext/` | the `system_ext` partition |
| `proprietary/product/` | the `product` partition |
| `radio/<name>.img` | the partition `<name>` |

## History

| Commit | What it does |
| --- | --- |
| The first | Imports the files exactly as the package has them, at the same paths |
| Each one after it | Makes one change to some of them, or adds or regenerates the build files that `extract-files.py` writes. Its message says what was done and with which command |
| The last | Adds this README |

So `git log` is the complete list of differences from stock.

## Verifying

### The first commit

You need `unzip`, `fsck.erofs` (erofs-utils; 1.7.1 was used) and `ota_extractor`, which is in `prebuilts/extract-tools/linux-x86/bin/` of a LineageOS tree. Run this from the top of a clone of this repository:

```sh
git checkout $(git rev-list --max-parents=0 HEAD)

mkdir -p work/img
unzip -o gold_global-ota_full-OS3.0.5.0.VNQMIXM-user-15.0-df9ff3aa93.zip payload.bin -d work
ota_extractor --payload work/payload.bin --output_dir work/img \
  --partitions vendor,system_ext,product,dpm,gz,lk,mcupm,md1img,pi_img,scp,spmfw,sspm,tee
fsck.erofs --extract=work/vendor work/img/vendor.img
fsck.erofs --extract=work/system_ext work/img/system_ext.img
fsck.erofs --extract=work/product work/img/product.img

check() { (cd "$1" && find . -type f) | while read -r f; do
    cmp -s "$1/$f" "$2/$f" || echo "DIFFERENT: $1/$f"; done; }
check proprietary/vendor work/vendor
check proprietary/odm work/vendor/odm
check proprietary/system_ext work/system_ext
check proprietary/product work/product
check radio work/img
```

The five `check` lines print nothing when every file matches. A file that does not is reported as `DIFFERENT: <path>`.

Seven of the paths are symbolic links in the package: `libhdrvideo.so`, `libmml.so`, `libscltm.so`, `libpq_sec.so`, `libpqparamparser.so` and `libmmagent.so` under `lib64/`, and `lib/libpq_sec.so`. The repository has the file each one points to, and `cmp` follows the link.

### The later commits

Do what a commit message says to the files of its parent commit and you get the files of that commit, byte for byte. `patchelf` is version 0.18, also in `prebuilts/extract-tools/linux-x86/bin/`.

The two patched applications are the exception. They are rebuilt with apktool from the patches in the device tree's `blob-patches/`.

## Regenerating

From the top of a LineageOS tree with the device tree checked out:

```sh
cd device/xiaomi/gold
./extract-files.py /path/to/gold_global-ota_full-OS3.0.5.0.VNQMIXM-user-15.0-df9ff3aa93.zip
cd -
```

- `extract-files.py` is the usual LineageOS extraction script. It copies the listed files out of the package and applies the changes recorded here.
- It needs `fsck.erofs` (erofs-utils) on the host. The other tools come with the LineageOS tree.
- It writes `vendor/xiaomi/gold`: the files under `proprietary/` and `radio/`, and the four build files.

## Credits

- [Dhterech](https://github.com/Dhterech/android_device_xiaomi_gold), whose device tree is built on claxten10's [mt6833-devs tree](https://github.com/mt6833-devs/android_device_xiaomi_gold): the list of files, and the first twelve changes after the import, from "Put the per-chip libraries where they are loaded from" to "Clear symbol versions that two blobs ask for and the build cannot give". This repository only writes them down as commits; a few were adjusted later.
- mt6897-devs' device_xiaomi_duchamp and LineageOS's android_device_xiaomi_rosemary: Dhterech's tree names the first as its source for the KeyMint and camera interface changes, and the second for clearing symbol versions.
- The later changes were made in [our fork](https://github.com/Star-caorui/android_device_xiaomi_gold) of Dhterech's tree. Where one of them follows another device tree, the commit message here or in the device tree names it.
