# TShark Challenge I: Teamwork

You get a pcap and have to dig through it with TShark to figure out what phishing site a user hit and what they gave away.

## What I did

Looked at the DNS queries first:

```
tshark -r teamwork.pcap -T fields -e dns.qry.name | sort | uniq -c
```

One domain was obviously fake. It had a well-known brand stuck at the front of a subdomain on some unrelated site. Threw it into VirusTotal to get the first-submission date and who flagged it.

For the email address I tried grepping for `@` and got nothing. Turns out the victim typed it into a login form, so it was sent URL-encoded and the `@` shows up as `%40`. Grepping for POST requests found the login form:

```
tshark -r teamwork.pcap | grep POST
```

Then I opened that frame with `-V` to read the form data:

```
tshark -r teamwork.pcap -Y "frame.number == <N>" -V
```

## Notes to self

- `grep` + `-V` on one frame gets you most of the way.
- If a search turns up nothing, the data might just be encoded (`%40`, base64, etc).
- Read hostnames right to left. The real domain is at the end.
