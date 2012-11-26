This page collects Google Refine recipes, small workflows and code fragments
that show you how to achieve specific things with Google Refine.

## String Manipulation

Here are some examples of possible types of common string manipulation
operations that you might encounter and how they can be achieved with the
[Google Refine Expression Language (GREL)](/p/google-
refine/wiki/GRELFunctions). See also [GREL String Functions](/p/google-
refine/wiki/GRELStringFunctions).

### Change `2010-05-31T01:10:05Z` to `05/31/2010`

One way to do this is use GREL's slice function.

    
       value.slice(5, 7) + '/' + value.slice(8, 10) + '/' + value.slice(0, 4)

### Trim whitespace from beginning and end of values

    
      value.trim()

or if you have non-breaking spaces `&nbsp` on both ends then you might try
this instead:

    
      split(escape(value,'xml'),"&#160;")[0]

### Titlecase that works on hyphenated names

If you have a "name" column with values like "FOTHERINGTON-THOMAS", this will convert correctly to "Fotherington-Thomas" (fuller explanation here: [Fotherington-Thomas](fuller explanation: http://www.rmorrison.net/mnemozzyne/google-refine-and-fotherington-thomas-issue))
    
      replace(toTitlecase(replace(value,"-",".")),".","-")


### "someprefix_a2343" -> "a2343"

You can use

    
      value.replace("someprefix_","")

basically replacing what you don't want with the empty string.

Note that "replace" replaces all instances of that first string in the cell
value so see the next recipe for that case.

#### "blah_2342_blah_1232" -> "2342_blah_1232"

Here we can't replace "blah" with "" because it would change the second
occurrence as well. So we can use this

    
       value.partition("blah_")[2]

that will chop the value in an array of three strings

    
       ["", "blah_", "2342_blah_1232" ]

and we get the 3rd value (remember that the arrays in Google Refine, like in
most programming languages start from 0, so `[2]` is actually the third
value).

### "a:b:c:d:e" -> "b:c:d"

Sometimes the strings are more complicated and the only thing they have in
common is a character separator. In that case it's handy to use "split" and
"join" together like this:

    
       value.split(":")[1,3].join(":")

which "splits" the string around the ":" separating character in an array of 5
strings

    
       ["a","b","c","d","e"]

then you select only the range between the 2nd and 4th and you get

    
       ["b","c","d"]

then you join them together with the same separator character ":" and you get
what you wanted "b:c:d".

### Pad with leading zeroes

If you need to fill out a number with for example leading zeroes up to four
characters try,

    
    "0000"[0,4-value.length()] + value

This provides a string of zeroes from which we ask for the first
4-length(value) characters using the string slicing action of [start, offset].

### Parse JSON and Create Custom Arrays using forEach()

    
    forEach(value.parseJson().result,v,[v.score,v.name,v.mid])

then getting your 1st custom result:

    
    forEach(value.parseJson().result,v,[v.score,v.name,v.mid])[0]

to get all instances from a JSON array called "keywords" having the same
object string name of "text", combine with the forEach() function to iterate
over the array:

    
    forEach(value.parseJson().keywords,v,v.text).join(":::")

### Removing duplicate rows when Exact values are found in a column

In Release 2.1+ there is now a Facet -> Customized Facets -> Duplicates Facet
that can be applied on more than one column as needed. This easily applies for
you a facetCount(value, 'value', 'Column') > 1 which shows ALL rows having
more than 1 duplicate value in the column.

However, to ONLY remove subsequent instances of duplicates (any string), use
this method:

  1. Sort (on the column that has duplicates or potentially) 
  2. Reorder rows permanently (this option is in blue color SORT link in header) 
  3. Edit cells -> Blank down 
  4. Facet -> Customized Facets -> Facet by blank (click True) 
  5. All -> Edit rows -> Remove all matching rows 

### Facet and Count duplicate patterns found in a cell value at each row

With strings that have duplicate words in them such as "Beef, Ground Beef",
you may wish to see a count of the duplicates in each column's row. Using
value.ngram(1) only separates the words, but doesn't give a count. So, first,
standardize spacing in the cell value or find good alternate separation chars
to use on your particular pattern if needed, and perhaps clean up spaces with
GREL tranform of

    
    value.replace(/\s+/,' ')

then using Jython, you can create a Custom Facet using something such as:

    
    from sets import Set
    # function to show count of Duplicates in a List returning an array
    def countDuplicatesInList(dupedList):
       uniqueSet = Set(item for item in dupedList)
       return [(item, dupedList.count(item)) for item in uniqueSet]
    
    v = countDuplicatesInList(value.split(" "))
    for x in v[:]: # makes a slice copy of the entire list
      if x[1]>1: # compares if the 2nd value in each mini list (x) is greater than 1, such as ["Beef", 2] 
        return x

### Find a sub pattern that exists at the end of a string

Sometimes you have a need beyond just `endsWith` to find a varying substring
and want to know if that variable pattern is found at the end of a string. You
can add a Custom Text Facet and find this variable pattern with regex delmited
by `/` characters such as `/.*(PR\d+)/`

    
    value.endsWith(get(value.match(/.*(PR\d+)/),0))

Here you have to wrap with `get()` in order to ask if it `endsWith` the string
version of the returned array from `match`. `endsWith` then returns a boolean
of True or False if the string ends with your regex pattern, in this case,
cells that end with reference numbers that begin with PR followed by any
number of decimal digits.

### Remove the last word in a string

This uses `smartSplit` to find the last word `[-1]` and then wraps `partition`
to keep only the first phrase or part of the string remaining with `[0]`.

    
    value.partition(smartSplit(value," ")[-1])[0]

### "00003400340300004" -> ["000034","0034","03","00004"]

Sometimes your content has no char separator because the separation is
performed by alignment (in the case of non-delimited files) and padded to
certain sizes. In that case you can use

    
       value.splitByLengths(6,4,2,5)

where those numbers are the length of the fields respectively.

### split / map / join

Running this expression

    
      forEach(value.split(" "), v, v + v.length()).join(";")

on the text

    
      abc defg hi

yields

    
      abc3;defg4;hi2

A similar trick can be used to compute, say, word length average

    
      with(value.split(" "), a, sum(forEach(a, v, v.length())) / a.length())

### Merging several columns

You'd need to use the 'cells' variable:

    
      cells["col1"].value + ", " + cells["col2"].value

If any of cells in the columns are blank, the merge will fail for that row. To
fix, create a facet of blank cells with "Text Facet" ⇒ "Customized Facets" ⇒
"Facet by Blank". Then use "Edit Cells" ⇒ "Transform ..." and enter a string
with a space: `' '`.

### "AT&amp;amp;T" --> "AT&T"

Sometimes you get content that was extracted from web pages or XML documents
and contain entities. These are very hard to deal with manually, mostly
because there are tons of potential string values between "&" and ";". Google
Refine provides you a convenient command to do this in GREL

    
      value.unescape("html")

Note the few entities are not part of HTML and are part of XML, so if you want
to be safe, you can do

    
      value.unescape("html").unescape("xml")

which will take care of them both.

Note that sometimes a string can get escaped several times:
"AT&amp;amp;amp;amp;amp;T" (yes, trust us, we've seen that in presumably good
data sets). When you do a transform on such cells, make sure you check the
"Re-transform until no change" checkbox. This causes the expression to get run
on each cell's content until the output is the same as the input (or up to 10
times).

### XML parsing & stripping

Suppose you have XML or ATOM feed data in your cells that you may have
gathered with the Add Column by Fetching URLs feature in Refine or some other
source and you need to parse and strip certain tag elements such as:

    
        <entry>
            <title>San Francisco chronicle.</title>
            <link href="http://chroniclingamerica.loc.gov/lccn/sn82003402/" />
            <id>info:lccn/sn82003402</id>
            <author><name>Library of Congress</name></author>
            <updated>2010-09-03T00:01:07-04:00</updated>
        </entry>
        
        <entry>
            <title>The San Francisco call.</title>
            <link href="http://chroniclingamerica.loc.gov/lccn/sn85066387/" />
            <id>info:lccn/sn85066387</id>
            <author><name>Library of Congress</name></author>
            <updated>2010-09-03T00:18:25-04:00</updated>
        </entry>

You can strip out the elements that your interested in from any XML or ATOM
feed and create an array of those elements values for further processing. In
the above example, I only want the lccn id and title for all the **entry**
records. This is useful when working with the
[http://chroniclingamerica.loc.gov](http://chroniclingamerica.loc.gov) API or
any XML or ATOM feed.

    
    forEach(value.split("<entry>"), v, v.partition("<id>info:lccn/")[2].partition("</id>")[0]+
    ","+v.partition("<title>")[2].partition("</title>")[0])

The above basically says that for each entry record (which creates an array),
I want only the string partition between my **id** tags and **title** tags.
The resulting output then looks something like:

    
    [ ",Chronicling America Title Search Results",
     "sn82003402,San Francisco chronicle.",
     "sn85066387,The San Francisco call.",
     "sn86091369,San Francisco Sunday examiner &amp; chronicle.",
     "sn82006825,The San Francisco examiner." ]

### "AÃ¯n TÃ©muchent" ---> "Aïn Témuchent"

Sometimes content is aggregated from various sources and some where improperly
decoded. This results in strings that contain 'garbled' content in the form of
weird spurious characters. Fixing that is normally a very tedious and time
consuming process, but Google Refine contains a handy GREL command
specifically for that

    
      value.reinterpret("utf-8")

that does exactly that.

As a tip, if you see two weird characters where you think there should be one,
try reinterpreting with "utf-8" which normally solves it. Other encoding
values that are useful to try are "latin-1" (for european content), "Big5"
(for chinese content"), etc. You can get a list of all the supported encodings
[here](http://java.sun.com/j2se/1.5.0/docs/guide/intl/encoding.doc.html). Re-
encoding can also be done outside of Google Refine with text tools such as
PSPad, Notepad++, etc.

### Question Marks � showing in your data

This is the "Replacement Character - used to replace an incoming character
whose value is unknown or unrepresentable." These will show if there are
encoding issues with your data. Because the data is actually changed at import
time, this may not be recoverable without re-importing, but you can try
reinterpreting as above. Also there may be non-breaking spaces `&nbsp`
Unicode(160). You can inspect with `unicode(value)` and replace with regular
spaces Unicode(32).

You can also try this quick fix to remove non-breaking spaces on both ends of
your string:

    
      split(escape(value,'xml'),"&#160;")[0]

**NOTE** To access and use the Non-breaking space character itself such as `&nbsp` or `\u00A0` or `[160]` in GREL expressions on Windows systems type on the numeric keypad: ALT 2 5 5 ,which will still display as a regular space in the expression window, but will actually be a non-breaking space character. Example: `value.replaceChars(" ", " ")`

## Numerical Conversions

### Convert to Decimal Latitude or Longitude

For a string holding longitude/latitude information in the format XX degrees,
XX minutes, XX.XX seconds the following can be used.

    
       forNonBlank(value[0,2], v, v.toNumber(), 0) +
       forNonBlank(value[2,4], v, v.toNumber()/60, 0) +
       forNonBlank(value[4,6], v, v.toNumber()/3600, 0) +
       forNonBlank(value[6,8], v, v.toNumber()/360000, 0)

## Error Handling

### forNonBlank

Sometimes the cells in a column are not uniform, but your expression has to be
flexible enough to handle them all. One case of nonuniformity is that the
strings in the cells are not all of the same length, e.g., 3 cells in a column
might contain

    
      200810
      20090312
      2010

This means that this formula

    
      value[0,4] + "-" + value[4,6] + "-" + value[6,8]

only works on the second cell. To catch the other cases, use the forNonBlank
control:

    
      value[0,4] + forNonBlank(value[4,6], v, "-" + v, "") + forNonBlank(value[6,8], v, "-" + v, "")

This new expression transforms those cells to

    
      2008-10
      2009-03-12
      2010

What forNonBlank does is test its first argument and decides what to do based
on whether that is blank (null or empty string) or not. If it's not blank,
bind that value to the variable name in the second argument (v) and evaluates
the third argument ("-" + v). Otherwise, it evaluates the forth argument ("").

## Spot Potential Encoding Issues

While Google Refine offers a convenient way to fix encoding issues with the
"reinterpret" GREL command, it also helps you find where these problems could
be since they can be hidden if most of the content has been properly decoded.

To do this, there is a special numeric facet called "Unicode charcode Facet"
that generates a distribution of which unicode characters are used in a
particular column. That distribution will allow you to spot outliers, meaning
characters that are used infrequently and that might suggest encoding issues.
You can use that char distribution facet to 'scan' and inspect the values for
yourself and then use fix their encoding with the 'reinterpret' GREL function.

## Spot Values Potentially Placed in the wrong Column

Another useful numeric facet is the "Text Length Facet" which builds a
distribution of lengths of the strings contained in a particular column. Here
too spotting outliers, meaning strings that are too big or too small, might be
an indication of a problem.

## ISBN Calculations & Manipulations

### ISBN-10

Compute ISBN-10 check
digit:`grel:sum(forRange(1,10,1,i,toNumber(value[i])*i))%11`

Check computed value against check digit:
`grel:if(value==10,'X',toString(round(value)))==cells['ISBN10'].value[10]`

Convert ISBN-10 check digit to number ('X' == 10)
`grel:if(toLowercase(value[10])=='x',10,toNumber(value[10]))`

### Convert an ISBN-10 to ISBN-13

Take the first 9 digits and prepend 978: `grel:'978'+value[1,10]`

Compute the ISBN-13 check digit:
`grel:(10-(sum(forRange(0,12,1,i,toNumber(value[i])*(1+((i)%2*2)))) %10))%10`

Append the check digit to the 12-digit base