---
layout: post
title: ObjectMapper
date: 2017-07-21 15:32:24.000000000 +09:00
---

#### What's the problem?

There a lot of data from server, right now developers manully handle those data, they should be handled in one place.

####  

We have two options, struct or class.

we use struct when

As a general guideline, consider creating a structure when one or more of these conditions apply:

The structure’s primary purpose is to encapsulate a few relatively simple data values.
It is reasonable to expect that the encapsulated values will be copied rather than referenced when you assign or pass around an instance of that structure.
Any properties stored by the structure are themselves value types, which would also be expected to be copied rather than referenced.
The structure does not need to inherit properties or behavior from another existing type.
Examples of good candidates for structures include:

The size of a geometric shape, perhaps encapsulating a width property and a height property, both of type Double.
A way to refer to ranges within a series, perhaps encapsulating a start property and a length property, both of type Int.
A point in a 3D coordinate system, perhaps encapsulating x, y and z properties, each of type Double.
In all other cases, define a class, and create instances of that class to be managed and passed by reference. In practice, this means that most custom data constructs should be classes, not structures.


````
import Foundation
import ObjectMapper

//because we only need to copy the data out instead of reference. 
struct Photo: Mappable {
    
    var id: Int!
    var name: String!
    var images: [Image]!
    var width: Int!
    var height: Int!
    var likes: Int!
    
    
    // MARK: JSON
    init?(map: Map) { }
    
    mutating func mapping(map: Map) {
        id          <- map["id"]
        name        <- map["name"]
        images      <- map["images"]
        width       <- map["width"]
        height      <- map["height"]
        likes       <- map["votes_count"]
    }
    
}

````

















