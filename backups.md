# Creating backups of your data

CAUTION: Hot backups are currently not supported, you must stop the services before backing up. 

## 1. Stop services

First stop the service. If using Upstart:

```
sudo stop ironmq
sudo stop ironauth
```

## 2. Copy data directory

Make a copy of the data directory to another directory. Data files will be stored at `$HOME/iron/data` by default if you 
haven't changed the configs.  

## 3. Start services

After you've copied the files, you can start the services back up again:

```
sudo start ironauth
sudo start ironmq
```

## 4. Archive copy to long term storage

Zip/tar the copy you made, then transfer it to a secure location. 
