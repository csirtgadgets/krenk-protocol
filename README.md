# 1. Goals

The KRENK protocol is an information disclosure protocol. It enables the sharing of intelligence by defining the rules in which intelligence can be, and should be shared amongst practitioners and/or automated systems. This protocol is based on the work by [the APWG](http://apwg.org/data-logistics-blog/data_logistics), [the IETF](http://tools.ietf.org/html/rfc5070), [the REN-ISAC](http://www.ren-isac.net/ses) and [the CSIRT Gadgets Foundation](http://csirtgadgets.org/rfc).

* Provide guidance if and how information should be shared
* Provide guidance for the circumstances in which information should not be shared

# 2. License

This Specification is free software; you can redistribute it and/or modify it under the terms of the GNU Lesser General Public License as published by the Free Software Foundation; either version 3 of the License, or (at your option) any later version.

This Specification is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License along with this program; if not, see <http://www.gnu.org/licenses>.

# 3. Change Process

This document is governed by the [Consensus-Oriented Specification System (COSS)](http://www.digistan.org/spec:1/COSS). In addition:

* Comments MUST be logged as [new issues](https://github.com/blog/411-github-issue-tracker).
* Contributions MUST be logged as [new pull-requests](https://help.github.com/articles/creating-a-pull-request).

# 4. Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

# 5. Protocol
## Krenk
```
+-----------------+
| Krenk           |
+-----------------+
| REAL version    |
| ENUM lang       |<>--{1..*}--[ Context        ]
| ENUM tlp        |<>--{0..1}--[ ReportTime     ]
| STRING ext-tlp  |<>--{0..1}--[ StartTime      ]
| UINT32 ttl      |<>--{0..1}--[ ExpirationTime ]
+-----------------+
```

#####*The aggregate classes that constitute Krenk are:*

### Context
Zero or many. Describes the context in which the data is represented. SHOULD be used to convey the care to be taken with the data.

### ReportTime
Zero or one. TIMESTAMP. A timestamp that represents when this data marking was generated.

### StartTime
Zero or one. TIMESTAMP. A timestamp that represents when the data marking takes effect.

### ExpirationTime
Zero or one. TIMESTAMP. A timestamp that represents when the data marking expires.

#####*The Krenk class has five attributes:*

### version
Required. REAL. The specification version number to which this class conforms. While the protocol itself conforms to a [semantic versioning](http://semver.org/), implemented, the protocol version should conform to a REAL (float/double) number. Examples:

* 0.0.1 SHALL BE implemented as 0.0010
* 1.0.01 SHALL BE implemented as 1.0001
* 2.1.2 SHALL BE implemented as 2.1020
* 0.01.01 SHALL BE implemented as 0.0101

Semver | REAL
0.0.1  | 0.0010
1.0.01 | 1.001


### lang
Optional.  ENUM.  A valid language code per RFC [4646](http://tools.ietf.org/html/rfc4646). The default is 'EN'.

### tlp
Optional. ENUM. This attribute indicates disclosure guidelines to which the sender expects the recipient to adhere for the information represented in this class and its children.  This guideline provides no security since there are no specified technical means to ensure that the recipient of the document handles the information as the sender requested.

The value of this attribute is logically inherited by the children of this class.  That is to say, the disclosure rules applied to this class, also apply to its children.

It is possible to set a granular disclosure policy. A child can override the guidelines of a parent class, be it to restrict or relax the disclosure rules (e.g., a child has a weaker policy than an ancestor; or an ancestor has a weak policy, and the children selectively apply more rigid controls).  The implicit value of the restriction attribute for a class that did not specify one can be found in the closest ancestor that did specify a value.

This attribute is defined as an enumerated value with a default value of "RED".  Note that the default value of the restriction attribute is only defined in the context of the Envelop class.  In other classes where this attribute is used, no default is specified.

Enumerated attributes MUST conform to the universally recognized Traffic Light Protocol[[1]](http://en.wikipedia.org/wiki/Traffic_Light_Protocol),[[2]](http://www.us-cert.gov/tlp):

1. **<font color="red">red.</font>** personal for named recipients only. In the context of a meeting, for example, RED information is limited to those present at the meeting. In most circumstances, RED information will be passed verbally or in person.
1. **<font color="amber">amber.</font>** limited distribution. The recipient may share AMBER information with others within their organization, but only on a ‘need-to-know’ basis. The originator may be expected to specify the intended limits of that sharing.
1. **<font color="green">green.</font>** targeted community distribution. Information in this category can be circulated widely within a particular community. However, the information may not be published or posted publicly on the Internet, nor released outside of the community.
1. **<font color="white" style="BACKGROUND-COLOR: black">white.</font>** unlimited, public. Subject to standard copyright rules, WHITE information may be distributed freely, without restriction.

### ext-tlp
Optional. STRING. A means for extending tlp.

### ttl
Optional. Uint32. Allows the specification of a "Time To Live" as similar to RFC [3443](http://tools.ietf.org/html/rfc3443) in relation to the TLP specification. The default value is 0, the max value is 3. A positive value allows the content to be re-shared to an extension of the original target contact (or community) while increasing the "TLP" restriction level.

For example:

  Information is transmitted to community1 from community2 with a TLP of "AMBER" and a TTL of "1". This means community1 may interpret the "need-to-know" clause as inclusive of their internal community, but the data must be re-shared as "RED" instead of "AMBER".

Information is transmitted to community1 from community2 with a TLP of "GREEN" and a TTL of "2". This means community1 may interpret the "need-to-know" clause as inclusive of their internal community, but the data must be re-shared as "RED" instead of "GREEN".

## Context
```
+-------------------+
| Context           |
+-------------------+
| STRING            |
|                   |
| ENUM ctype        |
| STRING ext-ctype  |
| ENUM rtype        |
| STRING ext-rtype  |
+-------------------+
```

The Context class represents the way in which data should be handled. The context is specified in the element content and the type attributes specify the context conduct.

The Context class has four attributes:

### ctype
Optional. ENUM. The context markings try to convey the care that should be taken with this data. The values range from an active operation is in place (i.e., publication of the data will compromise an activity), to the data was gathered via a known collector, to the data was collected by a covert collector, to the data being older. The default value SHALL be "default" meaning the "out of band, negotiated default value". The permitted values for this attribute are shown below:

1. **default**. The default, out-of-band agreement applies.
1. **historical**. The data is probably already known or is relatively old.
1. **live-overt**. The data was collected in a known fashion or collector.
1. **live-covert**. The data was collected via a private collection facility.
1. **active**. The data is currently part of an active investigation (LE and/or non-LE).
1. **ext-value**. An escape value used to extend this attribute.

### ext-ctype
Optional. STRING. A means for extending ctype.

### rtype
Optional. ENUM. A rule for how the data should be handled by the target Contact (eg: person, entity, community, etc). The permitted values for this attribute are shown below. The default value is "default".

1. **default**. The default, out-of-band agreement applies.
1. **shared**. At-least one entity in this community has been made aware of the data.
1. **investigation**. This data is currently being investigated (LE and/or non-LE).
1. **share-selectively**. This data may not be appropriate for some under the top level tag, share selectively.
1. **no-share**. This data should not be shared with the target Contact (eg: it may not be prudent to share with the LEO community).
1. **ext-value**. An escape value used to extend this attribute.

### ext-rtype
Optional. STRING. A means for extending rtype.

# 6. Examples
## XML

```
<?xml version="1.0" encoding="UTF-8"?>
    <krenk version="0.0001" lang="EN" tlp="AMBER" ttl="1">
        <context ctype="active" rtype="historical">FOUO</context>  
        <reporttime>2010-01-01T00:00:55Z</reporttime>
        <expirationtime>2012-01-01T23:59:59Z</expirationtime>
    </krenk>
</xml>
```

## JSON
```
{
   "@version":    "0.0001",
   "@lang":       "EN",
   "@ttl":        "1",
   "@tlp":        "AMBER",   
   "context": [
       {
          "@ctype": "active",
          "@rtype": "historical",
          "#text":   "FOUO",
       },
   ],
   "reporttime":     "2010-01-01T00:00:55Z",
   "expirationtime": "2012-01-01T23:59:59Z"
}
```

# 7. Data Types

## Enumerated Types

Enumerated types are represented by the ENUM data type, and consist of an ordered list of acceptable values.  Each value has a representative keyword.  Within the schema, the enumerated type keywords are used as attribute values.

## Date-Time Strings

Date-time strings are represented by the DATETIME data type.  Each date-time string identifies a particular instant in time; ranges are not supported. Date-time strings are formatted according to a subset of [ISO 8601: 2000 [13]](http://en.wikipedia.org/wiki/ISO_8601) as documented in [RFC 3339](http://www.ietf.org/rfc/rfc3339.txt).

# 8. References
## Known Implementations
* libkrenk [github.com/csirtgadgets](https://github.com/csirtgadgets/libkrenk)

## Misc
* CSIRT Gadgets Protocols - [csirtgadgets.org](http://csirtgadgets.org/rfc)
* Licenses for Protocols - [hintjens.com](http://hintjens.com/blog:41)
* ZeroMQ RFC - [rfc.zeromq.org](http://rfc.zeromq.org/)
* Consensus Oriented Specification System - [digistan.org](http://www.digistan.org/)
* Google Protocol Buffers Developer Guide - [developers.google.com](https://developers.google.com/protocol-buffers/docs/overview)
* Freeformatter (xml to xsd, json validation, etc) - [freeformatter.com](http://www.freeformatter.com/)