# Creating backups of your data

CAUTION: Hot backups are currently not supported, you must stop the services before backing up. 

First stop the service. If using Upstart:

```
sudo stop ironmq
sudo stop ironauth
```

Then copy data files to another directory. Data files will be stored at `$HOME/iron/data` by default if you 
haven't changed the configs.  

After you've copied the files, you can start the services back up again:

```
sudo start ironauth
sudo start ironmq
```

Then archive the copy you made and transfer it to a secure location. 
