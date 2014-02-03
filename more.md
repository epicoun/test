# Setting up an OSX Sever in order to „go private“ :-)


## Services

- Mail & Notes

- Calendar, Contacts & Reminders

- Dropbox / Google Drive Replacement
	for desktop, web & mobile
	with webdav access

- Bookmark Sync for Firefox

- FotoStream Replacement

- Secure Messaging, Voice & Video Chat

- Feedly RSS Reader Replacement

- Secure VPN for Desktop and Mobile Access



## Preparations


### Configuring Dynamic DNS

	get a dynamic hostname from no-ip.com
	
	e.g.: somename.no-ip.biz
	
	setup a dynamic dns updater either as software on your server or in your router
	
	verify your setup using
	
		nslookup somename.no-ip.biz
		http://wieistmeineip.de


### Setting up your Domain DNS

	
	get your own domain name with a provider that lets you configure your DNS settings
	
	choose a name for your server: e.g. „myserver"
	
	create cname alias for myserver in your domain to somename.no-ip.biz
	
	set the MX record for mydomain.tld to  myserver.mydomain.tld	


### Setting up your Local Network Router

	it’s best to use an apple airport !!!!
	
	set a good name server in your router e.g.:
		208.67.222.222   opendns.com
		
	assign a fixed ip to your server
	
	enable port forwarding to the server
	
		if it’s an airport, don’t do it in serveradmin, since it opens the non ssl ports too.
		or or do it in serveradmin and deactivate the non ssl ports manually in airport utility afterwards
		
		all mappings point to the fixed ip of the sever
			in a VM setup PTPP points to the host of the VM
			
				Src		Dst
	
	Web 		443		444		TCP		HTTPS		we do not want to expose the standard http & https server os osx server
													since they have the severadmin functions built in
	Mail		25				TCP		SMTP
				993				TCP		IMAP SSL
				587				TCP		SMTP Client Access
		
	CalDav		8443			TCP		CalDAV SSL
	
	CardDav		8843			TCP		CardDAV SSL
	
	VPN			500				UDP		ISAKMP/IKE
				1701			UDP		L2TP 
				4500			UDP		IPSEC/NAT
				
				1723			TCP		PTPP			-> map this to the host in a VM setup
				
	XMPP		5222			TCP		XMPP Client Access
				5269			TCP		XMPP Server 2 Server
				7777			TCP		XMPP File Transfer Proxy



### Running the Server in a Virtual Machine

	we want the host and the vm environment to startup automatically after power failure
	since we want the actual server run with file vault enabled we need access to it


on the host

		name the machine host / host.local
		
		create user admin with a strong password
		
		in security
			disable FileVault so the machine requires no password for booting
			enable the firewall
			
		in energy panel
			set restart after power failure  to ON
			set Power Nap to : OFF
			
		in sharing
			enable screen sharing
			
		set VM Ware Fusion to auto start on login
		
		set the VM for the server to start on VM Ware startup
		
		install the dynamic IP updater on the host 
		
		test whether you can access the machine with screen sharing
			finder > connetc to vnc://host.local
			
		turn on auto protect / automatic snapshots for the Server VM
		
		


enabling emergency VPN access to the host

		follow the instructions to setup a server on the host
		
			the machine does not need a FQDN, set it to local access only
			
			enable open directory (it a bug PPTP is failing without OD :-( )
			
			enable VPN , L2TP & PTPP
				you only need a few IPs for this VPN
				
		create a network user vpn-admin with a strong password
		
		on the router map PTPP to the host and L2TP to the real server in the VM
		
			now you have an emergency access the host by using PTPP
			regular VPN for all users of your domain is thru the server VM and L2TP
			
			
	now, after power failure
	
		your host will boot automatically and update your dynamic IP
		your host will start the VPN service
		your Server VM will start booting and wait for the FileVault password to be entered
			-> you can do this now even from a remote location using VPN PTPP and screen sharing



### Preparing the Server OS

	update system to latest version
	
	create user admin with a strong password
	
	in security preferences
		enable file vault
		
			don’t write the recovery key down
			don’t have it sent to apple
			
		enable FireWall
		
	in Energy Setting
		standby for computer: never
		standby for screen: 1 min


#### Preparing your Network Settings


in Sharing preferences

		set the name of the mac to myserver
		set the bonjour name to myserver


in Network Panel

		set ip to manually 
			pick a fixed value for your IP
			network mask is usually 255.255.255.0
			router as with DHCP
		
		in options
			if you are not using an airport as your router
				set your DNS server to 8.8.8.8 (goolge) so your router does not play any fancy tricks on you with overintelligent local dns
				
			remove any other DNS server settings


#### Setup a Local Backup

	get a time capsule
		install it in a different room / location then your server
		
	in Time Machine 
		enable Backup
		enable encryption


Consider Setting up Remote Backup

	use a tool like crashplan that offers free 448 bit encrypted backup to another computer across the net
 	backup /Library/Server the osx server uses to store all ist data
	exists once the server is installed




### Setting up the Sever


	Download  Server App from App Store
	Lauch Server App


#### General

	set server name to server.mydomain.tld
	set it to internet visibility (third option)


#### DNS

	a DNS server is automatically configured to resolve the local ip to the fqdn and vice versa on this machine
	
		the server needs to resolve myserver.mydomain.tld to its local IP and back
		the rest of the world should see myserver.mydomain.tld under the external ip

	check the setup
	
		open a shell an check your dns setup	
			sudo changeip -checkhostname
			
		in network > options > DNS
			your local IP should now be set as default DNS
			
		on any other machine the server should be seen under it’s external IP
		
		ufff :-)	


#### Security

	get a free server cert from startssl
	select show all certs
		click on plus and choose import certificate 
		drag the .crt and the private key to the window to the p12 file
		choose certificate in pull down to enable it for a lll services



#### Open Directory

	DO NOT ENABLE  Open Directory
	this is far to complex for a simple setup. if something fails we are doomed !



### Creating User Accounts

	 all accounts are local accounts
	 
	if your are paling to use only one domain on the server, you can have a simple naming scheme 
	
		name:  	name
		email: 	name@domain.tld
		
	to use multiple domains have unique user naming  
	
		name:  	name-domain-tld
		email: 	name@domain.tld
		
	no storage for users
	add additional e-mail aliases in extended options
	
	passwords should be at least 8 chars (mozialla sync won’t work with less)
	-> set a general rule for this in options menu



### Mail

Basic setup

		set your primary domain to „mydomain.tld“
			server supports multiple domains
			optionally add additional domain(s)
			
		set Authentication to Automatic
	
		filter settings
		
			activate 		virus filter 
			activate 		blacklist
			de-activate 	grey list
			activate 		spam filter on at least level 4
			
		e-mail addresses for user are configured in user manager		


Setting a mail relay

	your dynamic IP might be blacklisted by your provider on spamhouse.org as being untrustable as a mail sender
		thus you need an relay with a fixed IP
		
	this will result in your outgoing mail being not accepted by the receiving host
		
	be aware to put the domain name for the relay in brackets [ ]
		otherwise postfix will lookup the mx for the host and take that one
		[relay.example.com:25]


Increasing the maximum message size to 100MB (default is 10 MB)

	sudo serveradmin settings mail:postfix:message_size_limit = 100000000


Setting mail aliases for users

	servermananger > users > select user > right click > extended options > aliases
	
		aliases for the primary mail domain 	just the username
		aliases for sencodary mail domains 	user@domain


Sending mails from a fritz.box to your server

	e.g status information or mailbox messages
	
	unfortunately the fritz.box does not use a fqdn on smtp submit so we have to make up for this
		
	in /Library/Server/Mail/Config/postfix/main.cf at the end where apple save its overrides
	
			mynetworks = 127.0.0.0/8, [::1]/128, 10.0.0.0/8 (add whatever your network is)
			
			smtpd_helo_restrictions = permit_mynetworks reject_non_fqdn_helo_hostname reject_invalid_helo_hostname
			
	sudo postfix reload



### Calendar & Contacts

	Enable the Services
	
	CalDAV SSL 		will be available on port 8443
	CardDAV SSL 	will be available on port 8843
	
	We have no use for the non SSL services, so they are not mapped in our router



### Webserver (for Files & RSS Reader)

	create empty dir in /Library/Server/Web/Data/Sites/
	disable the http (port 80) server by setting server root to the empty dir
	
	enable php
	
	enable the server
	
	the SSL server will be 256 bit high grade encryption
	
	verify the operation of your server by surfing to https://myserver.local
	
	you should see the server start page
	
	this web server will not be exposed to the external world




### Files (using ownCloud)

Install ownCloud

	get ownCloud from http://owncloud.org/
	
	mkdir -p /Library/Server/Web/Data/Sites/myserver.mydomain.tld/
	copy owncloud folder here
	
	cd /Library/Server/Web/Data/Sites/myserver.mydomain.tld/
	sudo chown -R www   *
	sudo chgrp -R admin *
	
	in serveradmin > web
	
		create a new web-server 
		
		set the hostname myserver.mydomain.tld
		set server root of the web server to  /Library/Server/Web/Data/Sites/myserver.mydomain.tld/owncloud
		enable SSL with your StartSSL certificate
		
		set the port of the server to 444
	
		enable .htaccess in Options		

Running on Postgres SQL


		OSX ha a built-in dedicated postgres sql server for client apps
		
		sudo serveradmin start postgres
		
			to start the server
			server listens to localhost:5432
			additional IPs can be added in /Library/Server/PostgreSQL/Config/org.postgresql.postgres.plist
			
		get pgAdmin  -> http://www.pgadmin.org/download/macosx.php  (phpPgAdmin could not connect for whatever reason)
		
			create a connection to db localhost
				user _postgres, pw is admin password
				
			create login-role (user) 
				tab properties
					name: owncloud
				tab definition 
					set password (pick a new one)
				tab privileges
					grant "create table“ privilege
					
			create a connection to localhost with user owncloud to test


Optional:

if you want to disable the lost password functionality

	disable lost pwd function in ownCloud
	mv core/lostpassword -> core/x-lostpassword
	commented the part that shows lost passed if login fails in core/login (line 17)
	
	
	if you want to keep the user data separate from the server root
	
	sudo mkdir -p  /Library/Server/owncloud/data
	cd /Library/Server/owncloud/
	sudo chown -R www  data
	sudo chgrp -R admin data



Setup & Configuration

	setup using https://myserver.mydomain.tld/
	
	
	base config
	
		create user admin
		
		change 'datadirectory' => '/Library/Server/owncloud/data/',
		
		optionally using postgres
		
			host:		localhost
			user: 	owncloud
			pw:		
			db: 		own-cloud
			
		ignore WebDAV error after setup
		
	
	enable Mozilla sync app in apps panel
	
	disable apps for
		calendar
		contacts
		documents
		
	optionally using OD / LDAP (check if mozilla sync supports it)
	
		host:	127.0.0.1
		base-dn:	dc=myserver,dc=mydomain,dc=tld
		user-dn:	uid=diradmin,cn=users,dc=myserver,dc=mydomain,dc=tld
		passwd:		diradminpwd
		user  fltr: objectClass=posixAccount
		group fltr: objectClass=posixGroup
		emailfield: mail (for mozilla sync)
		
		
	add user accounts 
	
		either with the above naming scheme or their email addresses
		-> users must set their email address for mozilla sync in the prefs panel manually by themselves
		
		


Web Access	

	https://myserver.mydomain.tld/		


WebDAV

	https://myserver.mydomain.tld/remote.php/webdav/



### FireFox Bookmark Sync

is included in ownCloud

	-> USERS must add their email address in prefs panel in order to use mozilla sync		


in firefox - on first device 

	choose > creat new account
		
		email:		your email
		passwd:		your own ownCloud
		passed:		your own ownCloud
		
		Sync Server:	https://myserver.mydomain.tld/remote.php/mozilla_sync/

on other devies

	either use pairing function
			
	or connect to existing account using
	
		email:		your email
		passwd:		your ownCloud passed
		Sync Server:		https://myserver.mydomain.tld/remote.php/mozilla_sync/
		recovery key:	get this form the first device, sync panel, drop down menu
		

firefox sync interval in 10 minutes, manual sync is available in ff from the menu




### FotoStream Replacement

	install CameraSync on your iPhone
	
	point it to your ownCloud WebDAV photos folder
	
		https://myserver.mydomain.tld/photos




### VPN

	for proper L2TP you need to update server app to the latest version	
	
	Pick L2TP
	
		L2TP is more secure, but needs three UDP ports and might not work with all routers
		
		PTPP is less secure (still 128 bit), but more robust, since it only uses one TCP port, works even with double NAT
		BUT Auth for PTPP is currently (12/2013) is not working without OD turned on (which we don’t recommend)
		
		
	in a setup with the server in a VM, PTPP could be mapped to the host
	
	pick a shared secret
	
	pick an ip range not used by your router for DHCP
	
	start the service
	
	test the service wit a vpn connection to my server.local
	test the service wit a vpn connection to my myserver.mydomain.tld
		test using an iphone with 3G / 4G



### Messaging / XMPP

Optional: Domains configuration

	the server is configured to work with <user>@myserver.mydomain.tld
	
	make the server work with logins <user>@mydomain.tld and be available for other xmpp servers for federation
	
	/Library/Server/Messages/Config/jabberd/sm.xml
	
	add
	
	<local>
		<id>myserver.mydomain.tld</id>
		<id>mydomain.tld</id>
	 </local>
	 
	 
	/Library/Server/Messages/Config/jabberd/c2s.xml
	
        <id require-starttls="true" pemfile="/etc/certificates/myserver.mydomain.tld.xxx.concat.pem" private-key-
			password=„xxx“cachain="/etc/certificates/myserver.mydomain.tld.xxx.chain.pem“>
		mydomain.tld
		</id>
		
		
		set the srv records for xmpp in your dns to allow others to find the server for the domain
		
		_xmpp-client._tcp.mydomain.tld. 	18000 IN SRV 0 5 5222 myserver.mydomain.tld.
		_xmpp-server._tcp.mydomain.tld. 	18000 IN SRV 0 5 5269 myserver.mydomain.tld.


accessing your server from the mac


		don’t use ichat
		
		use jitsi for security including OTR encryption, voice & video chat
		
			https://jitsi.org/
			
		
		you can use audium as well 
		
			https://jitsi.org/
			
			settings > general 
				turn off message protocols
				
			settings > acoounts > account > options
				turn on require SSL / TLS 
				
			settings > acoounts > account > privacy
				consider force encryption and reject plain text 
					if chatting with users on other servers
 			

accessing your server from the iPhone 

		us IM+, supports OTR as well
		
			https://itunes.apple.com/de/app/im+-instant-messenger/id285688934?mt=8			



if your buddy list is populated with all other users on the server

		sudo serveradmin settings jabber:enableAutoBuddy = no
				
		looks like autobuddy = no is ignored on setups with OD active




### Feedly Replacement


baseds on tinytiny-rss, looks like feedly and even works on iOS Reeder 2 thru fever api


	create a virtual web host on port e.g. 555
	
	get tiny tiny rss
	
		http://tt-rss.org/redmine/projects/tt-rss/wiki
		
	 	move tt-rss folder to new server folder and set it as server root
		
		
	get the feedly theme
	
		https://github.com/levito/tt-rss-feedly-theme
		
		
	install theme and make it default
	
		copy feedly.css & feedly folder > themes
		
		mv default.css old.css
		
		cp feedly.css default.css
		
		
	install the fever api plugin
	
		http://tt-rss.org/forum/viewtopic.php?f=22&t=1981
		
		
	set the correct access rights for the tt-rss folder
	
		sudo chgrp -R admin *
		sudo chown -R www  *
		
		
	prepare postgress using pgadmin
	
		create login role tinyrss
		login with that role
		create db tinyrss
		
		
	restart the web server 
	
	
	access service
		setup & add users


setting up continuos updating

	execute the script first to test if the rights are yet correct
	
		/usr/bin/php /Library/Server/Web/Data/Sites/<homedir>/tt-rss/update.php —feeds
		
	add an entry to crontab
	
		export EDITOR='nano'
	
		crontab -e
		
			*/5 * * * * /usr/bin/php /Library/Server/Web/Data/Sites/<homedir>/tt-rss/update.php --feeds --quiet



moving feeds from feedly to tt-rss

		export your feeds in organize > export OPML
		import OPML in tt-rrs in settings > feeds > OPML


building a fluid app on OSX


	http://fluidapp.com/
	
	fluid supports unread count in badges
	nice reader fluid icon
		http://www.mactomster.de/wp-content/uploads/2010/03/reeder-icon2.png


using fever compatible feed readers apps

	in the tt-rss web interface
	
		enable api access for your account in settings > settings
		enable fever plugin in setting > plugins
		set extra passwd in  settings > fever plugin
		
	in the apps
	
		configure fever account
		
		https://mysererv.domain.tld:555/plugins/fever/
		username
		extra password


Best App for OSX

	https://itunes.apple.com/de/app/readkit/id588726889?mt=12


Best App for iOS

	https://itunes.apple.com/de/app/reeder-2/id697846300?mt=8

