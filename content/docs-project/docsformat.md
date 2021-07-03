Title: Module Format and Transformation
license: https://www.apache.org/licenses/LICENSE-2.0

# Module Format and Transformation #

This document describes the format of the Apache HTTP Server documentation
xml source files and the technique used to transform them to html.

# Format #

A DTD is located in the [style
directory](http://httpd.apache.org/docs/current/style/) of the manual. An
example of the format with extensive comments is also available in
[mod_template.txt](mod_template.txt). Obviously, the file extension should
be `xml`. It was changed to make online viewing simpler.

To ensure that your documentation follows the defined format, you should
parse it using the DTD. Some help using Emacs with XML files is available
from [IBM
developerWorks](https://www.ibm.com/developerworks/xml/library/x-emacs/).

Various formatting syntax can be used such as:

<table border="1">
<tr><th>Keyword</th><th>Purpose</th></tr>
<tr><td>&lt;example&gt; ... &lt;/example&gt;</td><td>Format an example as monospaced font</td></tr>
<tr><td>&lt;highlight language="config"&gt ... &lt;/highlight&gt;</td>
<td>A configuration example, syntax highlighted. Can also use language="perl", "lua", or "C"</td></tr>
</table>

# Transformation #

The easiest way to view the transformed docs is to simply open the xml file
directly in a recent verions of MSIE, Netscape, or Mozilla. (MSIE 6 seems
to work consistently. The Gecko/Mozilla based browsers, however, do not
incorporate the referenced DTDs, so they have problems with documents
containing character entities). These browsers will read the xsl file and
perform the transformation for you automatically, so that you can see what
the final output will look like. This means that you can work on the docs
and check your work without any special transformation setup.

For the final presentation, it is still necessary to transform to html to
accomodate older browsers. Although any standards-compliant xslt engine
should do, changing engines can lead to massive diffs on the transformed
files. Therefore, we have chosen a single recommended transformation system
based on Xalan+Xerces Java and Ant. These are all Apache projects
distributed under the Apache license.

The only prerequisite for the transformation is a Java 8 or greater JVM.
Assuming you already have
[httpd/httpd/branches/2.4.x/docs/](http://svn.apache.org/repos/asf/httpd/httpd/branches/2.4.x/docs/)
(or the equivalent from another branch) checked out from SVN, here is what
you need to do to build: (If you need instructions on setting up SVN, see
[this page](http://www.apache.org/dev/version-control.html).)

><code>$ cd docs/manual<br></br>$ svn co
https://svn.apache.org/repos/asf/httpd/docs-build/trunk build<br></br>$ cd
build<br></br>$./build.sh all</code>

If you are running under win32, the `build.sh` script will work if cygwin is
installed. Alternatively, on Win32, you should be able to run the
`build.bat` script.

If you don't want to get the build files from SVN, you can download a
pkzipped version of the current build tools from our [distribution
directory](http://www.apache.org/dyn/closer.lua/httpd/docs/).

The default target builds only the english-language docs. To build other
docs, you should specify the language-code (ja, de, etc) as an argument, or
use the `all` target. To find the available languages, please see our
[translations page](translations.html).

You can get an overview of all possible build targets by typing:

> `
./build.sh -projecthelp
` 

# Validating changes #

Before submitting or committing changes, you should check that the XML is
correct, and the generated HTML:

><code>./build.sh validate-xml<br></br>./build.sh validate-xhtml</code>

# Special Files #

When adding a new module, the transformation process tries to generate an
appropriate entry within `mod/allmodules.xml` and to create an accompanying
metafile ( `newfilename.xml.meta` ). Since these tasks are written in
[perl](http://www.perl.org/) , you'll need a working perl installation for
this. If not, you should take these steps manually, or just leave a note on
the project mailing list so that someone else can do it.

# Generating a PDF version #

The PDF version of the docs is generated by transforming the xml files to
LaTeX using the "latex-en" Ant target. The XSLT style files for the
transformation are under `style/latex/`. Once you have the `.tex`
equivalent of each `.xml` file, you can use `pdflatex` to convert this into
a pdf file. Recommended versions of pdflatex can be obtained as part of
TeTeX (unix) or MikTeX (win32), but any version of TeX should be fine, as
long as it is sufficiently complete and modern. To generate the PDF, you
should process the `sitemap.tex` file, which contains the main LaTeX
document code and will include all the other files. The outputed PDF will
then be called `sitemap.pdf` , which you can rename how you choose.

Some notes about the XML to LaTeX conversion are necessary. Although HTML
and LaTeX have many similarities, there are enough differences between the
two to make targetting both outputs a difficult task. In particular, the
methods for handling tables are very different. To aid in the conversion to
LaTeX of tables designed for HTML, a `&lt;columnspec&gt;` section should be
added to each table. Inside the `&lt;columnspec&gt;` , place a `&lt;column
width=".xx"/&gt;` for each column in the table, where `xx` is the
percentage of the line-width devoted to that column. This will assist in
the conversion of basic tables. More complex stuff (like spanning rows or
columns) is not currently convertible.

Further, `pdflatex` cannot incorporate GIF files in a generated PDF, so any
graphics must be available in PNG format.

There are various other restrictive assumptions embedded in the XSLT that
work for the current docs, but may need to be modified in the future. For
example, the code that transforms HTML-style links to LaTeX
cross-references will only work within the main directory and one level of
subdirectory. Also, `&lt;pre&gt;` sections rarely work well in LaTeX
because of differences in escaping and formatting rules in `verbatim`
sections.

Finally, there are various differences in escaping rules between XML/HTML
and LaTeX. Some characters need to be backslash-escaped in LaTeX, and all
XML entitites (&amp;whatever;) must be converted to LaTeX equivalents. This
is currently handled for a limited set of characters using a big, ugly
search-replace in the XSLT. This may need to be modified in the future,
especially to handle translations. Pre-processing with a perl script and a
substitution table may be a better solution.