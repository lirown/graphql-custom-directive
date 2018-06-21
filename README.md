# graphql-custom-directive
[![Build Status](https://travis-ci.org/lirown/graphql-custom-directive.svg?branch=master)](https://travis-ci.org/lirown/graphql-custom-directive)
[![Coverage Status](https://coveralls.io/repos/github/lirown/graphql-custom-directive/badge.svg?branch=master)](https://coveralls.io/github/lirown/graphql-custom-directive?branch=master)
[![npm version](https://badge.fury.io/js/graphql-custom-directive.svg)](https://badge.fury.io/js/graphql-custom-directive)
[![Dependency Status](https://david-dm.org/lirown/graphql-custom-directive.svg)](https://david-dm.org/lirown/graphql-custom-directive)
[![Known Vulnerabilities](https://snyk.io/test/github/lirown/graphql-custom-directive/badge.svg)](https://snyk.io/test/github/lirown/graphql-custom-directive)
[![License](http://img.shields.io/:license-mit-blue.svg)](http://doge.mit-license.org)

A custom directive for GraphQL with the ability to hook the query execution.

### Install
```
npm install --save graphql-custom-directive
```

### Usage
```javascript
import { 
  GraphQLString, 
  GraphQLSchema, 
  GraphQLObjectType, 
  graphql 
} from 'graphql';

import { 
  DirectiveLocation 
} from 'graphql/type/directives';

import { 
  GraphQLCustomDirective, 
  applySchemaCustomDirectives 
} from 'graphql-custom-directive';

// Define a directive that upper case the result

const GraphQLCustomDuplicateDirective = new GraphQLCustomDirective({
  name: 'toUpperCase',
  description:
    'change the case of a string to uppercase',
  locations: [
    DirectiveLocation.FIELD
  ],
  resolve(resolve) {
    return resolve()
        .then(result => result.toUpperCase());
  }
});

const schema = new GraphQLSchema({
  directives: [
    GraphQLCustomDuplicateDirective
  ],
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      value: {
        type: GraphQLString,
        resolve: () => 'test'
      }
    }
  })
});

applySchemaCustomDirectives(schema);

graphql(schema, `{ value @toUpperCase }`)
  .then(({ result, errors }) => {
     console.log(result); 
     // will print { value: "TEST" }
  });
```

### Options
```javascript
GraphQLCustomDirective({
  // name to be used in placing the directive (e.g @duplicate) 
  // [*required]
  name: String = 'duplicate',
  
  // explain the directive usage
  description: String = 'duplicate the string sperating them with space',
  
  // areas in the query you can place the directive 
  // [*required]
  locations: [String] = [ DirectiveLocation.FIELD ],
  
  // object of passed variables from directive to the resolve method
  args: Object = { by: { type: GraphQLInt, description: "foo bar" } } ),
  
  // method that hooks the execution and transforms the input to a new output
  // arguments:
  // 1. resolve - a field promise that will result in the field's value 
  //  (either the raw field or the previous directive output).
  // 2. source - a parent object of execution field result.
  // 3. args - a object of directive arguments defined in query exectution.
  // 4. context - a value to pass as the context to the graphql() function.
  // 5. info - a collection of information about the current execution state.
  resolve: Function = (resolve, source, args, context, info) => 
            { return resolve.then(input => input); }
})
```

### Examples 

This show the ability to configure directive on the query side or in the schema side

```javascript
import { 
  GraphQLString,
  GraphQLNotNull,
  GraphQLSchema, 
  GraphQLObjectType, 
  graphql 
} from 'graphql';

import { 
  DirectiveLocation 
} from 'graphql/type/directives';

import { 
  GraphQLCustomDirective, 
  applySchemaCustomDirectives 
} from 'graphql-custom-directive';

// Define a directive that duplicates the input

const GraphQLCustomDuplicateDirective = new GraphQLCustomDirective({
  name: 'duplicate',
  description:
    'duplicate the string sperating them with space',
  locations: [
    DirectiveLocation.FIELD
  ],
  args: {
    by: {
      type: new GraphQLNotNull(GraphQLInt),
      description: 'the times to duplicate the string'
    }
  },
  resolve(resolve, source, { by }, context, info) {
    return resolve().then(result => {    
      let times = [];
      
      for (let i = 0; i < by; i++) {
        times.push(result);
      }
      
      return times.join(' ');
    });
  }
});

// Use directive in a query

const schema = new GraphQLSchema({
  directives: [
    GraphQLCustomDuplicateDirective
  ],
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      input: {
        type: GraphQLString,
        args: {
          value: {
            type: GraphQLString
          }
        },
        resolve: (source, {value}) => value
      }
    }
 })
});

applySchemaCustomDirectives(schema);

graphql(schema, `{ input(value: "test") @duplicate(by:2) }`)
  .then(({ result, errors }) => {
     console.log(result); 
     // will print { input: "test test" }
  });
  
// Or use the directive in a schema

const schema2 = new GraphQLSchema({
  directives: [
    GraphQLCustomDuplicateDirective
  ],
  query:  new GraphQLObjectType({
    name: 'Query',
    fields: {
      input: {
        type: GraphQLString,
        args: {
          value: {
            type: GraphQLString
          }
        },
        directives: {
          duplicate: { // directive name
            by: 2 // directive args
          }
        },
        resolve: (source, {value}) => value
      }
    }
 })
});

applySchemaCustomDirectives(schema2);

graphql(schema2, `{ input(value: "test") }`)
  .then(({ result, errors }) => {
     console.log(result); 
     // will print { input: "test test" }
  });

```
### License
```
The MIT License (MIT)

Copyright (c) 2016 Lirown

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

