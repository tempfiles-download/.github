# TempFiles

**Share files securely over the internet for a day.**

## Why?

I, [@Carlgo11](https://github.com/Carlgo11/), created a site called [UploadMe](https://github.com/Carlgo11/UploadMe) back in 2014.  
The goal of the site was to help people securely share files with their friends without any spying eyes seeing the information.  
The main principle was to store the files with the same level of safety regardless of whether it was cat pictures or state secrets.

Due to the lack of resources I had to stop hosting UploadMe.  
TempFiles is a remastered version of UploadMe with updated encryption methods, improved storage mechanisms and better resource management resulting in a lower hosting bill.

## How?

### Components

<details>
<summary>Frontend</summary>

[__Frontend__](https://github.com/tempfiles-download/Frontend) is responsible for the website [tempfiles.download](https://tempfiles.download). It's a static website sending form data to the backend server.  
The frontend project is meant to be hosted by a CDN like GitHub Pages as it provides the most transparency with the least operating cost.
</details>

<details>
<summary>Backend</summary>

[__Backend__](https://github.com/tempfiles-download/Backend) is responsible for the encryption/decryption and storage of files. It's built using PHP and meant to be hosted on a read-only Linux file system.  
On [api.tempfiles.download](https://api.tempfiles.download) only the directory `/tmp` which stores the uploaded files is writable (as a [tmpfs](https://en.wikipedia.org/wiki/Tmpfs)).
</details>

### Procedure flows

<details>
<summary>Uploads</summary>

1. A user uploads a file through [tempfiles.download](https://tempfiles.download) or any other file-uploading client supporting TempFiles.
1. The file is sent via TLS 1.3 to the TempFiles [backend](https://github.com/tempfiles-download/Backend) server(s).
1. The received file is allocated a unique ID (UUID) starting with the letter `D` along with a random deletion password.
1. The file is encrypted using `AES-256-GCM` and stored in a JSON-file on the server. The time of the upload is also recorded in the file.
    The stored file looks like this:
    ```JSON
    {
    "expiry": 1652131135,
    "metadata": [
        "QWZUD7uTQI2u1Y3rAszFYBo94/ExKazJCkTu0unSMRWOamh3urriFAAGag==&GoElqdPyTxrEW9Mj&vK/WA8o0ZBVGLVKvAOSVlg=="
    ],
    "delpass": "$2y$10$G2DwZtWst2L8aSM44JijFO6jp6PJpBNZMXyw9tXo2sz.6iCXthCcy",
    "content": [
        "7PBZG3JRgh8=&bqoT732sVdwN6b25&IwLe91YwjRNiVQqu7HhM8g=="
    ]
    }
    ```
    * __Expiry__ - UNIX timestamp of file upload time + 24 hours
    * __Metadata__ - File metadata such as name, type, size _(AES-256-GCM)_
    * __Delpass__ - Deletion password hash _(bCrypt)_
    * __Content__ - File content _(AES-256-GCM)_

1. The user receives the UUID of the file and the deletion password.
</details>

<details>
<summary>Downloads</summary>

1. The unique ID (UUID) and decryption password is sent to the server `d.tempfiles.download`.
1. The server finds the corresponding file of the UUID and attempts to decrypt the file.
1. Assuming the file was successfully decrypted, the file metadata and file content is transferred back to the user.
</details>

<details>
<summary>Deletions</summary>

* __Automatic:__
1. The server goes through all files each hour and checks if the upload time was more than 24 hours ago.
1. If a file is older it is purged from the file system.

* __Manual:__
1. A user sends the unique ID (UUID) and deletion password to the server `api.tempfiles.download`.
1. If a file with the UUID exists, the deletion password is compared to a stored hash of the deletion password.
1. If the sent password matches the hash, the file is purged from the file system.
1. The server sends a notice of file deletion back to the user.
</details>

### API

A list of available API calls to the [Backend](https://github.com/tempfiles-download/Backend) service can be found on [Postman](https://documenter.getpostman.com/view/TzK2bEsi).

## FAQ

<details>
<summary>What is recorded?</summary>

* Access to files are not recorded.
* Error logs record unexpected errors unless a Do-Not-Track (DNT) header isn't used by the web browser.
</details>

<details>
<summary>How can I verify no logs are kept?</summary>

* As with any other website, it's impossible for a user to verify whether or not a service keeps logs so it all falls down to trust.
* If you don't trust my storage server I recommend you host your own.
</details>

<details>
<summary>Why not use client-side encryption?</summary>

* It is possible to encrypt a file in the client browser using JavaScript but that would require everyone to use a web browser to upload files. TempFiles is designed to be compatible with common file-uploading clients like ShareX and cURL.
* If you require client-side encryption I recommend you encrypt the file yourself before uploading it.
</details>

<details>
<summary>Why not use MySQL,Redis,MongoDB etc?</summary>

* File encryption requires a lot of memory and sending a large data blob to a database also requires a lot of memory. While MySQL support is implemented it only works for very small files or very powerful servers.
Eliminating the bottlenecks of database operations means much larger files can be stored.
</details>

<details>
<summary>What's the maximum allowed file size?</summary>

* There's no set maximum file size but due to the RAM required to encrypt uploaded files there's a practical limit where the server RAM runs out and requests are dropped.
* For [tempfiles.download](https://tempfiles.download) the limit is around __500MiB__.
* If you intend on hosting your own TempFiles [Backend](https://github.com/tempfiles-download/Backend) you should expect the RAM usage to be close to 1:1 with the file size if you store files using the Linux file system and 3:1 for MySQL storage.
</details>

<details>
<summary>Do you make money on this?</summary>

__No__. This is a hobby project of mine and for the moment requires very little resources to run.  
As such I'm able to provide this service for free without any ads or other compromises to user privacy.  
I do however accept [donations](https://github.com/sponsors/Carlgo11) through GitHub which are much appreciated! :heart:
</details>
