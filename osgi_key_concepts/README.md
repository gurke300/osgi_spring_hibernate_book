# OSGi Key concepts
I won't give here a detailed explanation what OSGi is there are lot already out there.
In short: OSGi is a standardized technology. It defines a runtime for software resources called bundles. Bundles are ... define metadata within the Manifest.mf file. **Export-Package**

## Some thoughts on IDEs
If you are coming from Web Development, no matter which of the big Java IDEs Intellij, Eclipse, Netbeans you use: if you develop Maven based Java EE Applications whether WAR or EAR projects you get a quite good support from your IDE. Also Spring is widly supported within the IDEs e.g. giving you code completion in spring xml configuration or hints in source files.

For OSGi development it looks different. If you are looking for plugins only Eclipse can offer a good plugin solution the [Bnd Tools](http://bndtools.org/): That is also used within the book from O'Reilly: [Building Modular Cloud Apps with OSGi](http://shop.oreilly.com/product/0636920028086.do) that I own (beside the Manning Books: [OSGi in Depth](http://www.manning.com/alves/) and [Enterprise OSGi in Action](http://www.manning.com/cummins/)). I can recommend those books if you seriously want to use OSGi in your software stack. All of the books are covering different aspects and complement each other.
But if you are used to develop software using Maven (or any other build tool capable of resolving dependencies) then you will face the first wall the needs to be taken.
In the beginning Bndtools felt quite good, allthough I had to get used to the project layout and the way you interact with it as well as the way to include dependencies.
Netbeans has some kind of support if you develop Netbeans Modules. But I ended up using plain Maven projects, because those can run in our CI environment out of the box.
About OSMORC the OSGi plugin from Intellij I cannot say anything because I have not used it.

