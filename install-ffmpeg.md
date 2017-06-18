# Install FFMPEG

## Step 1: Update the system
    ```
    sudo yum install epel-release -y
    sudo yum update -y
    sudo shutdown -r now
    ````
### Step 2: Install the Nux Dextop YUM repo
    No official FFmpeg rpm packages for CentOS for now.
    We can use a 3rd-party YUM repo, Nux Dextop.
    - CentOS 7.x :
        ```sh
        sudo rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
        sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
        ```
    - Centos 6.x
    ```sh
        sudo rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
        sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el6/x86_64/nux-dextop-release-0-2.el6.nux.noarch.rpm
    ```

### Step 3: Install FFmpeg and FFmpeg development packages
    ```sh
    sudo yum install ffmpeg ffmpeg-devel -y
    ```
