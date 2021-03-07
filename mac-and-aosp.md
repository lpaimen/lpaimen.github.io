# Working with AOSP on Mac

This page contains notes of building Android Open Source Project (AOSP) images on Macbook.

## Hardware

 - Macbook Pro. 8 cores, 16GB RAM, 1TB disk. This compiles AOSP from scratch in about 4 hours.
   - You want all cores you can get
   - 16GB RAM is minimum for compiling AOSP. I have had no issues with 16GB, but if I would choose now, I would go with 32.
   - Each Android environment is about 200-250GB so you can have three with 1TB disk. If you know you never need more than one environment, 500GB disk is enough.
 - USB hub. Flashing just works better with USB hub. Prefer powered and USB 3.0. Every time I've encountered odd read/write errors while flashing the phone, adding the hub has fixed it.
 - Phone. Not your daily use phone.

## Tips and tricks

**`kernel_task` burning your CPU?**

[Doc: TLDR; Charge from right side port](https://apple.stackexchange.com/questions/363337/how-to-find-cause-of-high-kernel-task-cpu-usage/363933#363933)


**File descriptor limit**

[Doc: Setting a file descriptor limit](https://source.android.com/setup/build/initializing#setting-a-file-descriptor-limit)

The recommended limit of 1024 open files is not enough with 8 cores. Use 2048.

    ulimit -S -n 2048

**Sparse image**

[Doc: Creating the case-sensitive disk image](https://source.android.com/setup/build/initializing#creating-a-case-sensitive-disk-image)

AOSP instructs to create sparse case-sensitive HFS+ image. But, when you remove lots of files from the image (`rm -rf environment` or `make clobber`), HFS is not so goot re-using the free'd image blocks. Unmount and compact the image (at the end of the day):

    umountAndroid
    hdiutil compact android.dmg.sparseimage
    mountAndroid

**Have external SSD? Use separate output directory**

[Doc: Separate output directory](https://source.android.com/setup/build/initializing#using-a-separate-output-directory)

If you have external SSD disk at hand, such as Samsung T7, use that as build output space. This way, I/O will be split between two drives - internal SSD will be doing the reads, external SSD will store the results.

Using `OUT_DIR_COMMON_BASE` _saves an hour_ of from-scratch compile time (3.5h -> 2.5h from-scratch).

    export OUT_DIR_COMMON_BASE=/Volumes/SamsungT7
    m

Remember to initialize the disk to case-sensitive format (HDF+ or AFS). Disk Utility helps on that. The external disk does not need to be sparse.