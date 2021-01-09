# Stage 0

This stage is located in the BootROM. Among others, it loads and executes stage 1 from NOR.

# Stage 1 (iBoot1)

This stage is the primary early loader, located in the on-board NOR. This boot stage very roughly goes as follows, given a target partition to boot from:

* Get the volume group UUID, so far seen as the UUID of the target partition Data subvolume;
* Get the local policy hash:
  - First try the local proposed hash (SEP command 11);
  - If that is not available, get the local blessed hash (SEP command 14)
* Read the local boot policy, located on the iSCPreboot partition at `/<volume-uuid>/LocalPolicy/<policy-hash>.img4`. This boot policy has the following specific metadata keys:
  - `vuid`: UUID: Volume group UUID - same as above
  - `kuid`: UUID: KEK group UUID
  - `lpnh`: SHA384: Local policy nonce hash
  - `rpnh`: SHA384: Remote policy nonce hash
  - `nsih`: SHA384: Next-stage IMG4 hash
  - `prot`: SHA384
  - `lobo`: bool
  - `smb0`: bool: Permissive security enabled, bit 0
  - `smb1`: bool: Permissive security enabled, bit 1
  - `smb2`: bool: Third-party kernel extensions enabled
  - `smb3`: bool: Manual mobile device management (MDM) enrollment
  - `smb4`: bool?: MDM device enrollment program disabled
  - `sip0`: u16: SIP customized
  - `sip1`: bool: Signed system volume (`csrutil authenticated-boot`) disabled
  - `sip2`: bool: CTRR ([configurable text region read-only](https://keith.github.io/xcode-man-pages/bputil.1.html)) disabled
  - `sip3`: bool: `boot-args` filtering disabled
  - `auxp`: SHA384: User-authorized kernel extensions hash
  - `auxi`: SHA384: User-authorized kernel cache IMG4 hash
  - `auxr`: SHA384: Kernel extension recept hash
  - `coih`: SHA384: kcOS (custom kernelcache) or fuOS (custom OS) IMG4 hash

  And optionally the following linked manifests, each located at `/<volume-uuid>/LocalPolicy/<policy-hash>.<id>.im4m`
  - `auxk`: kcOS manifest
  - `fuos`: fuOS manifest

* If loading the next stage:

  - The boot directory is located at the target partition Preboot subvolume, at path `/<volume-uuid>/boot/<local-policy.metadata.nsih>`;
  - Decrypt, verify and execute `<boot-dir>/usr/standalone/firmware/iBoot.img4` with the device tree and other firmware files in the same directory. No evidence towards other metadata descriptors yet.

* If loading a custom stage (kcOS/fuOS):

  - ...

# Stage 2 (iBoot2)

This stage is the OS-level loader, located inside the OS partition and shipped as part of macOS. It loads the rest of the system.