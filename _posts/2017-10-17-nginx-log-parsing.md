---
layout: post
title: Parsing logs with grep, awk, sed, and friends.
---

This post is about parsing an nginx log file to get information about the number of IPs that connected, frequency of certain HTTP requests and responses, etc. Basically, these are notes on grep, awk, sed, sort, and uniq.

Have to extract and process a bunch of info from an nginx log file, so I'm gonna learn a bit of awk.
(awk tutorial)[http://www.hcs.harvard.edu/~dholland/computers/awk.html]

The access log uses the default combined format, as described (here)[https://www.nginx.com/resources/admin-guide/logging-and-monitoring/].
```
log_format compression '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" "$gzip_ratio"';
```
On inspection, this log doesn't seem to include the gzip_ratio, and the referer seems to almost always be "-".

First, we need the number of unique IPs that reached the server. The remote address is the first column. We'll extract the first column then pipe it to sort -u.
```
awk < access.log '{ print $1 }' | sort -u | wc -l
```
NB, this works too:
```
awk '{ print $1 }' access.log
```
As does:
```
cat access.log | awk '{ print $1 }'
```

Next questions, how many 200 and 400 status responses. The status is after the request, and the request can have lots of spaces in it (whitespace is the default separator for awk), so I'm not sure if I can extract the status as a column easily with awk.
`awk < access.log '{ print $(NF - 3) }'` gives me the status EXCEPT when there's a user-agent string with spaces. NF is the number of fields in the line.

So, let's grep instead.
```
grep \b200\b access.log
```
No results. \b supposed to be word boundaries...
```
grep 200 access.log
```
works, but doesn't guarantee that it's the status. What if an IP or request or user-agent contains 200? Mind you, searching with word boundaries doesn't rule that out...
Aha! -w is the option to find a word enclosed by spaces.
```
grep -w 200 access.log
```
But 200 could also show up as the body_bytes_sent, so I can't just count those output lines. What I really wanna do is first strip out any strings in quotes. I could do that with a regex that matches from an opening quote to the first closing quote (not greedy, trying to match as much as it can, because that would get from the start of the request to the end of the user-agent), using sed. If I replaced all those strings with "-", I could maintain the number of columns corresponding to the fields of the log output format, then use awk to pull out the status column specifically.
sed starting point:
```
sed 's/"1234_NAME=TRUE",//g' "$file" > "$file.modified"
```

The regex I want to produce is approximately "a quote, 1 or more of any character, until another quote".
From regular-expressions.info, `"[^"\r\n]*"` is the appropriate way to match a string between double quotes. `".*"` is greedy, so it would match until the final quote it finds. Instead, you should use negated character classes to specify that you want to match any character that isn't a double quote or a line break. In my use case, it would be sufficient to exclude quotes, I believe.
```
sed 's/"[^"]*"/"-"/g' access.log > nostrings.log
```
BAM!
A line from nostring.log:
```
104.245.97.236 - - [29/Sep/2015:21:15:18 -0400] "-" 404 162 "-" "-"
```
The status is column 7, because the date takes up two columns with the space.
```
awk < nostrings.log '{ print $7 }' | grep 200 | wc -l
```
Using my new knowledge about other ways to give input to awk, I can make a one-liner:
```
sed 's/"[^"]*"/"-"/g' access.log | awk '{ print $7 }' | grep 200 | wc -l
```

Now I want to count the Firefox versions. Firefox shows up in the user-agent as Firefox/31.0 or Firefox/3.6.2 (or other versions). I want to use grep to just output the name and version. -o prints only the matched parts of a matching line. So I need a regex that matches the name and version. I could match a number plus 1 or more groups of a period and a number, but in this case, the Firefox version is at the very end of the user-agent and the next character is a quote, so I'm going to match `Firefox/[^"]*`.
```
grep -o 'Firefox/[^"]*' access.log
```
Aha! My laziness got the better of me. That matched more than I expected, with lines like this:
`Firefox/3.6.28 (.NET CLR 3.5.30729)`

That still looks like version info, but it demonstrates that I should use a more exact regex if I want to just match x.x[.x].

To count the versions programatically:
```
grep -o 'Firefox/[^"]*' access.log | sort | uniq -c
```
To print them in descending order:
```
grep -o 'Firefox/[^"]*' access.log | sort | uniq -c | sort -nr
```

We can use a similar command to count the HTTP methods:
```
awk '{ print $6 }' access.log | sort | uniq -c | sort -nr
```
