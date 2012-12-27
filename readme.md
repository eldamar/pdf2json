Introduction
====

A server side PDF parser Node.js module that converts PDF binaries to JavaScript objects, which can be easily serialized to

JSON when running in node.js based web service or web app.

Install:
====
>npm install pdf2json

Example:
====
```javascript

        var nodeUtil = require("util"),
            _ = require('underscore'),
            PFParser = require("./pdf2json/pfparser");

        var pdfParser = new PFParser();

        pdfParser.on("pdfParser_dataReady", _.bind(_onPFBinDataReady, self));

        pdfParser.on("pdfParser_dataError", _.bind(_onPFBinDataError, self));

        var pdfFilePath = "data/" + taxYear + "/" + stateName + "/" + pdfId + ".pdf";

        pdfParser.loadPDF(pdfFilePath);

```

API Reference
=====

* loadPDF:

        function loadPDF(pdfFilePath);

        load PDF file from specified file path asynchroniously

        if failed, event pdfParser_dataError will be raised with error object

        when success, event pdfParser_dataReady will be raised with JavaScript data object, which can be saved as

        JSON file in test or serialized to JSON when running in web service


Output format Reference
=====

Current parsed data has four main sub objects to describe the PDF document.

* 'Agency': the main text identifier for the PDF document
* 'Id': the XML meta data that embedded in PDF document
* 'Pages': array of 'Page' object that describes each page in the PDF, including sizes, lines, fills and texts within the page. More info about 'Page' object can be found at 'Page Object Reference' section
* 'Width': the PDF page width in page unit

Page object Reference
-----

Each page object within 'Pages' array describes page elements and attributes with 5 main fields:

* 'Height': height of the page in page unit
* 'HLines': horizontal line array, each line has 'x', 'y' in relative coordinates for positioning, and 'w' for width, plus 'l' for length. Both width and length are in page unit
* 'Vline': vetical line array, each line has 'x', 'y' in relative coordinates for positioning, and 'w' for width, plus 'l' for length. Both width and length are in page unit
* 'Fills': an array of rectangular area with solid color fills, same as lines, each 'fill' object has 'x', 'y' in relative coordinates for positioning, 'w' and 'h' for width and height in page unit, plus 'clr' to reference a color with index in color dictionary. More info about 'color dictionary' can be found at 'Dictionary Reference' section.
* 'Texts': an array of text blocks with position, actual text and stylying informations:
    * 'x' and 'y': relative coordinates for positioning
    * 'clr': a color index in color dictionary, same 'clr' field as in 'Fill' object
    * 'A': text alignment, including:
        * left
        * center
        * right
    * 'R': an array of text run, each text run object has two main foelds:
        * 'T': actual text
        * 'S': style index from style dictionary. More info about 'Style Dictionary' can be found at 'Dictionary Reference' section

Interactive Forms Elements
=====

v0.1.5 added interactive forms element parsing, including text input, radio button, check box, link button and dropdown list.

Interactive forms can be created and editted in Acrobat Pro for AcroForm, or in LiveCycle Designer ES for XFA forms. Current implementation for buttons only supports "link button": when clicked, it'll launch a URL specified in button properties. Examples can be found at f1040ezt.pdf file under test/data folder.

All interactive form elements parsing output will be part of corresponding 'Page' object where they belong to, radio buttons and check boxes are in 'Boxsets' array while all other elements objects are part of 'Fields' array.

Each object with in 'Boxset' can be either checkbox or radio button, the only difference is that radio button object will have more than one element in 'boxes' array, it indicates it's a radio button group. The following sample output illustrate one checkbox ( Id: F8888 ) and one radio button group ( Id: ACC ) in the 'Boxsets' array:

             Boxsets: [
                {//first element, check box
                    boxes: [ //only one box object object in this array
                    {
                        x: 47,
                        y: 40,
                        w: 3,
                        h: 1,
                        style: 48,
                        TI: 39,
                        AM: 4,
                        id: {
                            Id: "F8888",
                        },
                        T: {
                            Name: "box"
                        }
                     }
                     ],
                     id: {
                        Id: "A446",
                     }
                },//end of first element
                {//second element, radio button group
                    boxes: [// has two box elements in boxes array
                    {
                        x: 54,
                        y: 41,
                        w: 3,
                        h: 1,
                        style: 48,
                        TI: 43,
                        AM: 132,
                        id: {
                            Id: "ACCC",
                        },
                        T: {
                            Name: "box"
                        }
                    },
                    {
                        x: 67,
                        y: 41,
                        w: 3,
                        h: 1,
                        style: 48,
                        TI: 44,
                        AM: 132,
                        id: {
                            Id: "ACCS",
                            EN: 0
                        },
                        T: {
                            Name: "box"
                        }
                    }
                    ],
                    id: {
                        Id: "ACC",
                        EN: 0
                    }
                }//end of second element
             ] //end of Boxsets array

'Fields' array contains parsed object for text input (Name: 'alpha'), dropdown list (Name: 'apha', but has 'PL' object which contains label array in 'PL.D' and value array in 'PL.V'), link button (Name: 'link', linked URL is in 'FL' field). Some examples:

Text input box example:

                {
                    style: 48,
                    T: {
                        Name: "alpha",
                        TypeInfo: { }
                    },
                    id: {
                        Id: "p1-t4[0]",
                        EN: 0
                    },
                    TI: 0,
                    x: 6.19,
                    y: 5.15,
                    w: 30.94,
                    h: 0.85
                },

Dropdown list box example:

               {
                    x: 60,
                    y: 11,
                    w: 4,
                    h: 1,
                    style: 48,
                    TI: 13,
                    AM: 388,
                    mxL: 2,
                    id: {
                        Id: "ST",
                        EN: 0
                    },
                    T: {
                        Name: "alpha",
                        TypeInfo: {
                        }
                    },
                    PL: {
                        V: [
                            "",
                            "AL",
                            "AK"
                        ],
                        D: [
                        "%28no%20entry%29",
                        "Alabama",
                        "Alaska"
                        ]
                    }
               }


Link button example:

                {
                    style: 48,
                    T: {
                        Name: "link"
                    },
                    FL: "http://www.github.com",
                    id: {
                        Id: "quad8",
                        EN: 0
                    },
                    TI: 0,
                    x: 52.35,
                    y: 28.35,
                    w: 8.88,
                    h: 0.85
                }


Notes
=====

PDF.JS is designed and implemented to run within browsers that have HTML5 support, it has some depencies that's only available from browser's JavaScript runtime, including:

* XHR Level 2 (for Ajax)
* DOMParser (for parsing embedded XML from PDF)
* Web Worker (to enable parsing work run in a separated thread)
* Canvas (to draw lines, fills, colors, shapes in browser)
* Others (like web fonts, canvas image, DOM manipulations, etc.)

In order to run PDF.JS in Node.js, we have to address those dependencies and also extend/modify the fork of PDF.JS. Here below are some works implemented in this pdf2json module to enable pdf.js running with node.js:

* Global Variables
    * pdf.js' global objects (like PDFJS and globalScope) need to be wrapped in a node module's scope
* API Dependencies
    * XHR Level 2: I don't need XMLHttpRequest to load PDF asynchronously in node.js, so replaced it with node's fs (File System) to load PDF file based on request parameters;
    * DOMParser: pdf.js instantiates DOMParser to parse XML based PDF meta data, I used xmldom node module to replace this browser JS library dependency. xmldom can be found at https://github.com/jindw/xmldom;
    * Web Wroker: pdf.js has "fake worker" code built in, not much works need to be done, only need to stay aware the parsing would occur in the same thread, not in background worker thread;
    * Canvas: in order to keep pdf.js code intact as much as possible, I decided to create a HTML5 Canvas API implementation in a node module. It's named as 'PDFCanvas' and has the same API as HTML5 Canvas does, so no change in pdf.js' canvas.js file, we just need to replace the browser's Canvas API with PDFCanvas. This way, when 2D context API invoked, PDFCanvas just write it to a JS object based on the JSON format above, rather than drawing graphics on html5 canvas;
* Extend/Modify PDF.JS
    * Fonts: no need to call ensureFonts to make sure fonts downloaded, only need to parse out font info in CSS font format to be used in JSON's texts array.
    * DOM: all DOM manipulation code in pdf.js are commented out, including creating canvas and div for screen rendering and font downloading purpose.
    * Interactive Forms elements: (in process to support them)
    * Leave out the support to embedded images

After the changes and extensions listed above, this pdf2json node.js module will work either in a server environment ( I have a RESTful web service built with resitify and pdf2json, it's been running on an Amazon EC2 instance) or as a standalone commanline tool (something similar to the Vows unit tests).

Known Issues
===

This pdf2json module's output does not 100% maps from PDF definitions, some of them is because of time limitation I currently have, some others result from the 'dictionary' concept for the output. Given these known issues or unsupported features in current implementation, it allows me to contribute back to the open source community with the most important features implemented while leaving some improvement space for the future. All un-supported featurs listed below can be resolved technically some way or other, if your use case really requires them:

* Embedded content:
    * All embedded content are igored, current implementation focuses on static contents and interactive forms. Un-supported PDF embedded contents includes 'Images', 'Fonts' and other dynmatic contents;
* Text and Form Styles:
    * text and form elements styles has partial support. This means when you have client side renderer (say in HTML5 canvas or SVG renderer), the PDF content may not look exactly the same as how Acrobat renders. The reason is that we've used "style dictionary" in order to reduce the payload size over the wire, while "style dictionary" doesn't have all styles defined. This sort of partial support can be resolved by extending those 'style dictionaries'. Primary text style issues include:
        * Font face: only limit to the font families defined in style dictionry
        * Font size: only limit to 6, 8, 10, 12, 14, 18 that are defined in style dictionary, all other sized font are mapped to the closest size. For example: when a PDF defines a 7px sized font, the size will be mapped to 8px in the output;
        * Color: either font color or fill colors, are limited to the entries in color dictionry
        * Style combinations: when style combination is not supported, say in different size, face, bold and italic, the closest entry will be selected in the output;
* Text positioning and spacing:
    * Since embedd font and font styles are only honored if they defined in style dictionary, when they are not in there, the final output may have word positioning and spacing issues that's noticable.
* User input data in form element:
    * As for interactive forms elements, their type, poisitions, sizes, limited styles and control data are all parsed and served in output, but user interactive data are not parsed, like which radio button is selected, which checkbox is checked, text in text input box, etc., should be handled in client as part of user data, so that we can treat parsed PDF data as template data.


Run Unit Test
=====

Test suite for PDF2JSON is created with Vows.js, it'll parse 3 PDF files under 'test/data' directory in parallel and have 12 test cases need to be honored.

            node test/index.js






