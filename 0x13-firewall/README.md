0x13. Firewall ALX SE program tasks by Amr Samy

Steps solving advanced task :

# enabling port forwarding via UFW

# add this below rule to the /etc/ufw/before.rules file
# make sure its before the *filter line, VERY IMPORTANT

*nat
: PREROUTING ACCEPT [0:0]
-A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
COMMIT

# Then go to your /etc/sysctl.conf file to enable port forwarding by
# uncommenting the line #net.ipv4.ip_forward=1

-- before
#net.ipv4.ip_forward=1

--after
net.ipv4.ip_forward=1

# then reload the sysctl configuration
$ sudo sysctl -p

# congratulations !! you've done it
