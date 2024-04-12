
For details on each data type, see the linked documentation or the [specification document](https://github.com/puppetlabs/puppet-specifications/blob/master/language/types_values_variables.md).

- 字符型：引号可有可无；但单引号为强引用，双引号为弱引用；支持转义符；
-  数值型：默认均识别为字符串，仅在数值上下文才以数值对待；
-  数组：[]中以逗号分隔元素列表；
-  布尔型值：true, false；不能加引号；
-  hash：{}中以逗号分隔k/v数据列表； 键为字符型，值为任意puppet支持的类型；{ ‘mon’ => ‘Monday’, ‘tue’ => ‘Tuesday’, }；
-  undef：从未被声明的变量的值类型；


|   |   |   |
|---|---|---|
|Data type|Purpose|Type category|
|[Any](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|The parent type of all types.|Abstract|
|[Array](https://puppet.com/docs/puppet/7/lang_data_array.html#lang_data_array)|The data type of arrays.|Data|
|Binary|A type representing a sequence of bytes.|Data|
|[Boolean](https://puppet.com/docs/puppet/7/lang_data_boolean.html)|The data type of Boolean values.|Data, Scalar|
|[Callable](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_other)|Something that can be called (such as a function or lambda).|Platform|
|[CatalogEntry](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|The parent type of all types that are included in a Puppet catalog.|Abstract|
|[Class](https://puppet.com/docs/puppet/7/lang_data_resource_reference.html)|A special data type used to declare classes.|Catalog|
|[Collection](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|A parent type of Array and Hash.|Abstract|
|[Data](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|A parent type of all data directly representable as JSON.|Abstract|
|[Default](https://puppet.com/docs/puppet/7/lang_data_default.html)|The "default value" type.|Platform|
|Deferred|A type describing a call to be resolved in the future.|Platform|
|[Enum](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|An enumeration of strings.|Abstract|
|Error|A type used to communicate when a function has produced an error.||
|[Float](https://puppet.com/docs/puppet/7/lang_data_number.html#lang_data_number)|The data type of floating point numbers.|Data, Scalar|
|[Hash](https://puppet.com/docs/puppet/7/lang_data_hash.html#lang_data_hash)|The data type of hashes.|Data|
|Init|A type used to accept values that are compatible of some other type's "new".||
|[Integer](https://puppet.com/docs/puppet/7/lang_data_number.html#lang_data_number)|The data type of integers.|Data, Scalar|
|Iterable|A type that represents all types that allow iteration.|Abstract|
|Iterator|A special kind of lazy Iterable suitable for chaining.|Abstract|
|[NotUndef](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|A type that represents all types not assignable from the Undef type.|Abstract|
|[Numeric](https://puppet.com/docs/puppet/7/lang_data_number.html#lang_data_number)|The parent type of all numeric data types.|Abstract|
|Object|Experimental. Can be a simple object only having attributes, or a more complex object also supporting callable methods.||
|[Optional](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|Either Undef or a specific type.|Abstract|
|[Pattern](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|An enumeration of regular expression patterns.|Abstract|
|[Regexp](https://puppet.com/docs/puppet/7/lang_data_regexp.html#lang_data_regexp)|The data type of regular expressions.|Scalar|
|[Resource](https://puppet.com/docs/puppet/7/lang_data_resource_reference.html)|A special data type used to declare resources.|Catalog|
|[RichData](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|A parent type of all data types except the non serializeable types Callable, Iterator, Iterable, and Runtime.|Abstract|
|Runtime|The type of runtime (non Puppet) types.|Platform|
|[Scalar](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|Represents the abstract notion of "value".|Abstract|
|[ScalarData](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_parent)|A parent type of all single valued data types that are directly representable in JSON.|Abstract|
|[SemVer](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible-semver-data-type)|A type representing semantic versions.|Scalar|
|[SemVerRange](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible-semverange)|A range of SemVer versions.|Abstract|
|[Sensitive](https://puppet.com/docs/puppet/7/lang_data_sensitive.html)|A type that represents a data type that has "clear text" restrictions.|Platform|
|[String](https://puppet.com/docs/puppet/7/lang_data_string.html#lang_data_string)|The data type of strings.|Data, Scalar|
|[Struct](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|A Hash where each entry is individually named and typed.|Abstract|
|[Timespan](https://puppet.com/docs/puppet/7/lang_data_time.html#lang_data_time_timespan)|A type representing a duration of time.|Scalar|
|[Timestamp](https://puppet.com/docs/puppet/7/lang_data_time.html#lang_data_time_timestamp)|A type representing a specific point in time|Scalar|
|[Tuple](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|An Array where each slot is typed individually|Abstract|
|[Type](https://puppet.com/docs/puppet/7/lang_data_type.html#lang_data_type_type)|The type of types.|Platform|
|Typeset|Experimental. Represents a collection of Object-based data types.||
|[Undef](https://puppet.com/docs/puppet/7/lang_data_undef.html)|The "no value" type.|Data, Platform|
|URI|A type representing a Uniform Resource Identifier|Data|
|[Variant](https://puppet.com/docs/puppet/7/lang_data_abstract.html#lang_data_abstract_flexible)|One of a selection of types.|Abstract|

[https://github.com/puppetlabs/puppet-specifications/blob/master/language/types_values_variables.md](https://github.com/puppetlabs/puppet-specifications/blob/master/language/types_values_variables.md)

|   |   |
|---|---|
|Puppet Types|Puppet Types include the types that are meaningful in a Puppet Program - these are divided into the conceptual categories Data Types e.g.:<br><br>- Integer<br>- Float<br>- Boolean<br>- String<br>- Array<br>- Hash<br>- Undef<br>- URI<br>- Binary|
|Scalar Types|- Integer<br>- Float<br>- Boolean<br>- String<br>- Regexp<br>- SemVer<br>- Timespan<br>- Timestamp|
|Catalog Types|- Resource<br>- Class|
|Abstract Types|(The term abstract denotes that instances of such a type are always an instance of some other concrete type).<br><br>- Any; the parent type of all types<br>- CatalogEntry; the parent type of all types that are included in a Puppet Catalog<br>- Collection; a parent type of Array and Hash<br>- Data; a parent type of all data directly representable as JSON (alias for Variant[Undef, ScalarData, Array[Data], Hash[String, Data]])<br>- Enum; an enumeration of strings<br>- Iterator; a special kind of lazy Iterable suitable for chaining<br>- NotUndef; a type that represents all types not assignable from the Undef type<br>- Numeric; the parent type of all numeric data types (Integer, Float)<br>- Optional; either Undef or a specific type<br>- Pattern; an enumeration of regular expression patterns<br>- RichData'; a parent type of all data types except the non serializeable types Callable, Iterator, Iterable, and Runtime<br>- Scalar; the same as Variant[ScalarData, Regexp, SemVer, Timespan, Timestamp]<br>- ScalarData; a parent type of all single valued data types that are directly representable in JSON (alias for Variant[Integer, Float, String, Boolean])<br>- SemVerRange; a range of SemVer versions<br>- Struct; a Hash where each entry is individually named and typed<br>- Tuple; an Array where each slot is typed individually<br>- Variant; one of a selection of types<br>- Iterable; a type that represents all types that allow iteration|
|Platform Types|All types are organized into one Type System.<br><br>- Callable; something that can be called (function, lambda)<br>- Default; the "default value" type<br>- Runtime; the type of runtime (non Puppet) types<br>- Sensitive; a type that represents a data type that have "clear text" restrictions<br>- Type; the type of types<br>- Undef; the "no value" type<br>- Deferred; a type describing a call to be resolved in the future|
|||

Undef 从未被声明的变量的值类型
$abc=’hello world’  //声明变量赋值
$abc=undef    //撤销变量


hash
即为外键值数据类型，键和值之间使用”=>”分隔，键值对儿定义在”{}”中，彼此间以逗号分隔；
其键为字符型数据，而值可以为puppet支持的任意数据类型；
访问hash类型的数据元素要使用”键”当作索引进行；