# Python packages for AWS Lambda Layers

## How to use

1. Download the zip file of the package you want inside the folder "zip_files"
2. On AWS lambda, go to layers and click on "create layer"
3. Upload the zip file, specify the runtime and click create
4. Go to your lambda function and click "Add a layer" to add the layer you created
5. Import the package to your code and use it



## How to create these zip files
- Install docker on your machine

- Pull the aws lambda docker:

```bash
 docker pull amazon/aws-lambda-python
```

- Create 3 folders : pycurl, mysqlclient and pandas

- Inside all of them add a Dockerfile a requirements.txt and a the following folder structure:
"python/lib/python3.9/site-packages"


- Inside the folders pycurl and mysqlclient create a folder named "lib"

- Create a docker compose file to run all three containers at once

- Inside the pandas folder use the Dockerfile layout available on [aws documentation](https://docs.aws.amazon.com/lambda/latest/dg/python-image.html#python-image-create) to build a container
(change version of python from 3.8 to 3.9 and remove app.py)

- To write the Dockerfile of mysqlclient, add this line to  install mysqlclient dependencies before installing
the package using pip:

```bash
 RUN yum install gcc mysql-devel -y
```

- To write the Dockerfile of pycurl, add these lines to install pycurl dependencies before installing the package using pip:

```bash
RUN yum install gcc openssl-devel libcurl-devel python39-devel -y

ENV PYCURL_SSL_LIBRARY=openssl

RUN ln -s /usr/include /var/lang/include
```

- Write the requirements.txt on each folder specifying the package name and the version of the package

- On your terminal, run the command ```docker-compose build``` and then ```docker-compose up``` to run all
of the containers at once

- To create the zip file for pandas copy the contents inside /var/task (remove requirements.txt) in pandas container to "pandas/python/lib/python3.9/site-packages" folder using the command ```docker cp```.
- Then zip the file using:  
```bash
 zip -r pandas-<pandas-version>-py39-<arch>.zip python
```

- Repeat the same process you did with pandas on mysqlclient container but also copy the
contents of "/usr/lib64/mysql" inside the container to the lib folder

- Inside the mysql folder, run the command:
```bash
 zip -r mysqlclient-<mysqlclient-version>-py39-<arch>.zip python lib
```

- Repeat the same process you did with pandas inside the pycurl container

- To find the pycurl depedencies, you will have to open two terminals inside the container

- You can open a terminal inside the container with the command:
```bash
 docker exec -it pycurl bash
```

- Run python inside the first terminal and use the following commands:
```python
 import os
 os.getpid() # Returns PID 
```
- On the second terminal type the commands:
```bash
 yum install lsof -y
 /usr/sbin/lsof -p <PID_NUMBER> > before.txt
```
 - On the first terminal, import pycurl

- On the second terminal run:
```bash
 /usr/sbin/lsof -p <PID_NUMBER> > after.txt
 diff before.txt after.txt | cut -d / -f 2- | cut -d " " -f 1 | sed '1d' |  sed 's/^/\//'  > diff.txt 
 mkdir lib
 cat diff.txt | xargs -I {} find -L /usr/lib64 -samefile {} | xargs -I {} cp -P {} lib/
```

- After that, all the neccessary files should be inside the lib folder (The lib folder inside the pycurl container)

- Copy the contents of the lib folder inside the container to the local lib folder inside the pycurl folder

- Inside the pycurl folder, run the command "zip -r pycurl-<pycurl-version>-py39-<arch>.zip python lib"

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

