---
layout: post
title: ransack and ransackers
date: 2015-03-15 18:55:28 +0300
access: public
comments: true
categories: [ransack, arel, active admin]
---

Active Record uses Arel inside to construct SQL queries.

ransack is a gem to search for specific records of given model - pretty much the way custom scopes operate. it relies upon arel library to perform custom searching.

<!-- more -->

the point is that AA uses ransack inside and would apply all search conditions to resulting collection in the end. or else we can do it manually towards any model class or AR relation - ransack monkeypatches ActiveRecord::Base with its `#search` method aliased to `#ransack` (we should pass search conditions from `params[:q]` about which we'll talk next) - this method will return standard AR relation. these search conditions are located inside `params[:q]` hash and this is where they would be placed after applying any particular filter on AA page. every search condition has the form `key => value` where `key` is composed of model attribute and search predicate separated by underscore while `value` contains, well, value to filter by. in case of standard filter specifying existing model attribute AA in filter definition would append search predicate to attribute name by itself - in this case query string would contain the following substring:


```
q%5Bsite_id_eq%5D=100001
==
q[site_id_eq]=100001
```

that is filter on site_id = 100001: search predicate `eq` is appended by AA based on specified filter. and `params` variable inside AA controller would subsequently contain the following search condition:

```
{"utf8"=>"✓",
"q"=>{"site_id_eq"=>"100001"},
"commit"=>"Фильтровать",
"order"=>"created_at_asc",
"action"=>"index",
"controller"=>"admin/orders"}
```

in the end this condition would be applied to index collection (or any collection returned from overriden `scoped_collection` method).

in case of custom filter AA wouldn't be able to figure out what search predicate to append that is why it's necessary to specify it in the name of AA filter explicitly. as a result this new filter condition alongside its value will be added to query string (inside `q` hash to be precise).

so far so good but still we don't have a corresponding search method. obviously we need to implement it and this is where ::ransacker class method comes into play. it's used to define search method inside model (in our case we defined it in a class inheriting from model as to not pollute original model). important thing to note is that ransacker name should not contain search predicate suffixes itself - ransacker should take any value supplied to custom filter and return arel object so that ransack gem could apply search predicates (suffixes of custom filter name) to filter result even further.

to top it all, defining custom filter for AA page involves 2 basic steps:

1. define custom filter in AA resource page
2. define ransacker inside model to return filtered records

```ruby
class Order < ActiveRecord::Base
  ransacker :title do |parent|
    parent.table[:title]
  end
end
```

that is ransacker will return the 1st part of where clause, ransack gem will add predicate and user will supply value using filter title_cont (with `cont` predicate) would result into the following SQL:

```sql
SELECT * FROM "orders" WHERE "orders"."title" LIKE '%blablabla%';
```

it's also possible to use ransacker's formatter option and specify proc that will be used by ransack gem to format user supplied value before inserting it into SQL query.

ransacker should return arel object so that it's possible to chain other ransack predicates like eq, lt, cont, etc. it's not allowed to return simple relation. but different AR predicates (say, where) can accept arel object as conditions while returning relation in the end. vice versa is also possible: arel can accept relations but after converting them into arel objects using ast method (see [link](http://jpospisil.com/2014/06/16/the-definitive-guide-to-arel-the-sql-manager-for-ruby.html)).

there is no such arel predicate as `cont` e.g. - it's defined using matcher predicate and formatter proc in ransack (see [link](https://github.com/activerecord-hackery/ransack/blob/master/lib/ransack/constants.rb)). ransack eventually generates smth like this for `name_cont` filter:

```ruby
users[:name].matches("%#{user_name}%")
```

any preliminary processing of user-input data should go to formatter's proc - the latter should produce input values for where clause (that consists of model attribute from ransacker's block, search predicate from filter name and value from either user or formatter's proc). formatter's proc is invoked only when you specify some attribute in the block and after that block has been yielded.
important thing to notice about constructing custom filter! if using custom filter most likely you will supply some collection which will be displayed by AA in dropdown menu (or whatever). if collection member is an array itself the 1st element of that array will be used as a title while the 2nd one will be passed for further processing (e.g. this is what formatter's proc will receive):

```ruby
filter :seo_specialist_eq,
  as: :select,
  collection: [['Не назначен', :not_assigned]].concat(User.seo_specialists.pluck(:email))
```

in this case AA will display the following options: 'Любой' (displayed by default for any collection), 'Не назначен', 'seo_specialist1@company.ru', etc. if user selects option 'Любой' ransacker won't be called at all. Also don't use nil value as the 2nd element of collection member - AA (or ransack) will just ignore filter when user selects this option (no filter will be applied at all).

fake morr's ransackers just provided dummy methods so that chaining is possible but they wouldn't produce any valid results of course. in GroupedReportHelper all custom filter names are converted to standard ones which are happily applied by ransack. while all customs filters end up doing nothing because of dummy ransackers defined inside correspoding model (a class inheriting from model to be precise).

if you just need to search on associated models' attributes it's not necessary to define any ransacker - it's all the same you wouldn't be able to access the very association inside ransacker directly since self is equal to model class inside ransacker while association is available for model instance only. moreover the output of ransacker would come as the 1st part of where clause -> it's meaningless to specify associated model's attributes inside it without prior join but we cannot make that join in ransacker that is we cannot modify the rest of generated SQL query - only where clause.
to sum it all, ransackers are useless for searching on associated models' attributes but ransack gem natively provides this functionality: it's sufficient to prefix filter name with current model's association name followed by associated model's attribute and required search predicate. voila! it all magically works! (see [link](https://github.com/activerecord-hackery/ransack))

BTW inside ransacker's block we can get arel table object by calling parent.table (and access attributes through array notation) - it can be further used to make join with another table.

### COMPLETE EXAMPLE:

(search for order using `seo_specialist` association field - `email`)

*model*:

```ruby
class Order < ActiveRecord::Base
  belongs_to :seo_specialist, class_name: User.name

  ransacker :seo_specialist,
    formatter: -> (email) {
      email == 'not_assigned' ? nil : User.find_by(email: email).id
    } do |parent|
    parent.table[:seo_specialist_id]
  end
end
```

*model spec*:

```ruby
context :ransacker do
  describe 'seo_specialist' do
    let!(:order) { create :order, seo_specialist: seo_specialist }

    context 'with email seo_specialist@company.ru' do
      let(:seo_specialist) { create :user, :seo_specialist, email: email }
      let(:email) { 'seo_specialist@company.ru' }
      subject { Order.search({ seo_specialist_eq: email }).result }

      it { expect(subject.first).to eq order }
    end

    context 'with email not assigned' do
      let(:seo_specialist) { nil }
      let(:email) { 'not_assigned' }
      subject { Order.search({ seo_specialist_eq: email }).result }

      it { expect(subject.first).to eq order }
    end
  end
end
```

*active admin page for order*:

```ruby
filter :seo_specialist_eq,
  as: :select,
  label: 'Специалист',
  collection: [['Не назначен', :not_assigned]].concat(User.seo_specialists.pluck(:email))
```

***UPDATE (2015-02-05)***

we might override `scoped_collection` method and select a small set of attributes or create our own ones (using "select as" in SQL query). still when manually supplying additional search conditions (e.g. inserting them into `params[:q]` in `included` callback of some module that is meant to be included into AA page) it's necessary to specify original model attribute (plus search predicate) - not any one of SQL aliases defined inside SQL query! AA (or ransack) doesn't search on SQL aliases in scoped collection! also remember that if filtering on association attribute it's necessary to prefix search key in `params[:q]` with association name and underscore (as discussed above). we had such kind of bug when we prefixed search key with association name (using klass_with_date method) while filtering on model attributes directly.

***UPDATE (2015-03-02)***

this is how we can make default AA date range filter inclusive on upper bound:

```ruby
ActiveAdmin.register Order, as: 'natu_orders' do
 filter :user_registration_date, as: :date_range
end

class User < ActiveRecord::Base
 ransacker :registration_date,
   formatter: -> (date) { Time.zone.parse(date).end_of_day } do |parent|
   parent.table[:created_at]
 end
```

pay attention that custom ransacker filter is defined on association model.
this way filter will be applied (corresponding search conditions are present in URL) but filter values will not filled in filter itself (it will be empty).
instead it's necessary to specify ransacker type datetime:

```ruby
ransacker :registration_date, type: :datetime,
  formatter: -> (date) { date.end_of_day } do |parent|
  parent.table[:created_at]
end
```

pay attention that this time date is not a string but TimeWithZone object - we don't need to parse anymore. after this change date range filter is filled correctly after page refresh.

***UPDATE (2015-03-06)***

```ruby
ransacker :l_date_at, type: :datetime do |parent|
  Arel::Table.new('partner_transactions')[:l_date_at]
end
```

in this case `partner_transactions` is an alias of some table inside query of current model's scope. that is it's not even association table - it turns out we can specify any available table here if it makes sense of course. we use `l_date_at` name because this is the field used in AA filter but we don't have this field in resulting collection - this is the field of related table that is joined to current model's table.


## RESOURCES (IN ORDER OF DECREASING IMPORTANCE):

### ransack:

* <https://github.com/activerecord-hackery/ransack>
* <https://github.com/activerecord-hackery/ransack/wiki/Basic-Searching>
* <https://github.com/activerecord-hackery/ransack/wiki/Using-Ransackers>
* <https://github.com/activerecord-hackery/ransack/blob/master/lib/ransack/constants.rb> (list of ransack predicates with their definitions)
* <http://nikhgupta.com/code/activeadmin/custom-filters-using-ransacker-in-activeadmin-interfaces/> (turned out pretty useless in the end)
* <http://stackoverflow.com/questions/22473391/custom-search-with-ransacker> (example of custom processing inside formatter)
* <http://railscasts.com/episodes/370-ransack> (introduction screencast)

### arel:

* <http://jpospisil.com/2014/06/16/the-definitive-guide-to-arel-the-sql-manager-for-ruby.html> (the most comprehensive guide to arel)
* <http://www.slideshare.net/flah00/activerecord-arel> (very informative slideshow on arel - it makes sense to start with it)
* <https://github.com/rails/arel>
* <http://erniemiller.org/2010/05/11/activerecord-relation-vs-arel/> (difference between AR and arel with many examples)
* <http://rubydoc.info/github/rails/arel/master/Arel/Predications#matches_any-instance_method> (list of arel predications)
* <http://pivotallabs.com/using-arel-to-build-complex-sql-expressions/> (brief but has some examples - in particular about InfixOperation)
* <http://robots.thoughtbot.com/using-arel-to-compose-sql-queries> (haven't read yet)
