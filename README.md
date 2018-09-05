# Jupyter notebook access to an aws-ec2 instance from any browser for windows users using putty

> I hope everyone is familiar with the AWS (Amazon Web Services) and how to use Jupyter Notebook. If you are not familiar with this tool and you work with Python, I highly recommend you to learn it in order to not miss its importance in your future works.
The deployment of Jupyter Notebook Server on AWS allows to access all created notebooks from anywhere just using a browser.
According to this short tutorial, I'll list all the steps to create a Jupyter Notebook Server on an EC2 Instance in a step-wise fashion. 


## 1. Create an AWS account

The first step is to login to your Amazon Management Console. If you don't have an account yet, you can create one for it. You get one year of free access to some of the services, which you can check out at this link:
https://aws.amazon.com/free/


## 2. Create an EC2 Instance on Ubuntu 
Go to the EC2 main page, then you will see a '**Launch Instance**' button. If you are not familiar with how to create an EC2 instance, 
you can check out the video given in this link: https://www.youtube.com/watch?v=q4PrYQOShnE, in which the author goes through the steps from the beginning. 

By finishing this step, check that you have edit the inbound rules of the security groups of your EC2 instance as follows (**especially the SSH rule**):

| Type   |      Port Range      |  Source |
|----------|:-------------:|------:|
| **SSH** |  **22** | **Anywhere** |
| Custom TCP Rule |    8888   |   Anywhere |
| HTTPS | 443 |    Anywhere |


## 3. Connecting to your Linux Instance from Windows using PuTTY
SSH into the EC2 instance for Windows users, you’ll need to use PuTTY. Amazon has a really good set of instructions located here: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html 

Follow those to the point where you have the connection to the Ubuntu console.


## 4. Download the latest Anaconda Distribution for Linux
- Refer to the download website of anaconda distribution: https://www.anaconda.com/download/#linux 
- Right click on the chosen Python version, select '**copy link address**' then type ``wget`` function in the terminal PuTTy followed by the copied link address:    
```
$ wget https://repo.continuum.io/archive/Anaconda3-5.2.0-Linux-x86_64.sh
```


## 5. bash to run the downloaded .sh file 
```
$ bash Anaconda3-5.2.0-Linux-x86_64.sh
```


## 6. Check which python version is used
The goal is just to confirm whether you are using the one from Anaconda Distribution or not.
```
$ which python
```

## 7. Re-load your .bashrc
The above command lists the python that your system currently uses. If it does not mentions the one from ``".../anacondaX/..."`` folder, 
then you can use the following command to re-load your .bashrc, so as to set the correct python:
```
$ source .bashrc
```

## 8. iPython Terminal 
Open the iPython Terminal to get an encrypted password so as to use it for logging into our iPython Notebook Server. 
Remember to copy and save the output of this command, which will be an encrypted password, something like "sha1:..."
```
$ ipython
```

> 
```
In [1]: from IPython.lib import passwd
In [2]: passwd()
sha1:592a57cc3224:f190f1a25eb5f878e329f5...
```

## 9. Exit out from the iPython Terminal using "exit" command
> 
```
In [3]: exit
```

## 10. Create the configuration profile for your jupyter notebook server
To configure the Jupyter Notebook server on your EC2 instance, you should create a configuration file. In the configuration file, you set some of the values to use for web authentication, including the SSL certificate file path, and a password.
```
$ jupyter notebook --generate-config
```

This command creates a configuration file (jupyter_notebook_config.py) in the ``~/.jupyter directory``.


## 11. SSL certificate 
- Create a self-signed (SSL certificate) certificate for accessing to your Notebooks through HTTPS into a folder named as example 'certs'
- Connect to the EC2 instance then complete the following procedure. If you have set up a cluster of EC2 instances, connect to the master node.
```
$ mkdir certs
$ cd certs``
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mycert.key -out mycert.pem``
$ sudo chmod 400 mycert.pem
```

## 12. Configure FoxyProxy for Google Chrome
> **Note:** this step is essential in some cases for the smooth running of jupyter notebook servers

You can configure FoxyProxy for Google Chrome, Mozilla Firefox, and Microsoft Internet Explorer. 
FoxyProxy provides a set of proxy management tools that allow you to use a proxy server for URLs 
that match patterns corresponding to the domains used by the Amazon EC2 instances in your Amazon EMR cluster.
For more details look at the following website link: 
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-proxy.html


## 13. Update the configuration file to store your password and SSL certificate information
```
$ cd ~/.jupyter/
```
Open the .config file:
```
$ vi jupyter_notebook_config.py
```
Paste the following text at the beginning or the end of the ``jupyter_notebook_config.py`` file and leave the rest commented. You will need to provide your hash password.

```
c = get_config()
# Kernel config
c.IPKernelApp.pylab = 'inline' # if you want plotting support always in your notebook
# Notebook config
c.NotebookApp.certfile = u'/home/ubuntu/certs/mycert.pem' # location of your certificate file
c.NotebookApp.keyfile = u'/home/ubuntu/certs/mycert.key' # location of your certificate key
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False  # so that the ipython notebook does not open a browser by default
c.NotebookApp.password = u'sha1:68c136a5b064...' # the encrypted password you generated above
```

## 14. Update the putty parameters to access the EC2 instance
- Open a second session to the same EC2 instance using PuTTY. 
- In the Category pane, expand Connection, expand SSH, and then choose Tunnels. 
- Complete the following to add new forwarded port:
```
Source port = 8157
Select 'Dynamic'
Keep 'Auto'
Choose 'Add'
```

## 15. Start your Jupyter notebook server 
It's time to start your Jupyter notebook server. For this, you can create a new folder which will store all your notebooks
```
$ cd ~
$ mkdir Notebooks
$ cd Notebooks
```

Start your notebook server
```
$ jupyter notebook
```

## 16. Access to your notebooks using the web interfaces
- To open the web interfaces, in your browser's address bar, type master-public-dns followed by the port number or URL. You can access to your Notebook from anywhere through your browser. Just navigate to the DNS name, or Public IP of your instance, along with the port number (*https://Public_DNS_name:8888/*). 
- Remember to update your URL to "https" because the browser by default adds "http" to such a URL.
- You will be asked by your browser to trust the certificate, as you have signed it on your own, so you know that you can trust it.
- Login using the password you specified when you used the iPython Terminal to create an encrypted version.
