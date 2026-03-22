# Create Personal Key Pair for Public Key Authentication

## 1.0 Algorithm

There are multiple algorithms available, whereas most commonly RSA and Ed25519 are used.

## 2.0 Using PuTTYgen on Windows

In order to create a Key Pair, you need the program PuTTYgen, see: https://github.com/etkaar/help/tree/main/tools

Choose **1. EdDSA**, below select **Ed25519 (255 bits)** and then **2.** click **Generate**. Once that is done, two cryptographic values are generated: A private key which obviously must be kept private (if you would share that, it would be the same as sharing your password) and a public key.

Now, create a new directory on a drive where you are sure, the data is not lost, e.g. **D:\secret-keypairs**.

After that, save the public and private key using the buttons **3. Save public key** and **4. Save private key**. Once PuTTYgen asks you for confirmation that you don't want to protect the private key with a *passphrase* you can choose **Yes** if you're an educated user.

Since you will usually use a *username* for the environment (e.g. a dedicated server) you will use the keypair for, use your username so that you end up with two files:

**Public Key**
- C:\Users\%username%\.ssh\username.public

**Private Key**
- C:\Users\%username%\.ssh\username.private

You then will be asked for your *public* key. Never share your private key.

<img width="602" height="471" alt="image" src="https://github.com/user-attachments/assets/180e1627-2d93-459a-aa6d-2d8689a1d880" />

## 3.0 GNU/Linux

On GNU/Linux, just login with the user you want the keypair being created for and type in following command:

```
ssh-keygen -t ed25519
```
