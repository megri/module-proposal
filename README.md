# Proposal for a new language construct that combines `object`, `package` and `package object`

## Proposal outline

### Current state

#### Packages
Currently, both Scala and the upcoming Dotty treat *package*s as second class citizens: They are intangible, cannot be passed around and provides no more functionality than allowing code to be structured to fit the JVM model.
    
#### Objects
Scala *object*s allow the same structural organization as packages but lack the privilege of defining a package path.
  
#### Package objects
While packages cannot contain top level definitions, objects can. *Package object*s provide a place in which top-level definitions can exist and be allowed to import together with other package artifacts. I feel they add extra complexity by crudely mashing the two concepts together.

### Proposed state
This proposal aims at condensing `object`, `package` and `package object` into a single `module` concept.

#### The module
A module consists of two things:
* A path or module-path, consisting of path segments separated by '.' (just like a package path)
* An instance, from this point on simply refered to as 'module'

A module definition creates one or more paths and one or more modules. An empty module — that is, a module that doesn't contain any top level definitions — will not be reified during compilation.
  
For example, `module com.org` defines:
* the module 'com' with path '\_root_'
* the module 'org' with path '\_root_.com'


### Interoperability with current things

For a normal class definition
```
package com.org

class Foo
```

becomes


```
module com.org

class Foo
```
---
A package sequence such as 
```
package com.org
package communication

class HelloService(response: String)
```

becomes

```
module com.org
module communication

class HelloService(response: String)
```
---
A package object definition such as
```
package com.org

package object communication {
  def standardHello(response: String): HelloService = new HelloService(response)
}

```

becomes

```
module com.org

module communication {
  def standardHello(response: String): HelloService = new HelloService(response)
}
```
or even
```
module com.org.communication {
  def standardHello(response: String): HelloService = new HelloService(response)
}
```

### Implied syntactic pitfalls

Scala currently allows package members to be defined in blocks, like

```
package foo {
  class Bar
}
```

which would directly translate to

```
module foo {
  class Bar
}
```

which looks ambigous as to whether this declares a "path module" or a "package object module".

As this module contains no top level definitions it could be ignored during compilation and not generate bytecode  corresponding to today's package objects, allowing other module blocks to be declared without creating a collision between module instances.
  
However, in my experience this practice is not widely regarded as idiomatic or used in the wild. For this reason, I feel the better approach would be to disallow the package block syntax, or more specifically, always generate a module instance when a block is attached to the definition.
