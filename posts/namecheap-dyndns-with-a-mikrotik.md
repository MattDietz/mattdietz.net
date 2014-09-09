<!-- 
.. title: Namecheap DynDNS with a Mikrotik
.. slug: namecheap-dyndns-with-a-mikrotik
.. date: 2014-09-09 04:44:24 UTC
.. tags: Mikrotik, Networking, DynDNS
.. link: 
.. description: 
.. type: text
-->

I've been wanting to use my Mikrotik to update DynDNS for some time. After digging, I found out that newer firmware had a "Cloud" option that effectively accomplishes it, but I wanted to use a domain I registered through Namecheap instead. Some googling showed that the "Scripts" functionality of the router was perfectly suited to the task; coupled with scheduling, I was good to go.

You're going to want to go to Namecheap and enable DynDNS for the domain you'd like to use. Checkout [How do I setup a Host for Dynamic DNS?](https://www.namecheap.com/support/knowledgebase/article.aspx/43/11/how-do-i-setup-a-host-for-dynamic-dns) and [How do I enable Dynamic DNS for a domain?](https://www.namecheap.com/support/knowledgebase/article.aspx/595/11/how-do-i-enable-dynamic-dns-for-a-domain) for more information on getting that setup. After you've done so, Namecheap will show you a password; be sure to copy that.

Next, log into your router. I find that the Winbox interface is by far the easiest for this task. WINE does a pretty good approximation of Winbox if you don't have a windows box lying somewhere around. Meanwhile, I know there's an OSX client, but the download is suspiciously 151MB, whereas the Windows binary is in the neighborhood of 11MB.

After you've logged in, click on "System," then navigate down to "Scripts." Click on "Add New" and enter a name like "dyndns." Finally, paste the following in the "source" field:

    # Set local variables. If you want to update subdomain.example.com, put in "subdomain" under subdomain and "example.com" in the domain variable. If you want to update just example.com with no subdomain, enter in "@" for the subdomain.
    
    :local password "<namecheap provided password>"
    :local subdomain ""
    :local domain "<the domain>.<tld>"
    
    # Get Public IP
    
    /tool fetch url="http://bot.whatismyipaddress.com/" mode=http dst-path=pubIP.txt
    :local currentIP [/file get pubIP.txt contents]
    :log info "Current Public IP is:$currentIP"
    
    :local url1 "http://dynamicdns.park-your-domain.com/update\3Fhost=$subdomain&domain=$domain&password=$password&ip=$currentIP"
    /tool fetch url=($url1) mode=http
    :log info "DNS Successfully Updated"

Leaving the subdomain field as an empty string is perfectly acceptable.

Test the script out using the "Run Script" button. If everything has gone well, you should see new lines in your "Log" menu. Furthermore, you'll see some files in the root of the router under the "File" menu. These were created by the script above to shuttle the data around.

Next, you're going to want to set up a schedule. Ensure the clock is set correctly on your router: I recommend setting up SNTP. Once you've done that, click on "System" once again, and then "Scheduler." From there, click "Add New." The name of the job can be anything you'd like. Set the "Start Date" and "Start Time" to be today and in the near future. Finally, the "Interval" field is HH:MM:SS. I've seen people use it as frequently as every minute, but that seems like overkill. I'd also guess that Namecheap is likely to rate-limit you eventually. My own public IP only seems to change if there's a power outage, so I set the updates to be much less frequent. Finally, in the "On Event" field, put just the name of the script as you entered it previously. After that, you should have a scheduled Namecheap DynDNS updater.
