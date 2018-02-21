---
layout: post
title:  "[TIP] Debugging in Ruby and Python"
date:   2018-02-19 20:30:00 -0600
tags:
- python
- ruby
- ruby and python comparison
---
I've been writing code in Python for approximately three years and have become pretty
proficient at it. Within the last few months, I found myself needing to learn Ruby to
assist in developing some features in an application that were necessary for some of
the strategic goals that my team and I have been working towards.

I often find myself debugging applications attempting to find why things aren't working
like I expect them to. In Python, I utilize `type()` and `dir()` frequently attempting
to find out why data types an object is as well as what methods were available. As I
get deeper into Ruby, I find myself needing utilize its equivalent syntax.

Take the example Python code:

{% highlight python %}
class DataStructure():

    def __init__(self):
        pass

    def string_method(self):
        return 'string'

    def list_method(self):
        return ['list item']

    def dict_method(self):
        return { 'key': 'value' }

{% endhighlight %}

If you were attempting to debug the above code, I usually find `type()` and `dir()` two of the most
useful tools while debugging code.

Here is an example output of exploring the `DataStructure` class using the Python interpreter.

```python
>>> data = DataStructure()
>>> dir(data)
['__doc__', '__init__', '__module__', 'dict_method', 'list_method', 'string_method']
>>> type(data.dict_method())
<type 'dict'>
>>> dir(data.dict_method())
['__class__', '__cmp__', '__contains__', '__delattr__', '__delitem__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__iter__', '__le__', '__len__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'clear', 'copy', 'fromkeys', 'get', 'has_key', 'items', 'iteritems', 'iterkeys', 'itervalues', 'keys', 'pop', 'popitem', 'setdefault', 'update', 'values', 'viewitems', 'viewkeys', 'viewvalues']
>>> data.dict_method()
{'key': 'value'}
```

As you can see, I was able to quickly find the available methods within the `DataStructure` class using the `dir()` syntax. Then I was able to determine what the data type of the method is utilizing the `type()` syntax.

Here is the equivalent in Ruby code:

{% highlight ruby %}

class DataStructure

  def initialize; end

  def string_method
    'string'
  end

  def array_method
    ['list item']
  end

  def hash_method
    { 'key': 'value' }
  end
end

{% endhighlight %}

```ruby
irb(main):017:0> data = DataStructure.new
=> #<DataStructure:0x00007ff397267180>
irb(main):018:0> data.methods
=> [:string_method, :array_method, :hash_method, :remove_instance_variable, :instance_of?, :kind_of?, :is_a?, :tap, :public_send, :singleton_method, :instance_variable_defined?, :define_singleton_method, :method, :public_method, :instance_variable_set, :extend, :to_enum, :enum_for, :<=>, :===, :=~, :!~, :eql?, :respond_to?, :freeze, :inspect, :object_id, :send, :display, :to_s, :nil?, :hash, :class, :singleton_class, :clone, :dup, :itself, :taint, :tainted?, :untaint, :untrust, :untrusted?, :trust, :frozen?, :methods, :singleton_methods, :protected_methods, :private_methods, :public_methods, :instance_variable_get, :instance_variables, :!, :==, :!=, :__send__, :equal?, :instance_eval, :instance_exec, :__id__]
irb(main):019:0> data.hash_method.class
=> Hash
irb(main):020:0> data.hash_method.methods
=> [:index, :replace, :clear, :select!, :keep_if, :==, :delete_if, :to_h, :reject!, :<=, :[], :[]=, :include?, :assoc, :rassoc, :compact!, :compact, :empty?, :eql?, :flatten, :values_at, :default, :rehash, :store, :default=, :default_proc, :default_proc=, :key, :each_value, :each_key, :each_pair, :inspect, :transform_values, :transform_values!, :keys, :values, :fetch_values, :invert, :update, :merge!, :merge, :has_key?, :length, :size, :has_value?, :key?, :each, :compare_by_identity, :compare_by_identity?, :delete, :>=, :>, :<, :value?, :to_hash, :to_proc, :to_a, :to_s, :select, :reject, :dig, :any?, :member?, :hash, :fetch, :shift, :max, :min, :find, :entries, :sort, :sort_by, :grep, :grep_v, :count, :detect, :find_index, :find_all, :collect, :map, :flat_map, :collect_concat, :inject, :reduce, :partition, :group_by, :first, :all?, :one?, :none?, :minmax, :min_by, :max_by, :minmax_by, :each_with_index, :reverse_each, :each_entry, :each_slice, :each_cons, :each_with_object, :zip, :take, :take_while, :drop, :drop_while, :cycle, :chunk, :slice_before, :slice_after, :slice_when, :chunk_while, :sum, :uniq, :lazy, :remove_instance_variable, :instance_of?, :kind_of?, :is_a?, :tap, :public_send, :singleton_method, :instance_variable_defined?, :define_singleton_method, :method, :public_method, :instance_variable_set, :extend, :to_enum, :enum_for, :<=>, :===, :=~, :!~, :respond_to?, :freeze, :object_id, :send, :display, :nil?, :class, :singleton_class, :clone, :dup, :itself, :taint, :tainted?, :untaint, :untrust, :untrusted?, :trust, :frozen?, :methods, :singleton_methods, :protected_methods, :private_methods, :public_methods, :instance_variable_get, :instance_variables, :!, :!=, :__send__, :equal?, :instance_eval, :instance_exec, :__id__]
irb(main):021:0> data.hash_method
=> {:key=>"value"}
irb(main):022:0>
```

In Ruby, I was able to use the `.methods` syntax to determine the available methods. Then use the `.class` syntax to determine the data type of the method.

As I dig deeper into Ruby, I plan on continuing these comparison blogs. If you have any comments or blog suggestions, open a [github issue][netdevops-gh]. 

[jtdub][jtdub-gh]

[netdevops-gh]: https://github.com/netdevops/netdevops.github.io/issues
[jtdub-gh]: https://github.com/jtdub
