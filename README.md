NestHydrationJS
===============
[![Build Status](https://travis-ci.org/CoursePark/NestHydrationJS.svg?branch=master)](https://travis-ci.org/CoursePark/NestHydrationJS)
[![Coverage Status](https://coveralls.io/repos/CoursePark/NestHydration/badge.svg?branch=master&service=github)](https://coveralls.io/github/CoursePark/NestHydration?branch=master)

Converts tabular data into a nested object/array structure based on a definition object or specially named columns.

Tabular Data With Definition
----------------------------

```javascript
var NestHydrationJS = require('nesthydrationjs');
var table = [
	{
		id: '1', title: 'Tabular to Objects', required: '1',
		teacher_id: '1', teacher_name: 'David',
		lesson_id: '1', lesson_title: 'Definitions'
	},
	{
		id: '1', title: 'Tabular to Objects', required: '1',
		teacher_id: '1', teacher_name: 'David',
		lesson_id: '2', lesson_title: 'Table Data'
	},
	{
		id: '1', title: 'Tabular to Objects', required: '1',
		teacher_id: '1', teacher_name: 'David',
		lesson_id: '3', lesson_title: 'Objects'
	},
	{
		id: '2', title: 'Column Names Define Structure', required: '0',
		teacher_id: '2', teacher_name: 'Chris',
		lesson_id: '4', lesson_title: 'Column Names'
	},
	{
		id: '2', title: 'Column Names Define Structure', required: '0',
		teacher_id: '2', teacher_name: 'Chris',
		lesson_id: '2', lesson_title: 'Table Data'
	},
	{
		id: '2', title: 'Column Names Define Structure', required: '0',
		teacher_id: '2', teacher_name: 'Chris',
		lesson_id: '3', lesson_title: 'Objects'
	},
	{
		id: '3', title: 'Object On Bottom', required: '0',
		teacher_id: '1', teacher_name: 'David',
		lesson_id: '5', lesson_title: 'Non Array Input'
	}
];
var definition = [{
	id: {column: 'id', type: 'NUMBER'},
	title: 'title',
	required: {column: 'required', type: 'BOOLEAN'},
	teacher: {
		id: {column: 'teacher_id', type: 'NUMBER'},
		name: 'teacher_name'
	},
	lesson: [{
		id: {column: 'lesson_id', type: 'NUMBER'},
		title: 'lesson_title'
	}]
}];
result = NestHydrationJS.nest(table, definition);
/* result would be the following:
[
	{id: 1, title: 'Tabular to Objects', required: true, teacher: {id: 1, name: 'David'}, lesson: [
		{id: 1, title: 'Defintions'},
		{id: 2, title: 'Table Data'},
		{id: 3, title: 'Objects'}
	]},
	{id: 2, title: 'Column Names Define Structure', required: false, teacher: {id: 2, name: 'Chris'}, lesson: [
		{id: 4, title: 'Column Names'},
		{id: 2, title: 'Table Data'},
		{id: 3, title: 'Objects'}
	]},
	{id: 3, title: 'Object On Bottom', required: false, teacher: {id: 1, name: 'David'}, lesson: [
		{id: 5, title: 'Non Array Input'},
	]}
]
*/
```

SQL-ish Example
---------------

While not limited to SQL, it is common to want to define an SQL query and then just get back objects. Here's how.

In the following example you can see the same result but the definition of the object structure is contained in the column names of the tabular input. Nesting is achieved by using a underscore (_). A *x to one* relation is defined by a single underscore and a *x to many* relation is defined by preceeding properties of the many object with a 2nd underscore.

If a column alias ends in a triple underscore followed by either NUMBER or BOOLEAN then the values in those columns in the result data will be caste to the respective type unless the value is null.

**Note:** that this means that almost always bottom level properties will be prefixed with a underscore, as this is actually a *x to many* relation from the variable returned from the next function. If 

```javascript
var sql = ''
	+ 'SELECT'
	+ 'c.id        AS _id___NUMBER,'
	+ 'c.title     AS _title,'
	+ 'c.requried  AS _required___BOOLEAN,'
	+ 't.teacher   AS _teacher_id___NUMBER,'
	+ 't.name      AS _teacher_name,'
	+ 'l.id        AS _lesson__id___NUMBER,'
	+ 'l.title     AS _lesson__title'
	+ 'FROM course AS c'
	+ 'JOIN teacher AS t ON t.id = c.teacher_id'
	+ 'JOIN course_lesson AS cl ON cl.course_id = c.id'
	+ 'JOIN lesson AS l ON l.id = cl.lesson_id'
;
var table = db.fetchAll(sql);
/* table could result in the following:
[
	{
		_id: '1', _title: 'Tabular to Objects', _required: '1',
		_teacher_id: '1', _teacher_name: 'David',
		_lesson__id: '1', _lesson__title: 'Defintions'
	},
	{
		_id: '1', _title: 'Tabular to Objects', _required: '1',
		_teacher_id: '1', _teacher_name: 'David',
		_lesson__id: '2', _lesson__title: 'Table Data'
	},
	{
		_id: '1', _title: 'Tabular to Objects', _required: '1',
		_teacher_id: '1', _teacher_name: 'David',
		_lesson__id: '3', _lesson__title: 'Objects'
	},
	{
		_id: '2', _title: 'Column Names Define Structure', _required: '0',
		_teacher_id: '2', _teacher_name: 'Chris',
		_lesson__id: '4', _lesson__title: 'Column Names'
	},
	{
		_id: '2', _title: 'Column Names Define Structure', _required: '0',
		_teacher_id: '2', _teacher_name: 'Chris',
		_lesson__id: '2', _lesson__title: 'Table Data'
	},
	{
		_id: '2', _title: 'Column Names Define Structure', _required: '0',
		_teacher_id: '2', _teacher_name: 'Chris',
		_lesson__id: '3', _lesson__title: 'Objects'
	},
	{
		_id: '3', _title: 'Object On Bottom', _required: '0',
		_teacher_id: '1', _teacher_name: 'David',
		_lesson__id: '5', _lesson__title: 'Non Array Input'
	}
]
*/
result = NestHydrationJS.nest(table);
/* result would be the following:
[
	{id: 1, title: 'Tabular to Objects', required: true, teacher: {id: 1, name: 'David'}, lesson: [
		{id: 1, title: 'Definitions'},
		{id: 2, title: 'Table Data'},
		{id: 3, title: 'Objects'}
	]},
	{id: 2, title: 'Column Names Define Structure', required: false, teacher: {id: 2, name: 'Chris'}, lesson: [
		{id: 4, title: 'Column Names'},
		{id: 2, title: 'Table Data'},
		{id: 3, title: 'Objects'}
	]},
	{id: 3, title: 'Object On Bottom', required: false, teacher: {id: 1, name: 'David'}, lesson: [
		{id: 5, title: 'Non Array Input'},
	]}
]
*/
```

Custom Type Definition
----------------------

### As a custom type

New types can be registered using the `registerType(name, handler)` function. `handler(cellValue, name, row)` is a callback
function that takes the cell value, column name and the full row data.

#### Example Usage

```javascript
var NestHydrationJS = require('nesthydrationjs');
NestHydrationJS.registerType('CUSTOM_TYPE', function(value, name, row) {
	return '::' + value + '::';
});

var table = [
	{
		id: 1, title: 'Custom Data Types'
	}
];
var definition = [{
	id: 'id'
	title: {column: 'title', type: 'CUSTOM_TYPE'},
}];
result = NestHydrationJS.nest(table, definition);
/* result would be the following:
[
	{id: 1, title: '::Custom Data Types::'}
]
*/
```

### Type as a function

You can also define the type of a column in the definition object as a function and that function will be called for each
value provided. The arguments passed are the same as those passed to a custom type handler. This allows formatting of a 
type without defining it as a global type.

#### Example

```javascript
var NestHydrationJS = require('nesthydrationjs');

var table = [
	{
		id: 1, title: 'Custom Data Types'
	}
];
var definition = [{
	id: 'id'
	title: {column: 'title', type: function(value, name, row) {
		return '::' + value + '::';
	}},
}];
result = NestHydrationJS.nest(table, definition);
/* result would be the following:
[
	{id: 1, title: '::Custom Data Types::'}
]
*/
```

Related Projects
----------------

- [NestHydration for PHP](https://github.com/CoursePark/NestHydration) : The original. But a new algorithm was implemented for the JS (this) version and ported back to PHP.
- [KnexNest](https://github.com/CoursePark/KnexNest) : Takes a [Knex.js](http://knexjs.org/) query object and returns objects.
