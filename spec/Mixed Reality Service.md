MIXED REALITY SERVICE (MRS)
SECOND DRAFT, SEPTEMBER 2016 
// DEPRECATED OCTOBER 2016 - (DRAFT 3 IS CURRENT)

# AUTHOR

Mark D. Pesce (mpesce@gmail.com)
Honorary Associate, University of Sydney


# ABSTRACT

The Mixed Reality Service (MRS) provides registration and discovery services binding the real world of geospatial coordinates to the virtual world of Universal Resource Identifiers (URIs). The MRS protocol consists of three commands: ‘add’ and ‘delete’, which allow additions and deletions to a mapping of geographical coordinates to URIs; plus ‘search’, which performs searches by geospatial coordinates, returning a list of matching mapped URIs. Potential uses of MRS include mixed reality applications, guidance for autonomous vehicles and drones, and vastly simplified delivery of nearly all location-based services. Simple modifications to MRS make it suitable for shared virtual worlds.


# STATUS OF THIS DOCUMENT

This specification was published by the Mixed Reality System Working Group. It is not a W3C Standard nor is it on the W3C Standards Track. Please note that under the W3C Community Contributor License Agreement (CLA) there is a limited opt-out and other conditions apply. Learn more about W3C Community and Business Groups.
Changes to this document may be tracked at:
https://docs.google.com/document/d/1rXOK_n8CI_12Exz8oD0Dp0UogQ0EGp88sz06_sNJh3A/edit?usp=sharing
If you wish to make comments regarding this document, please send them to mpesce@gmail.com 


# INTRODUCTION

The advent of high-performance smartphones with access to geospatial positional services such as GPS, GLONASS and Galileo has created a global demand for locative services such as maps and directions. Locative services beyond these have historically been difficult to provide because the connectivity of the underlying networks bear no relation to locations in the physical world.

For thirty-five years, the Domain Name System (DNS) has provided an effective mapping between the human-centered space of domain names (i.e., w3c.org, hp.com, etc.) and IP addresses (i.e., 128.30.52.45, 15.217.232.245, etc.), a mapping that is becoming even more relevant in the transition to IPv6.

The Mixed Reality Service (MRS) proposed here provides an equivalent mapping between geospatial coordinates (i.e., Latitude -33.85939º, Longitude 151.20458º) and a URI as defined in IETF RFC 3986.

The Mixed Reality Service is composed of three interrelated and complementary operations.

Fundamentally, MRS maintains a registration of mappings between geospatial coordinates and URIs. These registration features of MRS are implemented in two atomic operations, ‘add’ and ‘delete’, which add or delete entries from the mapping.

MRS mappings are hard to change and slow to update, as is appropriate for a system that arbitrates virtual access rights to real-world properties and locations. 

MRS clients use the ‘search’ operation to interrogate and discover MRS mappings within proximity of a request. Where successful, a search request returns a list of the mappings between registered geospatial coordinates and their associated URIs.

Together these three atomic operations - ‘add’, ‘delete’ and ‘search’ - provide a complete Mixed Reality Service for creating, modifying and discovery of URIs mapped to geospatial coordinates.

The Mixed Reality Service is designed to be distributed, secure, resilient, and easy to implement. MRS is an application-level protocol relying relying on standard transport protocols (HTTPS) and encoding schemes (JSON) for its messaging.


# MRS ADD REQUEST

The Mixed Reality Service defines a simple protocol mechanism to make additions to a mapping binding geospatial coordinates to a URI.

The MRS ‘add’ request is stateless, with a request / response format.

The ‘add’ operation requests that a binding between provided geospatial coordinates and a URI be added to the MRS mapping.

In the simplest case, the MRS ‘add’ request operation includes the following data:

+ Latitude
+ Longitude
+ Elevation
+ FOAD
+ Service Point

Latitude is defined as a floating point number with five significant digits of precision, and a range from 90.0 to -90.0 degrees inclusive.

Longitude is defined as a floating point number with five significant digits of precision, and a range from 180.0 to -180.0 degrees inclusive. 

Elevation is defined as a floating point number with three significant digits of precision, as meters above sea level (or below sea level, in the case of negative values).

FOAD is a binary flag. If TRUE, the MRS mapping requests strict privacy and will provide no information about itself via a service point. Where FOAD is TRUE, the Service Point field is optional and undefined.

Service Point is defined as a URI as specified in IETF RFC 3986. It is strongly recommended that the Service Point resolve to a valid MRSE interrogation point, but the Service Point may be any well-formed URI. Where FOAD is TRUE, the Service Point field is optional and undefined.

An MRRS ‘add’ request packages these parameters into a JSON data structure as follows:

```javascript
{ “add”: {
    “lat”: <value>,
    “lon”: <value>,
    “ele”: <value>,
    “FOAD”: <boolean>,
    “Service_Point”: <URI>
} }
```

The MRS endpoint transmits a response to the ‘add’ request with a one required field, ‘added’. If the ‘add’ request has been successfully added to the MRS mapping, ‘added’ returns TRUE. If the MRS mapping was not successful, ‘added’ returns FALSE. 

This response is packed into a JSON data structure as follows:
```javascript
{ “response”: {
    “added”: <boolean>
} }
```

An MRS endpoint can reject any ‘add’ request on grounds of incompleteness for any reason whatsoever. 

The ‘add’ requests may include additional geospatial and verification data to help the responder determine the specific privileges of the request. 

Verification data is included as an optional field an ‘add’ requests. 

The specific content of the verification field is intentionally left undefined in this specification, but may be defined in further revisions of this specification.

In the case of an ‘add’ request, it would be added to the JSON data structure as follows:

```javascript
{ “add”: {
    “lat”: <value>,
    “lon”: <value>,
    “ele”: <value>,
    “FOAD”: <boolean>,
    “Service_Point”: <URI>,
    “verification”: <data>
} }
```

For security purposes, the contents of the verification field MUST be encrypted so that it is only visible to the receiver of the MRRS request, or some entity behind the MRRS request endpoint. This single-field encryption is above and beyond any transport-level encryption used to communicate the request.

MRS ‘add’ requests and responses can be transmitted via any secure transport mechanism (HTTPS, ssh, TLS, etc., see ‘Security Considerations’ below). Under no circumstances should MRRS traffic ever be transmitted over an unencrypted channel.

[ SEQUENCE DIAGRAM TBD ]


# MRS DELETE REQUEST

The Mixed Reality Service defines a simple protocol mechanism to delete the mapping between geospatial coordinates and a URI.

A ‘delete’ operation requests that the existing MRS mapping between a set of geospatial coordinates and a URI be removed. 

The MRS ‘delete’ request is stateless, with a request / response format.

In order to uniquely determine the MRS mapping to be removed, the same set of data used to create the mapping in an ‘add’ operation must be provided to the ‘delete’ operation. 

If there is any variation between ‘add’ and ‘delete’ data sets used to create and remove an MRS mapping, the ‘delete’ operation will fail, and the MRS mapping will not be removed. 

In the simplest case, the MRRS ‘delete’ request operation includes the following data:

+ Latitude
+ Longitude
+ Elevation
+ FOAD
+ Service Point

Latitude is defined as a floating point number with five significant digits of precision, and a range from 90.0 to -90.0 degrees inclusive.

Longitude is defined as a floating point number with five significant digits of precision, and a range from 180.0 to -180.0 degrees inclusive. 

Elevation is defined as a floating point number with three significant digits of precision, as meters above sea level (or below sea level, in the case of negative values).

FOAD is a binary flag. If TRUE, the MRS mapping has requested strict privacy and will provide no information about itself via a service point. Where FOAD is TRUE, the Service Point field is optional and undefined.

Service Point is defined as a URI as specified in IETF RFC 3986. Where FOAD is TRUE, the Service Point field is optional and undefined.

An MRRS ‘delete’ request packages these parameters into a JSON data structure as follows:

```javascript
{ “delete”: {
    “lat”: <value>,
    “lon”: <value>,
    “ele”: <value>,
    “FOAD”: <boolean>,
    “Service_Point”: <URI>
} }
```

The MRRS endpoint transmits a response to the ‘delete’ request with a one required field, ‘removed’. If the ‘delete’ request has been successfully deleted from the MRS mapping, ‘removed’ returns TRUE. If the MRS mapping was not successful deleted, ‘removed’ returns FALSE. 

This response is packed into a JSON data structure as follows:

```javascript
{ “response”: {
    “removed”: <boolean>
} }
```

An MRS endpoint can reject any ‘delete’ request on grounds of incompleteness for any reason whatsoever.  

A ‘delete’ requests may include additional geospatial and verification data to help the responder determine the specific privileges of the request. 

This verification data is included as an optional field in the ‘delete’ requests. 

The specific content of the verification field is intentionally left undefined in this specification, but may be defined in further revisions of this specification.

In the case of an ‘delete’ request, it would be added to the JSON data structure as follows:

```javascript
{ “delete”: {
    “lat”: <value>,
    “lon”: <value>,
    “ele”: <value>,
    “FOAD”: <boolean>,
    “Service_Point”: <URI>,
    “verification”: <data>
} }
```

The data provided within the verification field can be of arbitrary length. 

For security purposes, the contents of the verification field MUST be encrypted so that it is only visible to the receiver of the MRS ‘delete’ request, or some entity behind the MRS request endpoint. This single-field encryption is above and beyond any transport-level encryption used to communicate the request.

MRS ‘delete’ requests and responses can be transmitted via any secure transport mechanism (HTTPS, ssh, TLS, etc., see ‘Security Considerations’ below). Under no circumstances should MRRS traffic ever be transmitted over an unencrypted channel.

[ SEQUENCE DIAGRAM TBD ]


# MRS SEARCH REQUEST

The Mixed Reality Service defines a simple ‘search’ operation mechanism to interrogate the MRS mapping. 

The MRS ‘search’ request is stateless, with a request / response format.

An MRS client transmits a ‘search’ request with the following parameters:

+ Latitude
+ Longitude
+ Elevation
+ Range

Latitude is defined as a floating point number with five significant digits of precision, and a range from 90.0 to -90.0 degrees inclusive.

Longitude is defined as a floating point number with five significant digits of precision, and a range from 180.0 to -180.0 degrees inclusive. 

Elevation is defined as a floating point number with three significant digits of precision, as meters above sea level (or below sea level, in the case of negative values).

Range is defined as a floating point number with one significant digit of precision. The range value sets a radius in metres for the discovery query, and must be a positive number from 0.0 to 100000.0, setting the maximum range radius for any discovery request at 100 kilometers.

An MRS ‘search’ request packages these parameters into a JSON data structure as follows:

```javascript
{ “search”: {
    “lat”: <value>,
    “lon”: <value>,
    “ele”: <value>,
    “range”: <value>
} }
```

The MRS receiving endpoint, upon processing ‘search’ request, responds with the following data:

+ Number of matches
+ List of matches

Number of matches is defined as the number of intersections between the parameters in the discovery request and the MRRS data set. It is an integer value, set to zero if there are no matches, a positive integer if matches have been found, or -1 if an error has occurred.

List of matches is defined as an array of data representing the specific information about each match found during the MRS ‘search’ request. If Number of Matches is zero or negative, List of Matches should be omitted from the response.

Each entry in the List of Matches is a structure containing the following data:

+ Latitude
+ Longitude
+ Elevation
+ FOAD
+ Service Point

Latitude is defined as a floating point number with five significant digits of precision, and a range from 90.0 to -90.0 degrees inclusive.

Longitude is defined as a floating point number with five significant digits of precision, and a range from 180.0 to -180.0 degrees inclusive. 

Elevation is defined as a floating point number with three significant digits of precision, as meters above sea level (or below sea level, in the case of negative values).

FOAD is a binary flag. If TRUE, the match has requested strict privacy and will provide no information about itself, nor should it be queried via MRSE for any further information. Where FOAD is TRUE, the Service Point field is optional and undefined.

Service Point is defined as a URI as specified in IETF RFC 3986. It is strongly recommended that the Service Point resolve to a valid MRSE interrogation point, but the Service Point may be any well-formed URI. Where FOAD is TRUE, the Service Point field is optional and undefined.

An MRS ‘search’ response packages these parameters into a JSON data structure as follows:

```javascript
{ “response”: {
    “matches”: <value>,
    “matching”: [ array of entries... ]
} }
```

Each entry within the array in the “matching” field is a JSON data structure as follows:

```javascript
{
“lat”: <value>,
    “lon”: <value>,
    “ele”: <value>,
    “FOAD”: <boolean>,
    “Service_Point”: <URI>
}
```

MRS ‘search’ request and responses can be transmitted via any secure transport mechanism (HTTPS, ssh, TLS, etc., see ‘Security Considerations’ below). Under no circumstances should a ‘search’ request or response ever be transmitted over an unencrypted channel.

[ SEQUENCE DIAGRAM TBD ]


# ALTERNATIVE MAPPINGS

The Mixed Reality Service explicitly maps real-world geospatial coordinates to URIs. Other mappings are both possible and useful. 

A version of MRS suitable for immersive virtual worlds maps a three-dimensional coordinate system (x,y,z) to an arbitrary URI. 

That system has its own implementation of ‘add’, ‘delete’ and ‘search’ operations, with variations in the data requirements to reflect a generalised solution for three-dimensional coordinates.

MRS can also be used to provide a mapping of between arbitrarily high dimension space, of order N, and URIs. 

Again, this system has its own implementation of ‘add’, ‘delete’ and ‘search’ operations, with variations in the data requirements to reflect a generalised solution for N-dimensional coordinate spaces.


# DISTRIBUTED MIXED REALITY SERVICE

This specification is silent on the internal operations of MRS mapping. MRS defines how additions and deletions of mappings are performed, but the mechanism by which that mapping is created and maintained is beyond the scope of this specification.

In order to make MRS truly robust, fault-tolerant, resilient and distributed - that is, to make it a system that can scale to billions of simultaneous users - it needs to be conceived and implemented as such from its conception. 

Although this specification does not explore these implementation specific details, the following design guidelines are suggested by the author of this specification:

A distributed, global system of MRS mappings would likely be implemented through a distributed ledger technology (DLT).
This distributed ledger operates as the equivalent of a land title registry, in which ownership over a particular MRS mapping can be confirmed by consensus.
As such, this distributed ledger must be easy to share and difficult to modify.
As real property is at the core of the MRS mappings, any modifications in the distributed ledger must be accompanied by proofs appropriate to the modifications requested.
These proofs will have both legal and data requirements, and may vary based on the perceived significance of the request.
Proofs will have to be presented with the vast majority of requests, and further presented to every entity maintaining of the distributed ledger so that consensus can be reached about making a change in that ledger.


# DYNAMIC MIXED REALITY SERVICE

While MRS has been designed to work well for ‘immovable’ properties - land and buildings - portable properties, such as vehicles, would not be able to update their MRS mappings in real-time, as those mappings are intentionally designed to be hard to change and slow to update.

It is suggested that similar but more dynamic companion protocol to MRS will be required to fulfil this need.


# USE CASES

The capacity provided by the Mixed Reality Service to bind an arbitrary URI to a specific geospatial location has manifold uses.

Foremost, MRS allows entities to discover location-based information with no more data than their geospatial coordinates. Using MRS, every piece of land, house, or commercial property can publish its own highly-specific set of metadata via URI.

The content of this metadata varies entirely on application and situation. 

In the simplest case, a URI could refer to an HTML resource.

In a civic context, MRS can provide a mapping to metadata that allows the environment to ‘speak for itself’ about available resources and permitted uses.

In a commercial context, MRS mapped URIs could include commercially useful metadata such as hours of operation, or a building directory of tenants and services. 

In a health and safety context, MRS can provide real-time metadata warning of on-site hazards, remediations, and emergency contact details.

In an entertainment context, MRS can be used to permission or restrict certain kinds of activities (such as AR games like ‘Pokemon Go’), defining play areas and removing the threat of accidental trespass or nuisance violations.

In the context of autonomous vehicles, MRS can allow a vehicle to interactively discover the metadata necessary to guide itself through an unknown or dynamic environment.

This list of MRS use cases is meant to be illustrative rather than exhaustive.


# SECURITY CONSIDERATIONS

Requests for and the provision of geospatial information are both potentially highly sensitive operations. For that reason, all MRS traffic must be transmitted via a secure transport mechanism such as HTTPS, ssh, TLS, and so forth.

At no time should MRS traffic ever be sent in cleartext.

Beyond this, encryption is of vital importance in the MRS ‘verification’ field of both ‘add’ and ‘delete’ operations, and must be individually encrypted so they can only be read by the intended recipient.  

As “verification” data will regularly include highly sensitive information, such as proofs of identity and residence, it must always be separately encrypted, above and beyond any transport-level encryption, through the most effective means available. 

While these specifically enumerated considerations are essential for the implementation of the Mixed Reality Service, this specification does not preclude additional security measures, where appropriate, and encourages implementations of MRS to use best-practice security techniques whenever practical.


# CONCLUSION

The Mixed Reality Service provides a metadata layer for the real world.

This specification represents the start of an extensive period of development, refinement, and implementation across the entire range of real-world applications enabled by MRS.

Over the last quarter century the culture has taken full advantage of the capacity of the World Wide Web to link information and knowledge together into a seamless whole. 

For this whole period of time, the real world has existed almost entirely apart from and untouched by the project of the Web. The virtual world swelled with capacity while the real world remained effectively unchanged.

MRS closes the gap between the real and the virtual, binding them closely together in a global, distributed, secure and open geospatial-to-URI mapping. 

Adding a simple, extensible and open metadata layer to the real world allows it to ‘speak for itself’, and allows users to navigate within it interactively, through a process of discovery.

MRS provides a data layer the real world has needed for a generation, and needs more today than ever before.


# ACKNOWLEDGEMENTS

The author would like to thank the following individuals for their contributions and insight over the period MRS has been in gestation: Tony Parisi, Owen Rowley, Peter Kennard, Zac Pullen, Philippe Van Nedervelde, Michela Ledwidge, Chris Mountford, Hugo O’Connor and Raena Lea-Shannon. 

The author would also like to thank Sir Tim Berners-Lee for his continued support of this proposal.

Any errors or omissions in this specification are solely the responsibility of the author.

