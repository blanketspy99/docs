# Common Utilities Package

## Overview

This is common utilities used in day to day activities for operations.

```
python -m pylint src/mycommon/filesystem -r n --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" --output test-reports/pylint.txt --disable=R,C

pytest --junitxml=test-reports/test-results.xml --cov=mycommon --cov-report=xml --cov-report=html

cd docs
sphinx-apidoc -P -e -f -o source/ ../src/mycommon/
sphinx-apidoc -e -f -o source/ ../src/mycommon/
make html
or
sphinx-build -b html source/ build/html/docs/mycommon

### optional
cd docs
make clean
git submodule update --remote --merge
# now copy docs build html to submodule folder
```


```python
import os
#import local OS Filesystem ,S3 and Sharepoint
from mycommon.filesystem import OSFileSystem, S3FileSystem, SharePointFileSystem
fs1=OSFileSystem(os.getcwd())
fs1.chdir('..')
fs2=S3FileSystem('sp-s3bucket-cs')
auth=PublicClientAuthentication(client_id="accxxxx-1dxx-4bxx-xxxx-xxxxxxxxx",
                                        tenant_id=""xxxxxe87-7xxx3-4xxx-xxxxc-xxxxxxxxxx",
                                        username="user@mycompany.com",
                                        password="mysecretpassword")
fs3=SharePointFileSystem(
                    hostname="headcliff.sharepoint.com",
                    username="shaik.shahrukh@headcliff.com",
                    sitePath='/sites/Test',with_auth=True,auth=auth)
```
Every filesystem has common object handling method. All it differs on initialisation, such as for S3, you need the bucket name for basic init and for sharepoint, it requires details such as sharepoint hostname, sitepath and azure app registration client and tenant ids.
Reference:
1. S3: https://blanketspy99.avanverse.com/docs/mycommon/mycommon.filesystem.s3fs.html#module-mycommon.filesystem.s3fs
1. SharePoint: https://blanketspy99.avanverse.com/docs/mycommon/mycommon.filesystem.spfs.html#mycommon.filesystem.spfs.SharePointFileSystem

```python
from mycommon import utils
#copy as stream,  read both source and target in chunks
utils.copy_file_object_as_stream('PAV.zip',fs2,fs1,show_progress=True)

#copy as stream,  read both source and target in chunks with customisations
utils.copy_file_object_as_stream('PAV.zip',fs2,fs1,read_byte_size=4**10*30,write_byte_size=4**10*20,show_progress=True)

# copy by reading full data at once and write in chunks
utils.copy_file_object('PAV.zip',fs1,fs2,show_progress=False,chunk_write=True)

# copy by reading full data at once and write directly.
utils.copy_file_object('PAV.zip',fs1,fs2,show_progress=False,chunk_write=False)

```

### Folder Copy Example
```python

from mycommon.filesystem import S3FileSystem, OSFileSystem, SharePointFileSystem
from mycommon.utils import copy_file_object
s3fs=S3FileSystem('cf-templates-1oxxxxxxx94-us-east-1')
import os, posixpath
osfs=OSFileSystem(os.getcwd())
dir_w=osfs.listdir('docs',recursive=True)
# converting windows path to unix path if dir_w is run on windows
dir_u=[i.replace(os.sep, posixpath.sep) for i in dir_w]

#copy the folder and contents to s3fs folder inside a folder "new/"
for item in dir_u:
    if osfs.is_file(item):
        copy_file_object(item,osfs,s3fs,"new/"+item)
    if osfs.is_dir(item):
        s3fs.mkdir("new/"+item,exist_ok=True) #create directory in target system if full path not exists

#for optimisation, you can upload in parallel too
import concurrent
with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
    for item in dir_u:
        if osfs.is_file(i):
            executor.submit(copy_file_object_as_stream,item,osfs,s3fs,"new2/"+item)
        if osfs.is_dir(item):
            s3fs.mkdir("new/"+item,exist_ok=True) #create directory in target system if full path not exists


# Note: Concurrent Threadpool executor doesn't fail on exception, to get the list of failed objects at end you can
# query the executor futures post completion
#Example:

import concurrent
result=[]
with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
    for item in dir_u:
        if osfs.is_file(i):
            result.append(executor.submit(copy_file_object_as_stream,item,osfs,s3fs,"new2/"+item))
        if osfs.is_dir(item):
            s3fs.mkdir("new/"+item,exist_ok=True) #create directory in target system if full path not exists

for fut in result:
    try:
        print(fut.result())
    except Exception as e:
        print(e)
```

## #To-Do
1. Create function of create empty directory in each filesystem type.
1. Function to delete file/folder. **(completed in spfs)**
1. Function to rename/move file or folder internally. **(completed in spfs)**
1. Function to copy file/folder **(completed in spfs)**
1. **`Optional`** Utils copy operation can be optimised if the source and target system are same, then use internal copy functions for faster processing.
1. spfs authenticate part can be moved to utils as msal app authentication