# Lab Setup 
## MacOS 
Download `lab.pem` from the `keys` directory

### Set permission on SSH key 
```
chmod 600 /path/to/lab.pem
```

### SSH to lab servers 
The username for SSH is `ubuntu`
```
ssh -i /path/to/lab.pem ubuntu@<LAB IP> 
```


## Windows 
Download `lab.ppk` from `keys` directory

Open Putty and configure a new session. 
  
![](index/C4EC1E64-175D-4C84-8C49-D938337FA35A.png)

Expand “Connection_SSH_Auth and then specify the PPK file 
![](index/6FFB137C-1AD8-48A1-97E6-F5F6DA4BC55B.png)

 Now save your session    

![](index/FD3BA694-FD69-4C86-8EAF-4D5FC813EABA.png)
