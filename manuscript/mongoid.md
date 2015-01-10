Mongoid
=======

## Class using Mongoid

```
class User
  include Mongoid::Document
  include Mongoid::Timestamps # adds automagic fields created_at, updated_at

  references_many :posts  # store ObjectID of :posts, not embedded.
  embeds_one :profile, :dependent => :destroy
  field :first_name # will be stored as String.
  field :fieldname, :type => Array # or BigDecimal / Boolean / Date / DateTime
  index :fieldname #Float / Hash / Integer / String / Symbol / Time
  field :last_drink, type: Time, default: ->{ 10.minutes.ago } # lambda

end
```

## Mogoid Attribute get/set

### Read/Set an a Mongoid Field
```
# Get the value of the first name field.
person.first_name
person[:first_name]
person.read_attribute(:first_name)

### Write/Set a Mongoid Field
```
# Set the value for the first name field.
person.first_name = "Jean"
person[:first_name] = "Jean"
person.write_attribute(:first_name, "Jean")
# Set the field values in the document.
Person.new(first_name: "Jean-Baptiste", middle_name: "Emmanuel")
person.attributes = { first_name: "Jean-Baptiste", middle_name: "Emmanuel" }
person.write_attributes(
  first_name: "Jean-Baptiste",
  middle_name: "Emmanuel"
)
```


## Relations: Embedding and Referencing Models

### Embedding data in the same table:

```
embeds_one :profile
embeds_many :settings
embedded_in :user, :inverse_of => :settings
```

### Referencing data in an other table:

```
referenced_in :user
references_one :profile, :inverse_of => user
references_many :photos, :inverse_of => user
```

## Persistence

In order to make sure data is actually to be written to disk, you need to tell mongoid

```
.safely.save # ensures an actual write
.save  # queued for write
.save! # queued for write, throws exception if validations fails
```

Mongoid only fires the callback of the document that the persistence action was executed on, not it's children

## Queries

```
.where(:amount.gt => 100, :active => true)
.any_in(:category => array) #or .any_in({:category => array})
.all_in(:category => array)
.any_of({ :shape => "round" }, { :color => "red" })
.and(:amount.gt => 100, :account_status => "active")
.excludes(:status => "blocked")
.not_in(:status => ["blocked", "unverified"])
.only(:first_name, :last_name) # only retrieve these fields
.limit(20)
.skip(100)
```
### Criterias

note:   .where(age.gte: 18) fails. Muse be :age.gte => 18 since gte operator is at end

```
.where(:title.all => ["Sir"])
.where(:age.exists => true)
.where(:age.gt => 18)
.where(:age.gte => 18)
.where(:title.in => ["Sir", "Madam"])
.where(:age.lt => 55)
.where(:age.lte => 55)
.where(:title.ne => "Mr")
.where(:title.nin => ["Esquire"])
.where(:aliases.size => 2)
.where(:location.near => [ 22.5, -21.33 ])
.where(:location.within => { "$center" => [ [ 50, -40 ], 1 ] })
.where(name: /^Vancouver$/i)  # case insensitive search
```

### Logical or/and Taking an Array

```
# MongoDB style: note 'away_team_id' is string not symbol
.or({'away_team_id' => {'$in' => teams}}, {'home_team_id' => {'$in' => teams}}) 
.or(center_referee_id: ref_id, fourth_referee_id: ref_id).size # fails. cant figure out {}
# Arrays can be implied or explicit
.or({center_referee_id: ref_id}, {fourth_referee_id: ref_id}).size # implied
.or([{center_referee_id: ref_id}, {fourth_referee_id: ref_id}]).size # explicit
```

#### logical or mongo style (safer)

```
.where('$or' => [{center_referee_id: ref_id}, {fourth_referee_id: ref_id}]) 
# wordier but safer:
.where({'$or' => [{center_referee_id: ref_id}, {fourth_referee_id: ref_id}]}) 
```

### Order

```
.desc(:last_name).asc(:first_name)
.order_by(:last_name.desc, :first_name.asc, :city.desc)
```


### Calculations

```
.max(:age)
.min(:quantity)
.sum(:total)
```

## Rake Tasks

```
db:create
db:create_indexes
db:drop
db:migrate
db:schema:load
db:seed
db:setup
db:test:prepare
```

### Query Examples

```
*date.class==Date
schedule.matches.where('match_times.time' => {'$gte' => from_date, '$lte' => to_date}).all.size

ma=Match.first
teams=[ma.home_team_id, ma.away_team_id]
# 170
Match.any_in(:away_team_id => teams).size #170
Match.where({'away_team_id' => {'$in' => teams}}).size

# logical OR => 336, NOTE can not use any_in w/ OR
Match.or({'away_team_id' => {'$in' => teams}},
         {'home_team_id' => {'$in' => teams}}).size
```

## Special case: find/order by Hash value

givein the case where 

 * User.qualified is a Hash
 * qualified = {‘organization_id’ => {center_referee: true, assitant_referee: true}}

then:

```
User.where("qualified.#{mls.id}.center_referee" => {'$exists' => true}).
  order_by(:"qualified.#{mls.id}.center_referee".asc).size

User.where('qualified.scott.center_referee' => true).size #=> 0 FAIL
User.where('qualified' => {'$exists' => true}).size #1
User.where('qualified' => {'$exists' => true}).first.qualified

a=User.where('qualified.scott' => {'$exists' => true}).first #works
a=User.where('qualified.scott.center_referee' => {'$exists' => true}).first
a=User.where('qualified.51e4e5910d60c67c93000001' => {'$exists' => true}).first
a=User.order_by(:'qualified.scott.center_referee'.asc).pluck(:qualified)
str='qualified.51e4e5910d60c67c93000001'
a=User.where(str => {'$exists' => true}).first; a == scott


scott=User.where('qualified.scott' => {'$exists' => true}).first
str='qualified.' + mls.id + '.center_referee' # qualified[mls.id][center_referee]
a=User.where(str => {'$exists' => true}).first; a == scott # true

# example ordering by a particular Hash values' content
User.where("qualified.#{mls.id}.center_referee" => {'$exists' => true}).
  order_by(:"qualified.#{mls.id}.center_referee".asc).size


a=User.where("qualified.#{mls.id}.center_referee" => {'$exists' => true}).first; a == scott


=> "qualified.#{mls.id}.center_referee"
```


