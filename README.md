Description
===========

Provides a set of primitives for managing firewalls and associated rules.

PLEASE NOTE - The resource/providers in this cookbook are under heavy development.
An attempt is being made to keep the resource simple/stupid by starting with less
sophisticated firewall implementations first and refactor/vet the resource definition
with each successive provider.

Requirements
============

Platform
--------

* Ubuntu
* Debian

Tested on:

* Ubuntu 10.04
* Ubuntu 11.04
* Ubuntu 11.10
* Debian 7.0

Resources/Providers
===================

`firewall`
----------

### Actions

- :enable: *Default action* enable the firewall.  this will make any rules that have been defined 'active'.
- :disable: disable the firewall. drop any rules and put the node in an unprotected state.

### Attribute Parameters

- name: name attribute. arbitrary name to uniquely identify this resource
- log_level: level of verbosity the firewall should log at. valid values are: :low, :medium, :high, :full. default is :low.

### Providers

- `Chef::Provider::FirewallUfw`
    - platform default: Ubuntu

### Examples

    # enable platform default firewall
    firewall "ufw" do
      action :enable
    end

    # increase logging past default of 'low'
    firewall "debug firewalls" do
      log_level :high
      action :enable
    end

`firewall_rule`
---------------

### Actions

- :allow: the rule should allow incoming traffic.
- :deny: the rule should deny incoming traffic.
- :reject: *Default action: the rule should reject incoming traffic.

### Attribute Parameters

- name: name attribute. arbitrary name to uniquely identify this firewall rule
- protocol: valid values are: :udp, :tcp. default is all protocols
- port: incoming port number (ie. 22 to allow inbound SSH)
- ports: array of incoming port numbers (ie. [80,443] to allow inbound HTTP & HTTPS). NOTE: `protocol` attribute is required with `ports`
- port_range: range of incoming port numbers (ie. 60000..61000 to allow inbound mobile-shell. NOTE: `protocol` attribute is required with `port_range`
- source: ip address or subnet to filter on incoming traffic. default is `0.0.0.0/0` (ie Anywhere)
- destination: ip address or subnet to filter on outgoing traffic.
- dest_port: outgoing port number.
- position: position to insert rule at. if not provided rule is inserted at the end of the rule list.
- direction: direction of the rule. valid values are: :in, :out, default is :in
- interface: interface to apply rule (ie. 'eth0').
- logging: may be added to enable logging for a particular rule. valid values are: :connections, :packets. In the ufw provider, :connections logs new connections while :packets logs all packets.

### Providers

- `Chef::Provider::FirewallRuleUfw`
    - platform default: Ubuntu

### Examples

    # open standard ssh port, enable firewall
    firewall_rule "ssh" do
      port 22
      action :allow
      notifies :enable, "firewall[ufw]"
    end

    # open standard http port to tcp traffic only; insert as first rule
    firewall_rule "http" do
      port 80
      protocol :tcp
      position 1
      action :allow
    end

    # restrict port 13579 to 10.0.111.0/24 on eth0
    firewall_rule "myapplication" do
      port 13579
      source '10.0.111.0/24'
      direction :in
      interface 'eth0'
      action :allow
    end

    # open UDP ports 60000..61000 for mobile shell (mosh.mit.edu), note
    # that the protocol attribute is required when using port_range
    firewall_rule "mosh" do
      protocol :udp
      port_range 60000..61000
      action :allow
    end

    # open multiple ports for http/https, note that the protocol
    # attribute is required when using ports
    firewall_rule "http/https" do
      protocol :tcp
      ports [ 80, 443 ]
      action :allow
    end

    firewall "ufw" do
      action :nothing
    end

License and Author
==================

Author:: Seth Chisamore (<schisamo@opscode.com>)

Copyright:: Copyright (c) 2011 Opscode, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
