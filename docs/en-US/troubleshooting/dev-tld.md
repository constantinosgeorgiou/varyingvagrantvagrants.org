---
category: 5. Troubleshooting
order: 2
title: .dev domains
description: Issues with .dev sites
permalink: /docs/en-US/troubleshooting/dev-tld/
---

A lot of people use the `.dev` domain for local development, but this top-level domain (TLD) is actually owned by Google, and isn't free for development. They've begun enforcing special rules on sites using this domain when using Chrome. You also run the risk of clashes with Google owned domains.

The fix is simple, switch to `.test`, here, you'll find instructions on converting existing sites over. For this reason, the dashboard supports `http://vvv.test`

## Why not about `.local` or `.localhost`?

The `.test`, `.local`, and `.localhost` domains are all protected by standards documented in RFCs (Request for Comments) and cannot be bought as `.dev` was by Google. However, they have specific purposes and strings attached.

 - `.local` is used by zeroconf and bonjour to make computers on a local network easier to use. For example, if I name my machine `vvv`, other computers can reach it at `vvv.local`. This might stop you reaching VVV's dashboard however. You can use `.local` for VVV sites but it is not recommended.
 - `.localhost` can only by used for local loopback. This means you could never share the site with other computers on the same network for testing, so they can't be reliably sued for VVV.
- `.test` domains can never be registered, and are intended for testing purposes. The RFC prevents a company buying and running a registrar for `.test` domains, but does not place any restrictions on how it's intended to be used. This means you can use them for local development, or even for remote servers, but you have to define this yourself. ***This makes `.test` domains ideal for VVV***

## How To Migrate to `.test`

### Change The Hosts VVV uses

Modify your VVV config in `config/config.yml` by replacing the hosts for each site with `.test` versions. Once this is done, reprovision using `vagrant reload --provision` for the change to take effect.

This will make VVV serve the site at the correct domain, and update the dashboard site list

#### Custom Provisioners

If you've forked the custom site template, or modified the nginx/provisioner scripts, you should check these to make sure it doesn't install or try to serve `.dev` in the future.

### Search Replace

WordPress will have been installed with `.dev` in the site URL, so you need to use WP CLI to do a [search replace](https://developer.wordpress.org/cli/commands/search-replace/).

1. SSH into VVV using `vagrant ssh`
2.  `cd` into the `/srv/www` folder, then `cd` into the subfolder with your site. This should be the same as your `www` folder.
3. Enter the `public_html` subfolder so that we can use WP CLI
4. Run `wp search-replace '.dev' '.test' --recurse-objects`, adding ` --network` on the end if it's a multisite installation.
5. Run `exit` to exit the VM and return the terminal back to normal

If you've hardcoded the `.dev` domain in your plugins or themes, or `wp-config.php`, those will need manually changing

#### Example: Search Replacing `local.wordpress.dev` to `local.wordpress.test`

These are the commands that would change the default site over from `.dev` to `.test`:

First SSH into a running VVV instance  with `vagrant ssh`. Then we would run these commands inside the VM:

```sh
# change to the main WWW folder
cd /srv/www

# enter our sites folder
cd wordpress-default/public_html

# Run a WP CLI command to change the URL ( an SQL search replace would break WordPress )
wp search-replace '.dev' '.test' --recurse-objects

# Exit the VM
exit
```

#### Multisites

If you're using a multisite install, you need to add ` --network` on to the end of the search replace command so that it runs on all tables, not just those of the first site. Otherwise the instructions are identical, for example:

First SSH into a running VVV instance  with `vagrant ssh`. Then we would run these commands inside the VM:

```sh
# change to the main WWW folder
cd /srv/www

# enter our sites folder
cd wordpress-default/public_html

# Run a WP CLI command to change the URL ( an SQL search replace would break WordPress )
# Note the --network parameter
wp search-replace '.dev' '.test' --recurse-objects --network
exit
```

If you've already ran the commands without the network flag, you can run the process again without breaking things.

## Migrating from `.local`

The steps are identical to those for `.dev`, except, the search replace command swaps `.dev` for `.local`:

```sh
wp search-replace '.local' '.test' --recurse-objects
```

## Post Migration Troubleshooting

If after you've done this, you're getting a white screen of death, consider turning on XDebug or checking the PHP error logs of that site. If you're using a VVV custom site template, or one of the default sites, they should be in a `logs` subfolder adjacent to `public_html`.

Also keep in mind that some plugins and themes do unusual things, such as storing hashes of URLs. These will need adjusting manually.

### Sites Created With `vv`

The `vv` tool bypasses the normal flow, and reaches into VVV and places its own non-standard Nginx configs inside the virtual machine. If you can't migrate to a custom site template, you will need to SSH into the virtual machine using `vagrant ssh`, and modify the Nginx configs for `vv` sites manually. These can be found in `/etc/nginx` using vim or nano, with the filenames `~sitename~.conf` where `~sitename~` is the name of the `vv` site.
