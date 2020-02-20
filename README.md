Instructions on domain dns configuration 

Actual scan results:

## Connectivity

WARNING	All nameservers IPv4 addresses are in the same AS (19318).
WARNING	All nameservers are in the same AS (19318).

-AS refers to Autonomous System which in rough terms means "block of IPs that share common routing" or in more general terms "are from the same allocation block".

-You're getting this warning because the nameservers are all in the same block and if that single route goes offline for some reason, all your nameservers go down. It's generally best to spread these out geographically to minimize your exposure to localized events. 


## DNSSEC

NOTICE	There are neither DS nor DNSKEY records for the zone.
NOTICE	The zone is not signed with DNSSEC.

## Zone

(TTL)
**>>>>>NOTICE	SOA 'refresh' value (3600) is less than the recommended one (14400).<<<<<<<<<**
**>>>>>NOTICE	SOA 'retry' value (1800) is less than the recommended one (3600). <<<<<<<<<**
INFO	SOA 'expire' value (1209600) is higher than the minimum recommended value (604800) and not lower than the 'refresh' value (3600).
INFO	SOA 'minimum' value (86400) is between the recommended ones (300/86400).
INFO	SOA 'mname' nameserver (dns2030a.trouble-free.net) is authoritative for 'botxo.ai' zone.
INFO	SOA 'mname' value (dns2030a.trouble-free.net) refers to a NS which is not an alias (CNAME).
INFO	SOA 'mname' value (dns2030a.trouble-free.net) refers to a NS which is not an alias (CNAME).
INFO	SOA 'refresh' value (3600) is higher than the SOA 'retry' value (1800).
INFO	Target (MX=botxo.ai) found to deliver e-mail for the domain name.

### points to remember
1. an a record must always contain ip address (map host to ip)

-whenever you specify a record it must contain ip address on the right side. the a record is so important in dns without which the meaning of mapping hostnames to ip would be absurd. so remember this!

2. cname (alias) must contain hostnames. no ips here

3. ns an mx records must contain host names. no ips allowed.

4. use the dot in the end, whenever you specify a domain name in the dns zone file. this dot is so important and if you forget this you will have nightmares with your dns configuration.

for example

example.com. in    ns    ns1.example.com.

why dot? simply because it tells to start query from root servers (denoted by dot)

##the dot in a zone file

sometimes you need it, sometimes you don't. at first glance, and even at the fourth glance, it seems confusing.

the rule is simple and we call it the origin substitution rule.

if there is a dot at the end of a name in a resource record or directive, the name is qualified and if it contains the whole name including the host then it is a fully qualified domain name - fqdn. in this case the name as it appears in the rr is used unchanged.

if there is no dot at the end of the name (a.k.a. label(s) in dns jargon), the name is unqualified and dns software adds the value of the last or only $origin directive. unqualified names can appear on left-hand names, right-hand names or both. in the absence of an $origin directive the zone name from the named.conf file for this zone is used to synthesize an $origin directive. the fragment below illustrates this using a records and cname records.

with implicit $origin statement: in the zone fragment below an implicit $origin directive is created from the zone file name as noted. in general this is bad practise for two reasons. first, the file is not self-referencing - you need to know information from another source - in this case the zone name for which this file is being used which is defined in the named.conf file (for bind). second, it is not readily apparent what value is being substituted. at 3 a.m. when working under stress to bring back service it may not be so apparent. all to save typing a few characters. just not worth it.


 

### common dns errors in zone file writing

1. no cname pointing to ns records
domain.com.     in     ns     ns1.domain.com.
domain.com.     in    ns     ns2.domain.com.
domain.com.    in    cname    ns9.example-server.net -----> wrong

placing cname along with ns the all of namservers will fail and will result in lame delegation. don't do that!

refer to rfc1912 2.4 and rfc2181 10.3.
 

2. avoid running dns servers on ips on same subnet (/24) or on same server.
the whole purpose of dns is for nameservers to be spread over different geographical locations so that if one dns fails the other would work. although it is very common practice to run both nameservers on same server or subnet, it would not provide fault tolerance. if the server fails your nameservers will fail and your site wont load.

ns1 in a 75.33.22.xx -----> same subnet /24
ns2 in a 75.33.22.xx -----> same subnet /24

 

3. proper glue
always add glue to your ns records to the ip addresses using a record, failing which one of your nameservers will fail.

domain.com. in ns ns1.domain.com.
domain.com. in ns ns2.domain.com.

ns1 in a 1.2.3.4 -----> glue
ns2 in a 2.4.6.9 -----> glue

refer to rfc1912.

 

4. no duplicate mx records
domain.com. in mx mail.domain.com.
domain.com. in mx mail.domain.com  ----> duplicate

 

5. allow port 53 for both udp and tcp connections
if you use firewall make sure you do not block port 53 for dns tcp and udp requests. by default dns lookups use udp protocol while zone transfers and notifications use tcp protocol of port 53.

port 53 udp = dns requests
port 53 tcp = zone transfers

 

6. cnames cannot co-xist with mx hosts.
do not specify cname or aliases pointing to mx records.

domain.com. in mx 10 mail.domain.com.
mail in cname domain.com.  ----------> wrong

instead use a record to map directly to ip address.

mail in a 11.33.55.77 ---> correct

refer to rfc1912.

 

7. mx records should not contain ip addresses
domain.com. in 10 mx mail.domain.com. ----> correct
domain.com. in 20 mx 11.22.33.44  -----> wrong

the correct way of doing this is glue the mx host to a record.

domain.com. in mx 10 mail.domain.com. -----> correct
mail in a 11.33.55.77 ----------> correct

 

8. ns records should not contain ip address
always specify nameservers for your domain with ns records. it should be a name and not ip address.

domain.com. in ns dns0.domain.com. -----> correct
domain.com. in ns  75.xx.xx.xx -----------> wrong

 

reverse dns for mail delivery
for proper mail delivery, the following anti-spam methos are very important to make sure the email is delivered to users inbox. most public email service providers yahoo, hotmail and gmail do use these parameters to flag email is spam or not.

(i) setup reverse ip for your mail server with ptr in dns (needs dedicated ip)
(ii) setup spf record in your dns
(iii) setup domain keys

i have seen on many occasions if you use shared hosting plan, most emails do land up in spam/bulk folder in the users inbox. so use a dedicated server.

 

set up reverse ips for mailserver ips
in order to setup reverse ip, first you will need to place a request to your hosting provider (since they own ip address) and ask them to setup a reverse ip to your mail server. once that is done you have to place a line using ptr in your domain zone file.

to test reverse ip lookup:

  host <ip-address>
will show the output of reverse dns.

 

### domain keys

it basically contains 2 records (with and without selector) placed under txt using underscore along with generated public key.

the ’sel’ is a selector and can be selector name.

 

###how to set up dns nameservers for your domain using bind9
just go to your linux console and type the following commands.

yum install bind
create a new zone file for your domain and insert the sample zone file (see below). you have to change domain.com, serial and email parameters.

nano /var/named/domain.com.db

open /etc/named.conf and place the following lines:

 zone "domain.com" {
type master;
file "/var/named/domain.com.com.db";
};

save changes to the file and restart bind.

service named restart 
 

### sample dns zone file for bind 

use this file to set up your domains and nameservers in bind9:

;=================================================================
;sample bind dns zone file
;for any domain (just change domain.com to your site)
;================================================================

; before you start dont forget the dot and serial increment

$ttl 14400
$origin domain.com.

; soa record
; specify primary nameserver ns1.domain.com
; serial should increment every update
@ 14400 in soa ns1.domain.com. webmaster.domain.com. (
                                2009092902 ; serial in yyyymmddxx (xx is increment)
                                10800; refresh seconds
                                3600; retry
                                604800; expire
                                38400; minimum
                                );
; website ip address specified in a record

       in a xx.xx.xx.xx

; minimum 2 dns nameserver names

       in ns ns1.domain.com.
       in ns ns2.domain.com.

; mapping all nameservers and their corresponding ips (glue)

ns1  in a xx.xx.xx.xx
ns2  in a xx.xx.xx.xx

; specify any subdomains and www entry here using cname record

www     in cname domain.com.
ftp     in cname domain.com.
server     in cname domain.com.
webmail in cname domain.com.

; setup mx record (mail exchanger with priority)
domain.com. in mx 10 mail.domain.com.

; set a record for mail
mail in a xx.xx.xx.xx;====================================




#### chapter about $origin directives


$origin defines a base name from which 'unqualified' names (those without a terminating dot) substitutions are made when processing the zone file. zone files which do not contain an $origin directive, while being perfectly legitimate, can also be highly confusing. in general, always explicitly define an $origin directive unless there is a very good reason not to (sloth is not a very good reason).

$origin is a standard dns directive defined in rfc 1035.

$origin values must be 'qualified' (they end with a 'dot').

note for purists: technically it is legitimate to use an $origin without a terminating dot, it is then subject to the normal substitution rules. however, if used in this manner it can easily create configuation disasters than not even your mother could sort out. practically, an $origin must terminate with a dot.

if an $origin directive is not defined - bind synthesizes an $origin from the zone name in the named.conf file as illustrated below:

// named.conf file fragment

zone "example.com" in{
	type master;
	file "pri.example.com";
};
in the above fragment $origin example.com. is synthesized if none present in the zone file (pri.example.com).

$origin is used in two contexts during zone file processing:

the symbol @ forces substitution of the current (or synthesized) value of $origin. the @ symbol is replaced with the current value of $origin.
the current value of $origin is added to any 'unqualified' name (any name which does not end in a 'dot').
examples
@ symbol replacement:

; example.com zone file fragment 
; no $origin present and is synthesized from the 
; zone name in named.conf
....
@          in      ns     ns1.example.com. 
; ns1.example.com is the name server for example.com
....
$origin uk.example.com.
@          in      ns     ns2.example.com. 
; functionally identical to
; uk.example.com in ns ns2.example.com
; ns2.example.com is the name server for uk.example.com
unqualified name addition:

; example.com zone file fragment 
; no $origin present and is synthesized from the 
; zone name in named.conf
....
www          in      a    192.168.23.15 
; functionally identical to 
; www.example.com. in    a  192.162.23.15
; thus 
; www.example.com = ip 192.168.23.15
joe          in      cname www ;unqualified name
; joe.example.com = www.example.com
; could have written as
joe.example.com. in  cname www.example.com.
....
$origin uk.example.com.
ftp          in      a     10.0.16.34 
; functionally identical to
; ftp.uk.example.com in a 10.0.16.34
an $origin directive has scope (is effective) until either another $origin replaces it or until eof. the following examples illustrate $origin usage.

; example.com zone file fragment
...
$origin example.com.
...
smart a 192.168.2.3
; smart expands to smart.example.com.
...
$origin stupid.example.com.
...
smarter a 192.168.2.4
;smarter expands to smarter.stupid.example.com.
...
$origin stupider.example.com.
...
smartest a 192.168.2.5
;smartest expands to smartest.stupider.example.com.
...
$origin example.com.
...
genius a 192.168.2.6
; genius expands to genius.example.com.
...


#### time-to-live (ttl) values 

the $ttl directive is defined in rfc 2308.

ttl in the dns context defines the duration in seconds that the record may be cached by any resolver. zero indicates the record should not be cached. note: rfc 1912 cautions that 0 = no caching may not be widely supported, however most modern dns software does support the feature. ttl values of 0 should be used with extreme care since, depending on the type of rr to which they are applied, they can create significant loads on both the resolver (caching name server) and the authoritative name servers as well as significantly increase latency times for all transactions.

a more extensive discussion about sensible ttl values can be outsourced im sure.

the default ttl for the zone is defined in bind9 by the $ttl directive which must appear at the beginning of the zone file, that is, before any rr to which it will apply. this $ttl is used for any resource record which does not explicitly set the 'ttl' field.

the ttl field is defined to be an unsigned 32 bit value with a valid range from 0 to 2147483647 (clarified in rfc 2181) - which is a long time! - somewhere on the other side of 68 years.

the $ttl field may take any time value.

in bind 8 the soa record (minimum parameter) was used to define the zone default ttl value. in bind 9 the soa 'minimum' parameter is used as the negative (nxdomain) caching time (defined in rfc 2308).

rfc 1912 recommends that the $ttl value be set to 1 day or longer and that certain rrs which rarely change, such as the mx records for the domain, use an explicit ttl value to set even longer values such as 2 to 3 weeks. the value of any ttl is a balance between how frequently you think the dns records will change vs load on the dns server. in the example below the $ttl value of 2d (2 days) indicates that any change (in this case to the www entry only, since all other rrs have explicit ttl values) may not be fully propagated for 48 hours. in most cases this will not be a problem since ip address changes are normally planned in advance, in which case in advance of the change process the ttl could be reduced to 3h to 12h and then restored to a higher value when the change has stabilized. an alternative, lower risk, strategy may be to define the new ip address using a second www a rr (thus maintaining two web services during a transitional period). once the change has stabilized remove the old www a rr wait a further 48 hours and then retire the old web service.

example
; example.com zone file fragment 
$ttl 2d ; zone default
      3w   in      mx  10 192.168.254.2  ; overrides default     
joe   3h   in      a      192.168.254.3  ; overrides default     
www        in      a      192.168.254.3  ; uses zone default = 2 days


