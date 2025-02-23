---
sidebar_position: 3
---

# How to install apps in Azure (For a virtual desktop)

    - Fact : It's in the name... virtual : Hard to install any program directly or physicaly.
    - Problem : Need to install the apps for this project to work. In this case, I have only 4. (Not even Office 365)
    - Hypothese : Can I install these files through Azure by having them uploaded to a shared space?

## Options in Azure

    - There is a few options in Azure services in regards to storage :
        ![Storageoptions](/img/Azure/Storageoptions.png)

    - For what I understand, Storage account is the option needed. From here, need to create one :
        ![CreateStorage](/img/Azure/createstorage.png)

    - For security, networking, data protection, encryption and tags, I keeped everything witht the default options selected.

    - Once the storage account created, I created Containers to subdivide the application folders :
        ![Containers](/img/Azure/Containers.png)

    - Following this : https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-blobs-upload, I was able to upload the installation files in the proper containers.
        - Here the command I used : 
        ```js
            azcopy copy 'C:\myDirectory' 'https://mystorageaccount.blob.core.windows.net/mycontainer' --recursive
        ```
        - I changed : my local directory, the name of the storage account and container. Example :
        ```js
            azcopy copy 'C:\mul' 'https://storagetestena.blob.core.windows.net/mul' --recursive
        ```

    - **Make sure that only the files needed are uploaded to the container because I didn't find a way to move files from one container to another.**

## First approach
    - Since some of the apps don't have packages, I'm thinking that I could download the apps on to VD and manualy install them. 

    - To do this, I reversed the privious command from the terminal in the VD :

        ```js
            azcopy copy 'https://mystorageaccount.blob.core.windows.net/mycontainer' 'C:\Temp' --recursive
        ```
    - And voilà! All the apps are on the VD's C drive.



## What not to do 

    - Here's what I tried just for the hell of it : Download the apps directly on the desktop and install them. It worked but, as soon as I restarted the machine, the apps were gone (I forgot that the pool has 2 virtual machine and so, if one is shutdown, the other one steps in and I do not have the apps installed on that one yet).

## 

    - Now, I'm taking the approach of the web 