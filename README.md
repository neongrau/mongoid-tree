# mongoid-tree [![Build Status](https://secure.travis-ci.org/benedikt/mongoid-tree.svg?branch=master)](http://travis-ci.org/benedikt/mongoid-tree) [![Dependency Status](https://gemnasium.com/benedikt/mongoid-tree.svg)](http://gemnasium.com/benedikt/mongoid-tree)

A tree structure for Mongoid documents using the materialized path pattern

## Requirements

* mongoid (>= 4.0, < 9.0)

For a mongoid 3.x compatible version, please use mongoid-tree 1.0.x,
for a mongoid 2.x compatible version, please use mongoid-tree 0.7.x.


## Install

To install mongoid_tree, simply add it to your Gemfile:

    gem 'mongoid-tree', :require => 'mongoid/tree'

In order to get the latest development version of mongoid-tree:

    gem 'mongoid-tree', :git => 'git://github.com/benedikt/mongoid-tree'

You might want to add `:require => nil` option and explicitly `require 'mongoid/tree'` where needed and finally run

    bundle install

### Upgrade from mongoid-tree 1.x

To fix issues with the ordering of ancestors, mongoid-tree 2.0 introduces a new `depth` field to the documents that include the `Mongoid::Tree` module. In case your project uses its own `depth` field, you can now rely on mongoid-tree to handle this.

## Usage

Read the API documentation at http://benedikt.github.com/mongoid-tree and take a look at the `Mongoid::Tree` module

```ruby
class Node
  include Mongoid::Document
  include Mongoid::Tree
end
```

### Utility methods

There are several utility methods that help getting to other related documents in the tree:

```ruby
Node.root
Node.roots
Node.leaves

node.root
node.parent
node.children
node.ancestors
node.ancestors_and_self
node.descendants
node.descendants_and_self
node.siblings
node.siblings_and_self
node.leaves
```

In addition it's possible to check certain aspects of the document's position in the tree:

```ruby
node.root?
node.leaf?
node.depth
node.ancestor_of?(other)
node.descendant_of?(other)
node.sibling_of?(other)
```

See `Mongoid::Tree` for more information on these methods.


### Ordering

`Mongoid::Tree` doesn't order children by default. To enable ordering of tree nodes include the `Mongoid::Tree::Ordering` module. This will add a `position` field to your document and provide additional utility methods:

```ruby
node.lower_siblings
node.higher_siblings
node.first_sibling_in_list
node.last_sibling_in_list

node.move_up
node.move_down
node.move_to_top
node.move_to_bottom
node.move_above(other)
node.move_below(other)

node.at_top?
node.at_bottom?
```

Example:

```ruby
class Node
  include Mongoid::Document
  include Mongoid::Tree
  include Mongoid::Tree::Ordering
end
```

See `Mongoid::Tree::Ordering` for more information on these methods.

### Traversal

It's possible to traverse the tree using different traversal methods using the `Mongoid::Tree::Traversal` module.

Example:

```ruby
class Node
  include Mongoid::Document
  include Mongoid::Tree
  include Mongoid::Tree::Traversal
end

node.traverse(:breadth_first) do |n|
  # Do something with Node n
end
```

### Destroying

`Mongoid::Tree` does not handle destroying of nodes by default. However it provides several strategies that help you to deal with children of deleted documents. You can simply add them as `before_destroy` callbacks.

Available strategies are:

* `:nullify_children` -- Sets the children's parent_id to null
* `:move_children_to_parent` -- Moves the children to the current document's parent
* `:destroy_children` -- Destroys all children by calling their `#destroy` method (invokes callbacks)
* `:delete_descendants` -- Deletes all descendants using a database query (doesn't invoke callbacks)

Example:

```ruby
class Node
  include Mongoid::Document
  include Mongoid::Tree

  before_destroy :nullify_children
end
```


### Callbacks

There are two callbacks that are called before and after the rearranging process. This enables you to do additional computations after the documents position in the tree is updated. See `Mongoid::Tree` for details.

Example:

```ruby
class Page
  include Mongoid::Document
  include Mongoid::Tree

  after_rearrange :rebuild_path

  field :slug
  field :path

  private

  def rebuild_path
    self.path = self.ancestors_and_self.collect(&:slug).join('/')
  end
end
```

### Validations

`Mongoid::Tree` currently does not validate the document's children or parent associations by default. To explicitly enable validation for children and parent documents it's required to add a `validates_associated` validation.

Example:

```ruby
class Node
  include Mongoid::Document
  include Mongoid::Tree

  validates_associated :parent, :children
end
```

## Build Status

mongoid-tree is on [Travis CI](http://travis-ci.org/benedikt/mongoid-tree) running the specs on Ruby Head, Ruby 1.9.3, JRuby (1.9 mode), and Rubinius (1.9 mode).

## Known issues

See [https://github.com/benedikt/mongoid-tree/issues](https://github.com/benedikt/mongoid-tree/issues)


## Repository

See [https://github.com/benedikt/mongoid-tree](https://github.com/benedikt/mongoid-tree) and feel free to fork it!


## Contributors

See a list of all contributors at [https://github.com/benedikt/mongoid-tree/contributors](https://github.com/benedikt/mongoid-tree/contributors). Thanks a lot everyone!


## Support

If you like mongoid-tree and want to support the development, I would appreciate a small donation:

[![Pledgie](http://www.pledgie.com/campaigns/12137.png?skin_name=chrome)](http://www.pledgie.com/campaigns/12137)

[![Flattr](https://api.flattr.com/button/flattr-badge-large.png)](https://flattr.com/submit/auto?user_id=benediktdeicke&url=https://github.com/benedikt/mongoid-tree&title=mongoid-tree&language=&tags=github&category=software)

## Copyright

Copyright (c) 2010-2013 Benedikt Deicke. See LICENSE for details.
