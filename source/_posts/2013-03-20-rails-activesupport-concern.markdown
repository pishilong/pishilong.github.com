---
layout: post
title: "Rails ActiveSupport::Concern Object doesn't support#inspect"
date: 2013-03-20 11:06
comments: true
categories: ["技术"]
keywords: rails, Concerns, Object doesn't support#inspect ActiveSupport::Concern
---
这篇博客记录使用ActiveSupport::Concern对业务进行抽象时，遇到的Object doesn't support#inspect问题.

#背景

树形数据结构在系统中越来越多，而他的很多特性是可以复用的，如果对每一个model都定义相同的方法，声明相同的asscoiation肯定是不科学，于是打算将其抽取出来.

第一个方案打算使用继承,使用通用类继承ActiveRecord::Base 然后其他的树形类再继承这个通用类

```ruby
  TreeBase < ActiveRecord::Base
  end
  GrammarTree < TreeBase
  end
```

但是这样会报错，提示在数据库中找不到tree_bases这张表，这是肯定的，于是根据网上的方法，把TreeBase设置成没有表与其对应的虚类，但是Rails还是不吃这一套.

```ruby
  TreeBase < ActiveRecord::Base
    self.absract_class = true
    @columns = []
  end
```

于是打算采用module的方案，[Rails4重大设计决策：“胖”Model用ActiveSupport::Concern瘦身](http://ruby-china.org/topics/7709), 那就先采用这个做法试验一下.

```ruby
module TreeBase
  extend ActiveSupport::Concern

  included do
    attr_accessible :p_id, :_name, :order_idx, :level, :overall_idx
    belongs_to :parent, class_name: self.class.name, foreign_key: "p_id"
    has_many :children, class_name: self.class.name, foreign_key: "p_id", dependent: :restrict, order: :order_idx
    validates :node_name, presence: true

    default_value_for :level do
      parent.nil? ? 0 : (.parent.level + 1)
    end
    default_value_for :order_idx do
      if parent.nil?
        (self.class.where(p_id: nil).maximum(:order_idx) || 0) + 1
      else
        (parent.children.maximum(:order_idx) || 0) + 1
      end
    end
    default_value_for :overall_idx do
      self.compute_overall_idx
    end

    after_save :reset_index
  end

  module ClassMethods
  end

  module InstanceMethods
  end
end
```

以上代码的思路很容看清楚，树形结构肯定有`parent`,也有`children`,而`TreeBase`只是提供一个处理框架，具体的表还是得看`include`了这个`module`的类，所以在`class_name`中用的是`self.class.name`.
同理，在查找节点时，也用的`self.class.where(p_id: nil)`

#问题

这个时候，问题发生了，找打一个node没问题，但是`node.parent`的时候，就会报错`Object doesn't support#inspect`.

最终的原因在于`module`中`included`,`InstanceMethods`,`ClassMethods`的上下文不一样，`InstanceMethods`定义的是实例方法，所以默认的上下文就是实例，于是可以这样定义：

```ruby
module TreeBase
  extend ActiveSupport::Concern
  module InstanceMethods
   #判断是否有子节点
    def has_children?
      children.count > 0
    end

    #计算全局顺序
    def compute_overall_idx
      if parent.nil?
        order_idx.to_s.rjust(2,'0')
      else
        "#{parent.overall_idx}_" + order_idx.to_s.rjust(2,'0')
      end
    end
  end
end
```

`children`这样的属性或者实例方法能直接调用.

而在`included`中，上下文是具体的类，所以`self`是形如`GrammarTree`这样的具体类,因此`self.class`就是`Class`了，而`self.class.name`就是`'Class'`，而我们期待的实际是`'GrammarTree'`，

就这样，出现了`Object doesn't support#inspect`这个问题，衍生出来的，如果执行`node.has_children?`，因为`node.children`已经跪了，再来`chidlren.count > 0 `就会报错`undefined method 'scoped' for Class:Class`。

#解决

把`self.class.name`改成`name`,由于已经处于类的上下文，所以直接用`name`即可.

```ruby
  included do
    attr_accessible :p_id, :node_name, :order_idx, :level, :overall_idx
    belongs_to :parent, class_name: name, foreign_key: "p_id"
    has_many :children, class_name: name, foreign_key: "p_id", dependent: :restrict, order: :order_idx
    validates :node_name, presence: true

    default_value_for :level do |node|
      node.parent.nil? ? 0 : (node.parent.level + 1)
    end
    default_value_for :order_idx do |node|
      if node.parent.nil?
        (where(p_id: nil).maximum(:order_idx) || 0) + 1
      else
        (node.parent.children.maximum(:order_idx) || 0) + 1
      end
    end
    default_value_for :overall_idx do |node|
      node.compute_overall_idx
    end

    after_save :reset_index
  end
```

  其中，`default_value_for`是设置对象默认值的，其上下文应该是实例对象而不是类，所以需要在代码块中传入`node`.
