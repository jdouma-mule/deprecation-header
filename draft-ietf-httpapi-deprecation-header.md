---
coding: utf-8
abbrev:
title: The Deprecation HTTP Header Field
docname: draft-ietf-httpapi-deprecation-header-latest
category: std

ipr: trust200902
area: Applications and Real-Time
workgroup: HTTPAPI
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

venue:
  group: HTTPAPI
  type: Working Group
  home: https://ietf-wg-httpapi.github.io/
  mail: httpapi@ietf.org
  repo: https://github.com/ietf-wg-httpapi/deprecation-header

author:
  -
    ins: S. Dalal
    name: Sanjay Dalal
    email: sanjay.dalal@cal.berkeley.edu
    uri: https://github.com/sdatspun2
  -
    ins: E. Wilde
    name: Erik Wilde
    email: erik.wilde@dret.net
    uri: http://dret.net/netdret

normative:
  HTTP: RFC9110
  LINK: RFC8288


informative:


--- abstract

The Deprecation HTTP response header field is used to signal to consumers of a URI-identified resource that the resource will be or has been deprecated. Additionally, the deprecation link relation can be used to link to a resource that provides additional information about planned or existing deprecation, and possibly ways in which clients can best manage deprecation.


--- middle



# Introduction

Deprecation of an HTTP resource ({{Section 3.1 of HTTP}}) communicates information about the lifecycle of a resource. It encourages applications to migrate away from the resource, discourages applications from forming new dependencies on the resource, and informs applications about the risk of continued dependence upon the resource.

The act of deprecation does not change any behavior of the resource. It informs clients of the fact that a resource will be or is deprecated. The Deprecation HTTP response header field can be used to convey this at runtime to clients and carries information indicating when the deprecation will be in effect.

In addition to the Deprecation header field, the resource provider can use other header fields to convey additional information related to deprecation. This can be information such as where to find documentation related to the deprecation, what can be used as a replacement, and when a deprecated resource becomes non-operational.


##  Notational Conventions

{::boilerplate bcp14-tagged}

This specification uses the Augmented Backus-Naur Form (ABNF) notation of {{!RFC5234}} and includes, by reference, the IMF-fixdate rule as defined in {{Section 5.6.7 of HTTP}}.

The term "resource" is to be interpreted as defined in {{Section 3.1 of HTTP}}.

# The Deprecation HTTP Response Header Field

The `Deprecation` HTTP response header field allows a server to communicate to a client that the resource in context of the message is or will be deprecated.

## Syntax

The `Deprecation` response header field describes the deprecation of the resource identified with the response it occurred within (see {{Section 6.4.2 of HTTP}}). It conveys the deprecation date, which may be in the future (the resource context will be deprecated at that date) or in the past (the resource context has been deprecated at that date).

~~~ abnf
Deprecation = ISO-8601
~~~

Servers MUST NOT include more than one `Deprecation` header field in the same response.

The date is the date when the resource was or will be deprecated. It is in the form of an ISO-8601 timestamp.

### Example 1

The following example shows that the resource context has been deprecated on Sunday, November 11, 2018 at 23:59:59 Greenwich Mean Time (GMT):

    Deprecation: 2018-11-11T23:59:59Z
    
### Example 2

The following example shows that the resource context will be deprecated on Saturday, November 11, 2045 at 04:00:00 Pacific Standard Time (PST):

    Deprecation: 2023-01-02T04:00:00-08:00

The deprecation date can be in the future. This means that the resource will be deprecated at the indicated date in the future.


## Scope

The `Deprecation` header field applies to the resource identified with the response it occurred within (see {{Section 6.4.2 of HTTP}}), meaning that it announces the upcoming deprecation of that specific resource. However, there may be scenarios where the scope of the announced deprecation is larger than just the single resource where it appears.

Resources are free to define such an increased scope, and usually this scope will be documented by the resource so that consumers of the resource know about the increased scope and can behave accordingly. When doing so, it is important to take into account that such increased scoping is invisible for consumers who are unaware of the increased scoping rules. This means that these consumers will not be aware of the increased scope, and they will not interpret deprecation information different from its standard meaning (i.e., it applies to the resource only).

Using such an increased scope still may make sense, as deprecation information is only a hint anyway. It is optional information that cannot be depended on, and clients should always be implemented in ways that allow them to function without Deprecation information. Increased scope information may help clients to glean additional hints from related resources and, thus, might allow them to implement behavior that allows them to make educated guesses about resources becoming deprecated.

For example, an API might not use `Deprecation` header fields on all of its resources, but only on designated resources such as the API's home document. This means that deprecation information is available, but in order to get it, clients have to periodically inspect the home document. In this example, the extended context of the `Deprecation` header field would be all resources provided by the API, while the visibility of the information would only be on the home document.


# The Deprecation Link Relation Type

In addition to the `Deprecation` HTTP header field, the server can use links with the "deprecation" link relation type to communicate to the client where to find more information about deprecation of the context. This can happen before the actual deprecation, to make a deprecation policy discoverable, or after deprecation, when there may be documentation about the deprecation, and possibly documentation of how to manage it.

This specification places no restrictions on the representation of the linked deprecation policy. In particular, the deprecation policy may be available as human-readable documentation or as machine-readable description.


## Documentation

The purpose of the `Deprecation` header field is to provide a hint about deprecation to the resource consumer. Upon reception of the `Deprecation` header field, the client developer can look up the resource's documentation in order to find deprecation related information. The resource provider can provide a link to the resource documentation using a `Link` header field with relation type `deprecation` as shown below:

    Link: <https://developer.example.com/deprecation>;
          rel="deprecation"; type="text/html"

In this example the linked content provides additional information about deprecation of the resource context. There is no Deprecation header field in the response, and thus the resource is not (yet) deprecated. However, the resource already exposes a link where information is available how deprecation is managed for the resource context. This may be documentation explaining the use of the `Deprecation` header field, and also explaining under which circumstances and with which policies (announcement before deprecation; continued operation after deprecation) deprecation might be happening.

The following example uses the same link header field, but also announces a deprecation date using a `Deprecation` header field:

    Deprecation: 2018-11-11T23:59:59Z
    Link: <https://developer.example.com/deprecation>;
          rel="deprecation"; type="text/html"

Given that the deprecation date is in the past, the linked information resource may have been updated to include information about the deprecation, allowing consumers to discover information about the deprecation and how to best manage it.

Consider another scenario: a resource has been marked for deprecation to consolidate functionality, data,... in another resource. However, the effort for a consuming client to revert to a new resource can be limited to only pointing to the new resource if the API contract _itself_ does not constitute a breaking change. The `Depreciation` header, in conjunction with the relation type 'deprecation' can then naturally also be used to implement _Hypertext as Engine of Application State_ (HATEOAS) concepts, pointing clients to a new resource (being) implemented which can preemptively be tested, either manually or as part of automated testing, to assess the impact.

    Deprecation: 2018-11-11T23:59:59Z
    Link: <https://api.example.com/new-resource>;
          rel="deprecation"; type="application/json"

# Sunset

In addition to the deprecation related information, if the resource provider wants to convey to the client application that the deprecated resource is expected to become unresponsive at a specific point in time, the Sunset HTTP header field {{?RFC8594}} can be used in addition to the `Deprecation` header field.

The timestamp given in the `Sunset` header field MUST NOT be earlier than the one given in the `Deprecation` header field.

The following example shows that the resource in context has been deprecated since Sunday, November 11, 2018 at 23:59:59 GMT and its sunset date is Wednesday, November 11, 2020 at 23:59:59 GMT.

    Deprecation: 2018-11-11T23:59:59Z
    Sunset: 2020-11-11T23:59:59Z

# Resource Behavior

The act of deprecation does not change any behavior of the resource. Deprecated resources SHOULD keep functioning as before, allowing consumers to still use the resources in the same way as they did before the resources were declared deprecated.

# IANA Considerations

## The Deprecation HTTP Response Header Field

The `Deprecation` response header field should be added to the permanent registry of message header fields (see {{!RFC3864}}).

    Header Field Name: Deprecation

    Applicable Protocol: Hypertext Transfer Protocol (HTTP)

    Status: Standard

    Author: Sanjay Dalal <sanjay.dalal@cal.berkeley.edu>,
            Erik Wilde <erik.wilde@dret.net>

    Change controller: IETF

    Specification document: this specification,
                Section 2 "The Deprecation HTTP Response Header Field"



## The Deprecation Link Relation Type

The `deprecation` link relation type should be added to the permanent registry of link relation types ({{Section 4.2 of LINK}}).

    Relation Type: deprecation

    Applicable Protocol: Hypertext Transfer Protocol (HTTP)

    Status: Standard

    Author: Sanjay Dalal <sanjay.dalal@cal.berkeley.edu>,
            Erik Wilde <erik.wilde@dret.net>

    Change controller: IETF

    Specification document: this specification,
            Section 3 "The Deprecation Link Relation Type"



# Implementation Status

Note to RFC Editor: Please remove this section before publication.

This section records the status of known implementations of the protocol defined by this specification at the time of posting of this Internet-Draft, and is based on a proposal described in {{?RFC7942}}.  The description of implementations in this section is intended to assist the IETF in its decision processes in progressing drafts to RFCs.  Please note that the listing of any individual implementation here does not imply endorsement by the IETF. Furthermore, no effort has been spent to verify the information presented here that was supplied by IETF contributors.  This is not intended as, and must not be construed to be, a catalog of available implementations or their features.  Readers are advised to note that
other implementations may exist.

According to RFC 7942, "this will allow reviewers and working groups to assign due consideration to documents that have the benefit of running code, which may serve as evidence of valuable experimentation and feedback that have made the implemented protocols more mature. It is up to the individual working groups to use this information as they see fit".

## Implementing the Deprecation Header Field

This is a list of implementations that implement the deprecation header field:

Organization: Apollo

- Description: Deprecation header field is returned when deprecated functionality (as declared in the GraphQL schema) is accessed
- Reference: https://www.npmjs.com/package/apollo-server-tools

Organization: Zalando

- Description: Deprecation header field is recommended as the preferred way to communicate API deprecation in Zalando API designs.
- Reference: https://opensource.zalando.com/restful-api-guidelines/#deprecation

Organization: Palantir Technologies

- Description: Deprecation header field is incorporated in code generated by conjure-java, a CLI to generate Java POJOs and interfaces from Conjure API definitions
- Reference: https://github.com/palantir/conjure-java

Organization: IETF Internet Draft, Registration Protocols Extensions

- Description: Deprecation link relation is returned in Registration Data Access Protocol (RDAP) notices to indicate deprecation of jCard in favor of JSContact.
- Reference: https://tools.ietf.org/html/draft-loffredo-regext-rdap-jcard-deprecation

Organization:  E-Voyageurs Technologies

* Description: Deprecation header field is incorporated in Hesperides, a configuration management tool providing universal text file templating and properties editing through a REST API or a webapp.
* Reference: https://github.com/voyages-sncf-technologies/hesperides/blob/master/documentation/lightweight-architecture-decision-records/deprecated_endpoints.md

Organization: Open-Xchange

* Description: Deprecation header field is used in Open-Xchange appsuite-middleware
* Reference: https://github.com/open-xchange/appsuite-middleware

Organization: MediaWiki

* Description: Core REST API of MediaWiki would use Deprecation header field for endpoints that have been deprecated because a new endpoint provides the same or better functionality.
* Reference: https://phabricator.wikimedia.org/T232485



## Implementing the Concept

This is a list of implementations that implement the general concept, but do so using different mechanisms:

Organization: Zapier

- Description: Zapier uses two custom HTTP header fields named `X-API-Deprecation-Date` and `X-API-Deprecation-Info`
- Reference:  https://zapier.com/engineering/api-geriatrics/

Organization: IBM

- Description: IBM uses a custom HTTP header field named `Deprecated`
- Reference: https://www.ibm.com/support/knowledgecenter/en/SS42VS_7.3.1/com.ibm.qradar.doc/c_rest_api_getting_started.html

Organization: Ultipro

- Description: Ultipro uses the HTTP `Warning` header field as described in Section 5.5 of {{!RFC7234}} with code `299`
- Reference:  https://connect.ultipro.com/api-deprecation

Organization: Clearbit

- Description: Clearbit uses a custom HTTP header field named `X-API-Warn`
- Reference: https://blog.clearbit.com/dealing-with-deprecation/

Organization: PayPal

- Description: PayPal uses a custom HTTP header field named `PayPal-Deprecated`
- Reference: https://github.com/paypal/api-standards/blob/master/api-style-guide.md#runtime


# Security Considerations

The Deprecation header field SHOULD be treated as a hint, meaning that the resource is indicating (and not guaranteeing with certainty) that it will be or is deprecated. Applications consuming the resource SHOULD check the resource documentation to verify authenticity and accuracy. Resource documentation SHOULD provide additional information about the deprecation, potentially including recommendation(s) for replacement.

In cases where the Deprecation header field value is a date in the future, it can lead to information that otherwise might not be available. Therefore, applications consuming the resource SHOULD verify the resource documentation and if possible, consult the resource developer to discuss potential impact due to deprecation and plan for possible transition to recommended resource.

In cases where a `Link` header field is used to provide documentation, one should assume that the content of the  `Link` header field may not be secure, private or integrity-guaranteed, and due caution should be exercised when using it. Applications consuming the resource SHOULD check the referred resource documentation to verify authenticity and accuracy.


# Examples

The following examples do not show complete HTTP interactions. They only show those HTTP header fields in a response that are relevant for resource deprecation.

The first example shows a deprecation header field with date information:

    Deprecation: 2018-11-11T23:59:59Z

The second example shows a deprecation header field with a link for the resource's deprecation policy. In addition, it shows the sunset date for the deprecated resource:

    Deprecation: 2018-11-11T23:59:59Z
    Sunset: 2020-11-11T23:59:59Z
    Link: <https://developer.example.com/deprecation>; rel="deprecation"
    
## Migration to a new resource

Consider the following scenario: a client application is accessing a resource to retrieve an Employee User Name based on the Employee ID. On Jan 1, 2023, the `Deprecation` and `Link` header were added to the response to indicate future deprecation and point the client application and developer to relevant documentation and the new resource which will implement (part of) the functionality of the to-be deprecated resource.

    GET /employees/445667 HTTP/1.1
    Host: api.example.com

    HTTP/1.1 200 OK
    Host: api.example.com
    Deprecation: 2023-11-11T23:59:59Z
    Link: <https://api.example.com/new-resource>;
          rel="deprecation"; type="application/json"
          
    {
        "employeeId" : "445667",
        "employeeUserName" : "jsmith@example.com"
    }





--- back


# Acknowledgments


The authors would like to thank Nikhil Kolekar, Darrel Miller, Mark Nottingham, and Roberto Polli for their contributions.

The authors take all responsibility for errors and omissions.
