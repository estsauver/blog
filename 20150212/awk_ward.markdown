I was writing a `headers.rs` file for the toy server I'm writing, and I was manually writing out struct element definitions for the headers. And you may note that's a little bit tiring to do by hand. 

Then I remembered, oh yah, we can make computers do these things for us.

The headers we're interested in are available here as a csv:

http://www.iana.org/assignments/message-headers/message-headers.xml#perm-headers

First, we're looking only for the "standard" headers really. The struct definition is an optimization, and so it doesn't make sense to use every word anyone's ever tucked into an http request. 

```
cat perm-headers.csv \
| grep "standard"
```

and we're not trying to add default support for netnews or mial, so let's actually filter those out.

```
cat perm-headers.csv \
| grep "standard" \
| grep -v "netnews|mail"
```

Now we have line's that look like this:
```
Sec-WebSocket-Accept,,http,standard,[RFC6455]
```

So, we need the first element of each row of the csv. Let's fire up awk

```
cat perm-headers.csv \
| grep "standard" \
| grep -v "netnews|mail" \
| awk -F "," '{print $1}'
```

But it would also be great to get these into camel cased form for our struct definitions. But that's easy enough!

First we replace the dashes with underscores 
```
cat perm-headers.csv \
| grep "standard" \
| grep -v "netnews|mail" \
| awk -F "," '{print $1}' \
| sed s/\-/\_/g \
```
Then we lowercase everything
```
cat perm-headers.csv \
| grep "standard" \
| grep -v "netnews|mail" \
| awk -F "," '{print $1}' \
| sed s/\-/\_/g \
| awk '{print tolower($0)}' \
```

And then, since we've already come so far, let's also add the rest of the field definition at the end of the line. 

```
cat perm-headers.csv \
| grep "standard" \
| grep -v "netnews|mail" \
| awk -F "," '{print $1}' \
| sed s/\-/\_/g \
| awk '{print tolower($0)}' \
| awk '{print $1 ": Option<&str>,"}' > headers.tmp
```

Now, the output looks like

```
...
accept_charset: Option<&str>,
accept_encoding: Option<&str>,
accept_language: Option<&str>,
accept_ranges: Option<&str>,
...
```

And there we end up with a list of all of the standard headers we want. But, that list also leaves out a few headers that we want from the provisional headers. That's easy enough, let's grab all the http headers and append them to our file

```
cat prov-headers.csv \
| grep "http" \
| awk -F "," '{print $1}' \
| sed s/\-/\_/g \
| awk '{print tolower($0)}' \
| awk '{print $1 ": Option<&str>,"}' >> headers.tmp
```
Then, after sorting these
```
sort headers.tmp > sorted_headers.tmp
```
We have a very neat basis for a headers class. 

