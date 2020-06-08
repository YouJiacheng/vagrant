# Vagrant::Goodhosts

This vagrant plugin adds host file entries to the host pointing to the guest VM, using the [GoodHosts](https://github.com/goodhosts/cli) cli tool. This plugin is based on [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater) and aims to be compatible with the same config parameters.

On **up**, **resume** and **reload** commands, it tries to add the hosts if they do not already exist in your hosts file. If it needs to be added, you will be asked for the `sudo` password to make the necessary edits.

On **halt**, **destroy**, and **suspend**, those entries will be removed again. By setting the `config.goodhosts.remove_on_suspend  = false`, **suspend** and **halt** will not remove them. 


## Installation

```shell
vagrant plugin install vagrant-goodhosts
```

To uninstall :

```shell
vagrant plugin uninstall vagrant-goodhosts
```

To update the plugin:

```shell
vagrant plugin update vagrant-goodhosts
```

## Usage

You currently only need the `hostname` and a `:private_network` network with a fixed IP address.

```ruby
config.vm.network :private_network, ip: "192.168.3.10"
config.vm.hostname = "www.testing.de"
config.goodhosts.aliases = ["alias.testing.de", "alias2.somedomain.com"]
```

This IP address and the hostname will be used for the entry in the `/etc/hosts` file.

### Multiple private network adapters

If you have multiple network adapters i.e.:

```ruby
config.vm.network :private_network, ip: "10.0.0.1"
config.vm.network :private_network, ip: "10.0.0.2"
```

You can specify which hostnames are bound to which IP by passing a hash mapping the IP of the network to an array of hostnames to create, e.g.:

```ruby
config.goodhosts.aliases = {
    '10.0.0.1' => ['foo.com', 'bar.com'],
    '10.0.0.2' => ['baz.com', 'bat.com']
}
```

This will produce `/etc/hosts` entries like so:

```
10.0.0.1 foo.com
10.0.0.1 bar.com
10.0.0.2 baz.com
10.0.0.2 bat.com
```

### Keeping Host Entries After Suspend/Halt

To keep your `/etc/hosts` file unchanged simply add the line below to your `VagrantFile`:

```ruby
config.goodhosts.remove_on_suspend = false
```

This disables `vagrant-goodhosts` from running on **suspend** and **halt**.


## Suppressing prompts for elevating privileges

These prompts exist to prevent anything that is being run by the user from inadvertently updating the hosts file. 
If you understand the risks that go with supressing them, here's how to do it.

### Linux/OS X: Passwordless sudo

To allow vagrant to automatically update the hosts file without asking for a sudo password, add one of the following snippets to a new sudoers file include, i.e. `sudo visudo -f /etc/sudoers.d/vagrant_goodhosts`.  
Change the cli path as printed in the vagrant output.

For Ubuntu and most Linux environments:

    %sudo ALL=(root) NOPASSWD: [the-path]

For MacOS:

    %admin ALL=(root) NOPASSWD: [the-path]

### Windows: UAC Prompt

You can use `cacls` or `icacls` to grant your user account permanent write permission to the system's hosts file. 
You have to open an elevated command prompt; hold `❖ Win` and press `X`, then choose "Command Prompt (Admin)"

    cacls %SYSTEMROOT%\system32\drivers\etc\hosts /E /G %USERNAME%:W 

## Installing The Development Version

If you would like to install `vagrant-goodhosts` to make contributions or changes, run the following commands::

```shell
git clone https://github.com/goodhosts/vagrant vagrant-goodhosts
cd vagrant-goodhosts
git checkout develop
./package.sh
vagrant plugin install vagrant-goodhosts-*.gem
```
