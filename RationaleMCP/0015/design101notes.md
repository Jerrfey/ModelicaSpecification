Discussion whether it should only be at the top?
Seems reasonable first restriction, would be straightforward to lift later. Poll on merging [PR#2419](https://github.com/modelica/ModelicaSpecification/pull/2419)

What about non-file based storage? State that it should have the same information, but not specified how.

Tools don't always fully implement a version; how to handle that case in practice?

How does it solve the case where users are using different tools (or different versions of the same tool) that support different versions:
1. If tools always write the current version, it will be annoying in old versions of tools.
2. If tools keep the version in the package then tools should ideally implement a check that you cannot introduce new operators (etc) in old versions.
 
What to do when language is clarified? It shouldn't change anything, but some "clarfications" are changes.
For real clarifications it is reasonable to always use the new semantics; and similarly for bug-fixes in the specification.
But sometimes it is a grey area.

The current version only have the language-version without any extra information, there was some concern that it was too little information and adding that it is Modelica-Language version could help (but previous decisions have removed that).
