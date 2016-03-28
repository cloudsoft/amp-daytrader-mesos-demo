Public and private keys added to this directory are copied to the `/home/vagrant/.ssh/` directory on startup so as to be available to AMP Pro. For example if you use the `openstack` keypair to connect to your IBM Blue box tenant you should copy them as follows:

```
cp ~/.ssh/openstack.pem ~/.ssh/openstack.pub ./my_keys/
```

These keys will subsequently be available to AMP in the vagrant users `~/.ssh ` directory, for example `~/.ssh/openstack.pem` and `/home/vagrant/.ssh/openstack.pub`