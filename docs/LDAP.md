## LDAP

If you wish to use SARK's LDAP support to provide directory lookups to your phones, proceed as follows:

### Check the LDAP Base Name (Base DN)

You can check by running slapcat as follows:

```sh
sudo slapcat
```

```sh
dn: dc=nodomain
objectClass: top
objectClass: dcObject
objectClass: organization
o: nodomain
dc: nodomain
```

The base name appears in the first line of output. In the above example, it is "dc=nodomain". We've used "nodomain" as an example because you'll likely see it a lot, particularly when you are spinning up LXC instances and the like. It just means that LDAP couldn't find the FQDN, probably because it wasn't set. It's no big deal, you can just go ahead and use dc=nodomain if you like. In any event, you should enter whatever YOUR base name is into the Globals=>LDAP panel along with the LDAP password you chose during the install. If "dc=nodomain" bothers you, then you'll need to enter a domain name in SARK, issue a commit and then do a reconfigure for slapd (the LDAP daemon):

```sh
sudo dpkg-reconfigure slapd
```

This will allow it to have another look.

### Adding the Contacts OU

Now we need to add an organizationalUnit (ou) for the address book. If you are already a whiz with LDAP, then just go ahead and do it, using the base name. If you don't know LDAP, then proceed as follows:

You will find a file on your SARK box at `/opt/sark/cache/ldapcontactou.ldif`:

```sh
dn: ou=contacts,dc=sark,dc=aelintra,dc=com
objectClass: organizationalUnit
objectClass: top
ou: contacts
```

Without getting into the details (you can read about LDAP elsewhere); in the first line of the file is the name of the organizationalUnit we want to create. In our example, it is "contacts". The remainder of the line (dc=sark,dc=aelintra,dc=com) is the base name we discussed earlier. You must edit the file to make the file match your base name from the slapcat you did a couple of steps back. If, for example, you have a base name of `dc=splodge,dc=soap,dc=com` then you should make your file look like this:

```sh
dn: ou=contacts,dc=splodge,dc=soap,dc=com
objectClass: organizationalUnit
objectClass: top
ou: contacts
```

Ok, let's add it. We are going to use the LDAP slapadd utility, which is old-fashioned nowadays, but easy to use:

```sh
sudo service slapd stop
sudo slapadd -l /opt/sark/cache/ldapcontactou.ldif
sudo service slapd start
```

Refresh your browser and navigate to SARK. You should see a "Directory" option when you click on the "Settings" drop-down. In this page, you can add and modify your telephone book entries or upload a vCard file you have exported from Google contacts or whatever. You can also have your phones browse your contacts if you have SIP phones which are LDAP aware (many are). You'll need to add a couple of entries to your SARK firewall for TCP ports 389 and 686 (restrict them to net:$LAN) to allow the phones to query the database - don't forget to restart the firewall.

### Set Up Your Phones to Use LDAP

Most major SIP Phone types can use LDAP; however, the implementation varies from type to type. Snom, Panasonic (KX-HDV), Yealink, and Fanvil all support LDAP and have on-board provisioning already set up for openLDAP on SARK. Cisco small business phones (SPA) support LDAP and Provu have a section on their website showing how to set them up [here](http://blog.provu.co.uk/item/234). Polycom phone set-up is more complex (isn't it always?), however, there is good documentation provided by Polycom so you should be able to get it running with a little work although we don't show it here. Gigaset professional (N series) phones support LDAP and there is documentation on their website.