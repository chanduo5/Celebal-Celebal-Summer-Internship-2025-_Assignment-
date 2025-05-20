


# **DevOps001 "weekly Assignments" {week-1}**




------------------------

## Create a file, assign permissions (read, write, execute) to different user categories (owner, group, others), and practice changing permissions using chmod.

####  **Create a File**


```bash
touch myfile.txt
```

You can check  with:

```bash
ls -l myfile.txt
```

####  **Change Permissions Using `chmod`**

#####  **Add Execute Permission to Owner**

```bash
chmod u+x myfile.txt
```


---

#####  **Give Read, Write to Group**

```bash
chmod g+rw myfile.txt
```

Now group has read & write.

---

#####  **Remove All Permissions for Others**

```bash
chmod o-rwx myfile.txt
```


###  Summary of Permission Numbers

| Symbol | Meaning | Binary | Octal |
| ------ | ------- | ------ | ----- |
| `r`    | Read    | 100    | 4     |
| `w`    | Write   | 010    | 2     |
| `x`    | Execute | 001    | 1     |
| `-`    | None    | 000    | 0     |



![Screenshot](Screenshorts/Screenshot%202025-05-20%20232612.png)


------------------------------------------------


-------------------------------------------------

##  Execute basic Linux commands (e.g., ls, cd, mkdir, rm, touch) to manipulate files and directories, with an emphasis on understanding their usage.
####  Create a File

```bash
touch chandufile.txt
```

####  Create a Directory

```bash
mkdir chandu
```

####  Change into a Directory

```bash
cd chandu
```

####  Go Back to Parent Directory

```bash
cd ..
```

####  List Files

```bash
ls          # Basic list
ls -l       # Long format (permissions, owner, etc.)
```

####  Remove File or Directory

```bash
rm chandufile.txt        # Delete a file
rm -r chandu            # Delete a directory
```

---------------------------------------------------

![Screenshot](Screenshorts/Screenshot%202025-05-20%20232916.png)

--------------------------------------------------


##  Using the terminal, practice navigating through directories, listing file contents, and moving files to different locations.

####  Create Structure

```bash
mkdir chandu
cd chandu 
touch test.txt project.txt
mkdir doc
```

####  Move and Rename Files

```bash
mv test.txt doc/        # Move file to doc
mv oldname.txt newname.txt    # Rename file
```

####  Copy Files and Directories

```bash
cp test.txt copy_test1.txt       # Copy a file
cp -r dir1/ dir2/                 # Copy a directory
```


----------------------------------------------------------

![Screenshot](Screenshorts/Screenshot%202025-05-20%20233828.png)

----------------------------------------------------------

##  Create a new user and group, set their permissions, and explore user management commands like useradd, usermod, and userdel.

####  Create a User

```bash
sudo useradd new1
sudo passwd new1
```

####  Create a Group

```bash
sudo groupadd dev
```

####  Add User to Group

```bash
sudo usermod -aG dev new1
```

####  View Group Membership

```bash
groups new1
```

####  Delete User and Group

```bash
sudo userdel new1
sudo groupdel dev
```

-----------------------------------------------------------------

![Screenshot](Screenshorts/Screenshot%202025-05-20%20234323.png)

-----------------------------------------------------------------

### Useful Commands

| Command       | Description                         |
| ------------- | ----------------------------------- |
| `pwd`         | Print current working directory     |
| `whoami`      | Display current user                |
| `man ls`      | Manual page for command             |
| `history`     | Show command history                |
| `clear`       | Clear terminal screen               |
| `df -h`       | Disk usage in human-readable format |
| `top`         | Show running processes              |
| `cat`         | Print file content                  |
| `nano file`   | Open file in nano editor            |
| `echo "text"` | Print message to terminal           |

---


---
