# NetLib

[![Build Status](https://travis-ci.org/netopsio/netlib.svg)](https://travis-ci.org/netopsio/netlib)

Netlib is an attempt at re-writing
['pyRouterLib'](https://github.com/jtdub/pyRouterLib). The goal is to create a
library that is much more efficient and easier to use to establish SSH and Telnet
connections to network devices, such as routers and switches.

## Install

Install the Python library.

```
git clone https://github.com/jtdub/netlib.git
cd netlib

# For Python2 Support:
sudo python setup.py install

# For Python3 Support:
sudo python3 setup.py install
```

## Access via Telnet and SSH

Currently, the SSH and Telnet modules have been created. Both modules have a
very similar API structure, with the biggest difference between the two are how
connections are established to network devices.

To use either the SSH or Telnet module, you need to import the library into your script:

```
from netlib.conn_type import SSH
from netlib.conn_type import Telnet
```

From there, you define your connection parameters:

```
telnet = Telnet('somerouter', 'username', 'password')
ssh = SSH('somerouter', 'username', 'password')
```

Once the basic parameters have been set, you establish a connection to the
device.

```
telnet.connect()
ssh.connect()
```

Once you are connected, you are free to send commands to you network device. If
you intend to iterate through output that is long, then you can disable paging
on the output.

```
telnet.disable_paging()
ssh.disable_paging()

telnet.command('show version')
ssh.command('show version')
```

If you need to enter a privileged mode, you can use the set_enable api.

```
telnet.set_enable('supersecretpassword')
ssh.set_enable('supersecretpassword')
```

When you've completed your task on the device, you can close your connections.

```
telnet.close()
ssh.close()
```

At this point, those are the features that both libraries share. As the telnet
library and ssh library vary on how they parse data, there is a need for extra
functionality on the ssh library. Here is the API functionality that is
specific to the ssh library:

```
ssh.clear_buffer()
```

The SSH library stores output into a buffer. Sometimes this buffer can present
results that aren't expected. Clearing the buffer should mitigate the
unexpected results.

## User Credentials

When working with a large number of devices, it's inconvenient to have to type
your credentials in a large number of times and storing your credentials
directly into a script can be insecure. Therefore, I created a library that
allows you to store them, securely utilizing the 'keyring' python library.
'keyring' utilizes your operating systems native method for storing passwords.
For example, in MacOS X, the Keychain utility is utilized.

The past methods of password storage, simple_creds and simple_yaml have been depricated
and removed from netlib.

Utilizing the keyring is simple. Import the module:

```
>>> from netlib.user_keyring import KeyRing
```

Asign it to a variable, calling your username:
```
>>> user = KeyRing(username='jtdub')
```

If there are no credentials for the keyring, the get_creds() method will call the set_creds()
method:
```
>>> user.get_creds()
No credentials keyring exist. Creating new credentials.
Enter your user password: 
Confirm your user password: 
Enter your enable password: 
Confirm your enable password: 
```

Otherwise, the creds will be pulled from the keyring:
```
>>> user.get_creds()
{'username': 'jtdub', 'enable': u'enablepass', 'password': u'testpass'}
```

The set_creds() method can be called directly. It will over-write existing creds if they exist:
```
>>> user.set_creds()
Enter your user password: 
Confirm your user password: 
Enter your enable password: 
Confirm your enable password: 
>>> user.get_creds()
{'username': 'jtdub', 'enable': u'newenable', 'password': u'newpass'}
```

Of course, the keyring can be deleted all together utilizing the del_creds() method:
```
>>> user.del_creds()
Enter your user password: 
Deleting keyring credentials for jtdub
>>> 
```
