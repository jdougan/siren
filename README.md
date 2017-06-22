# Siren: a hypermedia specification for representing entities

[![DOI](https://zenodo.org/badge/4917422.svg)](https://zenodo.org/badge/latestdoi/4917422)

Your input is appreciated.  Feel free to file a GitHub Issue, a Pull Request, or contact us.  Thank you!

- [Official Siren Google Group](https://groups.google.com/forum/#!forum/siren-hypermedia)
- Kevin on Twitter [@kevinswiber](https://twitter.com/kevinswiber)

## Example

Below is a JSON Siren example of an order, including sub-entities.  The first sub-entity, a collection of items associated with the order, is an embedded link.  Clients may choose to automatically resolve linked sub-entities.  The second sub-entity is an embedded representation of customer information associated with the order.  The example also includes an action to add items to the order and a set of links to navigate through a list of orders.

The media type for JSON Siren is `application/vnd.siren+json`.

```json
{
  "class": [ "order" ],
  "properties": { 
      "orderNumber": 42, 
      "itemCount": 3,
      "status": "pending"
  },
  "entities": [
    { 
      "class": [ "items", "collection" ], 
      "rel": [ "http://x.io/rels/order-items" ], 
      "href": "http://api.x.io/orders/42/items"
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
```

## Introduction

Siren is a hypermedia specification for representing entities.  As HTML is used for visually representing documents on a Web site, Siren is a specification for presenting entities via a Web API.  Siren offers structures to communicate information about entities, actions for executing state transitions, and links for client navigation.  

Siren is intended to be a general specification of a generic media type that can be applied to other types that are not inherently hypermedia-powered.  The initial implementation is JSON Siren.  Other implementations, such as XML Siren, may also be implemented using the Siren specification.

All examples in this document are in the JSON Siren format.

## Entities

An Entity is a URI-addressable resource that has properties and actions associated with it.  It may contain sub-entities and navigational links.

Root entities and sub-entities that are embedded representations SHOULD contain a `links` collection with at least one item contain a `rel` value of `self` and an `href` attribute with a value of the entity's URI.

Sub-entities that are embedded links MUST contain an `href` attribute with a value of its URI.

### Entity

#### `class`

Describes the nature of an entity's content based on the current representation.  Possible values are implementation-dependent and should be documented.  MUST be an array of strings.  Optional.

#### `properties`

A set of key-value pairs that describe the state of an entity.  In JSON Siren, this is an object such as `{ "name": "Kevin", "age": 30 }`.  Optional.

#### `entities`

A collection of related sub-entities.  If a sub-entity contains an `href` value, it should be treated as an embedded link.  Clients may choose to optimistically load embedded links.  If no `href` value exists, the sub-entity is an embedded entity representation that contains all the characteristics of a typical entity.  One difference is that a sub-entity MUST contain a `rel` attribute to describe its relationship to the parent entity.

In JSON Siren, this is represented as an array.  Optional.

#### `links`

A collection of items that describe navigational links, distinct from entity relationships.  Link items should contain a `rel` attribute to describe the relationship and an `href` attribute to point to the target URI.  Entities should include a link `rel` to `self`.  In JSON Siren, this is represented as `"links": [{ "rel": ["self"], "href": "http://api.x.io/orders/1234" }]`  Optional.

#### `actions`

A collection of action objects, represented in JSON Siren as an array such as `{ "actions": [{ ... }] }`.  See Actions.  Optional

#### `title`
Descriptive text about the entity.  Optional.

  
### Sub-Entities

Sub-entities can be expressed as either an embedded link or an embedded representation.  In JSON Siren, sub-entities are represented by an `entities` array, such as `{ "entities": [{ ... }] }`.

#### Embedded Link

A sub-entity that's an embedded link may contain the following:

##### `class`

Describes the nature of an entity's content based on the current representation.  Possible values are implementation-dependent and should be documented.  MUST be an array of strings.  Optional.

##### `rel`

Defines the relationship of the sub-entity to its parent, per [Web Linking (RFC5988)](http://tools.ietf.org/html/rfc5988) and [Link Relations](http://www.iana.org/assignments/link-relations/link-relations.xhtml).  MUST be an array of strings.  Required.

##### `href`

The URI of the linked sub-entity.  Required.

##### `type`

Defines media type of the linked sub-entity, per [Web Linking (RFC5988)](http://tools.ietf.org/html/rfc5988).  Optional.

##### `title`
Descriptive text about the entity.  Optional.

#### Embedded Representation

Embedded sub-entity representations retain all the characteristics of a standard entity, but MUST also contain a `rel` attribute describing the relationship of the sub-entity to its parent.

### Classes vs. Relationships

It's important to note the distinction between link relations and classes.  Link relations define a relationship between two resources.  Classes define a classification of the nature of the element, be it an entity or an action, in its current representation.

### Sub-Entities vs Links

Another distinction is the difference between sub-entities and links.  Sub-entities exist to communicate a relationship between entities, in context.  Links are primarily navigational and communicate ways clients can navigate outside the entity graph.

## Links

Links represent navigational transitions.  In JSON Siren, links are represented as an array inside the entity, such as `{ "links": [{ "rel": [ "self" ], "href": "http://api.x.io/orders/42"}] }`

Links may contain the following attributes:

### `rel`

Defines the relationship of the link to its entity, per [Web Linking (RFC5988)](http://tools.ietf.org/html/rfc5988) and [Link Relations](http://www.iana.org/assignments/link-relations/link-relations.xhtml).  MUST be an array of strings. Required.

### `class`

Describes aspects of the link based on the current representation.  Possible values are implementation-dependent and should be documented.  MUST be an array of strings.  Optional.

### `href`

The URI of the linked resource.  Required.

### `title`

Text describing the nature of a link.  Optional.

### `type`

Defines media type of the linked resource, per [Web Linking (RFC5988)](http://tools.ietf.org/html/rfc5988).  Optional.

## Actions

Actions show available behaviors an entity exposes.

### `name`

A string that identifies the action to be performed.  Action names MUST be unique within the set of actions for an entity. The behaviour of clients when parsing a Siren document that violates this constraint is undefined.  Required.

### `class`

Describes the nature of an action based on the current representation.  Possible values are implementation-dependent and should be documented.  MUST be an array of strings.  Optional.

### `method`

An enumerated attribute mapping to a protocol method.  For HTTP, these values may be `GET`, `PUT`, `POST`, `DELETE`, or `PATCH`.  As new methods are introduced, this list can be extended.  If this attribute is omitted, `GET` should be assumed.  Optional.

### `href`

The URI of the action.  Required.

### `title`

Descriptive text about the action.  Optional.

### `type`

The encoding type for the request.  When omitted and the `fields` attribute exists, the default value is `application/x-www-form-urlencoded`.  Optional.

### `fields`

A collection of fields, expressed as an array of objects in JSON Siren such as `{ "fields" : [{ ... }] }`.  See Fields.  Optional.

### Fields

Fields represent controls inside of actions.  They may contain these attributes:

#### name

A name describing the control.  Field names MUST be unique within the set of fields for an action. The behaviour of clients when parsing a Siren document that violates this constraint is undefined.  Required.

### `class`

Describes aspects of the field based on the current representation.  Possible values are implementation-dependent and should be documented.  MUST be an array of strings.  Optional.

#### type

The input type of the field. This may include any of the following [input types](http://www.w3.org/TR/html5/single-page.html#the-input-element) specified in HTML5:

`hidden`, `text`, `search`, `tel`, `url`, `email`, `password`, `datetime`, `date`,
`month`, `week`, `time`, `datetime-local`, 
`number`, `range`, `color`, `checkbox`,
`radio`, `file` 

When missing, the default value is `text`.  Serialization of these fields will depend on the value of the action's `type` attribute. See [`type`](#type) under Actions, above. Optional.

#### value

A value assigned to the field.  Optional.

#### title

Textual annotation of a field.  Clients may use this as a label.  Optional.

## Usage Considerations

Siren supports a resource design style that doesn't have to be primarily CRUD-based.  A root entity may take ownership of facilitating changes to sub-entities via actions.  Using Siren allows you to easily provide a task-based interface through your Web API.

## Feedback

Siren is still a work in progress looking for some real world usage and feedback.  Please contribute!  Feel free to use GitHub Issues to make suggestions or fire up a Pull Request with changes to be reviewed.  Thank you!



## Appendix: The Siren data model

If Siren is to become a multi-encoding format, then the underlying data model needs to be cleanly defined, along with a bijective mapping to each of the encodings. I don't think we need to go to the lengths Patrick Winston did with formalizing RDF, but some struture would be helpful to understand what is core to the representation versus what is an accident of the JSON context.

The _type_ attribute in the JSON encoding is unclearly defined as it is overloaded several different ways:
    * Link.type means encoding type to expect in GET response body
    * Action.type means encoding type to put in request body
    * Field.type means the HTML 5 field type (both interpreting the Field.value and validation...like a schema)
For clarity this model splits _type_ into 3 attributes and will rely on the mappings to change the names if necessary.
    * _req-enctype_ (**Action**._type_)
    * _resp-enctype_ (**Link**._type_)
    * _field-schema_ (**Field**._type_)

### Model Attributes
* _class_ => **list** of **class-id**
* _title_ => **string**
* _properties_ => **dict** of (**string**, **jdata**???)
* _entities_ => **list** of **Link** and (Sub?)**Entity**
* _links_ => **list** of **Links**
* _actions_ => **list** of **Action**
* _name_ => **string** (unique in parent collection context)
* _method_ => **http-method** 
* _href_ => **uri** (required in **Link**, **Action**)
* _req-enctype_ => **media-type** ("type"in **Action**)
* _resp-enctype_ => **media-type** ("type" in **Link**)
* _field-schema_ => **html5-field-type** ("type" in **Field**)
* _rel_ => **list** of **link-relation** (Req. in **Link**, Sub**Entity**)
* _value_ => **jdata**????

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
	* _name_ (Req. Unique in parent collection.)
	* _method_
	* _href_ (Req.)
	* _resp-enctype_ (Key is "type" in JSON mapping)
	* _fields_

* **Link**
	* _rel_ (Req.)
	* _href_ (Req.)
	* _resp-enctype_ (Key is "type" in JSON mapping)

* **Field**
	* _name_ (Req. Unique in parent collection.)
	* _field-schema_ (Key is "type" in JSON mapping)
	* _value_

### Data types
* **string**
    * Unicode
    * Because JSON is always UTF-8.
* **uri**
    * Unicode characters reduced to 7-bit ASCII via URLencoding for max compatibility?
    * Should this really be an IRI? https://stackoverflow.com/questions/2742852/unicode-characters-in-urls#2742985
* **link-relation**
    * spec'd in RFC-YYYY
    * extension mechanism uses URIs. Should we care about that or just treat as an opaque string?
* **media-type**
    * Unicode string.
    * spec'd in RFC-ZZZZ
* **html5-field-type**
    * Unicode string as documented in RFC-AAAA
* **http-method**
    * Unicode string as documented in RFC-BBBB
* **classId**
    * arbitrary str?  Should we encourage URIs?
* **jdata**
    * Jdata is the set of entities that can be most naturally representied by JSON across platforms. Dict, Array, Bool, Number, etc.
    * Is the repn power here accidental or core? If I were to make an XML serialization, would the properties look more like json or just be arbitrary XML data?
* **opaque-data**
    * serialization specific data inserted verbatim.
    * Nothing uses this, I think.

## JSON Enoding Mapping


### Extended Order Example
This is the Order example at the top with some extensions to exercise more aspects of Siren. All mappings will uese this document as an example.
~~~~
{
  "class": [ "order" ],
  "properties": { 
      "orderNumber": 42, 
      "itemCount": 3,
      "status": "pending"
      "arbitrary-jdata": {"parent":"widget", a":42, "b":"thingy", "c":[1,2,3,4,5]}
      "arbitrary-json": {"parent":"widget", a":42, "b":"thingy", "c":[1,2,3,4,5]}
  },
  "entities": [
    { 
      "class": [ "items", "collection" ], 
      "rel": [ "http://x.io/rels/order-items" ], 
      "href": "http://api.x.io/orders/42/items",
      "title":"Link to my items",
      "type":"application/vnc.siren+json"
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




## S-expressions Mapping
S-expressions have a lot of the advantages of XML, with less of the complexity and overhead. A quality streaming parser can be done in very little code. One of the earliest attemps at a consistent network data encoding was built around s-expressions (RFC-) and Ron Rivest's version of S-expressions are used in some crypto applications.

### Extended Order Example, no implicits
This is the Order example at the top with some extensions to exercise more aspects of Siren.
~~~~
(entity 
  (class "order") 
  (properties 
    (prop "orderNumber" 42) 
    (prop "status" "pending") 
    (prop "itemCount" 3) 
    (prop "arbitrary-jdata" (dict "parent" "widget" "a" 42 "b" True "c" (arr 1 2 3 4 5))) 
    (propa "arbitrary-s-expr" (quote (1 2 3 "foo" (this is a test))))
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
        (prop "customerId" "pj123") 
        (prop "name" "Peter Joseph)) 
      (links 
        (link 
          (rel "self") 
          (href "http://xrfb.net/id/3"))))))
~~~~


## XML Mapping
