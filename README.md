## 🚀 Android Pentesting: Beginning with InsecureBankV2

Hello Everyone!
This marks the beginning of something new — **Android Pentesting**. Here, I'll be sharing everything I learn along the way.

---

## 🏦 InsecureBankV2: *Gaining Access to Admin Features*

To begin exploring the inner workings of an Android APK, we'll use a tool called **Bytecode Viewer**, which allows us to inspect the code inside the APK file.

### 🔧 Installing Bytecode Viewer

Install it using the following command:

```bash
apt install bytecode-viewer
```

> 📌 *Note: If you’re on a system that doesn’t support `apt`, such as Windows, you can download Bytecode Viewer from its [official ByteCode-Viewer](https://github.com/Konloch/bytecode-viewer) or use a Java decompiler like JADX.*

---

## 🔍 Investigating the APK File

In the file `AndroidManifest.xml`, we can see that the **first activity launched** is `LoginActivity`.

![AndroidManifest.xml-screen-shot](https://github.com/povzayd/Android-RE/blob/main/uploads/image1.png)

Navigate to `LoginActivity.class` inside the following directory structure:

```
com/android/InsecureBankV2/
```

Now open the `LoginActivity.class` file using Bytecode Viewer and look for any vulnerable code.

### 🧪 Vulnerable Code Snippet
![loginactivity.class-screen-shot](https://github.com/povzayd/Android-RE/blob/main/uploads/image4.png)

Around lines 66–72, you might see the following:

```java
protected void onCreate(Bundle var1) {
    super.onCreate(var1);
    this.setContentView(2130968605);
    if (this.getResources().getString(2131165258).equals("no")) {
        this.findViewById(2131558510).setVisibility(8);
    }
}
```

### 🧠 What This Code Does:

This block of code checks a specific string resource (probably named something like `"is_admin"` or `"show_admin_features"`).
If the value of this string is `"no"`, it hides a specific UI element using:

```java
.setVisibility(8)
```

> 📝 `8` corresponds to `View.GONE`, which means the element is **completely hidden and doesn't take up space**.

So, by changing that resource string from `"no"` to `"yes"`, we can potentially **unlock admin features** in the app.

---

## 🛠️ Using APKTool for Reversing & Rebuilding

Next, we’ll use `apktool` to **decompile** the APK and modify the resources.

### 🔧 Installing Required Tools

Run the following command to install all necessary tools:

```bash
apt install apktool openjdk-11 jarsigner zipalign
```

**What these tools do:**

* `apktool` – Decompiles and rebuilds APKs.
* `openjdk-11` – Java runtime environment (required for keytool and jarsigner).
* `jarsigner` – Used to sign the APK with a custom keystore.
* `zipalign` – Optimizes the APK for installation on Android.

---

### 📦 Decompiling the APK

To decompile the APK:

```bash
apktool d InsecureBankV2.apk
```

After running this, a folder named `InsecureBankV2/` will be created, containing the decompiled files.

---

### ✍️ Modifying the Resource File

Navigate to the values directory:

```bash
cd InsecureBankV2/res/values
```

Open the `strings.xml` file in your favorite text editor. Look for a string with the value `"no"` (likely named something like `show_admin_features`, `is_admin`, etc.) and change it to `"yes"`.
![is-admin-no-screen-shot](https://github.com/povzayd/Android-RE/blob/main/uploads/image2.png)
![is-admin-yes-screen-shot](https://github.com/povzayd/Android-RE/blob/main/uploads/image3.png)

```xml
<string name="is_admin">yes</string>
```

Save and close the file.

---

## 🔨 Rebuilding the APK

Use the following command to rebuild the APK:

```bash
apktool b InsecureBankV2
```

This will create a new APK inside the `InsecureBankV2/dist/` directory.

---

## 🔏 Signing the APK

Before installing the modified APK, you need to **sign it** using a keystore. Android won’t install unsigned APKs.

### 📜 Generate a Keystore File

Run:

```bash
keytool -genkey -v -keystore ctf.keystore -alias ctfKeystore -keyalg RSA -keysize 2048 -validity 20000
```
---

## 📖 Explanation:

| **Flag**                 | **Meaning**                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| `keytool`                | A Java tool used to generate and manage cryptographic keys and certificates. Comes with the Java Development Kit (JDK).         |
| `-genkey`                | Tells `keytool` to **generate a new key pair** (private + public keys).                                                         |
| `-v`                     | Enables **verbose** output, showing detailed information about the process.                                                     |
| `-keystore ctf.keystore` | Specifies the name of the **keystore file** to be created. This file will securely store your keys and certificates.            |
| `-alias ctfKeystore`     | A **nickname** for the key entry you're creating. You’ll use this alias later to refer to this key when signing the APK.        |
| `-keyalg RSA`            | Sets the **encryption algorithm** to RSA (Rivest–Shamir–Adleman), one of the most commonly used algorithms.                     |
| `-keysize 2048`          | Specifies the **length of the key** in bits. `2048` is a secure and commonly used size. Larger keys are more secure but slower. |
| `-validity 20000`        | Specifies the **validity period of the key** in days. Here, the key will be valid for \~54 years.                               |

---

## 📝 What Happens When You Run This?

1. You are prompted to enter a password for the keystore.
2. Then, you’ll provide personal information for the certificate (e.g., name, organization, country).
3. A keystore file named `ctf.keystore` will be generated, containing a new key with alias `ctfKeystore`.

---

## 🧠 Why Is This Important?

* Android requires APKs to be **signed** before installation.
* For **custom or reverse-engineered APKs**, you can’t use the original developer’s signature (you don’t have their private key), so you need to create your **own**.
* The key you generate here is used to **sign** the modified APK using tools like `jarsigner`.

---

### 🖊️ Sign the APK Using Jarsigner

```bash
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ctf.keystore InsecureBankV2/dist/InsecureBankV2.apk ctfKeystore
```
## 📖 Explanation:

| **Flag / Argument**                      | **Meaning**                                                                                                                       |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `jarsigner`                              | A Java tool used to **digitally sign JAR or APK files** and verify their signatures.                                              |
| `-verbose`                               | Enables **detailed output**, showing the signing process and entries being signed.                                                |
| `-sigalg SHA1withRSA`                    | Specifies the **signature algorithm**: SHA-1 hashing + RSA encryption. <br>🔴 *Note: SHA1 is outdated — use SHA256 for security.* |
| `-digestalg SHA1`                        | Specifies the **digest (hashing) algorithm** used to create a checksum of the file before signing.                                |
| `-keystore ctf.keystore`                 | Path to the **keystore file** containing the private key you'll use to sign the APK.                                              |
| `InsecureBankV2/dist/InsecureBankV2.apk` | The path to the **APK file to be signed**.                                                                                        |
| `ctfKeystore`                            | The **alias** of the key inside the keystore that will be used for signing.                                                       |

---

## 🔐 What Happens When You Run This?

1. You are prompted to enter the **keystore password**.
2. `jarsigner` accesses the key (identified by the alias `ctfKeystore`) inside `ctf.keystore`.
3. It signs the APK (`InsecureBankV2.apk`) using the selected key and algorithm (`SHA1withRSA`).
4. The signature is embedded inside the APK's metadata.

---

## 📌 Why This Is Important

* Android **requires all APKs to be signed** before they can be installed.
* By signing the APK with your own keystore, you're **proving the integrity and authenticity** of the file.
* Unsigned APKs (or those signed incorrectly) will be **rejected by Android**, especially on newer versions.

---

## ⚠️ Important Security Note:

* `SHA1withRSA` is **outdated and insecure**. It's fine for **testing or CTFs**, but for production use, prefer:

  ```bash
  -sigalg SHA256withRSA -digestalg SHA-256
  ```
---
## ✅ Summary:

This command:

* Signs your **modified APK** using the key from `ctf.keystore`.
* Ensures the APK is **recognized by Android as valid**.
* Is the final step before **zipaligning** and installing the app.

---
This signs the APK using the keystore you created.
---

### 🗜️ Zipalign the APK (Optional but Recommended)

Zipalign ensures that all uncompressed data starts with a particular byte alignment — which is required for performance and installation.

```bash
zipalign -v 4 InsecureBankV2/dist/InsecureBankV2.apk mod-bank.apk
```

> 🎉 `mod-bank.apk` is now your **modified and signed APK**, ready to be installed on an Android device.

---

## ✅ Final Step: Install & Test

Install the APK on your device or emulator using or you can do it manually:

```bash
adb install mod-bank.apk
```

Open the app and verify whether the admin features are now visible or accessible. If so, congratulations — you’ve successfully modified your first APK!
<!------->
<!------->
![no-mod-screen-shot](https://github.com/povzayd/Android-RE/blob/main/uploads/image5.jpg)
![modded-screen-shot](https://github.com/povzayd/Android-RE/blob/main/uploads/image6.jpg)

---

## 🧠 Summary

### What You Learned:

* How to decompile and analyze an APK using Bytecode Viewer.
* Identifying logic in the code that hides functionality based on resource values.
* Modifying string resources to unlock hidden features.
* Rebuilding, signing, and zipaligning an APK to make it installable.

This is a **fundamental skill** in Android pentesting, and it lays the groundwork for more advanced analysis like:

* Debugging APKs in real-time.
* Bypassing root/jailbreak detection.
* Hooking methods using Frida or Xposed.
