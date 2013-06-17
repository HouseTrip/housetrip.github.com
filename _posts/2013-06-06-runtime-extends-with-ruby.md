---
layout: post
title: Runtime extends with Ruby
published: true
author: Dawid Sklodowski
author_role: Lead Developer
author_url: http://github.com/dawid-sklodowski
author_avatar: http://www.gravatar.com/avatar/073c19a8d1fd30baa6dba34eaa55fe90.png
summary: Ruby is a dynamic language which supports many ways to organize logic. We can use class inheritance or/and compose our classes by including selected modules (mixins). We can define or undefine methods on the fly. We can even use methods that aren not really defined (using method_missing). Another powerful feature is its ability to extend object with new methods in run-time by including modules to its class or its singleton class (if he/she wants to extend only the one object). To present this design pattern, lets assume that we want to create an application which will entertain our users — an RPG game, where the plot takes place in a fantasy world.
---
<img src="/images/2013-06-06/ruby.jpg" style="float:left; width: 150px; height: 150px; margin: 3px 20px 10px -9px;"/>

Ruby is a dynamic language which supports many ways to organize logic.
We can use class inheritance or/and compose our classes by including selected modules (mixins).
We can define or undefine methods on the fly. We can even use methods that aren't really defined (using method_missing).
Another powerful feature is its ability to extend object with new methods in run-time by including modules to its class
or its singleton class (if he/she wants to extend only the one object).

To present this design pattern, lets assume that we want to create an application which will entertain
our users — an RPG game, where the plot takes place in a fantasy world.

For simplicity of this example we will model character class. The player can pick one of the six races:
dwarf, elf, gnome, hobbit, human or ogre. Then the player must chose their characters occupation including:
priest, programmer, smith, thief, warrior or wizard.

Our character class is going to have a public method: greeting, where results will vary depending on character's
race and occupation. Lets start with rspec test, to illustrate our needs.

{% highlight ruby%}
#spec/character_spec.rb

require 'spec_helper'
require 'character'

describe Character do
  describe '#greeting' do
    it 'works for ogre warrior' do
      ogre = Character.new(:race=>'ogre', :occupation=>'warrior')
      ogre.greeting.should == 'Grumph! I will kill you!'
    end

    it 'works for elven wizard' do
      elf = Character.new(:race=>'elf', :occupation=>'wizard')
      elf.greeting.should == 'Heil! Did you see my staff?'
    end

    it 'works for hobbit thief' do
      hobbit = Character.new(:race=>'hobbit', :occupation=>'thief')
      hobbit.greeting.should == "Good Morning. Haven't you lost something?"
    end

    it 'works for gnome programmer' do
      gnome = Character.new(:race=>'gnome', :occupation=>'programmer')
      gnome.greeting.should == 'Guten Tag.Do You Know Ruby?'
    end

    it 'works for dwarf smith' do
      dwarf = Character.new(:race=>'dwarf', :occupation=>'smith')
      dwarf.greeting.should == 'Humpf! Your sword needs to be fixed.'
    end

    it 'works for human priest' do
      human = Character.new(:race=>'human', :occupation=>'priest')
      human.greeting.should == 'Good Day. Only Chosen One knows his path!'
    end

    it 'works for human programmer' do
      human = Character.new(:race=>'human', :occupation=>'programmer')
      human.greeting.should == 'Good Day. Do you know Ruby?'
    end
  end
end
{% endhighlight %}

Character#greeting is composed of two parts. The first part depends on how the given race says hello, for example the ogre will say Grumph!. The second part is influenced by character's occupation, for example a programmer will ask about Ruby knowledge. By summing up the above examples the ogre programmer will produce a greeting such as: Grumph! Do you know Ruby?

Having our all our tests in red state, lets think about possible implementations.

The easiest way to make the tests pass is by the creation of one class – Character – composed of multiple if (or case) statements, modifying the output of greeting method. In case of our tests it could be shortest implementation also. However, it is most likely that other attributes would be introduced in this class in the future, resulting in the modification of various methods. This leads to a complicated logic which would be very difficult to maintain.

Another possible way to make the tests pass is to separate logic into classes inheriting from the Character class. This solution is nicely supported by Rails framework with its Single Table Inheritance (STI). Going with this approach is good when there is one layer of logic separation. Fer instance, when we separate logic based on the character's race, we can create classes corresponding to races such as: Dwarf, Elf, Gnome, Hobbit, Human, Ogre, but this is not our case. We want to separate logic by race and occupation. This would lead to two layers of logic separation and results in 36 clases like following: OgreProgrammer, OgrePriest, GnomeThief, HobbitWizard, and so on. It is easy to imagine growth of this number when adding new layers of logic separation. We could end with thousands of classes like FemaleYoungWoodenElfArcher.

A solution for this problem that I would like to present is using run-time extends with Ruby. With this solution we create a module for each race and occupation. Also we need some logic to glue things together.

Lets start with Character class:

{% highlight ruby %}
#lib/character.rb

class Character
  include Character::Race
  include Character::Occupation

  def greeting
    "#{race_greeting} #{occupation_greeting}"
  end

  def race_greeting
    raise 'Not implemented'
  end

  def occupation_greeting
    raise 'Not implemented'
  end
end
{% endhighlight %}

Character class implements greeting method, which depends on two other methods: race_greeting and occupation_greeting. Those two methods are expected to be implemented in modules included in lines: 4 and 5. Also those methods are defined in Character class, but they raise an error to indicate that they should be defined elsewhere.

Lets continue with implementation of modules that were included in Character class.

Race module:

{% highlight ruby %}
#lib/character/race.rb

class Character
  module Race

    def initialize(options={})
      @race = options[:race]
      include_race
      super if defined? super
    end

    def race_module
      ActiveSupport::Inflector::constantize("Character::Race::#{@race.capitalize}")
    end

    private
    def include_race
      singleton_class = class << self;self;end;
      singleton_class.send(:include, race_module)
    end
  end
end
{% endhighlight %}

Occupation module:

{% highlight ruby %}
#lib/character/occupation.rb

class Character
  module Occupation

    def initialize(options={})
      @occupation = options[:occupation]
      include_occupation
      super if defined? super
    end

    def occupation_module
      ActiveSupport::Inflector::constantize("Character::Occupation::#{@occupation.capitalize}")
    end

    private
    def include_occupation
      singleton_class = class << self;self;end;
      singleton_class.send(:include, occupation_module)
    end
  end
end
{% endhighlight %}

Those two modules look similar and surely can be refactored, but we will examine that later. For now we will closely examine the Occupation module. First line of initialize method sets instance variable @occupation to hold occupation of character. Second line calls include_occupation method, which includes chosen occupation module to object's singleton class (this means that this module is available only for this particular object, but not for all Character's objects). Described method depends on other method — occupation_module, which returns Module that is going to be included. It uses ActiveSupport's constantize for that (constantize makes constant out of string).

Last line of initialize method:

{% highlight ruby %}
super if defined? super
{% endhighlight %}

calls initialize method that is defined in any other module/class, which is higher in inheritance line. This is important, because it calls not only initialize method of Character‘s class, but it also calls initialize methods defined in all modules, which had been included before the described one was included. It assures that both: initialize defined in Race module and initialize defined in Occupation module would be called.

Race module is analogous to Occupation module, so we can skip its examination.

The last thing we need to implement is modules for particular races and occupations. Since all of the modules are quite similar, I’m only going to list here two of each kind:

Ogre module:
{% highlight ruby %}
#lib/character/race/ogre.rb

class Character
  module Race
    module Ogre
      def race_greeting
        'Grumph!'
      end
    end
  end
end
{% endhighlight %}

Human module:
{% highlight ruby %}
#lib/character/race/human.rb

class Character
  module Race
    module Human
      def race_greeting
        'Good Day.'
      end
    end
  end
end
{% endhighlight %}

Programmer module:
{% highlight ruby %}

#lib/character/occupation/programmer.rb

class Character
  module Occupation
    module Programmer
      def occupation_greeting
        'Do you know Ruby?'
      end
    end
  end
end
{% endhighlight %}

Wizard module:
{% highlight ruby %}

#lib/character/occupation/wizard.rb

class Character
  module Occupation
    module Wizard
      def occupation_greeting
        "Did you see my staff?"
      end
    end
  end
end
{% endhighlight %}

Implementing all required modules makes our tests pass. However we need to require ActiveSupport in Character‘s class by adding those lines at the begging:

{% highlight ruby %}

#lib/character.rb

require 'rubygems'
require 'active_support'
{% endhighlight %}


Changing existing objects in runtime
------------------------------------

So far we’ve implemented a structure that allows us to set character’s race and occupation at the time of the objects creation, using the new method.
However, this doesn’t fulfil our needs, because we need to be able to change the existing character’s occupation and race (this is some kind of magic) in run-time. This can be easily achieved by improving our modules. However lets write some tests first:

{% highlight ruby %}

#spec/character_spec.rb

describe Character do
  context 'attribute readers' do
    it 'are being set during initialization' do
      ogre = Character.new(:race=>'ogre', :occupation=>'warrior')
      ogre.race.should == 'ogre'
      ogre.occupation.should == 'warrior'
    end

    it 'are changeable' do
      character = Character.new(:race=>'ogre', :occupation=>'warrior')
      character.race = 'elf'
      character.occupation = 'smith'
      character.race.should == 'elf'
      character.occupation.should == 'smith'
    end
  end
  describe '#greeting' do
    #Some code removed for clarity
    it 'works for hobbit priest who was dwarf thief' do
      character = Character.new(:race=>'dwarf', :occupation=>'thief')
      character.race = 'hobbit'
      character.occupation = 'priest'
      character.greeting.should == 'Good Morning. Only Chosen One knows his path!'
    end
  end
end
{% endhighlight %}

To make this pass we need to add two methods to Character and Occupation class:

{% highlight ruby %}

#lib/character.rb

class Character
  module Race

    def self.included(base)
      base.send(:attr_reader, :race)
    end

    def race=(value)
      @race = value
      include_race
    end

    #Rest of code removed for clarity.
  end
end
{% endhighlight %}

First method is called upon when the module is included in other module or class (parent class/module is being held by variable base). This is Character class in our case. Only line of this method sets attribute reader for race variable on Character class. Second method is attribute writer, which assigns value to object’s instance variable, and then it includes appropriate race module.

Playing with it even more
-------------------------

To make things complicated, lets implement gnome’s speaking dialect for our characters. Gnomes are very smart and they have a lot to say, so their dialect should speak faster. Gnomes will omit the pauses in between words and use special accents to substitute that. They will say for example:
HowAreYouDoing? instead of How are you doing?.

To depict our needs lets start with modification of character’s tests:

{% highlight ruby %}
#spec/character_spec.rb

describe Character do
  describe '#greeting' do
    it 'works for gnome programmer' do
      gnome = Character.new(:race=>'gnome', :occupation=>'programmer')
      gnome.greeting.should == 'GutenTag.DoYouKnowRuby?'
    end
    #Rest of tests omited for clarity.
  end
end
{% endhighlight %}

To implement that we need to modify Character class, to let it use race speech modifiers if defined any:

{% highlight ruby %}

#lib/character.rb
class Character
  include Character::Race
  include Character::Occupation

  def greeting
    if race_module.methods.include?(:race_modifier)
      race_module.race_modifier(clean_greeting)
    else
      clean_greeting
    end
  end

  def clean_greeting
    "#{race_greeting} #{occupation_greeting}"
  end
  #Rest of code omitted for clarity
{% endhighlight %}

This listing says that if there is race_modifier implemented in race_module then it should be used, otherwise unmodified clean_greeting should be returned. Last thing we need to do is to implement this race_modifier in Gnome module:

{% highlight ruby %}

#lib/character/race/gnome.rb

class Character
  module Race
    module Gnome
      def race_greeting
        'Guten Tag.'
      end
      def self.race_modifier(value)
        value.split(' ').map(&:capitalize).join
      end
    end
  end
end
{% endhighlight %}

This simple example of speech modification for gnomes illustrates how easily and cleanly logic can be extended when run-time extends design pattern is being used.

Some refactoring
----------------

So far we have a very evident code duplication. Race and Occupation modules looks almost the same. We can address this issue by creation of module named Common which will be included in Race and Occupation class as follows:

{% highlight ruby %}
#lib/character/race.rb

class Character
  module Race
    include Common
  end
end



#lib/character/occupation.rb

class Character
  module Occupation
    include Common
  end
end



#lib/character/common.rb

module Common
  def self.included(base)
    base_name = base.name.split('::').last.downcase
    base.send(:attr_reader, base_name.to_sym)
    base.class_eval <<-EOS
      def initialize(options={})
        @#{base_name} = options[:#{base_name}]
        include_#{base_name}
        super if defined? super
      end

      def #{base_name}=(value)
        @#{base_name} = value
        include_#{base_name}
      end

      def #{base_name}_module
        ActiveSupport::Inflector::constantize("Character::#{base_name.capitalize}::\#{@#{base_name}.capitalize}")
      end

      private
        def include_#{base_name}
          singleton_class = class << self;self;end;
          singleton_class.send(:include, #{base_name}_module)
        end
    EOS
  end
end
{% endhighlight %}

This code uses included hook to read name of including module and then, having that name (race or occupation), set reader for attribute named with that name and define methods (using class_eval), which definitions needs this name upfront.

Wrapping up
-----------

Complete code from this blog post can be found here:
[http://github.com/dawid-sklodowski/runtime-extends](http://github.com/dawid-sklodowski/runtime-extends "Runtime Extends with Ruby")
