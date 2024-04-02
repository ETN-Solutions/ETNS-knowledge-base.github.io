---
layout: home
parent: JsDocs
title: Installation
---

# Installation

- To install JSDoc globally, run the following command ---- <br>
 `npm install -g jsdoc`

- Configuring JSDoc
In the`script` property of package.json, we will need to add the jsdoc command to run JSDoc and generate documentation, Add the command similar to given below in the package.json file <br>

```Json
       { 
        "scripts":  {
    "jsdoc": "jsdoc -c jsdoc.json"
  } 
}

 This command has a -c tag which denotes that jsdoc will run with a custom config file, Hence let’s create a config file for JSDoc.
```
- In the root of your project directory create a file named `“jsdoc.json”` , add the following code in that file: <br>
   
```Json 
{  
  "plugins": ["plugins/markdown"], 
  "recurseDepth": 10, 
  "source": { 
    "include": ["src"], 
    "includePattern": ".js$", 
    "excludePattern": "(node_modules/|docs)"
  }, 
  "templates": { 
    "cleverLinks": true, 
    "monospaceLinks": true
  }, 
  "opts": { 
    "destination": "./jsdoc", 
    "recurse": true, 
    "readme": "./readme.md"
  } 
}