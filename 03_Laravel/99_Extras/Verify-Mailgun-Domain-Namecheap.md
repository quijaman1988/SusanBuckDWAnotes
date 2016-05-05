## Verifying a domain in MailGun with Namecheap

In Mailgun, if you attempt to verify a domain, it will ask you to set up the following records with your DNS provider:

1. A SPF `TXT` record
2. A DKIM `TXT` record

__SPF (Sender Policy Framework)__ and __DKIM (DomainKeys Identified Mail)__ are systems used to verify domain ownership and prevent email spoofing. When you send email from your domain, the receiver will check your SPF/DKIM settings to make sure your domain us authorized to send email from that domain.

The information Mailgun provides you for the SPF/DKIM records needs some adjustments before they'll work with Namecheap.



### SPF
First, let's focus on the SPF record.

Mailgun tells you the following details:

+ Type: `txt`
+ Host: `dwa15-practice.biz`
+ Value: `v=spf1 include:mailgun.org ~all`

What needs to change is the hostname; for Namecheap it needs to be `@`.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-mailgun-spf@2x.png' style='max-width:1082px; width:100%' alt=''>



### DKIM
Next, let's focus on the DKIM record.

Mailgun tells you the following details:
+ Type: `txt`
+ Host: `mailo._domainkey.dwa15-practice.biz` (where `dwa15-practice.biz` is your domain)
+ Value: `k=rsa; p=LONGSTRINGOFCHARACTERS`

What needs to change is both the host and the value.

The __host__ should not include your domain; it should just be `mailo._domainkey`.

The __value__ needs to have `v=DKIM1;` prefixed to it. I.e. `k=rsa; p=LONGSTRINGOFCHARACTERS` becomes `v=DKIM1;  k=rsa; p=LONGSTRINGOFCHARACTERS`.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-mailgun-dkim@2x.png' style='max-width:1082px; width:100%' alt=''>

## CNAME
The third record Mailgun instructs you to create is a CNAME record. No changes are needed for that record so follow the instructions exactly as given.



## Verify
After you save the above settings, wait a few minutes, then click __Check DNS Records Now__ within MailGun to see if your settings have updated. If they have not, you may need to wait longer.



## Reference
The above information was adapted from this blog post: [MailGun and NameCheap: SPF and DKIM Validation](https://julienliabeuf.com/mailgun-namecheap-spf-dkim-validation/)
