---
title: Tracking Organizations in Vertex Synapse
tags: synapse storm
---

In this blog post, I will provide insight into tracking organizations using the
Vertex [Synapse](https://vertex.link/synapse) system. This centralized
intelligence platform will store all important data and intelligence. The core
component of the system is open source.

Structured data and data modeling enables us to gain a deeper comprehension of
the world and how it works. We can query our system to uncover correlations,
retrieve information or create statistics, warnings, etc. In Synapse, for
example, I model organizations with `ou:org` type which is referred to as "A
human organization such as a company or military unit."

With `ou:org`, there is an abundance of properties available. But rather than
being overwhelmed by the multitude of possibilities, focus on your specific
needs to determine what fields would be essential for you to record information
and answer any future questions. Utilize your primary intelligence and
collection requirements to identify which elements you should prioritize to
achieve optimal results.

| Property   | Description                                                                  | Type        |
| ---------- | ---------------------------------------------------------------------------- | ----------- |
| .seen      | The time interval for first/last observation of the node.                    | ival        |
| loc        | Location for an organization.                                                | loc         |
| name       | The localized name of an organization.                                       | ou:name     |
| type       | The type of organization.                                                    | str         |
| orgtype    | The type of organization.                                                    | ou:orgtype  |
| vitals     | The most recent/accurate ou:vitals for the org.                              | ou:vitals   |
| desc       | A description of the org.                                                    | str         |
| logo       | An image file representing the logo for the organization.                    | file:bytes  |
| names      | A list of alternate names for the organization.                              | array       |
| alias      | The default alias for an organization.                                       | ou:alias    |
| phone      | The primary phone number for the organization.                               | tel:phone   |
| sic        | The Standard Industrial Classification code for the organization.            | ou:sic      |
| naics      | The North American Industry Classification System code for the organization. | ou:naics    |
| industries | The industries associated with the org.                                      | array       |
| us:cage    | The Commercial and Government Entity (CAGE) code for the organization.       | gov:us:cage |
| founded    | The date on which the org was founded.                                       | time        |
| dissolved  | The date on which the org was dissolved.                                     | time        |
| url        | The primary url for the organization.                                        | inet:url    |
| subs       | An set of sub-organizations.                                                 | array       |
| orgchart   | The root node for an orgchart made up ou:position nodes.                     | ou:position |
| hq         | A collection of contact information for the "main office" of an org.         | ps:contact  |
| locations  | An array of contacts for facilities operated by the org.                     | array       |
| dns:mx     | An array of MX domains used by email addresses issued by the org.            | array       |
| goals      | The assessed goals of the organization.                                      | array       |

It's interesting to note the distinction between `type` and `orgtype`, as both
captures essentially the same thing. The `ou:orgtype` type captures an
organization-type taxonomy with a more specific set of values than what is
found in the free text field. Having such constrained values can be extremely
useful when creating statistics; for example, you could define types such as
corporation, non-profit organization, government agency, or cooperative.

The `ou:org` type is based on a GUID, so finding a unique identifier for
deconfliction can prove difficult. However, we can use an alternative property
- such as the organization's name – to identify if it already exists when
importing or creating our organizations. This saves us from having to conduct
searches every time! To achieve this goal, Synapse proposes GUID generators
driven by an extra property. We are particularly interested in the
`$lib.gen.orgByName(name)` generator which will generate an ou: org with the
specified name if it does not already exist yet. Furthermore, another generator
of interest is `$lib.gen.orgByFqdn(fqdn)`, which returns a generated
organization node based on a domain used for emailing purposes - usually taken
from an email address itself!

As an initial case study, let's discuss my enterprise Cyantlabs. The following
steps will generate the node (or retrieve it if it has already been created).

```
storm> $lib.gen.orgByName("cyantlabs")
```

Afterward, we can broaden the node's properties with more detailed information.

```
storm> ou:org:name="cyantlabs" [ loc="be.bru.brussels" dns:mx=$lib.list("cyantlabs.com") ]
```

By introducing the organization, we can now transition from an individual email
address to that of the company. For instance

```
storm> inet:email="spam@cyantlabs.com" -> inet:fqdn -> ou:org
```

We can also utilize the second generator based off of a domain name. For
instance, after starting with an email address, we acquire its related domain
to subsequently create (if necessary) the organization node.

```
storm> [ inet:email="info@nviso.eu" ] -> inet:fqdn $lib.gen.orgByFqdn($node)
```

An intriguing element is the 'loc' datatype, which forms a hierarchy with
dot-separated strings. For instance, we can easily fetch all Belgian
organizations stored in our system:

```
storm> ou:org:loc=be.*
```

or organizations situated in the Brussels region

```
storm> ou:org:loc=be.bru.*

ou:org=8a9d383c25ebf82bd710b274e75ef5fd
    .created=2023/02/04 01:03:05.826
    name=nviso
    loc=be.bru.etterbeek
    dns:mx=(nviso.eu)
ou:org=17c8a7cdaae78909c954a28ea7cec2b9
    .created=2023/02/01 23:21:06.839
    dns:mx=(cyantlabs.com)
    name=cyantlabs
    loc=be.bru.schaerbeek
```

By tracking organizations in Vertex Synapse, you can leverage the `ou:org` node
and its various properties to effectively manage your organization roster. Once
the nodes are created, several useful properties can be utilized - such as
location and DNS MX records - to gain a better understanding of the
organizations you're tracking. As this article shows, Vertex Synapse provides
an intuitive way to record and monitor your (cyber) structured data.

Happy modeling! :)