# Model Name Migration Guide

This guide describes how to rename a Rails model. We'll use renaming `Circle` to `Shape` as an example.

## Step 1: Rename the database table

Create a migration to rename the table:

```ruby
rename_table :circles, :shapes
```

Commit and deploy this change before proceeding. The old model code will
continue to work because Rails looks up the table name from the model class,
and the model still references the old table name until we update it.

## Step 2: Create new model file

Before:
```ruby
class Circle < ApplicationRecord
  # contents of model
end
```

After:
```ruby
class Circle < Shape
  # class is now empty
end
```

```ruby
class Shape < ApplicationRecord
  # contents of model
end
```
