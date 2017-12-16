# puppet-st2

[![Build Status](https://travis-ci.org/StackStorm/puppet-st2.svg)](https://travis-ci.org/StackStorm/puppet-st2)
[![Coverage Status](https://coveralls.io/repos/StackStorm/puppet-st2/badge.svg?branch=master&service=github)](https://coveralls.io/github/StackStorm/puppet-st2?branch=master)
[![Puppet Forge Version](https://img.shields.io/puppetforge/v/stackstorm/st2.svg)](https://forge.puppet.com/stackstorm/st2)
[![Puppet Forge Downloads](https://img.shields.io/puppetforge/dt/stackstorm/st2.svg)](https://forge.puppet.com/stackstorm/st2)
[![Join our community Slack](https://stackstorm-community.herokuapp.com/badge.svg)](https://stackstorm.com/community-signup)

Module to manage [StackStorm](http://stackstorm.com)

## Supported platforms

* Ubuntu 14.04/16.04
* RHEL/Centos 6/7

## Quick Start

For a full installation on a single node, a profile already exists to
get you setup and going with minimal effort. Simply:

```
include ::st2::profile::fullinstall
```

## Configuration

This module aims to provide sane default configurations, but also stay
out of your way in the event you need something more custom. To accomplish
this, this module uses the Roles/Profiles pattern. Included in this module
are several modules that come with sane defaults that you can use directly
or use to compose your own site-specific profile for StackStorm installation.

Configuration can be done directly via code composition, or set via
Hiera data bindings. A few notable parameters to take note of:

* `st2::version` - Version of ST2 to install. (Latest version w/o value)

All other classes are documented with Puppetdoc. Please refer to specific
classes for use and configuration.

### Profiles:

* `st2::profile::client` - Profile to install all client libraries for st2
* `st2::profile::fullinstall` - Full installation of StackStorm and dependencies
* `st2::profile::mistral` - Install of OpenStack Mistral
* `st2::profile::mongodb` - st2 configured MongoDB installation
* `st2::profile::nodejs` - st2 configured NodeJS installation
* `st2::profile::python` - Python installed and configured for st2
* `st2::profile::rabbitmq` - st2 configured RabbitMQ installation
* `st2::proflle::server` - st2 server components
* `st2::profile::web` - st2 web components
* `st2::profile::chatops` - st2 chatops components

### Installing and configuring Packs

StackStorm packs can be installed and configured directly from Puppet. This
can be done via the `st2::pack` and `st2::pack::config` defined types.

Installation/Configuration via modules:

```ruby
  # install pack from the exchange
  st2::pack { 'linux': }
  
  # install pack from a git URL
  st2::pack { 'private':
    repo_url => 'https://private.domain.tld/git/stackstorm-private.git',
  }
  
  # install pack and apply configuration
  st2::pack { 'slack':
    config   => {
      'post_message_action' => {
        'webhook_url' => 'XXX',
      },
    },
  }
```

Installation/Configuration via Hiera:

```yaml
st2::packs:
  linux:
    ensure: present
  private:
    ensure: present
    repo_url: https://private.domain.tld/git/stackstorm-private.git
  slack:
    ensure: present
    config:
      post_message_action:
        webhook_url: XXX
```

### Configuring Hubot (ChatOps)

Configuration via Hiera:

```yaml
  # install and configure hubot adapter (rocketchat, nodejs module installed by ::nodejs)
  st2::chatops_adapter:
    hubot-adapter:
      package: 'hubot-rocketchat'
      source: 'git+ssh://git@git.company.com:npm/hubot-rocketchat#master'

  # adapter configuration (hash)
  st2::chatops_adapter_conf:
    HUBOT_ADAPTER: rocketchat
    ROCKETCHAT_URL: "https://chat.company.com:443"
    ROCKETCHAT_ROOM: 'stackstorm'
    LISTEN_ON_ALL_PUBLIC: true
    ROCKETCHAT_USER: st2
    ROCKETCHAT_PASSWORD: secret123
    ROCKETCHAT_AUTH: password
    RESPOND_TO_DM: true
```

## Module Dependencies

This module installs and configures all of the components required for StackStorm.
In order to not repeat others work, we've utilized many existing modules from the
foge. We manage the module dependenies using a `Puppetfile` for each OS we support.
These `Puppetfile` can be used both with [r10k](https://github.com/puppetlabs/r10k)
and [librarian-puppet](http://librarian-puppet.com/).

### Puppetfiles

 * RHEL/CentOS 6 - [build/centos6/Puppetfile](build/centos6/Puppetfile)
 * RHEL/CentOS 7 - [build/centos7/Puppetfile](build/centos7/Puppetfile)
 * Puppet 4.0 - [build/puppet4/Puppetfile](build/puppet4/Puppetfile)
 * Puppet 5.0 - [build/puppet5/Puppetfile](build/puppet5/Puppetfile)
 * Ubuntu 14.04 - [build/ubuntu14/Puppetfile](build/ubuntu14/Puppetfile)
 * Ubuntu 16.06 [build/ubuntu16/Puppetfile](build/ubuntu16/Puppetfile)

## Known Limitations

### MongoDB (Puppet < 4.0)

When running the initial install of `st2` you will see an error from the 
MongoDB module :

```
Error: Could not prefetch mongodb_database provider 'mongodb': Could not evaluate MongoDB shell command: load('/root/.mongorc.js'); printjson(db.getMongo().getDBs())
```

This error is caused by a deficiency in this module trying to use authentication
in its prefetch step when authentication hasn't been configured yet on
the database. The error can be safely ignored. Auth and databases will be 
configured normally. Subsequent runs of puppet will not show this error.

### MongoDB (Puppet >= 4.0)

When running the initial install of `st2` you will see an error from the 
MongoDB module :

```
Error: Could not prefetch mongodb_database provider 'mongodb': Could not evaluate MongoDB shell command: load('/root/.mongorc.js'); printjson(db.getMongo().getDBs())
```

This error is caused by a deficiency in this module trying to use authentication
in its prefetch step when authentication hasn't been configured yet on
the database. This results in a failure and stops processing.

In these cases we need to disable auth for MongoDB using the `mondob_auth` variabe.
This can be accomplished when declaring the `::st2` class:

``` puppet
class { '::st2':
  mongodb_auth => false,
}
```

Or in hiera:

``` yaml
st2:
  mongodb_auth: false
```

### Ubuntu 14.04

Because 14.04 ships with a very old version of puppet (3.4) and most puppet modules
no longer support this version of puppet, we recommend upgrading to 3.8.7 at a
minimum.

``` shell
# 14.04 trusty
# By default this ships with puppet 3.4.x (very old), need a newer version to 
# work with with of the required puppet module dependencies for this module. 
wget https://apt.puppetlabs.com/puppetlabs-release-trusty.deb
sudo dpkg -i puppetlabs-release-trusty.deb
sudo apt-get update
sudo apt-get install puppet=3.8.7-1puppetlabs1
```

### Ubuntu 16.04

In StackStorm < `2.4.0` there is a known bug [#3290](https://github.com/StackStorm/st2/issues/3290) 
that when first running the installation with this puppet module the `st2` pack
will fail to install. Simply invoking puppet a second time will produce
a fully running st2 installation with the `st2` pack installed. This has
been fixed in st2 version `2.4.0`.


## Maintainers

* Nick Maludy 
  * GitHub - [@nmaludy](https://github.com/nmaludy)
  * Email - <nick.maludy@encore.tech>
* StackStorm <info@stackstorm.com>
* James Fryman
* Patrick Hoolboom

## Help

If you're in stuck, our community always ready to help, feel free to:
* Ask questions in our [public Slack channel](https://stackstorm.com/community-signup) in channel `#puppet`
* [Report bug](https://github.com/StackStorm/puppet-st2/issues), provide [feature request](https://github.com/StackStorm/puppet-st2/pulls) or just give us a ✮ star

Your contribution is more than welcome!
