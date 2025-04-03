## 🔐 File Encryption and Decryption Demo using OpenSSL and AWS KMS

This guide demonstrates how to encrypt and decrypt a file (`password.txt`) using **AWS KMS** and **OpenSSL**. We'll use a CMK called `demokey` located in the `ap-northeast-1` region.

---

### 🔑 Step 1: Generate a Data Encryption Key (DEK) using AWS KMS
We will generate a DEK using your KMS key (`alias/demokey`). This returns both a plaintext key and an encrypted key.  

```sh
aws kms generate-data-key --key-id alias/demokey --key-spec AES_256 --region ap-northeast-1 --output json > keys.json
```

Your `keys.json` will look like:

```json
{
    "CiphertextBlob": "AQIDAHjOz8gaBAHMXz7mV6Iq1glscRwo2rT4i+FxbbxWOZcoWwGTdWoM2gTz73VESuBwWu1xAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMs9rQpmdDwjsfs08HAgEQgDvojYI5ptKoE3DawuCQqSODRNKDPOaRpa7u0J6ivgowEDH+94hS9Glk+ebvgRPXJqadhwQwB7CUZJg4Zg==",
    "Plaintext": "FmQYswQnXt7g4edIaiV3OXQ45CWaa5jOiqU5EUHGzhI=",
    "KeyId": "arn:aws:kms:ap-northeast-1:123456789012:key/abcd-1234..."
}
```

---

### 🔑 Step 2: Save the Keys Locally

#### Save the Plaintext Key (Decoded)
```sh
echo "FmQYswQnXt7g4edIaiV3OXQ45CWaa5jOiqU5EUHGzhI=" | base64 -d > plaintext.key
```

#### Save the Encrypted Key (CiphertextBlob)
```sh
echo "AQIDAHjOz8gaBAHMXz7mV6Iq1glscRwo2rT4i+FxbbxWOZcoWwGTdWoM2gTz73VESuBwWu1xAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMs9rQpmdDwjsfs08HAgEQgDvojYI5ptKoE3DawuCQqSODRNKDPOaRpa7u0J6ivgowEDH+94hS9Glk+ebvgRPXJqadhwQwB7CUZJg4Zg==" | base64 -d > ciphertext.key
```

---

### 🔒 Step 3: Encrypt `password.txt` using OpenSSL

We will use the plaintext key to encrypt the file. Note the use of `-pbkdf2` for stronger key derivation.

```sh
openssl enc -aes-256-cbc -salt -in password.txt -out password.txt.enc -pass file:./plaintext.key -pbkdf2
```

---

### 🔓 Step 4: Decrypt the Encrypted Key using KMS

Now, to decrypt the file, you need to **recover the plaintext key** from the encrypted key (`ciphertext.key`).

```sh
aws kms decrypt --ciphertext-blob fileb://ciphertext.key --region ap-northeast-1 --output json
```

The output will provide a new `Plaintext` value. Save it to `plaintext.key` using:

```sh
echo "FmQYswQnXt7g4edIaiV3OXQ45CWaa5jOiqU5EUHGzhI=" | base64 -d > plaintext.key
```

---

### 🔓 Step 5: Decrypt the File using OpenSSL

Now, use the recovered key to decrypt the file.

```sh
openssl enc -d -aes-256-cbc -salt -in password.txt.enc -out password.txt -pass file:./plaintext.key -pbkdf2
```

---

### ✅ Verification

Compare the original and decrypted files:

```sh
diff password.txt decrypted_password.txt
```

If there's no output, the decryption was successful! 🎉

---

## ✅ How to Copy an Encrypted EBS Snapshot (Using Customer Managed Key) to Another AWS Account via Console

AWS **console does not support sharing snapshots encrypted with the default AWS-managed KMS key**. So, we will first **re-encrypt the snapshot using a Customer Managed Key (CMK)** and then share it.

### 🔑 Step 1: Create and Share a Customer Managed KMS Key (CMK)
1. **Go to the KMS Console:** Navigate to **AWS Management Console > KMS > Customer managed keys > Create key**.
2. **Select Key Type:** Choose **Symmetric** (recommended for EBS encryption).
3. **Configure Key:** Set a **Name**, **Description**, and other settings as needed.
4. **Define Key Policy:**
   - Go to **Key Policies** and click on **Add Account**.
   - Enter the **AWS Account ID** of the target account you want to share the snapshot with.
   - Save Changes.
5. **Save Changes** and **Create Key.**

---

### 🔒 Step 2: Re-Encrypt the Snapshot Using Your CMK
1. **Go to the Amazon EC2 Console:** Navigate to **Snapshots** under **Elastic Block Store**.
2. **Select the Snapshot to be Copied.**
3. Click on **Actions > Copy**.
4. In the **Copy Snapshot** dialog, select your **CMK**.
5. Click **Copy**.

---

### 📤 Step 3: Share the Snapshot with Another AWS Account
1. **In the EC2 Console**, go to **Snapshots**.
2. Select the **Copied Snapshot (Encrypted with CMK)**.
3. Click on **Actions > Modify Permissions**.
4. Click **Add Account** and enter the **AWS Account ID** of the target account.
5. Click **Save**.

---

### 📥 Step 4: Copy the Snapshot in the Target Account
1. **Log into the Target Account**.
2. Go to **Snapshots** in the **EC2 Console**.
3. Select the **shared snapshot**.
4. Click on **Actions > Copy**.
5. Choose **a CMK from the target account**.
6. Click **Copy**.


