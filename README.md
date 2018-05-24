# @CentralPing/express-pagination

[![Build Status](https://travis-ci.org/CentralPing/express-pagination.svg?branch=master)](https://travis-ci.org/CentralPing/express-pagination)
[![Coverage Status](https://coveralls.io/repos/github/CentralPing/express-pagination/badge.svg)](https://coveralls.io/github/CentralPing/express-pagination)
[![Dependency Status](https://david-dm.org/CentralPing/express-pagination.svg)](https://david-dm.org/CentralPing/express-pagination)
[![Greenkeeper Status](https://badges.greenkeeper.io/CentralPing/express-pagination.svg)](https://greenkeeper.io/)
[![Known Vulnerabilities](https://snyk.io/test/github/centralping/express-pagination/badge.svg)](https://snyk.io/test/github/centralping/express-pagination)

A middleware wrapper for [expressjs](http://expressjs.com) for validating pagination parameters and generating pagination links by [`@CentralPing/list-pagination`](https://github.com/CentralPing/express-pagination).

## Installation

`npm i --save https://github.com/CentralPing/express-pagination`

## API Reference

<a name="module_express-pagination..tokenize"></a>

### express-pagination~tokenize([options]) ⇒ <code>function</code>
Configures Express middleware for composing optional pagination tokens.

**Kind**: inner method of [<code>express-pagination</code>](#module_express-pagination)  
**Returns**: <code>function</code> - Configured Express middleware  

| Param | Type | Default |
| --- | --- | --- |
| [options] | <code>Object</code> |  | 
| [options.readFrom] | <code>String</code> | <code>query</code> | 
| [options.writeTo] | <code>String</code> | <code>pagination</code> | 
| [options.uuidKey] | <code>String</code> |  | 
| [options.secret] | <code>String</code> |  | 

<a name="module_express-pagination..validate"></a>

### express-pagination~validate(options) ⇒ <code>function</code>
Configures Express middleware for validating pagination query parameters.

**Kind**: inner method of [<code>express-pagination</code>](#module_express-pagination)  
**Returns**: <code>function</code> - Configured Express middleware  

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| options | <code>Object</code> |  |  |
| [options.countValid] | <code>\*</code> | <code>&#x27;1&#x27;</code> | Indicates what a truthy value is (outside  of `true`). Can be a single value or an array of values. |
| [options.max] | <code>Number</code> | <code>25</code> | The maximum and default list items  allowed. |
| [options.sort] | <code>String</code> \| <code>Array.&lt;String&gt;</code> | <code>id</code> | The default field(s) to sort  the list by. |
| [options.sortValid] | <code>String</code> \| <code>Array.&lt;String&gt;</code> | <code>[id]</code> | The allowed fields to  sort the list by. The schema will automatically match descending annotation  (i.e. `-id`). |


## Examples

### For Default Validation

```js
const express = require('express');
const { tokenize, validate } = require('express-pagination');

const app = express();

app.get('/foos',
  validate,
  (req, res, next) => {
    res.locals.list = getList(res.locals.query);
    next();
  },
  tokenize,
  (req, res) => {
    res.json({
      pages: res.locals.pagination,
      list: res.locals.list
    })
  }
);

/*
GET request -> /foos
// After validate
res.locals would be set to:
  {
    query: {
      // Defaults
      limit: 25,
      sort: ['id'],
      filter: {}
    }
  }
// After tokenize (assuming a list of 25 items)
res.locals would include:
  {
    pagination: {
      self: TOKEN,
      first: TOKEN,
      next: TOKEN // Omitted if less than 25 items
    }
  }

GET request -> /foos?limit=5&sort=-foo&filter.foo='bar'
// After validate
res.locals would be set to:
  {
    query: {
      limit: 5,
      sort: ['-foo', 'id'],
      filter: {foo: 'bar'}
    }
  }
// After tokenize (assuming a list of 5 items)
res.locals would include:
  {
    pagination: {
      self: TOKEN,
      first: TOKEN,
      next: TOKEN
    }
  }

GET request -> /foos?page=NEXT_TOKEN
// After validate (assuming orignal limit, sort, and filter from previous example)
res.locals would be set to:
  {
    query: {
      limit: 5,
      sort: ['-foo', 'id'],
      filter: {foo: 'bar'},
      // Subsequent next tokens will add more cursors, e.g. [6, 3],
      //  where the current cursor will always be at the 0 index position
      cursors: [3],
      type: 'next'
    }
  }
// After tokenize (assuming another list of 5 items)
res.locals would include:
  {
    pagination: {
      self: TOKEN,
      first: TOKEN,
      next: TOKEN,
      prev: TOKEN
    }
  }
*/
```

## License

MIT
