This is a complete disaster, but at least I learned some things about Android internals I guess?

Basic TWRP build for the Cosmo Communicator. This uses the kernel and the TrustKernel system files from the stock firmware to allow the userdata partition to be decrypted and accessed.

# Caveats

- Most features are untested (especially SD card and external storage)
- Doesn't seem to work with Planet's OTA zips
- Will require change to `BoardConfig.mk` if Planet moves Android past security patch version 2019-07-05
- Not compatible with SELinux enforcing
- Only works when compiled as eng and not as userdebug

# Compiling

The following patch is required to stop TWRP from trying to access a Keymaster2 device as Keymaster0 and segfaulting:

```
diff --git a/crypto/fde/cryptfs.cpp b/crypto/fde/cryptfs.cpp
index 8352296..cc0e039 100644
--- a/crypto/fde/cryptfs.cpp
+++ b/crypto/fde/cryptfs.cpp
@@ -424,6 +424,7 @@ static int keymaster_sign_object(struct crypt_mnt_ftr *ftr,
 #if TW_KEYMASTER_MAX_API >= 1
     keymaster0_device_t *keymaster0_dev = 0;
     keymaster1_device_t *keymaster1_dev = 0;
+    goto initfail;
     if (keymaster_init(&keymaster0_dev, &keymaster1_dev)) {
 #else
     keymaster_device_t *keymaster0_dev = 0;
```
