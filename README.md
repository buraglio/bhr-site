BHR Site
========

This is the BHR site / API endpoint.

It does not make any policy decisions except for the block duration auto scaling.

It basically acts as a message queue between bhr clients adding blocks, and bhr
clients implementing blocks.

Blocks flow through the system like so:

* A BHR Client calls `block(cidr='192.168.254.254', source='readme', why='because!', duration=300)`
* This entry is now in the system but not marked as blocked.
* A BHR Client calls `get_block_queue()` which will return a list containing that record
* That BHR Client will then add a firewall rule, bgp entry, whatever
* That BHR Client calls `set_blocked` and marks it as blocked

300 seconds pass

* A BHR client calls `get_unblock_queue` which returns a list containing that record
* That BHR client will remove the firewall rule, bgp entry, whatever
* That BHR client calls `set_unblocked` and marks it as unblocked

Bhr clients have an 'ident' associated with them, and blocks/unblocks are
tracked per ident.  This enables a single BHR system to be used to apply blocks
across multiple backend systems.

Configuration
=============

Create `bhr_site/settings_local.py` with something like:

    LOCAL_SETTINGS = True  # do not touch
    from settings import * # do not touch

    DEBUG = False
    ALLOWED_HOSTS = ['bhr.example.com', 'bhr']

    STATIC_ROOT="/home/bhr/static"

    ADMINS = (("You", "root@localhost"), )

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'bhr',
        }
    }

    BHR = {
        'time_multiplier':              2.0,
        'time_window_factor':           2.0,
        'minimum_time_window':          43200.0,
        'penalty_time_multiplier':      2.0,
        'return_to_base_multiplier':    2.0,
        'return_to_base_factor':        2.0,
    }

And configure apache similar to examples/apache.conf

Related projects
================

* [bhr-client](https://github.com/JustinAzoff/bhr-client) - BHR python client
* [bhr-client-exabgp](https://github.com/JustinAzoff/bhr-client-exabgp) - ExaBGP block manager
* [bhr-bro](https://github.com/JustinAzoff/bhr-bro) - Basic Bro integration for BHR
