# PHP - The Wonderful World of XML Parsers
## Basic background, understanding, misconceptions & usage of XML parsers in PHP

**Published:**  2015/04/07 (April 7 2013)

XML is one of the most universal and versatile markup languages in coding. It is used everywhere and can be adapted for almost anything. Many languages have parsers built in for XML and are used for rss feeds, news list, storing of data, API's & more.

Its no wonder the PHP has more than one type of XML parser, however the question is why? What's the difference between SimpleXML, SAX, XML Parser & XMLReader? What's libXML? Whats the different from a SAX implementation and DOM implementation? And why are some XML better than others? All that and much much more in the Wonderful World of XML Parsers!

If your lost from reading the above, seeing new XML parsers/terms or just interested and want to learn more continue reading!

*Quick Note: Before we start your should be fairly familiar with XML. Not an expert but understand some basics of it.*

## The Basics
First things first, let's start off with a few definitions so we are all on the same page.

**Parser**
> A program for parsing or reading data and turning it into a usable form.

**SAX (Simple API for XML) & SAX 2**
> Developed by David Megginson; A widely-used specification that tells XML how to pass information efficiently from a XML documents to software applications, script, etc... It is not official and was not created by a XML consortium however it is so well constructed it is used like a standard; SAX 2 is the updated and preferred version.

**DOM (Document Object Model)**
> DOM allows scripts to dynamically access and update the content, structure and style of documents by loading the entire document.

**Pull parser**
> Acts as a cursor going forward on the document stream and stopping at each node on the way. 

**LibXML & libXML 2**
> Library that is used for a bunch of extensions such as simpleXML; libXML 2 is the updated and preferred version.

**SimpleXML**
> Is a XML Parser that loads the entire document and then allows access/editing of the data; DOM implementation.

**XML Parser**
> The most basic XML parser in PHP; It allows you to create your XML Reader from scratch; SAX implementation

**XMLReader**
> A type of XML Parser that scan the XML document for a signaled key and returns the value. It is important to note that it does not load the entire document the first time; SAX implementation/Pull parser 

## What's the best XML Parser?
The question its self is flawed. No one XML parser is the best. Many of the XML parsers borrow from each other. So each XML parser benefits each type of scenario. Lets see the advantages and disadvantages of each XML Parser.

First lets understand the difference and trade offs of Sax implementation vs DOM implementation.

SAX is a very fast and efficient by not loading the entire XML document and scanning for only specific tags and returning those values. It is far superior in retrieving data in large XML files. SAX drawback is that it is harder to setup. It's API thought fast and efficient is slow to code and requires addition thinking/logic. It may also require slightly more time to maintain and add extensions to the code.

DOM is extremely easy to use and takes little to no time to setup. However DOM is slow and very memory heavy. DOM requires the code to load the XML document and then can access it.

TLDR; SAX is fast & efficient, however slower to deploy. DOM is easy to setup and use, however slower for single items.

You should now understand the fundamental differences of SAX and DOM, lets see how the implementation effect the different parsers.

**SAX & DOM**
> Implemented in PHP 4.0.0. >=
> We have already covered the differences and advantages/disadvantages.

**SimpleXML**
> Implemented in PHP 5.0.0 >=
> SimpleXML is very simple to use, hence the name "simple"XML. However this has a tradeoff (you probably already guessed it) SimpleXML is slower, not by a lot (check the speed test far into the article) but still slightly slower. With these characters it's obvious to see it was built off DOM.

**XMLReader**
> Implemented in PHP 5.0.0 >=
> XMLReader is fast and efficient and hence it is based off the SAX API. It however is rather weird to setup and has one of the worst documentation on PHP.net I have ever seen giving you a laundry list and no examples to work off. It is also worth noting that it calls it's self a "Pull parser " rather than a SAX parser. They are basically the same thing.

Next we will look at some examples of SAX & DOM Parsers, and then try simpleXML and XMLReader.

## Examples
Before we dive into simpleXML and XMLReader it would be a good idea to start with the basics and learn SAX & DOM, we will than move to SimpleXML/XMLReader.

*Note: The code will not be broken-up but better yet commented at every step so you can read the code while playing around it. I will also supply a single item version and a list version.*

The XML for these examples:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<blogs>
    <blog id="1">
        <title>Hello World</title>
        <author>FireDart</author>
        <date>07/06/2012</date>
        <content>This is some content</content>
    </blog>
    <blog id="2">
        <title>Hello World 2</title>
        <author>FireDart</author>
        <date>07/010/2012</date>
        <content>This is some content 2</content>
    </blog>
</blogs>
```

### SAX
*Note: Due to the fact the SAX requires specific code to setup for each xml feed this code will need to be changed if you plan on using it. I have also create a class for it to make it easier.*

```php
<?php
/*
 * saxParser
 * 
 * Create an easy to use class for SAX Parsing
 */
class saxParser {
    /*
     * @var $sax The SAX variable
     */
    public $sax;
    /*
     * @var $xml The XML data
     */
    public $xml;
    /*
     * @var $tags Stores the xml array
     */
    public $tags;
    /*
     * @var $blog_id Stores the current blog id
     */
    public $blog_id;
    /*
     * @var $blog_content Stores the current blog content
     */
    public $blog_content;
    
    /*
     * saxParser Constant
     * 
     * Sets SAX settings & loads xml file
     * 
     * @param xml $xml File path to xml
     */
    public function __construct($xml) {
        $this->sax = xml_parser_create();

        xml_set_object($this->sax, $this);
        
        xml_parser_set_option($this->sax, XML_OPTION_CASE_FOLDING, false);
        xml_parser_set_option($this->sax, XML_OPTION_SKIP_WHITE, true);
        xml_set_element_handler($this->sax, 'start_element', 'end_element');
        xml_set_character_data_handler($this->sax, 'content_element');
        
        $this->xml = file_get_contents($xml);
    }
    /*
     * start_element
     * 
     * This declars what is wrapped before the content
     * 
     * @param string $sax The handler
     * @param string $tag The tag
     * @param array $attribute Any attributes on the tags
     */
    private function start_element($sax, $tag, $attribute) {
        if($tag == 'blog') {
            $this->blog_id = $attribute['id'];
            $this->tags[$this->blog_id] = array();
        }
    }
    /*
     * end_element
     * 
     * This declars what is wrapped after the content
     * 
     * @param string $sax The handler
     * @param string $tag The tag
     */
    private function end_element($sax, $tag) {
        switch($tag) {
            case("title"):
                $this->tags[$this->blog_id]['title'] = $this->blog_content; 
                break;
            case("author"):
                $this->tags[$this->blog_id]['author'] = $this->blog_content; 
                break;
            case("date"):
                $this->tags[$this->blog_id]['date'] = $this->blog_content; 
                break;
            case("content"):
                $this->tags[$this->blog_id]['content'] = $this->blog_content; 
                break;
        }
    }
    /*
     * content_element
     * 
     * This returns the value of the tag
     * 
     * @param string $sax The handler
     * @param string $data The content of the node
     */
    private function content_element($sax, $data) {
        $this->blog_content = trim($data);
    }
    
    /*
     * parse
     * 
     * return data
     *
     * @param array $tags Tags to look for
     */
    public function parse() {
        xml_parse($this->sax, $this->xml, true);
    }
    
    /*
     * destruct
     * 
     * Free the parsered data
     */
    public function __destruct() {
        xml_parser_free($this->sax);
    }
}

if(isset($_GET['display']) && $_GET['display'] == 'list') {

    $sax = new saxParser('xml.xml');

    $sax->parse();

    foreach($sax->tags as $tag) {
        echo $tag['title'] . '<br />';
        echo $tag['author'] . '<br />';
        echo $tag['date'] . '<br />';
        echo $tag['content'] . '<br />';
        echo '<hr />';
    }
    
} else {

    $sax = new saxParser('xml.xml');

    $sax->parse();
    
    echo $sax->tags['1']['title'] . '<br />';
    echo $sax->tags['1']['author'] . '<br />';
    echo $sax->tags['1']['date'] . '<br />';
    echo $sax->tags['1']['content'] . '<br />';
    
}
```

### DOM

Select the first item:
```php
/* Laod DOM */
$doc = new DOMDocument();
/* Fetch XML File */
$doc->load('xml.xml');
/* Select the node BUT select the first item only! */
$blog = $doc->getElementsByTagName("blog")->item(0);
/* Get the tags and the node value and echo it out */
$title   = $blog->getElementsByTagName('title')->item(0)->nodeValue;
$author  = $blog->getElementsByTagName('author')->item(0)->nodeValue;
$date    = $blog->getElementsByTagName('date')->item(0)->nodeValue;
$content = $blog->getElementsByTagName('content')->item(0)->nodeValue;
/* Echo Everything */
echo $title . '<br />' . $author . '<br />' . $date . '<br />' . $content . '<hr />';
```

List all the items:

```php
/* Laod DOM */
$doc = new DOMDocument();
/* Fetch XML File */
$doc->load('xml.xml');
/* Select the node */
$blogs = $doc->getElementsByTagName("blog");
/* Foreach node named "blog" load the data inside it */
foreach($blogs as $blog) {
    /* Get the tags and the node value and echo it out */
    $title   = $blog->getElementsByTagName("title")->item(0)->nodeValue;
    $author  = $blog->getElementsByTagName("author")->item(0)->nodeValue;
    $date= $blog->getElementsByTagName("date")->item(0)->nodeValue;
    $content = $blog->getElementsByTagName("content")->item(0)->nodeValue;
    /* Echo Everything */
    echo $title . '<br />' . $author . '<br />' . $date . '<br />' . $content . '<hr />';
}
```

### SimpleXML
Select the first item:

```php
/* Load the XML File */
$xml = simplexml_load_file('xml.xml');
/* Select the nodes */
echo $xml->blog->title . '<br />';
echo $xml->blog->author . '<br />';
echo $xml->blog->date . '<br />';
echo $xml->blog->content . '<br />';
```
List all the items:
```php	
/* Load the XML File */
$xml = simplexml_load_file('xml.xml');
/* List each node */
foreach($xml->blog as $blog) {
    echo $blog->title . '<br />';
    echo $blog->author . '<br />';
    echo $blog->date . '<br />';
    echo $blog->content . '<br />';
    echo '<hr />';
}
```
### XMLReader

```php
<?php
/*
 * XMLReader Class
 * 
 * Since XMLReader is kinda complicated this class should help
 */
class XMLR {
    /*
     * @var $reader Contains the XMLReader class
     */
    public $reader;
    /*
     * XMLR Constant
     * 
     * Creates a new instance of XMLReader to be used in this class
     * Also opens the XML File
     * 
     * @param xml $xml File path to xml
     */
    public function __construct($xml) {
        /* Create new instance of XMLReader */
        $this->reader = new XMLReader();
        /* Open XMl Reader */
        $this->reader->open($xml, 'UTF-8');
    }
    /*
     * Read
     * 
     * This function is used to read xml using the XMLReader() class.
     * It finds on element tag at a time and outputs the data
     * 
     * @param string $tag The tag/node to look for in the xml document
     */
    public function read($tag) {
        /* Read the XML File */
        while($this->reader->read()) {
            /* Search for any nodes that re elements and with the name of the tag */
            if($this->reader->nodeType == XMLReader::ELEMENT && $this->reader->localName == $tag) {
                /* Read the node value */
                $this->reader->read();
                /* Return Node Value */
                return $this->reader->value;
            }
        }
        /* Close XMLReader */
        $this->reader->close();
    }
    
    /*
     * Read
     * 
     * Creates an array of data with all the data
     * 
     * @param string/array $tags The tag or tags to find
     */
    public function xmlList($tags) {
        
        $lists = array();
        /* Read the XML File */
        while($this->reader->read()) {
            $i   = 0;
            $len = count($tags);
            foreach($tags as $tag) {
        
                /* Search for any nodes that re elements and with the name of the tag */
                if($this->reader->nodeType == XMLReader::ELEMENT && $this->reader->localName == $tag) {
                    /* Read the node value */
                    $this->reader->read();
                    /* If first item in array create $list array */
                    if($i == 0) {
                        $list = array();
                    }
                    /* Return Node Value */
                    $list[$tag] = $this->reader->value;
                    /* If list item in array compile everything in $lists array */
                    if($i == $len - 1) {
                        $lists[] = $list;
                    }
                }
                $i++;
            }
        }
        
        return $lists;
        
        /* Close XMLReader */
        $this->reader->close();
    }
}

if(isset($_GET['display']) && $_GET['display'] == 'list') {

    $xmlr = new XMLR('xml.xml');

    $data = $xmlr->xmlList(array('title', 'author', 'date', 'content'));

    foreach($data as $xml) {
        echo $xml['title'] . '<br />' . $xml['author'] . '<br />' . $xml['date'] . '<br />' . $xml['content'] . '<hr />';
    }

} else {
    $xmlr = new XMLR('xml.xml');
    echo $xmlr->read('title') . '<br />' . $xmlr->read('author') . '<br />' . $xmlr->read('date') . '<br />' . $xmlr->read('content');
}
```

As you can see SAX & XML Reader are significantly more complicated to setup and modify while DOM and SimpleXML are much easier, however, lets now talk about the parsing speeds of SAX & DOM.

## What about speed?
It's no secret that XML is sometimes slow to parser, however this is not always true. Like already stated above SAX does a great job speed wise and has very low usage but only on very large files IF you are looking for a specific node. After a certain amount of calls to the xml file a DOM implementations become more efficient since the XML is loaded into the memory and does not need to be cached again and again.

| Parser              | Speed          |
| ------------------- | -------------- |
| SAX [Single]        | 0.0007 seconds |
| SAX [List]          | 0.0007 seconds |
| DOM [Single]        | 0.0006 seconds |
| DOM [List]          | 0.0030 seconds |
| SimpleXML [Single]  | 0.0038 seconds |
| SimpleXML [List]    | 0.0007 seconds |
| XML Reader [Single] | 0.0006 seconds |
| XML Reader [List]   | 0.0009 seconds |

## Resources, Discussions & Papers
Below is a list of resources I used to create this tutorial, please visit them to learn more information

**From php.net:**
* http://www.php.net/manual/en/refs.xml.php
* http://php.net/manual/en/book.xml.php (Global XML stuff and XML Parser)
* http://php.net/manual/en/book.simplexml.php (SimpleXML)
* http://php.net/manual/en/book.xmlreader.php (XMLReader, terrible documentation)

**SAX**
* http://www.saxproject.org/ (Official Site)
* http://www.megginson.com/downloads/SAX/
* http://www.brainbell.com/tutorials/php/Parsing_XML_With_SAX.htm
* http://www.pixel2life.com/publish/tutorials/35/the_ultimate_guide_to_parsing_xml_part_1_using_sax_/
* http://php.net/manual/en/ref.xml.php

**DOM**
* http://www.php4every1.com/tutorials/php-domdocument-tutorial/ (Very good, best examples I could find)
* http://www.ibm.com/developerworks/library/os-xmldomphp/

**SimpleXML**
* http://php.net/manual/en/book.simplexml.php (SimpleXML)

**XMLReader**
* http://www.codem.com.au/streams/2009/web-development/consuming-xml-fast-with-php-and-xmlreader.html
* http://www.itsalif.info/content/php-5-xmlreader-reading-xml-namespace-part-2
* http://www.video2brain.com/en/lessons/reading-xml-with-xmlreader-1
* http://stackoverflow.com/questions/5128811/how-to-use-xmlreader-domdocument-with-large-xml-file-and-prevent-500-error
* http://posterous.richardcunningham.co.uk/using-a-hybrid-of-xmlreader-and-simplexml
* https://blog.liip.ch/archive/2004/05/10/processing_large_xml_documents_with_php.html

**Benchmarks**
* http://www.devx.com/xml/Article/16922/1954
