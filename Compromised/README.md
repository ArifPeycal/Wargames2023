# Compromised
## Description
> Where aRe you?

## Challenge Overview

We were given a zip file that contain user directories. 

![image](https://github.com/user-attachments/assets/96a1e5d3-0c14-4e8b-8fbc-7cb2bb180512)

There a `flag.png` and `flag.txt` in `Desktop`. But, the `flag.png` cannot open and `flag.txt` is empty.

![image](https://github.com/user-attachments/assets/39dd69a6-7133-4523-8327-54b174695084)

When I send the image to <a href="https://hexed.it/">hexedit</a>, the magic bytes indicate that the file is a zip file. 

![image](https://github.com/user-attachments/assets/ae7471d1-9511-45a5-b4c9-9d94daea2037)

After changing the file extension from `.png` to `.zip` and trying to unzip it, I found out that it is password-protected.  

Going to `Documents` and we can see `Default.rdp`. It means that the computer had been accessed by `RDP`. 

> Default.rdp is a file created by Microsoft's Remote Desktop Connection client (mstsc.exe) when a user initiates a remote desktop session. It stores the user's default settings for Remote Desktop sessions, making it easier to reuse configuration settings for future connections.

After going through all directories, I found out two files, `bcache24.bmc` and `Cache0000.bin` located at `C:\Users\<USER>\AppData\Local\Microsoft\Terminal Server Client\Cache\`. 

Asking ChatGPT, I found out that the files are `RDP Bitmap Cache`.

> The RDP Bitmap Cache is a feature of the Remote Desktop Protocol (RDP) that improves the performance and responsiveness of remote desktop sessions, particularly over slower network connections. Here's how it works:

### How RDP Bitmap Cache Works:
- Caching Bitmaps: When you connect to a remote desktop, the images (or bitmaps) displayed on the screen, such as windows, icons, and other graphical elements, are transferred from the remote machine to your local device. Instead of sending the same graphical data repeatedly, the RDP client stores (caches) these bitmaps locally on your machine.
  
- Reusing Cached Bitmaps: Once the bitmap is cached, if the same or similar graphical element appears again, the RDP client can pull it from the local cache rather than requesting the server to resend the data. This reduces network bandwidth usage and speeds up screen refresh rates.

## Solution
We can use BMCTools to parse the `RDP Bitmap Cache`. 

```
python3 bmc-tools.py -s Cache0000.bin -d parsed -b
```

![image](https://github.com/user-attachments/assets/b48ed10e-0c5a-4c76-a76d-125c3e296604)

We can see the password for zip file but the images are quite messy. 

![image](https://github.com/user-attachments/assets/02a3fb7e-f328-4046-9639-b0a6c9f79ef7)

We can use `RdpCacheStitcher`, an open source GUI tools that allows us to construct bitmap by dragging and dropping the images. After constructing the bitmap, we get the password for zip file.

```
WGMY_P4ssw0rd_N0t_V3ry_H4rd!!!
```
![image](https://github.com/user-attachments/assets/25b0c74e-a123-44fa-ae63-b0210a854486)

## Flag
```
wgmy{d1df8b8811dbe22f3dce67ef2998f21c}
```
