# OP-TEE + fTPM using QEMU.
This git containst repo manifests to be able to clone all source code needed to
be able to setup a full OP-TEE developer build which also enables Microsofts
fTPM implementation.

Note that we this setup contains a couple of different forks (fTPM, Linux
kernel, ibmtpm20tss). Likewise some steps will be done after booting up the
system. 

### Get the source
Pre-condition, install the pre-reqs mentioned at the official OP-TEE docs:
https://optee.readthedocs.io/building/prerequisites.html

When done, continue here.

```bash
$ mkdir optee-ftpm && cd optee-ftpm
$ repo init -u https://github.com/jbech-linaro/manifest.git -b ftpm -m qemu-ftpm.xml 
$ repo sync -j4 --no-clone-bundle
```

Next we need to get WolfSSL
```bash
$ cd <optee-ftpm>/ms-tpm-20-ref
$ git submodule init
$ git submodule update
```

### Compile
The initial build takes quite some time, have patience!
```bash
$ cd <optee-ftpm>/build
$ make toolchains -j2
$ make -j`nproc`
```

### Run
```bash
$ cd <optee-ftpm>/build
$ make -j`nproc` run
```

#### Shell 1 - QEMU:
```
(qemu) c
```

#### Shell 2 - QEMU: NW - 54320
Here we will login, run un alias `ftpm` which will

- Mount the host folder
- Symlink the fTPM TA into `/lib/optee_armtz`
- Copy the ibmtpm20tss `libibmtss*` libraries to `/lib`
- And then go into the ibmtpm20tss folder and run `getrandom`.

The command and the output should look like below. If you see that, then you
can be sure that you have a working setup! (Note, it's a little bit slow here,
so give it a few seconds).

```bash
buildroot login: root
# ftpm
fTPM: before open context
fTPM: after open context
random: fast init done
urandom_read: 2 callbacks suppressed
random: getrandom: uninitialized urandom read (32 bytes read)
 randomBytes length 8
 81 a1 06 bb 4c e7 cc 7a 
```

#### Re-run without re-compile
```bash
$ cd <optee-ftpm>/build
$ make run-only
```

### Debugging with GDB
In `Shell 3 - QEMU: SW - 54321` look for the load address after you've loaded
the module.
```bash
D/TC:? 0 load_elf_from_store:801 ELF load address 0x113000
```
That + `0x20` in offset is the address where you load the symbols, it could look like this in GDB.

```bash
add-symbol-file <optee-ftpm>/ms-tpm-20-ref/out/bc50d971-d4c9-42c4-82cb-123456789123.elf 0x113020
```

After that you can try setting some breakpoints, there are a couple of good candidates:
```bash
b TPM2_GetRandom
b fTPM_Submit_Command
b TA_InvokeCommandEntryPoint
```

Note that address changes, to it might be that you have to stop on another
break point in `optee_os`, `tee.elf` first (after seeing the load address) and
first thereafter add symbols and breakpoints as described above.

# Official OP-TEE documentation
All other official OP-TEE documentation has moved to
http://optee.readthedocs.io. The information that used to be here in this git
can be found under [manifest].

// OP-TEE core maintainers

[manifest]: https://optee.readthedocs.io/building/gits/manifest.html
