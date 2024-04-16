
## How to Set Up a Custom Email with Cloudflare and Mailgun

<img alt="How to Set Up Custom Email along with Domain" src="https://i.imgur.com/Op3IJis.png">

As a software engineer, you may want to consider having a professional email account along with your own website like "_info@example.com_". But you may realized that it must cost a certain amount that you are not willing to pay. How would you behave if you knew you could do it for free? There is actually a way to do it, and besides the fact that having a professional email account is free, it will help you be more efficient, reliable and secure in your daily work.

In this article, you'll learn how to create and set up your own email address using Cloudflare and Mailgun to manage emails in Gmail. It means you can send and receive emails directly on your Gmail inbox.
I've already done this for personal use and have taken screenshots of the entire process to show it in this article. So I'll share here all the necessary steps you need to follow to set up your own email.


### Introduction

Let's figure out what you need to have before you start, what you are going to do and how it will work.

#### What you need to have before you start:

At first, we assume, you already have a domain, let's call it "_yourdomain.com_", on which you have control. Specifically, you need to have accessibility to connect your domain with Cloudflare and setup DNS records there. A classic example of that is having a domain on some domain registrar (like GoDaddy, Namecheap), and adding your domain to Cloudflare by setting DNS records provided by Cloudflare on your domain registrar account.

Adding a domain to Cloudflare involves updating your domain's DNS nameservers to point to Cloudflare's nameservers. Once the domain is added, Cloudflare acts as an intermediary for web traffic, providing security features like DDoS protection, firewall, and SSL encryption, as well as performance enhancements through caching and content optimization.
If you haven't done that yet, here's the official [video on YouTube](https://www.youtube.com/watch?v=7hY3gp_-9EU) on how to connect your domain to Cloudflare.

Additionally, Cloudflare manages DNS records for your domain, allowing you to control how traffic is routed and ensuring reliable delivery of services like email.
So, our work in this article will be focusing exactly on that, on how to setup your domain on Cloudflare Email.
[Cloudflare Email](https://blog.cloudflare.com/email-routing-leaves-beta/) is one of the services of Cloudflare since 2021, which can be used for free (as for now at least).

The second assumption is that you have account at Gmail, and you have access to its email settings. Simply, if you have just a regular "_youremail@gmail.com_" email, which isn't under control of any administrator or kind of that, then you're have nothing to worry about for this. We'll explore and work on email settings later on.

#### What you are going to do:

In simple words, you're going to create a custom email like "_something@yourdomain.com_", which you can use to send and receive emails from that email by using Gmail's platform. So you will be able receiving and reading emails sent to "_something@yourdomain.com_" in Gmail, as well as sending emails from that custom email using Gmail.
For all of that you'll use Cloudflare Email for the email routing, and Mailgun's SMTP server for sending emails.


#### How it will work technically:

When composing an email from Gmail with the sender set as "_something@yourdomain.com_", Gmail utilizes Mailgun's SMTP server through the provided credentials, transmitting the email. Mailgun then processes the message and forwards it to the recipient's email server, likely involving DNS lookups to find the recipient's server.
Emails sent to "_something@yourdomain.com_" are received by Cloudflare's email servers, configured via MX records in the domain's DNS settings. Cloudflare stores the received emails in the associated account, accessible through Gmail, which periodically connects to Cloudflare's servers (using IMAP or POP3 protocols) to retrieve new messages, enabling seamless access to incoming emails.


### Email Routing on Cloudflare

> Cloudflare Email Routing is designed to simplify the way you create and manage email addresses, without needing to keep an eye on additional mailboxes. With Email Routing, you can create any number of custom email addresses to use in situations where you do not want to share your primary email address, such as when you subscribe to a new service or newsletter. Emails are then routed to your preferred email inbox, without you ever having to expose your primary email address. ([Docs](https://developers.cloudflare.com/email-routing/))

Sign in to your Cloudflare account and navigate to Dashboard.
Choose and click to the desired website. For me it's "_boolfalse.com_", as I want to create a custom email like "_email@boolfalse.com_".

<img alt="Cloudflare: Websites" src="https://i.imgur.com/VfManqF.png">

Navigate to the "Email Routing" for the selected website.

<img alt="Cloudflare: Email Routing" src="https://i.imgur.com/wR7fam2.png">

If you don't have email routing configured, you may see something similar to the screenshot above. Click "Get started". You may be able to create your own address to receive emails and take action.
We'll skip this without creating our own address for doing it manually.

<img alt="Cloudflare: Custom Email" src="https://i.imgur.com/w1VRfrs.png">

By default, email routing is disabled, so you need to enable it. Click the link to navigate to the "_Email Routing_" page.

<img alt="Cloudflare: Email Routing" src="https://i.imgur.com/59gCU3C.png">

Submit it by clicking "Enable Email Routing".

<img alt="Cloudflare: Enable Email Routing" src="https://i.imgur.com/ds34wVH.png">

For get it done, you need to have three MX and one TXT records:

- Type: **_MX_**; Name: **_@_**; Mail Server: **_route1.mx.cloudflare.net_**; TTL: **_Auto_**; Priority: **_69_**
- Type: **_MX_**; Name: **_@_**; Mail Server: **_route2.mx.cloudflare.net_**; TTL: **_Auto_**; Priority: **_99_**
- Type: **_MX_**; Name: **_@_**; Mail Server: **_route3.mx.cloudflare.net_**; TTL: **_Auto_**; Priority: **_40_**
- Type: **_TXT_**; Name: **_@_**; TTL: **_Auto_**; Content: **_v=spf1 include:_spf.mx.cloudflare.net ~all_**

You can see them at the bottom of the "_Email Routing_" page.

<img alt="Cloudflare: DNS records for Email Routing" src="https://i.imgur.com/ZYSuiMA.png">

So, as already said, in the left menu, go to "DNS" -> "Records" and add the following records there.

<img alt="Cloudflare: DNS records added" src="https://i.imgur.com/pk2Im6F.png">

After creating these records, go to the "_Email Routing_" page again.

There you only need to have the records you just created.
So, if you have any other records, just delete them.
For example, I already had an unnecessary entry there that I should delete.

<img alt="Cloudflare: existing records for Email Routing" src="https://i.imgur.com/2QCQvaC.png">

Submit to delete existing unnecessary records.

<img alt="Cloudflare: deleting unnecessary records" src="https://i.imgur.com/gWtxnVV.png">

After removing unnecessary DNS records, you will see only the ones you need there.
You will now be able to enable email routing by clicking the "Add records and enable" button.

<img alt="Cloudflare: Enable Email Routing" src="https://i.imgur.com/ciCjtnR.png">

After enabling it you'll see something like this:

<img alt="Cloudflare: Email DNS records configured" src="https://i.imgur.com/V4AZxMn.png">


### Creating Custom Email on Cloudflare

Now go to the "_Routes_" tab and create an email by clicking the "Create address" button.

<img alt="Cloudflare: Email Routing (enabled)" src="https://i.imgur.com/9d53xZo.png">

In this example, we'll create "_email@boolfalse.com_" email address, by adding "_email_" as a custom address, and a destination email address, where I'll be able to receive emails.

<img alt="Cloudflare: Email Routing" src="https://i.imgur.com/lB3QzxE.png">

You'll see a notification about that.

<img alt="Cloudflare: creating a custom email" src="https://i.imgur.com/BB8JF00.png">

You will also get an email for confirming this action.

<img alt="Verifying the destination email" src="https://i.imgur.com/9eUFg6x.png">

Just verify the email address.

<img alt="Verify email address" src="https://i.imgur.com/U8bDTVz.png">

Once you've verified the email address, you may get this page:

<img alt="Cloudflare: custom email address is verified" src="https://i.imgur.com/AK8cLHd.png">

You probably will also get an email that you've verified your domain with Mailgun:

<img alt="Notification about custom email address verification" src="https://i.imgur.com/IO1zmiQ.png">


### Receiving Emails into the Custom Email

Now, your email address is activated, and you can see that here:

<img alt="Cloudflare: custom email address is active" src="https://i.imgur.com/YqwGRJX.png">

At this point you already can send emails to the custom email you just set up. In this case, it's "_email@boolfalse.com_".
Below is a test email sent from a different email.

<img alt="Testing email receiving" src="https://i.imgur.com/zDXBS8j.png">

You'll receive a test email to the custom email.

<img alt="Test email has been received" src="https://i.imgur.com/wZQdzUE.png">


### Mailgun: Adding New Domain

You can now successfully receive emails, but you can't send emails from that custom email yet.

So, it's time to switch to the Mail Service provider. In our case it will be [Mailgun](https://www.mailgun.com/).
To do this, you just need to register and attach the card to your Mailgun account. After activating your account with the card attached, you can set up a domain for your email.

You don't have to worry about the card, because Mailgun does not charge for limited quantities. I think the amount it gives is quite suitable for a free package.
You can find the price packages in detail [here](https://www.mailgun.com/pricing/).

Go to "_Sending_" -> "_Domains_" page, and click the "Add New Domain" button.

In our case it will be "_mg.boolfalse.com_", as Mailgun recommends to use like that to be able to send emails from your root domain, e.g. "_email@boolfalse.com_".
You can see that recommendation on the right in below image:

<img alt="Mailgun: create a new domain" src="https://i.imgur.com/PE2O4tT.png">

You can also select the domain region and DCIM key length, but you can leave everything as default.
I will leave DCIM key lenght as 1024 and "US" as a domain region.

After creating the domain, you may be shown some tips on how to verify your domain.

<img alt="Mailgun: adding a new domain" src="https://i.imgur.com/Ur3B57w.png">

Mailgun will give you two TXT records, two MX records and one CNAME record to add to your provider.

- Type: **_TXT_**; Name: **_mailto._domainkey.mg.boolfalse.com_**; TTL: **_Auto_**; Content: **_SECRET_**
- Type: **_TXT_**; Name: **_mg.boolfalse.com_**; TTL: **_Auto_**; Content: **_v=spf1 include:mailgun.org ~all_**
- Type: **_MX_**; Name: **_mg.boolfalse.com_**; Mail Server: **_mxa.mailgun.org_**; TTL: **_Auto_**; Priority: **_10_**
- Type: **_MX_**; Name **_mg.boolfalse.com_**; Mail Server: **_mxb.mailgun.org_**; TTL: Auto; Priority: **_10_**
- Type: **_CNAME_**; Name: **_email_**; Target: **_mailgun.org_**; TTL: **_Auto_**; Proxy Status: **_On_**

In our case, we will add them to Cloudflare.

Below is the first TXT record:

<img alt="Mailgun: first TXT record for a new domain" src="https://i.imgur.com/Y265ayL.png">

Below is the second TXT record:

<img alt="Mailgun: second TXT record for a new domain" src="https://i.imgur.com/AQPMrdn.png">

Below is the first MX record:

<img alt="Mailgun: first MX record for a new domain" src="https://i.imgur.com/SpSYc3q.png">

Below is the second MX record:

<img alt="Mailgun: second MX record for a new domain" src="https://i.imgur.com/nsqyF5V.png">

After you've added two TXT and two MX records, you can check and verify them by clicking the "Verify DNS Records" button.

<img alt="Mailgun: checking TXT and MX records for a new domain" src="https://i.imgur.com/Dfoa8bi.png">

Lastly, add CNAME record.

<img alt="Mailgun: adding CNAME record for a new domain" src="https://i.imgur.com/mbPG4E1.png">

You may see a warning icon on the left of the CNAME record. You don't need to worry about that, cause what [official documentation says](https://developers.cloudflare.com/ssl/edge-certificates/additional-options/total-tls/error-messages) about it:

> If you recently [added your domain](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/) to Cloudflare - meaning that your zone is in a [pending state](https://developers.cloudflare.com/dns/zone-setups/reference/domain-status/) - you can often ignore this warning.
> Once most domains becomes **Active**, Cloudflare will automatically issue a Universal SSL certificate, which will provide SSL/TLS coverage and remove the warning message.

After adding a CNAME record, you can check and verify it again by clicking the second "Verify DNS Records" button.

<img alt="Mailgun: checking CNAME record for a new domain" src="https://i.imgur.com/Xm58Ziq.png">

If you have added all 5 records on the Cloudflare successfully, after clicking the verifying button, Mailgun will automatically redirect you to the "_Overview_" page.

<img alt="Mailgun: 2 TXT, 2 MX and 1 CNAME records added for a new domain" src="https://i.imgur.com/bULjKsQ.png">

It means you're ready to add a Sending API key on Mailgun.


### Mailgun: Sending API key & SMPT User

Go to "_Sending_" -> "_Domain Settings_" page. Choose the "_Sending API keys_" tab at the top. Probably you won't see any API keys there. You just need to create a new Sending API key. Click "Add sending key" from the top right corner, and in the popup fill the name of the key you about to create.

<img alt="Mailgun: creating a Sending API key" src="https://i.imgur.com/FqBEgYp.png">

After pressing "Create sending key", you'll get the secret API key that you need to copy and save somewhere safe. After saving the key, you can just close the popup.
You'll see the created key listed:

<img alt="Mailgun: Sending API key created" src="https://i.imgur.com/bjneG7e.png">

You also need to create a new SMTP user in Mailgun dashbaord.
Go to "_Sending_" -> "_Domain Settings_" page. Choose the "_SMTP credentials_" tab at the top and press "Add new SMTP user" button on the top left corner. It will open up a popup. Type user credentials there. In our case I'll create a user with the name "_email_". It will be like a login for your email on Gmail.

<img alt="Mailgun: creating SMTP user" src="https://i.imgur.com/w1XQAiW.png">

Once you create an SMTP user in Mailgun, you'll see it listed and a password for that user will be generated automatically. To get this password, copy it by clicking the "Copy" button in the pop-up notification in the lower right corner.

<img alt="Mailgun: SMTP user created" src="https://i.imgur.com/sc3o62Z.png">

Keep this in a safe place for future use. You will need this login and password to authenticate on Gmail.

You are now ready to set up email configurations with your email provider. In our case, we will do this in Gmail.
Open your Gmail account in your desktop browser and go to Settings by clicking the settings icon in the top right corner and click the "See all settings" button.

<img alt="Mailgun: new domain is verified" src="https://i.imgur.com/CIffzot.png">


### Gmail Authentication with Mailgun SMTP Server

In the Gmail settings page choose the "_Accounts and Import_" tab and click on the "Add another email address" from the "_Send mail as_" section:

<img alt="Gmail: Settings" src="https://i.imgur.com/TmeLLP9.png">

It will open a popup for the authentication. Use the login and the password you just got by creating an SMTP user on Mailgun. Make sure to fill out the credentials correctly.

<img alt="Gmail: authenticate a new user using a created SMTP server on Mailgun" src="https://i.imgur.com/olxBStq.png">

Submit the form by clicking the "Add Account" button.
It probably will ask you to save the username/password in your browser. It's up to you.
And the last important thing here, that it will ask you to verify adding an account.

<img alt="Gmail: authentication confirmation for a new user" src="https://i.imgur.com/WcOZGMA.png">

For the verification, the confirmation email will be sent to your primary email.

<img alt="Gmail: authentication verification email" src="https://i.imgur.com/SpwMVc4.png">

You can either use the confirmation code to verify it using the pop-up window or simply follow the link provided in the confirmation email.
In this case, we'll click on a link that will open the page, where will be asked to confirm. Click on "Confirm" and simply close the previously opened pop-up window without worrying.

<img alt="Gmail: verifying the authentication" src="https://i.imgur.com/3nVbH1T.png">

Now you're ready to send and receive emails from the custom email you just created.

For sending an email from the custom email, you just need to choose that email as a sender email:

<img alt="Gmail: sending emails" src="https://i.imgur.com/gMkmDKW.png">

**That's it!**

An additional thing that may be useful to you is that you can set the custom email address you just created as the default address for sending emails from Gmail.
You can set this on the settings page in the "_Send mail as_" section:

<img alt="Gmail: Settings (default sender)" src="https://i.imgur.com/bOPXRmt.png">

I hope this guide will be a good resource for setting up your custom email.


### Conclusion

In this article, you learned how to set up your own email to manage emails in Gmail using Cloudflare Email and Mailgun.
In conclusion, it is worth noting that this choice of tools is not mandatory, other tools could be used instead, but the basic idea and logic would be similar.

You can check out my website at: [**boolfalse.com**](https://boolfalse.com/)

Feel free to share this article. 😇
