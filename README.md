1. Goals
==
The KRENK protocol is an information disclosure protocol. It enables the sharing of intelligence by defining the rules in which intelligence can be, and should be shared amongst practitioners and/or automated systems.

* Provide guidance if and how information should be shared

2. License
==

This Specification is free software; you can redistribute it and/or modify it under the terms of the GNU Lesser General Public License as published by the Free Software Foundation; either version 3 of the License, or (at your option) any later version.

This Specification is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License along with this program; if not, see <http://www.gnu.org/licenses>.

3. Change Process
==
This document is governed by the [Consensus-Oriented Specification System (COSS)](http://www.digistan.org/spec:1/COSS).


4. Language
==
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC [2119](http://www.ietf.org/rfc/rfc2119.txt).

## 5. Examples
## 5.1 JSON
```
{
    version        => '0.0.1',
    lang           => 'EN',
    tlp            => 'red',
    rule           => 'shared',
    ttl            => 1,
    Description    => {
                            lang => 'EN',
                            content => 'FOUO',
                      },
    Sensitivity    => [{
                          stype    => 'historical',
                          Description    => [{ content => 'already shared with LE' }],
                      }],
    ReportTime     => '2012-01-01T00:00:50Z',                             
}
```

## 5.2 XML

```
<?xml>
    <Krenk version="0.0.1" lang="EN" tlp="red" rule="shared", ttl="1">
        <Description lang="EN">FOUO</Description>
        <Sensitivity stype="historical">
            <Description>already shared with LE</Description>
        </Sensitivity>
    </Krenk>
</xml>
```

## 5.3 Protocol Buffer
see [source](https://github.com/csirtgadgets/krenk-protocol/blob/master/src/pb/main.proto)

6. Protocol
==
6.1 Krenk
--
```
+------------------+
| Krenk            |
+------------------+
| REAL version     |
| ENUM lang        |
|                  |<>--{0..1}--[ Description    ]
| ENUM tlp         |<>--{0..*}--[ Sensitivity    ]
| ENUM rule        |<>--{0..1}--[ ReportTime     ]
| STRING ext-rule  |<>--{0..1}--[ StartTime      ]
| UINT32 ttl       |<>--{0..1}--[ ExpirationTime ]
+------------------+
```

#### version
Required. REAL. The specification version number to which this Envelope conforms.

##### lang
Required.  ENUM.  A valid language code per RFC [4646](http://tools.ietf.org/html/rfc4646).

### Description
Zero or one. [MLSTRING](#multilingual-strings). A free-form textual representation of the localized restriction policy (eg: 'RESTRICTED' or 'PRIVATE' instead of 'RED'). The specifics of this are to be negotiated out-of-band.

### Sensitivity
Zero or many.

### ReportTime
Zero or one. TIMESTAMP. A timestamp that represents when this data marking was generated.

### StartTime
Zero or one. TIMESTAMP. A timestamp that represents when the data marking takes effect.

### ExpirationTime
Zero or one. TIMESTAMP. A timestamp that represents when the data marking expires.

### tlp
Required. ENUM. This attribute indicates disclosure guidelines to which the sender expects the recipient to adhere for the information represented in this class and its children.  This guideline provides no security since there are no specified technical means to ensure that the recipient of the document handles the information as the sender requested.

The value of this attribute is logically inherited by the children of this class.  That is to say, the disclosure rules applied to this class, also apply to its children.
 
It is possible to set a granular disclosure policy. A child can override the guidelines of a parent class, be it to restrict or relax the disclosure rules (e.g., a child has a weaker policy than an ancestor; or an ancestor has a weak policy, and the children selectively apply more rigid controls).  The implicit value of the restriction attribute for a class that did not specify one can be found in the closest ancestor that did specify a value.

This attribute is defined as an enumerated value with a default value of "RED".  Note that the default value of the restriction attribute is only defined in the context of the Envelop class.  In other classes where this attribute is used, no default is specified.
    
Enumerated attributes SHALL conform to the universally recognized Traffic Light Protocol[[1]](http://en.wikipedia.org/wiki/Traffic_Light_Protocol),[[2]](http://www.us-cert.gov/tlp):
    
1. **<font color="red">red.</font>** personal for named recipients only. In the context of a meeting, for example, RED information is limited to those present at the meeting. In most circumstances, RED information will be passed verbally or in person.

2. **<font color="amber">amber.</font>** limited distribution. The recipient may share AMBER information with others within their organization, but only on a ‘need-to-know’ basis. The originator may be expected to specify the intended limits of that sharing.

3. **<font color="green">green.</font>** targeted community distribution. Information in this category can be circulated widely within a particular community. However, the information may not be published or posted publicly on the Internet, nor released outside of the community.

4. **<font color="white" style="BACKGROUND-COLOR: black">white.</font>** unlimited, public. Subject to standard copyright rules, WHITE information may be distributed freely, without restriction.

### rule
Optional. ENUM. A rule for how the data should be handled by the target Contact (eg: person, entity, community, etc). The permitted values for this attribute are shown below. The default value is "default".

1. **default**. The default, out-of-band agreement applies.
2. **shared**. At-least one entity in this community has been made aware of the data.
3. **investigation**. This data is currently being investigated (LE and/or non-LE).
4. **share-selectively**. This data may not be appropriate for some under the top level tag, share selectively in conjunction with [Sensitivity](#52-sensitivity).
5. **no-share**. This data should not be shared with the target Contact (eg: it may not be prudent to share with the LEO community).
6. **ext-value**. An escape value used to extend this attribute.

### ext-rule
Optional. String. A means for extending rule.

### ttl
Optional. Uint32. Allows the specification of a "Time To Live" as similar to RFC [3443](http://tools.ietf.org/html/rfc3443) in relation to the TLP specification. The default value is 0, the max value is 3. A positive value allows the content to be re-shared to an extension of the original target contact (or community) while increasing the "TLP" restriction level.

For example:

  Information is transmitted to community1 from community2 with a TLP of "AMBER" and a TTL of "1". This means community1 may interpret the "need-to-know" clause as inclusive of their internal community, but the data must be re-shared as "RED" instead of "AMBER".
  
Information is transmitted to community1 from community2 with a TLP of "GREEN" and a TTL of "2". This means community1 may interpret the "need-to-know" clause as inclusive of their internal community, but the data must be re-shared as "RED" instead of "GREEN".

6.2 Sensitivity
--
```
+------------------------+
| Sensitivity            |
+------------------------+
| ENUM stype             |<>--{0..*}[ Description   ]
| STRING ext-sensitivity |
+------------------------+
```

### Description
Zero or many. MLString. A localized description of the sensitivity.

### stype
Required. ENUM. The sensitivity markings try to convey the care that should be taken with this data. The values range from an active operation is in place (i.e., publication of the data will compromise an activity), to the data was gathered via a known collector, to the data was collected by a covert collector, to the data being older. The default value SHALL be "default" meaning the "out of band, negotiated default value". The permitted values for this attribute are shown below:

1. default. The default, out-of-band agreement applies.
2. historical. The data is probably already known or is relatively old.
3. live-overt. The data was collected in a known fashion or collector.
4. live-covert. The data was collected via a private collection facility.
5. active. The data is currently part of an active investigation (LE and/or non-LE).
6. ext-value. An escape value used to extend this attribute.

### ext-sensitivity
Optional. String. A means for extending stype.

7. Data Types
==
Multilingual Strings
--
```
+----------------+
| MLString       |
+----------------+
| ANY            |
|                |
| ENUM lang      |
+----------------+
```
String data that represents multi-character attributes in a language different than the default encoding of the document is of the MLSTRING data type. The default lang (language) attribute MUST be 'EN'.

Bytes
--
A binary octet is represented by the BYTE data type.  A sequence of binary octets is represented by the BYTE[] data type.  These octets are encoded using base64.


8. References
==
8.1. Implementations
--
1. libkrenk-perl [github.com/csirtgadgets](https://github.com/csirtgadgets/libkrenk-perl)