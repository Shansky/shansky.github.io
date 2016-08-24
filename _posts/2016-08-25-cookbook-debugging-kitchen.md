---
layout: post
title: Cookbooks debugging in kitchen
---

Sometimes you just need debug your cookbooks

## chef-shell

Easiest way to debug execution of your cookbook is using chef-shell.

To do so you run chef-zero as daemon on provisioned kitchen instance
(note: every command runs from ```/tmp/kitchen``` directory):

```bash
root@ubuntu-1404:/tmp/kitchen# /opt/chef/embedded/bin/chef-zero -d
```

Then you need to upload your cookbook to new instance of chef-zero:

```bash
root@ubuntu-1404:/tmp/kitchen# knife cookbook upload -a -c client.rb
Uploading debugbook    [0.1.0]
Uploaded all cookbooks.
```

Then just run chef-shell:

```bash
root@ubuntu-1404:/tmp/kitchen# chef-shell -z -c client.rb -o 'debugbook'
loading configuration: client.rb
Session type: client
Loading..[2016-08-24T23:12:35+00:00] WARN: Run List override has been provided.
[2016-08-24T23:12:35+00:00] WARN: Original Run List: []
[2016-08-24T23:12:35+00:00] WARN: Overridden Run List: [recipe[debugbook]]
resolving cookbooks for run list: ["debugbook"]
.Synchronizing Cookbooks:
  - debugbook (0.1.0)
done.

This is the chef-shell.
 Chef Version: 12.13.37
 http://www.chef.io/
 http://docs.chef.io/

run `help' for help, `exit' or ^D to quit.

Ohai2u vagrant@default-ubuntu-1404!
chef (12.13.37)>
```

### pry-remote

Or you can use harder way with pry-remote.

Define private network and port forwarding in kitchen so we can connect to pry server. In ```.kitchen.yml``` for your cookbook add network parameters:

```yaml
driver:
  name: vagrant
  network:
    - ['private_network', {ip: '33.33.33.33'}]
    - ['forwarded_port', {guest: 9876, host: 9876}]
```

In your recipe add directives to install pry-remote and set break point:

```ruby
chef_gem 'pry-remote'
require 'pry-remote'

binding.remote_pry '0.0.0.0'
```

Run your kitchen converge or test and you get something like this:

```bash
[...]
       Installing Cookbook Gems:
       Compiling Cookbooks...
       Recipe: debugbook::default
         * chef_gem[pry-remote] action install (up to date)
       [pry-remote] Waiting for client on druby://0.0.0.0:9876
```

Now in other shell just run:

```bash
pry-remote -s 33.33.33.33 -p 9876
```

And you're connected to pry session.

Example cookbook can be find here: [debugbook](https://github.com/Shansky/debugbook)