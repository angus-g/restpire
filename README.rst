========
Restpire
========

A Breathe_ replacement specifically for Fortran. Breathe is a Sphinx_
extension designed to process the XML output of Doxygen_ into
reStructuredText (RST) nodes through docutils_. There is a pretty
significant drawback to this approach, which is that either
documentation has to be manually broken up into pages, or if generated
automatically, it all appears on the same page. This may be alright
for small an infrequent references, but as a replacement to the HTML
output of Doxygen it is much too limited.

.. _Breathe: https://breathe.readthedocs.io/en/latest/
.. _Sphinx: http://www.sphinx-doc.org/en/stable/
.. _Doxygen: http://www.stack.nl/~dimitri/doxygen/
.. _docutils: http://docutils.sourceforge.net/

Design Approach
---------------

The most flexible output format from Doxygen is XML. The main file
generated here is ``index.xml``, which contains a cross-referenced
list of every single output item from the parsing. As the name
suggests, this can be used for generating an index. Every tag has a
``refid`` attribute, which when appended with the suffix "``.xml``",
becomes a compound definition file. These are generated for things
like *modules* (sometimes called *namespaces*), *types* and
*files*. Within the compound definition files lies the actual
processed documentation from the source code.

We're interested in fairly closesly mimicking the output structure of
Doxygen, albeit in a format that integrates better with Sphinx. I
think the easiest approach to this would be to run a program on the
XML output of Doxygen, and generate multiple RST files, such as
module, type and subroutine indices, and individual pages for all
modules with their overview documentation and their contents.

Some things we have to be careful about include handling links from
Sphinx to Doxygen, from Doxygen to Sphinx, and from Doxygen to
Doxygen. Indeed, this is one of the things about Breathe that makes it
difficult to port to Fortran. Breathe uses Sphinx "domains" to parse
some extra language-dependent information out of the Doxygen output,
so that usual links from Sphinx into the Breathe output behave as
expected. Unfortunately, this involves a lot of strange parsing code,
for which the Fortran domain available for Sphinx is inadequate. I
don't think we need to make this an extremely general solution, we can
just handle the case we need.

Essential Elements
------------------

module list
+++++++++++

This is a list of all the modules present in the source, along with their brief documentation statement if available. For this, we want to get all elements with a kind of "namespace" from the index file.

module reference
++++++++++++++++

This is the page that you're taken to after following a link from the `module list`_. It starts with a brief description of the defined *data types* and *functions/subroutines*, followed by a *detailed description* of the module. Below this is the detailed documentation for all functions/subroutines, including the location in the source (with a link), references to other functions/subroutines, a source listing, and a call graph.

type reference
++++++++++++++

When following a link to a type description from the `module reference`_, or the `type list`_ you get a separate page for each type. This includes a *collaboration diagram* (which shows the types of all constituent members), followed by a list of all public attributes with brief documentation, the detailed description of the type, and then the detailed description of all the type members.
