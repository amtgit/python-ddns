python-ddns
===========

##A simple dynamic DNS howto using BIND9's DNSSEC update system and python.

1. Needed: linux machine with bind9, python and a web server with a cgi-bin folder setup. This how-to assumes you are using a Debian-based distro such as Debian or Ubuntu.

2. Generate keys needed for nsupdate. Change dynamic.mydomain.com to whatever you want. Making them be the same is easiest.

        $ dnssec-keygen -a HMAC-MD5 -b 512 -n USER dynamic.mydomain.com.

 This will create two files something like: Kdynamic.mydomain.com.+157+12505.[key|private]

3. Move the files to /etc/dyndns, or anywhere. Set the permissions or ownership so that the web server process can read them. You could do this:

        sudo mkdir /etc/dyndns
        sudo chown root.www-data /etc/dyndns
        sudo chmod 660 /etc/dyndns

4. In your bind config file (/etc/bind/named.conf.local or wherever you define your mydomain.com zone) add this into the zone definition:

        allow-update {
            key dynamic.mydomain.com.;
        };

 So for example, the whole zone entry might look like this:

        zone "mydomain.com." {
            type master;
            allow-transfer { 1.2.3.4;};
            file "/etc/bind/pri.mydomain.com";
            allow-update {
                key dynamic.mydomain.com.;
            };
        }

5. Create a file in /etc/bind/ called mydomain.com.keys.conf. The secret is from the "private" file created in step 2.

        key dynamic.mydomain.com. {
            algorithm HMAC-MD5;
            secret "kQQvmdkTk8l6uMYfdnIBn5Ys7uhJFvlqSD73WNzEMX42kYZ4H0cuyykTjHY7mB8N9nnZL0gpA9LJmepPYKaU0g==";
        };

6. Somewhere in named.conf.local, also add a line like this:

        include "/etc/bind/mydomain.com.keys.conf";

7. Bind needs to be able to write to the zone files. You may need to update permissions if you are storing your zone files in /etc/bind.

        sudo chown root.bind /etc/bind
        sudo chown 775 /etc/bind

8. Create the cgi script in /usr/lib/cgi-bin/dyndns, or wherever your apache configuration puts the cgi-bin folder at. The package "sh" might not be installed. Go to http://www.pip-installer.org to install pip, and install sh with "sudo pip install sh". You can rewrite it to use os.system if you prefer.

 Change the 1234 in the "adler32" function to some random number that only you know. It needs to match the number in the next step. This is just a secret used to verify the hash key and allow updates.

 Set it to be executable by the web server:

        sudo chown root.www-data /usr/lib/cgi-bin/dyndns
        sudo chmod 550 /usr/lib/cgi-bin/dyndns

9. You'll need python, and you may need to install some modules with pip

        sudo apt-get install python

 You can try:

        sudo apt-get install python-sh

 If that doesn't work you'll have to try:

        sudo apt-get install python-pip
        sudo pip install sh

10. Create getddkey in /usr/local/bin. Change the 1234 to some random number that only you know. This needs to match the number you changed in the previous step. Set this as executable

        sudo cp getddkey /usr/local/bin
        sudo chown root.root /usr/local/bin
        sudo chmod +x /usr/local/bin/getddkey

11. You can enable updates for any subdomain, such as x.mydomain.com or home.mydomain.com, by getting the hash key for it. To get the key, run getddkey and pass the subdomain name.

        $ getddkey x
        http://mydomain.com/cgi-bin/dyndns?name=x&key=38472837

 (Note that your keys should be different if you changed the 1234 to some other number.)

12. Leave the ip parameter off to use the IP you are calling from:

        http://mydomain.com/cgi-bin/dyndns?name=x&key=38472837&ip=10.0.0.1

 If you have https setup pointing to the same cgi-bin folder you can use that to be more secure.
