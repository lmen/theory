# Xpath What is
XPath is an expression language that allows to identify nodes from the a DOM tree.

Types of nodes
- Root node
- Element node
- Attribute node
- Text node
- Comment node
- Processing instructions node
- Namespaces nodes

Kinds do nodes
- nodeSet : a set of zero or more nodes
- node : one node from the


# XPath syntax

## Selection nodes

 Expr | description | Example
--- | --- | ---
 `nodename` | selects all nodes with the name *nodename* | recepi
  /  | selects from the root node | /recepies/recepi
 // | selects node in the document from the current node that matchs the selection no matter where they are | //ingredient
 @ | selects attributes | @name

 ## Predicates
 They serve to find nodes with a specific caracteristic thery are placed inside **[**  **]**

 Example:

 `recepi[@name="pudim"]` the predicate is attribute name with the value pudim

 ## Whildcards

 Wildcard | Description
 --- | ---
 * | Matches any element code
 @* | matches any attribute node
 node() | matches any node of any kind

## XPath axes
XPath was designed to work as the path of unix terminal, that is, instead of specifing always an absolute path it is possible to define a path relative from the the node last selected. In XPath land this is called axis.

Some of the existing axis are:

AXIS name | description | example
---|---|---
ancestor | selects all ancestors (parent, grandparent, ...) of the current node | ancestor::recepi
ancestor-or-self | selects all ancestor of the current node and the current node itself
attribute | selects all attributes of the current node
child | selects all children of current node
parent | selects the parent of the current node (same as ..)
self | selects the current node (same as .)

## Operatores

An XPath can have operators inside some of the operators are:

Operator | description | example | return value
---|---|---|---
\| | Computes two node-sets | //book \| //cd | returns a node-set with all book and cd elements
+ - * div mod | Addition, subtraction. Multiplication, dividion, modulus | 6+4 6-4 8 div 4 2 mod 4
= != < <= > => | equal different | price = 2 price > 3
or and | or and | price > 8 and price < 9


## Some examples


 Xpath | result
 --- | ---
//book[@year>2001]/title/text() | the title text of all  books with attribute year bigger than 2001
//book[price<8]/title/text() | the title text of all books with element price less than 8
//book[1]/title/text() | the title text from the first book element
//book[last()]/title/text() | the title text from the last book element
//book/author/text() | the author text of all books
count(//book/title) | return the number of all titles from a book
//book[starts-with(author,'Neal')] | all book elements where author element text starts with Neal
//book[contains(author,'Niven')] | all book elements where author element text contains Niven
//book[author='Neal Stephenson']/title/text() | the title text from all books where author is 'Neal Stephenson'
count(//book[author='Neal Stephenson']) | return the number of books where author is 'Neal Stephenson'
//inventory/comment() | reading a comment node inside an inventory

## How to use Xpath in Java

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setNamespaceAware(true); // never forget this!
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse("sample.xml");

XPathFactory xpathfactory = XPathFactory.newInstance();
XPath xpath = xpathfactory.newXPath();

System.out.println("n//1) Get book titles written after 2001");
// 1) Get book titles written after 2001
XPathExpression expr = xpath.compile("//book[@year>2001]/title/text()");
Object result = expr.evaluate(doc, XPathConstants.NODESET);
NodeList nodes = (NodeList) result;
for (int i = 0; i < nodes.getLength(); i++) {
    System.out.println(nodes.item(i).getNodeValue());
}
```


### References
[Content copied from](https://howtodoinjava.com/xml/how-to-work-with-xpaths-in-java-with-examples/)










