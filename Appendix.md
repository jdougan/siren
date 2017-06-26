# Siren: a hypermedia specification for representing entities

[![DOI](https://zenodo.org/badge/4917422.svg)](https://zenodo.org/badge/latestdoi/4917422)

Your input is appreciated.  Feel free to file a GitHub Issue, a Pull Request, or contact us.  Thank you!

- [Official Siren Google Group](https://groups.google.com/forum/#!forum/siren-hypermedia)
- Kevin on Twitter [@kevinswiber](https://twitter.com/kevinswiber)

## Appendix: The Siren Data Model

If Siren is to become a multi-encoding format, then the underlying data model needs to be clearly defined, along with a bijective mapping to/from each of the encodings. I don't think we need to go to the lengths Patrick Hayes and the W3C did with formalizing RDF, but some structure would be helpful to understand what is core to the representation versus what is tentative or an accident of the JSON encoding context. 

Being based in REST principles, Siren views the world as a large collection of web resouorces, some of which are represented as Siren Entities.

The JSON encoding defines a lot implicitly, so a bare reference to an attribute _attr-name_ isn't definitive, so I'm going to use a **EntityName**._attribute-name_ notation when referring to elements of the JSON encoding. Entity classes are in boldface with an initial capital (**Entity**), attributes are italicized within initial lowercase (_attr-name_), and other types are boldfaced and lowercased (**some-datatype**).
> Comments are in blockquotes like this.

The _type_ attribute in the JSON encoding is unclearly defined as the attributes semantics are overloaded several different ways:

* **Link**._type_ means encoding type to expect in GET response body
* **Action**._type_ means encoding type to put in request body
* **Field**._type_ means the HTML 5 field type (both interpreting the Field.value and validation...like a schema)

For clarity this model splits _type_ into 3 attributes and will rely on the mappings to generae the right names as necessary for the JSON encoding.

* _req-enctype_ (**Action**._type_)
* _resp-enctype_ (**Link**._type_)
* _field-schema_ (**Field**._type_)

#### Commentary
> It feels like hhe's weaseling on definitions, to much looseness  and you have SOAP/WS-*. Protocols are about restrictions.

> ANSWERED. The _value_ and _properties_ attributes need clarification of the type of their values. They are listed here as **jdata**, but could conceiveably be interpreted as **opaque-data**. Is the representation capability of **jdata** accidental or core? If I were to make an XML serialization, would the properties look more like json or just be arbitrary XML data? Answer: After loking at the github issues, it appears to be core, adding that to the sexpr repn.

> The elements values for the collection attributes _action_ and _fields_ have uniquness requirements with regards to _name_. Is this core, tentative or incidental? If it is core would modelling using using a **dict** of (_name_, value) make sense?
> Should the collection attributes be modelled as annotated relations? It might help in dealing with uniqueness requirements. OTOH, If we think of the _actions_ and _fields_ as subcomponent relations (which the spec appears to say), where the children are never shared, we don't have to manage the _name_ uniqueness in a global sense.
> The _links_ relation may be best modelled as the same. The subentities in _entities_ are a bit harder though , as they are already either full **Entity** or attenuated shadows of the same.
> Bag it, this is a comms format and not an object distribution format.

> I've been an idiot, it is unclear where the ordering in lists in intended to be significant.  Is "class":["foo","bar"] supposed to be the same as "class":["bar","foo"] ??? More imortantly, we have to think about  field order fow the web standrd form encodings as (I think) they do by default in HTML.

> An oddity: all the entity collection name are plural, except _class_.

> Make the _rel_ attry in an **Entity** possible in the top level but specify it is to be ignored if the entity is the top level. This might make parsers a bit easier.

> With _type_ split, we could potentially put _resp-enctype_ on **Action** which might help clients trying to search for compatible actions.

> Might be useful to specify that the **Field**s and such are only currently specified to work with the _req-enctype_ values "application/x-www-form-urlencoded" and "multipart/form-data", thus opening up space for later extensions.

> Doing the above would enable a new media type "multipart/vnd.siren-field-data+json" for **Action**._resp-enctype_. Take the field values and write them out in the POST/PUT/etc. body as a JSON object with the names as keys. This would have the plus that we could define more useful **Field**._field-schema_s since they would no longer collide with the HTML forms input types. From my perspective this eliminates a bunch of annoying encoding and translation issues.

> My inclination is to specify "MUST IGNORE" as the primary affordance for expansions. Ths is pretty natural for JSON as if you don't know about a key you are less likely to access it navigationally. It has failure cases. such as name colisions, but it has worked pretty well for HTTP. OTOH, HTTP does have a version number in the request which seems to have been useful for marking major changes.
> Another affordance for expansions is to promise that the official spec will NEVER use some character(s) in the key and _class_ strings. For instance, if the reserved character is "." then ".registered-expansion-name.fred" or "net.xrfb.schema.fred" would be a legit expansion keys (in the latter case using a reverse DNS name scheme, which might be useful for namespacing without a registry). Obvious possibilities for reserved characters re: ";:-_*#$‖‽" or almost anything in the Unicode math section.
> This is taken from some ideas at [this Mark Nottingham blog post](https://www.mnot.net/blog/2011/10/12/thinking_about_namespaces_in_json)

> Does properties with the value null have the same itended semantics as not having a properties element? Or a properties element with an empty {}?
> This question generalizes to other aspects.

> If I end up forking this, call it JSON Dynamic Access Media (JDAM)



### Model Attribute Types
* _class_ => **list** of **class-id**
    * >  Is order intended to be insignificant?
    * > Alternate: **set** df **class-id**
* _title_ => **string**
* _properties_ => **dict** of (**string**, **jdata**)
* _entities_ => **list** of **Link** and (Sub?)**Entity**
    * > Alternate, may be more natural to use:
    * > **dict** of (_rel_ element, **list** of **Entity** or **Link**)
* _links_ => **list** of **Links**
    * > Alternate, may be more natural to use:
    * > **dict** of (_rel_ element, **list** of **Link**)
* _actions_ => **list** of **Action**
    * > Alternate, enforces _name_ uniqueness:
    * > **dict** of (**Action**._name_, **Action**)
* _name_ => **string** (unique in parent collection context)
* _method_ => **http-method** 
* _href_ => **uri** (required in **Link**, **Action**)
* _req-enctype_ => **media-type** ("type"in **Action**)
* _resp-enctype_ => **media-type** ("type" in **Link**)
* _fields_ => **list** of **Field**
    * > Alternate, enforces _name_ uniqueness:
    * > **dict** of (**Field**._name_, **Field**)
* _field-schema_ => **html5-field-type** ("type" in **Field**)
* _rel_ => **list** of **link-relation** (Req. in **Link**, (Sub **Entity**)
* _value_ => **jdata**

### Model Class Hierarchy and attributes
* **Object** (abstract)
    * **Action**
    * **Entity**
    * **Link**
    * **Field**

* **Object**
	* _class_
	* _title_

* **Entity**
	* _properties_
	* _entities_
	* _links_
	* _actions_
	* _rel_ (Required only in subentities)

* **Action**
	* _name_ (Required, and MUST be unique in parent collection.)
	* _method_
	* _href_ (Required)
	* _resp-enctype_ (Key is "type" in JSON mapping)
	* _fields_

* **Link**
	* _rel_ (Required)
	* _href_ (Required)
	* _resp-enctype_ (Key is "type" in JSON mapping)

* **Field**
	* _name_ (Req. Unique in parent collection.)
	* _field-schema_ (Key is "type" in JSON mapping)
	* _value_

### Data types
* **list** of element
    * ordered collection of values
    * JSON array, Python list, Smalltalk OrderedCollection or Array, etc.
    * In general, do we have any indication if the order of a list with any particular attribute  is significant?
* **set** of element
    * unordered collection of values
    * JSON object where (key, value) are always the same. Python set, Smalltalk Set, etc.
* **dict** of (key => **string**, value)
    * unordered set of pairs of (key=>**string**, value), with the key unique.
    * JSON object, Python dict with string keys, Smalltalk Dictionary with String keys, etc.
    * Key lookup semantics are as defined in the JSON spec, where equality is a determined via direct codepoint comparison.
* **string**
    * Unicode string.
    * [The JavaScript Object Notation (JSON) Data Interchange Format (RFC7159)](http://tools.ietf.org/html/rfc7159) says JSON SHALL be encoded with UTF-8, UTF-16 or UTF-32, which makes Unicode a reasonable default.
* **uri**
    * Unicode string ?with characters reduced to 7-bit ASCII via URLencoding for compatibility?
    * > Should this really be an IRI? https://stackoverflow.com/questions/2742852/unicode-characters-in-urls#2742985
    * > Lots of environments don't have good URI/IRI convertion tools but handle plain URIs well enough.
* **link-relation**
    * Unicode string.
    * Specified in [Web Linking (RFC5988)](http://tools.ietf.org/html/rfc5988) and the [associated registry](http://www.iana.org/assignments/link-relations/link-relations.xhtml)
    * > The extension mechanism uses URIs, not IRIs. Should we care about that and the relevant encoding issues or just treat as an opaque Unicode string?
* **media-type**
    * Unicode string.
    * Specified in [Media Type Specifications and Registration Procedures (RFC6838)](http://tools.ietf.org/html/rfc6838) and the [associated registry](http://www.iana.org/assignments/media-types/media-types.xhtml)
* **html5-field-type**
    * Unicode string as documented for the field tag type attribute in W3C/WHATWG docs.
* **http-method**
    * Unicode string of the names as documented by W3C/WHATWG/IETF for the HTTP method documentation. 
    * "GET", "PUT", "POST", "DELETE", "HEAD", "PATCH", "OPTIONS", "TRACE", "CONNECT", etc.
    * Also a bunch of new ones added for WebDAV that may not be as necessary. 
* **class-id**
    * Arbitrary opaque string?  Should we encourage URIs?
    * > It might be useful to standardize a small vocabulary for cases where interop is very important.
* **jdata**
    * jdata is the set of data types that can be most naturally representied by JSON across platforms. Dict/Hash/Object, Array/List/OrderedCollection, Bool/Boolean, Number/Float/Integer, etc.
    * Non-JSON encodings are going to need to explicitly handle this.
* **opaque-data**
    * Encoding specific data inserted verbatim.
    * Nothing uses this at present. Having an escape into the representation might be useful for some encodings.

## JSON Encoding Mapping
The original JSON encoding is not very explicit, with the client having to infer the kinds of entitys from their position in the JSON structure and some attribute semantics changing due to position. This is not a killer issue, unless the protocol gets significantly more complex.

### Extended Order Example
This is the Order example at the top with some extra data to exercise more aspects of Siren. All mappings will uese this document as an example.
~~~~
{
  "class": [ "order" ],
  "properties": { 
      "orderNumber": 42, 
      "itemCount": 3,
      "status": "pending"
      "arbitrary-jdata": {"parent":null, "a":42, "b":true, "c":[1,2,3,4,5]},
      "arbitrary-opaque-json":  {"parent":null, "a":42, "b":true, "c":[1,2,3,4,5]}
  },
  "entities": [
    { 
      "class": [ "items", "collection" ], 
      "rel": [ "http://x.io/rels/order-items" ], 
      "href": "http://api.x.io/orders/42/items",
      "title":"Link to my items",
      "type":"application/vnd.siren+json"
    },
    {
      "class": [ "info", "customer" ],
      "rel": [ "http://x.io/rels/customer" ], 
      "properties": { 
        "customerId": "pj123",
        "name": "Peter Joseph"
      },
      "links": [
        { "rel": [ "self" ], "href": "http://api.x.io/customers/pj123" }
      ]
    }
  ],
  "actions": [
    {
      "name": "add-item",
      "title": "Add Item",
      "method": "POST",
      "href": "http://api.x.io/orders/42/items",
      "type": "application/x-www-form-urlencoded",
      "fields": [
        { "name": "orderNumber", "type": "hidden", "value": "42" },
        { "name": "productCode", "type": "text" },
        { "name": "quantity", "type": "number" }
      ]
    }
  ],
  "links": [
    { "rel": [ "self" ], "href": "http://api.x.io/orders/42" },
    { "rel": [ "previous" ], "href": "http://api.x.io/orders/41" },
    { "rel": [ "next" ], "href": "http://api.x.io/orders/43" }
  ]
}
~~~~

## S-expressions Encoding Mapping
S-expressions have a lot of the advantages of XML, with less of the complexity and overhead. A quality streaming parser can be done in very little code. The drawbacks of sexprs is less standardization and no standard namespace support, but for the semantics of a JSON based protocol that shouldn't be an actual problem. The examples here have as atoms: tokens, double quoted strings with JSON-style character quoting, numbers, true, false, and nil.
https://people.csail.mit.edu/rivest/sexp.html

### Extended Order Example, almost no implicits
This is the Order example at the top with some extensions to exercise more aspects of Siren. The encoding is fairly explicit, with most component lists with a token in the first place to indicate intended semantics.
~~~~
(entity 
  (class "order") 
  (properties 
    ("orderNumber" 42) 
    ("status" "pending") 
    ("itemCount" 3) 
    ;;; possible explicit alternate repn for individual properties
    (prop "arbitrary-jdata" (dict "parent" nil "a" 42 "b" true "c" (liist 1 2 3 4 5))) 
    (prop-opaque "arbitrary-sexpr" (10.0 (1 2 3 "foo" (this is a test))))
  (links 
    (link (rel "self") (href "http://api.x.io/orders/42")) 
    (link 
      (rel "previous") 
      (href "http://api.x.io/orders/41")) 
    (link 
      (rel "next") 
      (href "http://api.x.io/orders/43"))) 
  (actions 
    (action 
      (name "add-item") 
      (href "http://api.x.io/orders/42/items") 
      (title "Add Item") 
      (req-enctype "application/x-www-form-urlencoded") 
      (fields 
        (field 
          (name "orderNumber") 
          (field-schema "hidden") 
          (value "42")) 
        (field 
          (name "productCode") 
          (field-schema "text")) 
        (field 
          (name "quantity") 
          (field-schema "number"))))) 
  (entities 
    (link 
      (rel "http://x.io/rels/order-items") 
      (href "http://api.x.io/orders/42/items") 
      (title "Link to my items") 
      (class "items" "collection") 
      (resp-enctype "application/vnd.siren+json")) 
    (entity 
      (rel "http://x.io/rels/customer") 
      (class "info" "customer") 
      (properties 
        ("customerId" "pj123") 
        ("name" "Peter Joseph")) 
      (links 
        (link 
          (rel "self") 
          (href "http://xrfb.net/id/3"))))))
~~~~


## XML Mapping
The Extensible Markup Language (XML) is designed to be an encoding that could represent any data semantics for transport or storage. It does this at the cost of a lot of verbosity and complexity. The advantage is that with the immense amount of XML standardiztion and extension that has already taken place, you almost never need to invent somthing to do a job.

~~~~
<!-- ugh   -->
~~~~


