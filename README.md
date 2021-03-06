# file

## Name

*file* - enables serving zone data from an RFC 1035-style master file with semi-authoritative options.

## Description

The *file* plugin is used for an "old-style" DNS server. It serves from a preloaded file that exists on disk. If the zone file contains signatures (i.e., is signed using DNSSEC), correct DNSSEC answers are returned. Only NSEC is supported! If you use this setup *you* are responsible for re-signing the zonefile.

Additionally, this modified plugin supports a semi-authoritative mode, where results not known by the nameserver are forwarded to the next chain, allowing for a horizon view if the record is not locally known for a particular record in a given zone.

## Syntax

~~~
file DBFILE [ZONES...]
~~~

* **DBFILE** the database file to read and parse. If the path is relative, the path from the *root*
  plugin will be prepended to it.
* **ZONES** zones it should be authoritative for. If empty, the zones from the configuration block are used.

If you want to round-robin A and AAAA responses look at the *loadbalance* plugin.

~~~
file DBFILE [ZONES... ] {
    reload DURATION
    fallthrough
}
~~~

* `reload` interval to perform a reload of the zone if the SOA version changes. Default is one minute.
  Value of `0` means to not scan for changes and reload. For example, `30s` checks the zonefile every 30 seconds
  and reloads the zone when serial changes.
* `fallthrough`, if specified will allow NXDOMAIN and other failures to be forwarded to the next plugin chain.

If you need outgoing zone transfers, take a look at the *transfer* plugin.

## Examples

Load the `example.org` zone from `example.org.signed` and allow transfers to the internet, but send
notifies to 10.240.1.1

~~~ corefile
example.org {
    file example.org.signed
    transfer {
        to * 10.240.1.1
    }
}
~~~

Or use a single zone file for multiple zones:

~~~ corefile
. {
    file example.org.signed example.org example.net
    transfer example.org example.net {
        to * 10.240.1.1
    }
}
~~~

Note that if you have a configuration like the following you may run into a problem of the origin
not being correctly recognized:

~~~ corefile
. {
    file db.example.org
}
~~~

We omit the origin for the file `db.example.org`, so this references the zone in the server block,
which, in this case, is the root zone. Any contents of `db.example.org` will then read with that
origin set; this may or may not do what you want.
It's better to be explicit here and specify the correct origin. This can be done in two ways:

~~~ corefile
. {
    file db.example.org example.org
}
~~~

Or

~~~ corefile
example.org {
    file db.example.org
}
~~~

## See Also

See the *loadbalance* plugin if you need simple record shuffling. And the *transfer* plugin for zone
transfers. Lastly the *root* plugin can help you specificy the location of the zone files.
