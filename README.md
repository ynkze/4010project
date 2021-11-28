# SecureFiles
SecureFiles is an online file hosting service specifically designed to host user files with user's privacy as a priority. Users can upload files that could be accessed over the internet and choose from a list of registered users to share files with them. The intermediary, our hosting service SecureFiles, will be end-to-end encrypted for file sharing to all users and the website will not know the content of the unencrypted file.

To achieve this, our project goals are as followed:

* User management module needs to be authenticated to ensure entity authentication and non-repudiation.

* File sharing should follow end-to-end encryption so that the platform does not see unencrypted shared files.

* Confidentiality and Integrity of the shared files must be assured even in case of attacks on the intermediary.

 ### Motivation

Many online file hosting service, such as Dropbox, do not provide end-to-end encryption for files. This means that the hosting service has the necessary key to encrypt and decrypt files. Cybersecurity threats are constantly evolving and seek to exploit vulnearabilities in software, and if the service suffer from a data breach, the files shared over hosting services are not safe from content being leaked. Even major companies like Twitch, Facebook, using state of the art cybersecurity has suffered from data breaches recently as well. Many companies also need access to the file content to cooperate with government surveillance or to analyze personal data for targeted ads. SecureFiles was designed as an alternative to tackle the aforementioned problems, as we feel that the intermediary cannot be trusted fully with our files.

 ### Research and Design

From our research, we found that most file hosting services use AES-256 to encrypt files at rest. End-to-end encryption is usually only used for messaging platform such as Whatsapp. This is because files are long-lived and rather large, making them difficult to protect, compared to messages. We looked into how we can implement end-to-end encryption for file sharing, and we came up with the following: 

1. Alice upload a file and select the user they want to share to, in this case Bob.
2. The file will be encrypted on Alice's end using the shared Diffie-Hellman key, generated from Alice's private key and Bob's public key.
3. The file stored in the database can only be decrypted when either Alice and Bob's private key is known, which only Alice and Bob has access to. The intermediary or other 3rd parties will not be able to decrypt the content of the file.
4. When Bob wants to download the file, the file will be decrypted with the same shared Diffie-Hellman key, but generated from his private key and Alice's public key, and he will be able to see the content of the file.

We opt to keep the size of file to a maximum of 10MB to ensure we are able to protect it well and make sure encryption is fast enough. 

 ### Development

To achieve our project goals, we did the following:

1. User management module: We use PHP and MYSQL database to store login information of users.
    * When user login, authenticate.php will verify if the user login information matches the database. If it matches, a session is created that will be destroyed on logging out. Any unregistered user or wrong password will be denied access to the website.
    * Every page of the websites can only be accessed when the user is login. Trying to manually enter the site without credentials will be redirected back to the login page.

2. End-to-end encryption: We uses mainly the OpenSSL library for the implmentation of the security.
    * OpenSSL is used to generate a 512-bit private and public key for both Alice and Bob, using the same dh parameters p and g. The openssl_dh_compute_key() function will be used to compute the dh key for encryption/decryption, using a local private key and remote public key.

    * The openssl_encrypt() and openssl_decrypt function is use for AES-256 encryption/decryption of the file contents.

3. For the implementation of integrity of the shared files:
    * md5_file() hashing is used to hash the file content before encryption. The result will be used to compare after decryption for verification. This can be used to check if a file has been tampered by an intermediary.

 ### Code usage
 
 1. .pem files: use to store the generated private and public keys
 2. createTable.php: use to create table into database
 3. filesLogic.php: algorithm for upload, download and remove of files 
 4. security.php: functions for generating dh key and for AES-256 encryption/decryption
 5. authenticate.php: check whether the login credentials are correct
 6. Rest of files: software and UI design of website
 
 ### Pre-requistes

To test this project, the following necessary:

1. Download XAMPP control panel and launch Apache and MYSQL module.
2. Place this folder named 4010project in drive location C:\xampp\htdocs
3. Open web browser and type localhost/4010project/createTable.php to create database tables. Ensure to create 4 sql databases named 'phplogin', 'file-management', 'integrity-management' and 'key-management'  beforehand.
4. Go to localhost/4010project and you should see a login page. We have 2 accounts for testing:

    * Username: alice
    * Password alice

    * Username: bob
    * Password bob

5. Upload files of type .zip, .pdf, .docx under 10MB to verify it is working for different users.

