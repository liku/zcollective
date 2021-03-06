# ZCollective

ZCollective is a tool used to configure Zabbix using data discovered using MCollective.

At [The Scale Factory](http://www.scalefactory.com/), we've used Zabbix for monitoring for some time.  Our main complaint with this software has been that configuration has always been a bit too GUI-focussed.  This has been addressed in recent releases with an improvement in the functionality exposed via the API.

We've written ZCollective to remove some of the GUI legwork required in configuring monitoring for individual hosts - we think it's a cleaner alternative to Zabbix host discovery.

## How does it work?

First of all, ZCollective connects to the Zabbix API to fetch a list of templates that Zabbix knows about.  These have been constructed in the usual way inside the Zabbix UI, and given the same name as the configuration management class they support.

It then pulls a list of hosts, interfaces and templates for each host configured in Zabbix.

Then it uses MCollective to build a view of the hosts on your infrastructure.  For each host, it fetches a name, some IP address details and a list of configuration management classes associated with the node.

Hosts that are found by MCollective but are not monitored will be added to Zabbix, and linked to any templates that match configuration management class names.

Hosts found in Zabbix but not by MCollective are left alone, but reported on.

Hosts found in both Zabbix and MCollective will be linked to any missing templates.  IP address mismatches for the hostname are reported on, but no changes are made.

## Usage

```
Usage: zcollective [options]
        --zabbix-api-url url         JSON-RPC endpoint for zabbix server
        --zabbix-user user           Zabbix API username
        --zabbix-pass pass           Zabbix API password
        --debug                      Enable debugging
        --noop                       Don't make changes
        --interface-cidr CIDR        Only consider interfaces matching the given CIDR
```

The URL, username and password options are self-explanatory.

```--interface-CIDR```, if passed will filter the interfaces MCollective finds by CIDR range.  This is useful for multihomed hosts where you only want to monitor the interface on an administrative network.

```--debug``` will generate a lot of console noise, but helps with working out what's going on.

Passing ```--noop``` will report on the changes to be made, but not make any.




## Example

A run against a fresh Zabbix install on a modest infrastructure, using the default username/password:

```
$ ./zcollective.rb --zabbix-api-url http://zabbix/zabbix/api_jsonrpc.php --interface-cidr 10.0.44.0/24 
I, [2012-11-21T16:43:32.899696 #8727]  INFO -- : Host ljn-live-db2 found by mcollective but not in zabbix
I, [2012-11-21T16:43:32.986700 #8727]  INFO -- : Host ljn-live-db2 added as ID 100100000010157 with 1 templates
I, [2012-11-21T16:43:32.986854 #8727]  INFO -- : Host ljn-live-db1 found by mcollective but not in zabbix
I, [2012-11-21T16:43:33.103230 #8727]  INFO -- : Host ljn-live-db1 added as ID 100100000010158 with 1 templates
I, [2012-11-21T16:43:33.103389 #8727]  INFO -- : Host ljn-core2 found by mcollective but not in zabbix
I, [2012-11-21T16:43:33.456250 #8727]  INFO -- : Host ljn-core2 added as ID 100100000010159 with 5 templates
I, [2012-11-21T16:43:33.456342 #8727]  INFO -- : Host ljn-core1 found by mcollective but not in zabbix
I, [2012-11-21T16:43:33.823787 #8727]  INFO -- : Host ljn-core1 added as ID 100100000010160 with 5 templates
I, [2012-11-21T16:43:33.823875 #8727]  INFO -- : Host ljn-log1 found by mcollective but not in zabbix
I, [2012-11-21T16:43:33.997864 #8727]  INFO -- : Host ljn-log1 added as ID 100100000010161 with 2 templates
I, [2012-11-21T16:43:33.998029 #8727]  INFO -- : Host ljn-live-app2 found by mcollective but not in zabbix
I, [2012-11-21T16:43:34.316868 #8727]  INFO -- : Host ljn-live-app2 added as ID 100100000010162 with 4 templates
I, [2012-11-21T16:43:34.317034 #8727]  INFO -- : Host ljn-live-app1 found by mcollective but not in zabbix
I, [2012-11-21T16:43:34.654262 #8727]  INFO -- : Host ljn-live-app1 added as ID 100100000010163 with 4 templates
```

A second run against the same infrastructure returns nothing on the commandline, as nothing is modified.

If we remove a couple of templates from ljn-live-app1 and run again:

```
$ ./zcollective.rb --zabbix-api-url http://zabbix/zabbix/api_jsonrpc.php --interface-cidr 10.0.44.0/24 
I, [2012-11-21T16:48:59.915380 #10272]  INFO -- : Zabbix ljn-live-app1 not linked to a template for sf_varnish
I, [2012-11-21T16:48:59.915588 #10272]  INFO -- : Zabbix ljn-live-app1 not linked to a template for sf_apache
I, [2012-11-21T16:49:00.184213 #10272]  INFO -- : Added missing templates to ljn-live-app1
```

## Assumptions

Because we wrote this to scratch our own itch, we've made assumptions that may only hold for a Scale Factory zabbix setup.

We assume that your Zabbix hosts are named the same as the hostname of the monitored server - so ```ljn-live-web1.scalefactory.net``` will be referred to as ```ljn-live-web1``` by Zabbix.

## Requirements

You need to be runing Zabbix >=2.0,  MCollective >=2.2, and Facter >=1.6.14.  You'll also need the netaddr Ruby Gem.

## "Missing features"

ZCollective currently doesn't know how to cope with hosts that are only visible to Zabbix via a proxy.

Giving a password on the commandline isn't very secure.

ZCollective has only been tested against a small number of use cases - use it at your own risk.

