---
layout: post
title: "Simple Data Mapper"
date: 2012-03-02 15:35
comments: true
categories: 
  - rails
  - ruby
  - Karle Durante
author: Karle Durante
published: true
---

I recently tackled a pretty typical data migration task where some source model had to be transformed into some target model.  About 80% of the elements were mapped field for field, and the other 20% had to be mutated in some way.  Tired of writing one off rake tasks to pull and transform data, I came up with a little data mapper class that I cold reuse in the future. 

```ruby
class Mapper
  attr_accessor :source_model
  attr_accessor :mappings
  
  class AttributeMapping;end
  
  class ListMapping < AttributeMapping
    attr_accessor :fields
    
    # fields - array of field symbols we want to map data into
    def initialize(fields)
      self.fields = fields
    end
    
    def transform(source_obj, source_attribute)
      {}.tap do |values|
        fields.each do |field|
          values[field] = source_obj.send(source_attribute)
        end
      end
    end
  end

  class ComplexMapping < AttributeMapping
    attr_accessor :field
    attr_accessor :instruction
    
    # field       - field symbol we want to map data into
    # instruction - lambda which accepts source object and source attribute. 
    #               expected to return value to be mapped to field (on target obj)
    def initialize(field, instruction)
      self.field = field
      self.instruction = instruction
    end
    
    def transform(source_obj, source_attribute)
      { field => instruction.call(source_obj, source_attribute) }
    end
  end
  
  def initialize(source_model, mappings)
    self.source_model = source_model
    self.mappings = mappings
  end

  def conjure(model)
    values = map_values_for( self.source_model, self.mappings )
    model.to_s.camelize.constantize.new(values)
  end
  
  def map_values_for(source_model, mappings)
    {}.tap do |values|
      mappings.each do |attr, mapping|
        if AttributeMapping === mapping
          values.update(mapping.transform(source_model, attr))
        else
          values[mapping] = source_model.send(attr)
        end
      end
    end
  end
end
```

Using the mapper is really simple.  Let's say my source model, LegacyCustomer, is based is based off of a legacy table from an older system and looks like:

```ruby
LegacyCustomer(
  userid: integer,        # primary key
  creationdate: datetime, # date record was created
  accountnum: string      # customer number. it's prefixed with 
                          # "LGCY-" string that we no longer need!
)
```

And we want to migrate the LegacyCustomer data to a new Customer model that looks like:
```ruby
Customer(
  id: integer,
  created_at: datetime, 
  updated_at: datetime, 
  account_number: string
)
```

My rake task to run the migration would look like:
```ruby
namespace :migrate do
  task :legacy_customers do
    field_mappings = {
      :userid         => :id,
      :creation_date  => Mapper::ListMapping.new(
                          [:created_at, :update_at_]
                         ),
      :user_data      => Mapper::ComplexMapping.new(
                          :account_number, 
                          lambda {|obj, attr| obj.send(attr).gsub('LGCY-','')}
                         )
    }

    LegacyCustomer.all.each do |legacy_customer|
      mapper = Mapper.new(legacy_customer, field_mappings)
      customer = mapper.conjure(:customer)
      customer.save!
    end
  end
end
```

As my migration marched on 'one off' data errors would pop up causing the script to fail.  This is what ultimately led me to create the ComplexMapping class.  Every time some white space, funny character, or field split requirement bombed my script I was able to add some code to my ComplexMapping requirement to solve it.  

I wanted to share this experience for two reasons:

Ruby is awesome.  Metaprogramming and Procs made this mapper possible.  When I first started programming with Ruby, these were the two hardest concepts for me to wrap my head around.  Investing time into learning these aspects of ruby have made me such a better ruby programmer.

The second reason is to reinforce the lesson that doing things the lazy (comfortable) way will rarely ever benefit you.  This mapper class not only made writing and maintaining my migration script easier.  It has also found it's way into some production code. 

Abstracting concepts (or remembering the [single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle)) will always benefit you in the future.  
