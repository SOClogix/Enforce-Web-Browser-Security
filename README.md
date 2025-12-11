# Enforce Web Browser Security

## Notice
#### Opera Browser Management (Opera, Opera GX, and Opera Air)
This will come at a later date when more time for research and testing is allocated.
Opera does not provide documented Windows enterprise policy / ADMX support for disabling its password manager via registry or a central machine-wide policy, unlike Chrome/Edge/Firefox. Microsoft’s own guidance for Intune/SCCM notes that there’s currently no device configuration support for Opera browser management.
#### Firefox Browser Management


## Disable Browser Password Managers - Introduction

### Google Chrome
The `PasswordManagerEnabled = 0` policy disables Chrome’s built-in password manager and prevents users from saving new passwords.  
Chrome does **not** provide an enterprise policy to disable the **“Sign in automatically”** setting. That UI toggle may still appear, but without a functioning password manager, Chrome cannot save new passwords or prompt users to store credentials.

### Mozilla Firefox
The `PasswordManagerEnabled = 0` policy disables Firefox’s built-in password manager entirely.  
When enforced, this also disables Firefox’s **“Fill usernames and passwords automatically”** feature, because it is a function of the password manager.  
Firefox does not provide a separate enterprise policy for autofill beyond disabling the password manager itself.

### Microsoft Edge (Chromium)
Edge supports both password-manager and autofill/passkey controls.  
- `PasswordManagerEnabled = 0` disables saving passwords.  
- `PrimaryPasswordSetting = 3` additionally disables **“Autofill passwords and passkeys”** and locks the toggle.  
Edge is the only major browser that exposes a dedicated enterprise policy for disabling autofill and passkeys.

## Disable Browser Password Managers via PowerShell Scripts

### Google Chrome
```
# Disable Chrome built-in password manager (machine-wide)
$base = 'HKLM:\SOFTWARE\Policies\Google\Chrome'
New-Item -Path $base -Force | Out-Null
New-ItemProperty -Path $base -Name 'PasswordManagerEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
```
### Mozilla Firefox
```
# Disable Firefox built-in password manager (machine-wide)
$base = 'HKLM:\SOFTWARE\Policies\Mozilla\Firefox'
New-Item -Path $base -Force | Out-Null
New-ItemProperty -Path $base -Name 'PasswordManagerEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
```
### Microsoft Edge (Chromium)
```
# Disable Edge built-in password manager (machine-wide)
$base = 'HKLM:\SOFTWARE\Policies\Microsoft\Edge'
New-Item -Path $base -Force | Out-Null
New-ItemProperty -Path $base -Name 'PasswordManagerEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
New-ItemProperty -Path $base -Name 'PrimaryPasswordSetting' -PropertyType DWord -Value 3 -Force | Out-Null
```

## Disable Browser Password Managers via GPO

### Steps to Configure Group Policy (GPO)

1. Open **Group Policy Management** (`gpmc.msc`).
2. Right-click the target OU or the domain root and select  
   **Create a GPO in this domain, and Link it here…**  
   Name it: **Enforce – Disable Browser Password Managers**
3. Right-click the new GPO and select **Edit…**
4. Navigate to:  
   **Computer Configuration** → **Preferences** → **Windows Settings** → **Registry**
5. Right-click **Registry** → **New** → **Registry Item**  
   Configure the following entries:

---

### Google Chrome – Disable Built-in Password Manager  
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Google\Chrome`  
- **Value name:** `PasswordManagerEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

---

### Mozilla Firefox – Disable Built-in Password Manager  
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Mozilla\Firefox`  
- **Value name:** `PasswordManagerEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

---

### Microsoft Edge (Chromium) – Disable Built-in Password Manager  
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Microsoft\Edge`  
- **Value name:** `PasswordManagerEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

---

### Microsoft Edge (Chromium) – Disable *“Autofill passwords and passkeys”*

To additionally disable Edge’s **Autofill passwords and passkeys** feature for all users, create **another** Registry Item:

- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Microsoft\Edge`  
- **Value name:** `PrimaryPasswordSetting`  
- **Value type:** `REG_DWORD`  
- **Value data:** `3`  

> Effect: Disables “Autofill passwords and passkeys” in Edge and prevents users from turning it back on.

---

### Verification  
1. Run `gpupdate /force` on a client (or allow normal refresh).  
2. Verify the registry keys exist under their respective policy paths:
   - `HKLM\SOFTWARE\Policies\Google\Chrome`
   - `HKLM\SOFTWARE\Policies\Microsoft\Edge`
   - `HKLM\SOFTWARE\Policies\Mozilla\Firefox`
3. Open each browser to confirm password saving is disabled.

## Disable Browser Saving Payment Methods and Addresses - Introduction

### Google Chrome – Payment Methods & Addresses
Chrome provides enterprise policies to disable autofill of **payment methods** and **addresses**, but these are separate from the password manager:
- `AutofillCreditCardEnabled = 0` disables saving and autofilling payment cards.
- `AutofillAddressEnabled = 0` disables saving and autofilling addresses.

These settings do **not** affect the "Sign in automatically" option and are independent of `PasswordManagerEnabled`.

### Microsoft Edge (Chromium) – Payment Methods & Addresses
Edge provides enterprise policies to disable autofill of **payment methods** and **addresses**, but these are separate from the password manager:
- `AutofillCreditCardEnabled = 0` disables saving and autofilling payment cards.
- `AutofillAddressEnabled = 0` disables saving and autofilling addresses.

These are separate from Edge’s password-related controls (`PasswordManagerEnabled` and `PrimaryPasswordSetting`).

### Mozilla Firefox – Payment Methods & Addresses
Firefox provides enterprise policies to disable autofill of **payment methods** and **addresses**, but these are separate from the password manager:
- `AddressAutofillEnabled = 0` disables saving and autofilling payment cards.
- `AutofillAddressEnabled = 0` disables saving and autofilling addresses.

These settings are independent of `PasswordManagerEnabled`.

## Disable Saving Payment Methods & Addresses via PowerShell

### Google Chrome
```
# Disable Chrome payment method autofill (credit cards)
$base = 'HKLM:\SOFTWARE\Policies\Google\Chrome'
New-Item -Path $base -Force | Out-Null
New-ItemProperty -Path $base -Name 'AutofillCreditCardEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
# Disable Chrome address autofill
New-ItemProperty -Path $base -Name 'AutofillAddressEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
```
### Microsoft Edge (Chromium)
```
# Disable Edge payment method autofill (credit cards)
$base = 'HKLM:\SOFTWARE\Policies\Microsoft\Edge'
New-Item -Path $base -Force | Out-Null
New-ItemProperty -Path $base -Name 'AutofillCreditCardEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
# Disable Edge address autofill
New-ItemProperty -Path $base -Name 'AutofillAddressEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
```
### Mozilla Firefox
```
# Disable Firefox payment method autofill (credit cards)
$base = 'HKLM:\SOFTWARE\Policies\Mozilla\Firefox'
New-Item -Path $base -Force | Out-Null
New-ItemProperty -Path $base -Name 'AutofillCreditCardEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
# Disable Chrome address autofill
New-ItemProperty -Path $base -Name 'AutofillAddressEnabled' -PropertyType DWord -Value 0 -Force | Out-Null
```
## Disable Saving Payment Methods & Addresses via GPO

### Steps to Configure Group Policy (GPO)

1. Open **Group Policy Management** (`gpmc.msc`).
2. Right-click the target OU or the domain root and select:  
   **Create a GPO in this domain, and Link it here…**  
   Name it: **Enforce – Disable Autofill for Payment Methods & Addresses**
3. Right-click the GPO and choose **Edit…**
4. Navigate to:  
   **Computer Configuration** → **Preferences** → **Windows Settings** → **Registry**
5. Right-click **Registry** → **New** → **Registry Item**  
   Then create the following registry entries for each browser.

---

### Google Chrome – Disable Saving Payment Methods
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Google\Chrome`  
- **Value name:** `AutofillCreditCardEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

### Google Chrome – Disable Saving Addresses
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Google\Chrome`  
- **Value name:** `AutofillAddressEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

---

### Microsoft Edge (Chromium) – Disable Saving Payment Methods
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Microsoft\Edge`  
- **Value name:** `AutofillCreditCardEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

### Microsoft Edge (Chromium) – Disable Saving Addresses
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Microsoft\Edge`  
- **Value name:** `AutofillAddressEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

---

### Mozilla Firefox – Disable Saving Payment Methods  
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Mozilla\Firefox`  
- **Value name:** `AutofillCreditCardEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

### Mozilla Firefox – Disable Address Autofill
- **Action:** Update  
- **Hive:** `HKEY_LOCAL_MACHINE`  
- **Key Path:** `SOFTWARE\Policies\Mozilla\Firefox`  
- **Value name:** `AddressAutofillEnabled`  
- **Value type:** `REG_DWORD`  
- **Value data:** `0`

---

### Verification  
1. Run `gpupdate /force` on a client system or wait for automatic refresh.  
2. Verify the registry paths exist under:
   - `HKLM\SOFTWARE\Policies\Google\Chrome`
   - `HKLM\SOFTWARE\Policies\Microsoft\Edge`
   - `HKLM\SOFTWARE\Policies\Mozilla\Firefox`
3. Launch each browser and confirm:
   - Autofill of payment methods is disabled.
   - Autofill of addresses is disabled.  
