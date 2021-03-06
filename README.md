pyschema
========

This project provides a way to validate recursive data structures created using
dictionaries, tuples, lists etc.  Proper way of adding validations should be
to add python classes in the mix, but lot of times we end up with simple data
structures as they are easy to start.  This project provides a way to create
schema definition for such objects. 

Basic Usage
------------
```
>>> from pyschema import Schema
>>> s = Schema({
...     'type': 'number',
...     'minimum_value':0,
...     'maximum_value':120,
...     'display_name':'Age'
... })
>>> s.validate(10)
[]
>>> s.validate(200)
['Value 200 is greater than 120 at root(Age)']

```


Basic Schema
------------

Schema definition is done recursively using attributes given here and allow any depth to of the metadata to be specified.  

Each level is described by the following properties in a python dictionary:

| Name | Description | Required | Lambda Support | Default |
| ---- | ---- | ---- | ---- | ---- |
| display_name | Name to be used in the display | True | False | n/a |
| description | Helpful information about this element | False | False | Blank string |
| type | Data type of the value. Should be one of map, list, string, number, boolean or any(4) | False | True | n/a |
| value_schema | Schema to be used for any of the child values by default. | Depends(1) | True | n/a |
| verbatim | Any information to be saved along with schema. (3) | False | False | None |
| allow_none | Allow None value.  No validations are made if the value is None | False | False | False |
| custom_validation | Python function that peforms custom validation (in addition to other). In the realized schema, it will just say True. | False | False | None |
| **Map Specific** |  |  |  |  |
| known_children | A map of named children that are allowed for a map.  Each named children can have their own schema the value. If it is blank dictionary, or None, value_schema is used. | False | True | Blank Map | 
| allow_unknown_children | Should the UI allow adding children whose names are not known.  If this is set to true, value schema attribute above must be defined. | False | False | False |
| mandatory_children | Names for which values must be provided.  Useful for a map only.	| False | True | No Restrictions | 
| allow_list | Allows list of maps in addition to map.  If List is given each item is validated.	| False | False | False |
| **Number Specific** |  |  |  |  |
| minimum_value | Used for int type leaf nodes only. | False | True | No Limit |
| maximum_value	| Used for int type leaf nodes only.| False | True | No Limit | 
| **String Specific** |  |  |  |  |
| allowed_pattern | Used for string type values only.  Regular expression to validate given input strings. | False | False | No restrictions |
| allowed_values | Used for string type values only.  List of values allowed to be set. | False | True | No restrictions |
| **Boolean Specific** |  |  |  |  |
| true_value | Used for boolean type leaf nodes only.  String to be displayed if the underlying value is True. For example: Yes, Enabled, True etc. | False | False | True |
| false_value | Used for boolean type leaf nodes only. String to be displayed if the underlying value is False. For example: No, Disabled, False etc. | False | False | False |
| **List Specific** |  |  |  |  |
| minimum_size | Minimum number of entries in list | False | False | 0 |
| maximum_size | Maximum number of entries in list | False | False | Unlimited |
| unique | Require all list entries be unique (5) | False | False | No restriction |


1. For each complex type of object, schema is required either directly or through inheritance.  Schema defined here is used for all children where an explicit schema is not defined.
2. Data is only validated and never modified.  Validation errors are listed as a simple list of strings.
3. Verbatim data is for other consumers for example UI to drive the widgets or to give hints.  It has no meaning for the backend.
4. Any type has no validations done except for any custom validations.  Use with care.  You are creating the schema to avoid any in the first place :-)
5. If a list contains complex values unique check can become complicated.  Current implementation simple adds values to a set for comparison.

Lambda Support
--------------

Lambda support allows the schema to be generated by python code instead of defining it as a static
python data. Advantage of such approach is to give out list of names or children that must be valid.
While we can make everything dynamic, we choose to support a sensible subset.

A process called schema realization must be executed to run the python code and realize
the true schema. First call to validate or an explicit call performs the realization of schema.
However note that, validate realizes only the part of the schema that is actually used, where as realize
call truly realizes the entire schema dynamically.

See the samples/basics.py code for an example of lambda support. Running this code produces the following output:

```
Realized schema
---------------
{'display_name': 'Root Configuration',
 'known_children': {'listed_names': {'display_name': 'Name List',
                                     'type': 'list',
                                     'value_schema': {'allowed_values': ['a',
                                                                         'b',
                                                                         'c'],
                                                      'display_name': 'Names',
                                                      'type': 'string'}},
                    'notify_via_mail': {'display_name': 'Mail Preference',
                                        'false_value': 'False',
                                        'true_value': 'True',
                                        'type': 'boolean'},
                    'number_of_names': {'display_name': 'Name Count Limit',
                                        'minimum_value': 2,
                                        'type': 'number'}},
 'mandatory_children': ['listed_names'],
 'type': 'map'}



Sample Validation Errors: 1
---------------
["Expecting value to be (<type 'list'>, <type 'set'>, <type 'tuple'>) but got <type 'int'> for root.listed_names(Name List)"]



Sample Validation Errors: 2
---------------
["Expecting value to be (<type 'str'>, <type 'unicode'>) but got <type 'int'> for root.listed_names.n(Names)",
 'invalid-name is not a allowed value for root.listed_names.n(Names). Expect it to be one of: a,b,c']



Sample Validation Errors: 3 (No errors)
---------------
[]
```
