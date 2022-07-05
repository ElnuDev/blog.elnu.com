---
title: "Automatically Update Matomo Excluded IPs"
date: 2022-02-28
tags:
  - programming
description: "Going into Matomo to manually update excluded IPs to prevent your own visits from being counted is a pain. Learn how to write a command that updates them so you don’t have to!"
---

> :warning: Edited from [my forum posts on Matomo forums](https://forum.matomo.org/t/cant-find-global-website-settings-global-excluded-ips-etc-in-database-nor-configuration-files/44888)

I have a dynamic IP address, and every time it changes I had to manually go into [Matomo](https://matomo.org/) (a free and open-source self-hosted Google Analytics alternative that is endorsed by the EU and has [<abbr title="General Data Protection Regulation">GDPR</abbr>](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation)-compliant privacy tools), check to see if it logged any of my own visits, delete them in the GDPR tools, and update my IP in the general website settings. It was moderately annoying to do, and after doing this every once and a while for a year I wanted to find a way to automate this.

Unfortunately, I couldn't find where the global list of excluded IPs field is in neither the PHP configuration files nor the [Matomo database schema](https://developer.matomo.org/guides/database-schema).

However, I managed to find a workaround. While I couldn't find the global excluded IP field in any of the database tables, I was able to find the site-specific `excluded_ips` column in the `sites` table. Instead of using the global excluded IP field, I decided to make a script that automatically sets the excluded IPs for all of the available sites at once.

Here's what I did if anyone else encounters the same issue.

I made the following template SQL script, `excluded_ips_updater.sql`, that excludes all visits from an empty wildcard field, `{}`. This is where I format in my public IP address later. (In my database, all of the table names are prefixed with `matomo_`, but this might not be the case for you.)

```SQL
USE matomo;
UPDATE matomo_site SET excluded_ips="{}";
```

Then, in order for this to work, I run the following Bash command. What this does is `curl`s the contents of the [My IP API](https://api.my-ip.io/ip) to get our public IP address (the `-s` flag makes `curl` silent), replaces the `{}` in the SQL script with the public IP, and finally pipes into MySQL using my login credentials. Not that there is **no space** between the `-p` flag and your password.

```SH
sed "s/{}/$(curl -s https://api.my-ip.io/ip)/g" excluded_ips_updater.sql | mysql -u matomo -pmatomoUserPassword
```

And that's done! One can get this to run automatically on boot or every X amount of time using a cronjob or similar.

I'd still like to know whether or not there's a way to do this using the global excluded IPs list as it would be a lot more elegant, but this works for now. I hope this might be useful to anyone who ran into the same problem as I did.
