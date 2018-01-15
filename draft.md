---
title: Transclude Preference for the HTTP Prefer Header
abbrev: Transclude for HTTP Prefer
docname: inadarei-prefer-transclude
date: 2018
category: info

ipr: trust200902
area: General
workgroup:
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: I. Nadareishvili
    name: Irakli Nadareishvili
    street: 114 5th Avenue
    city: New York
    country: United States
    email: irakli@gmail.com
    uri: http://www.freshblurbs.com/
 -
    ins: E. Wilde
    name: Erik Wilde
    email: dret@ca.com
    uri: http://dret.net/netdret/

normative:
  RFC5988:
  RFC2119:
  RFC7240:

informative:
  # RFC7230:
  # RFC6838:

--- abstract

The Transclude preference is a registered behavior for the HTTP Prefer header 
field. Its purpose is for clients to specify preferences that certain links in 
a response should be resolved by the server, instead of being served as links 
that the client can resolve.

--- note_Note_to_Readers

**RFC EDITOR: please remove this section before publication**

The issues list for this draft can be found at <https://github.com/inadarei/draft-prefer-transclude/issues>.

The most recent draft is at <https://inadarei.github.io/draft-prefer-transclude/>.

--- middle

# Introduction

"Prefer Header for HTTP" {{RFC7240}} proposes an HTTP header that can be used 
to indicate that particular server behaviors are preferred by the client but 
are not required for successful completion of the request. It further defines 
several standard Preferences, such as the "return" preference. The "return" 
preference lets the server know that the client would prefer a specific 
representation of a resource in a response payload, e.g. full representation 
vs. a minimal one.

Preferences like the "return" one are critical for mobile scenarios as mobile 
applications are very sensitive to network latency, throughput and anything 
that can improve end-user experience, even on resource-constrained networks. 
Prefer header allows servers to tune and optimize its response payload based 
on client preferences, for instance: to only send mobile app a minimal response 
when it doesn't need full resource.

The size of the payload is not the only preference that can improve 
user-experience in mobile scenarios, however. Closely related to the size of 
the payload, is the number of HTTP requests a client needs to make to get all 
of the required data. This is the challenge that "transclude" preference 
addresses.

When server sends hypermedia responses (e.g. in the case of Hypermedia APIs) 
some of the response data may be referenced via a URI link instead of being 
embedded in the payload itself. The need to grab data from a link can degrade 
experience of mobile applications, since they are forced to make multiple 
requests to per single end-user request. This is sometimes referred to as 
"chatty interface" and is a significant problem for mobile and Internet of 
Things scenarios.

Transclude preference notifies the server that the client would prefer the
server to proactively transclude certain content represented by links of
indicated link relation types. The notion of "link relation type", in this
context is as defined by {{RFC5988}}

As a result of using a transclude preference, a client receives all of the
required data already embedded in the response output, without the need to make
additional network calls.

# Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{RFC2119}}.

# Transclude Usage Example

Following is an example of a client asking server to transclude data represented 
at the copyright, edit-form and "other-form" links. Since "other-form" is not 
a standard, IANA link relation type, client is using a URI for identifying the 
link relation type.

~~~
Get /blog/1223 HTTP/1.1
Host: api.example.org
Content-Type: application/json  
Prefer: transclude="copyright;edit-form;http://rels.io/other-form"
Vary: Prefer,Accept,Accept-Encoding
~~~

As you can see from the example, the transclude preference expects the value 
to be enclosed in double quotes, if there are multiple link relation types 
provided. Further, standard link relations SHOULD be indicated by name while 
custom link relation types SHOULD be indicated with a unique URI representing 
that link relation. Multiple link relation types MUST be separated by 
semicolons.

Example response may look something like the following:

~~~
HTTP/1.1 200 OK
Server: nginx/1.10.3 (Ubuntu)
Date: Sat, 27 Jun 2015 11:03:32 GMT
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: close
Vary: Prefer,Accept,Accept-Encoding
Preference-Applied: transclude="copyright;edit-form"
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Content-Type
~~~

As you can see from this output, the server only transcluded copyright and 
edit-form link relation types, but not the custom type client requested. This 
is because preferences are just suggestions and server has no obligation related 
to them. In this case, we can assume that the server skipped the last link 
relation type because maybe it wasn't familiar with it, or for some other 
reason.

# Implementation Considerations

Transclude preference is media-type agnostic. It should work with any response 
content-type. The mechanics of transclusion, however will either depend on 
capabilities of the response media-type or require a multi-part response with 
multiple media types in the response.

# Multipart Response Example

~~~
HTTP/1.1 200 OK
Server: nginx/1.4.6 (Ubuntu)
Date: Thu, 15 Sep 2016 11:03:32 GMT
Content-type: multipart/mixed; boundary="some boundary string"

--some boundary string
Content-Type: application/hal+json; charset=utf-8

{
    "_links": {
        "self": { "href": "/orders" },
        "edit-form": { "href": "/create" }
    },
    "currentlyProcessing": 14,
    "shippedToday": 20
}

--some boundary string
Content-Type: application/prs.hal-forms+json

{
    "_links" : {
        "self" : { "href" : "/create" }
    },
    "_templates" : {
        "default" : {
            "title" : "Create",
            "method" : "post",
            "contentType" : "application/json",
            "properties" : [
                {"name": "title", "required": true, 
                 "prompt": "Title"},
                {"name": "completed", "value": "false", 
                 "prompt": "Completed"}
            ]
        }
    }
}
--some boundary string--
~~~

# Security Considerations

None.

# IANA Considerations

The HTTP Preference below is being registered with IANA per Section 5.1 of 
{{RFC7240}}:

Preference: transclude
Value:
Optional Parameters:
Description:
Reference: RFC XXXX

--- back

# Acknowledgments