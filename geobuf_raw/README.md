# geobuf_new

# What is it?

Protobuf representation of geojson features is something I've been trying for a while now. Previously things like not being able to implement bufio and having a bad protobuf struct to start with sort of limited the project. This is another attempt at creating such a structure and hopefully will be worth it. The protobuf structure looks like this: 

```
syntax = "proto3";

// Variant type encoding
// The use of values is described in section 4.1 of the specification
message Value {
	// Exactly one of these values must be present in a valid message
	string string_value = 1;
	float float_value = 2;
	double double_value = 3;
	int64 int_value = 4;
	uint64 uint_value = 5;
	sint64 sint_value = 6;
	bool bool_value = 7;
}

// GeomType is described in section 4.3.4 of the specification
enum GeomType {
	UNKNOWN = 0;
	POINT = 1;
	LINESTRING = 2;
	POLYGON = 3;
}

message Feature {
	uint64 Id = 1;
	map<string, Value>  Properties = 2;
 	GeomType type = 3;
	repeated uint64 Geometry = 4 [ packed = true ];
	repeated int64 BoundingBox = 5 [ packed = true ]; // N,S,E,W
}

message FeatureCollection {
	repeated Feature Features = 1;
}
```

**As you can see it takes a few cues from vector-tile protobuf spec, this protobuf structure is essentially an intermediate feature object for a regular geojson object.** 

#### How do Geojson Features and Geobuf Features Differ

As you can see not by much, firstly I currently implement a scorched earth approach to multi geometries. That means polygons can be represented easily with only one number indicating the ring size and points / lines are fine converted into uint64 with of course the usual delta-encoding. The map of properties has a value as the value instead of normally and interface, but other than that its basically the same. 

**While the implementation details are worth explaining for end-user purposes you will never see a geobuf feature.**

# Performance

Obviously at a single feature rate I/O is much much faster the problem previously was reading from a file iteratively was extremely hacky and I ended up splitting up code where I should have used an io.Reader. My new repo [protoscan](github.com/murphy214/protoscan) fixes this. 

# Differences between previous implementation

* New proto struct
* Writer / Reader methods will be completely independent which will help alot as well use io.Reader / io.Writer
* Reader will use protoscan which hopefully should speed things up quite a bit
* Cleaner abstractions between pure read / writes and file or writer objects