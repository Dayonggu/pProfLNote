# Things to know

## To learn list
* TreeMap 

## Principles

#### SOLID
* **S**: single responsibility
* **O**: Open for extension but close for modification
* **L**: Liskov substitution; if *S* is a subtype of *T*, then in program, any object of *T* can be replaced with an object of *S*, without any changes
  * example of violation: Square is not a Rectangle
* **I**: Interface segregation (ISP)
  * No client should be forced depend on methods it does not need
  * So an interface should not have method that some client of it does not need
  * Change a big interface to smaller one to keep the system *decoupled*, and easy to *refactor*
* **D**: Dependency inversion
  * High level model should *NOT* depend on low-level modules; both should depend on *abstraction*
  * Details depend on *abstraction*, not the other direction

## Technology

### Geo Hash
* a hierarchical spatial data structure which subdivides space into buckets of grid shape
* encodes a geographic location into a short string of letters and digits.
* use 32-bits binary to represent the Latitude/Longitude:
  * the first bit would tell you whether it is in (-90c, 0c), or (0c, 90c): e.g.
  * then the second bit cut the current range to 2, e.g.:
    * from "00": we know since the first bit is 0, it is in (-90c, 0c), and then the second bit is also 0, so it would be in the first half of the range (represented by the first bit), so would be (-90c, -45c)
    * "01" would be (-45c, 0c)
    * "10" would be (0c, 45c)
    * "11" would be (45c, 90c)
  * then a 32-bits long of 0/1 would be pretty accurate
  * next step is map long 0/1 string to a 32-bit-based format (use 0-9, b,c,d,...z)
* Geohashes can be used to find points in proximity to each other based on a *common prefix*

### Mono-increasing unique Id
* 64bits long:
  * bit o: reserved
  * bit 1-41: timestamp
  * next 10 bits : machine Id
  * next 12 bits: sequence number
