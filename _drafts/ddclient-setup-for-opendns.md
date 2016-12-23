
Network filtering is a technique that you can configure your firewall to control the flow of incoming and outgoing packets based on the source and destination Ip address.

Generally, we define rules in a firewall device to allow and deny a certain traffic based on source IP address and destination IP address.
 Let's say you want to block facebook.com in your home network. How do you do it? You create a rule in a gateway router(router has an embedded firewall) of your home or office, such that  `Block -> domainName -> facebook.com`.

What if your home router or an office router doesn't have a packet filtering capability,  then you are out of luck, You **cannot** filter the traffic. Oh! just kidding you can.

 There is a service called OpenDNS which provide the network filter service based on the DNS request.

According to them,
>> OpenDNS is a company and service which extends the Domain Name System by adding features such as phishing protection and optional content filtering


To use OpenDNS service, first, you need to create an account on OpenDNS. OpenDNS has a free plan for home internet security. Follow this [link](https://www.opendns.com/home-internet-security/) to signup for free home security from OpenDNS.

Free service supports only one public IP address. If you want to support more than one public IP, then you need to switch to other plan or create multiple OpenDNS account.

If your Public IP address changes frequently, then you can use ddclient or Dynamic DNS updater to update the changed IP address. Take a look on this [post](http://localhost) how to deal with dynamic IP address in OpenDNS.

To Configure OpenDNS, signup to the OpenDNS website and choose OpenDNS home service plan as shown below.
<img src="http://i.imgur.com/riK7IV8.png">
Enter the required fields and signup as shown below.
<img src="http://i.imgur.com/UYDWvLH.png">
After signing up, verify your email address and your Public IP address.

Now it's time to add your IP address to OpenDNS services and filter the traffic as shown below.

1. Go to your account and add a Network as shown below.
 <img src="http://imgur.com/a/FqjIF">

 <img src="http://imgur.com/a/PU29Z">

 *open DNS will automatically detect your public IP address **

 3. Give a name to your filtering rules and click done as shown below.
 <img src="http://imgur.com/a/VykgY">

 4. Apply the filtering rules as shown below and click on apply.

 <img src="http://imgur.com/hkhQ4pi" >

 Now, the content filtering has been almost finished. It's time to configure your router or a firewall to use this Filtering policy.

 The configuration on router varies from manufacturer to manufacturer. I have a Digicom DSL router and I will show you how to configure this DigiCom DSL router to use OpenDNS content filtering policy.

 To configure the router, log in to a router and go to Local Network setup and change the DNS server address as shown below.

 >>>
 DNS Server 1:    208.67.222.222
 DNS Server 2:    208.67.220.220

 You can visit the [link](https://use.opendns.com/) to see how to configure the specific routers to use OpenDNS service.

So, now it's time to test the configuration. Open your browser and try to visit any social networking site. On visiting social networking sites, you saw that your connection is not private. Voila!, that's it.

I hope this post helped to set up content filtering policy in your office or in your home network using OpenDNS service.
