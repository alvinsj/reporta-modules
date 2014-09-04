# Reporta Modules

__reporta-modules__ is a Rails gem packed with modules and helpers to help you build your reports. 

It is also a [fork](https://github.com/uts/reporta/issues/2) from the original [__reporta__](github.com/uts/reporta) Rails engine gem. 

## Installation

Add __reporta-modules__ to your Gemfile

```ruby
gem 'reporta-modules'
```

Generate default view templates    
`$ rails generate reporta:views` 

Include `Reporta::Reportable` in your view model.  

```ruby
class YourReport
  include Reporta::Reportable
end
```

## Example

### View Model

```ruby
class ProjectReport
  include Reporta::Reportable

  filter :start_date, default: '2013-01-01', required: true
  filter :finish_date, default: '2013-12-13', required: true

  column :name
  column :created_at

  def rows
    Project.where(created_at: start_date..finish_date)
  end
end
```

### Controller

```ruby
class ProjectReportsController < ApplicationController
  def show
    @report = ProjectReport.new(params[:reporta_form])
  end
end
```

### View Helpers

```erb
<%= filters_for @report %>
<%= table_for @report %>
```



## Reporta Modules

### Reporta::Reportable - DSL for report's view model

`Reporta::Reportable` added DSL for your convenience to create your report's [__view model/object__](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/).

#### Defining Filters

Reports are normally generated based on filters or input from users. Define those filters with `filter filter_name [options]`.  
For example: 

```ruby
class ProjectReport
  include Reporta::Reportable

  filter :start_date, as: :date, default: '2013-01-01', required: true 
  filter :finish_date, as: :date, default: '2013-12-31', required: true
  filter :status, collection: Status.all, include_blank: false
  filter :exclude_deleted, as: :boolean, default: false
  
  def rows 
    # use filters above to manipulate the results
    projects = Project.where(created_at: start_date..finish_date)
      .where(status: status)
    projects = projects.where(deleted_at: nil) if exclude_deleted
    projects
  end
end
```

**Filter Options**

* `required` - set to `true` to specify it as required, defaults to `false`.
* `collection` - specify a collection to render a select input, will be used in `options_for_select(collection)`
* `include_blank` - specify whether to include blank in the select input, defaults to `true`.
* `as` - specify the type of field. e.g. `:boolean`, `:string`, `:check_boxes`, `:radio`, `:date`, defaults to `:string`.  
* `default` - set the default value for the filter.
* `label` - set the label for the filter.

#### Defining Columns

Tables will be generated in your report, defining columns with DSL allows the flexibility of changes on what will be display. Define those columns with `column column_name [options]`.     
For example:  

```ruby
class ProjectReport
  include Reporta::Reportable
  
  def indexed; @index||=0; @index += 1; end
  
  column :indexed, title: 'No.:'
  column :name
  column :created_at, helper: :readable_date, title: 'Created at'
  column :manager, data_chain: 'manager.full_name'
  column :cost, class_names: 'sum currency'

  def rows
    Projects.all
  end

  def readable_date(value)
    value.strftime("%b %d, %Y")
  end
end
```

**Column Options**

* `title` - set a custom title, defaults to the `column.humanize`
* `data_chain` - provide a chain of methods to call in order to fetch the data.
* `class_names` - specify html class attribute for column, useful for custom styling or Javascript hooks.
* `helper` - specify view helper for the column

#### Defining Required Methods

Define the `rows` method to represent data rows to be used for display.  
__Note: For the return array of records, each record should comform(respond_to) to the specified columns.  
For example:  

```ruby
class ProjectReport
  include Reporta::Reportable

  column full_name
  
  def rows
    Project.select('concat(first_name, last_name) as full_name').all
  end
end
```


#### Generating View Templates

View templates are available after running `rails generate reporta:views`, it will generate views in `app/views/reporta` folder for you to customize. These templates are required for the view helpers:

##### filters_for
The `filters_for` helper generates a html form based on the `filter`s you have defined.

```erb
<%= filters_for @report %>
```

Template:  
```erb
<%= form_for @report.form do |f| %>
  <% @report.filters.each do |filter| %>
    <%= f.label filter.name %>
    <%= f.text_field filter.name %>
  <% end %>
  <%= f.submit %>
<% end %>
```
##### table_for
The `table_for` helper generates a html table based on the `column`s you have defined.

```erb
<%= table_for @report %>
```

Template:  
```erb
<table>
  <thead>
  	<tr>
      <% @report.columns.each do |column| %>
        <th class="#{column.class_names}">
       	  <%= column.title %>
        </th>
      <% end %>
    </tr>
  </thead>
  <tbody>
    <% @report.rows.each do |record| %>
      <tr>
        <% @report.columns.each do |column| %>
          <th class="#{column.class_names}">
            <%= @report.value_for(record, column) %>
          </th>
        <% end %>
      </tr>
    <% end %>
  </tbody>
</table>
```

#### Other useful instance methods

After `Reporta::Reportable` is included, useful instance methods are also added.  
**Methods**

* `filters` - returns hash of filters definition. e.g. `filters[:name] #output struct of filter definition` 
* `params` - return hash of parameters received from a request.

## License  
see MIT-LICENSE 
