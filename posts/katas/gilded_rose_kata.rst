.. post:: 27 Sept, 2024
   :tags: python, blog, kata, software-design
   :category: python
   :author: Maksym Shalenyi
   :excerpt: 2
   :image: 1
   :nocomments:


Gilded Rose Refactoring Kata
============================

Not too long ago I tried "Gilded Rose Refactoring Kata". It is an excellent
exercise, and I highly recommend to try it. I've also seen many people sharing
great solutions on GitHub and in personal blogs. However, almost all of
the solutions I've seen did not reflect the reasoning about "why?" and "how?".
Why the current code needs to be refactored? How do I want to change code?
How do I want to go about changing code? Why I refactored the code in this
way and not another? So this post is my reflections on the "why?" and "how?"
which I hope you will find useful and/or entertaining.


The kata and its test is kindly provided by Emily Bache and can be found on
https://github.com/emilybache/GildedRose-Refactoring-Kata. I highly encourage
to give it a go, as I had a lot of fun working on it. The kata goes like this:


    Hi and welcome to team Gilded Rose. As you know, we are a small inn with a
    prime location in a prominent city ran by a friendly innkeeper named
    Allison. We also buy and sell only the finest goods. Unfortunately, our
    goods are constantly degrading in Quality as they approach their sell by
    date.

    We have a system in place that updates our inventory for us. It was
    developed by a no-nonsense type named Leeroy, who has moved on to new
    adventures. Your task is to add the new feature to our system so that we
    can begin selling a new category of items. First an introduction to our
    system:

    * All items have a SellIn value which denotes the number of days we have to
      sell the items
    * All items have a Quality value which denotes how valuable the item is
    * At the end of each day our system lowers both values for every item

    Pretty simple, right? Well this is where it gets interesting:

    * Once the sell by date has passed, Quality degrades twice as fast
    * The Quality of an item is never negative
    * "Aged Brie" actually increases in Quality the older it gets
    * The Quality of an item is never more than 50
    * "Sulfuras", being a legendary item, never has to be sold or decreases
      in Quality
    * "Backstage passes", like aged brie, increases in Quality as its SellIn
      value approaches:

      - Quality increases by 2 when there are 10 days or less and by 3 when
        there are 5 days or less but
      - Quality drops to 0 after the concert

    We have recently signed a supplier of conjured items. This requires an
    update to our system:

    * "Conjured" items degrade in Quality twice as fast as normal items

    Feel free to make any changes to the UpdateQuality method and add any new
    code as long as everything still works correctly. However, do not alter the
    Item class or Items property as those belong to the goblin in the corner
    who will insta-rage and one-shot you as he doesn't believe in shared code
    ownership (you can make the UpdateQuality method and Items property static
    if you like, we'll cover for you).

    Just for clarification, an item can never have its Quality increase above
    50, however "Sulfuras" is a legendary item and as such its Quality is 80
    and it never alters.

That is a nice set of rules, as it is large and diverse enough so that its
implementation won't be too trivial. Surely, it can't be compared to the requirements
of complex production systems, but it gives some space for reasoning about
the domain and designing. And the system's requirements are big
enough for the implementation to go wrong. Here is original code which we must
modify to accommodate selling "conjured items":

.. code:: python


    class GildedRose(object):

        def __init__(self, items):
            self.items = items

        def update_quality(self):
            for item in self.items:
                if item.name != "Aged Brie" and item.name != "Backstage passes to a TAFKAL80ETC concert":
                    if item.quality > 0:
                        if item.name != "Sulfuras, Hand of Ragnaros":
                            item.quality = item.quality - 1
                else:
                    if item.quality < 50:
                        item.quality = item.quality + 1
                        if item.name == "Backstage passes to a TAFKAL80ETC concert":
                            if item.sell_in < 11:
                                if item.quality < 50:
                                    item.quality = item.quality + 1
                            if item.sell_in < 6:
                                if item.quality < 50:
                                    item.quality = item.quality + 1
                if item.name != "Sulfuras, Hand of Ragnaros":
                    item.sell_in = item.sell_in - 1
                if item.sell_in < 0:
                    if item.name != "Aged Brie":
                        if item.name != "Backstage passes to a TAFKAL80ETC concert":
                            if item.quality > 0:
                                if item.name != "Sulfuras, Hand of Ragnaros":
                                    item.quality = item.quality - 1
                        else:
                            item.quality = item.quality - item.quality
                    else:
                        if item.quality < 50:
                            item.quality = item.quality + 1


    class Item:
        def __init__(self, name, sell_in, quality):
            self.name = name
            self.sell_in = sell_in
            self.quality = quality

        def __repr__(self):
            return "%s, %s, %s" % (self.name, self.sell_in, self.quality)


Before jumping into refactoring and implementation of the new requirements,
let's first reflect the main objectives:

#. we need to add a new type of an item called "conjured item"
#. "conjured item" has its own quite distinct set of rules
#. the system must make sure "conjured items" have different rule for changing quality
#. for the rest of the items the system must keep working as before

While refactoring is not part of the objectives as given by the requirements,
the goals cannot be reached by simply adding a new if statement. The code in its
current state looks more like a house of cards rather than something that can
be safely modified. It is even hard to reason and communicate about the code
as it is, and that is major red flag. Let's highlight what makes the current code
challenging to work with:

* A single type of item. While domain mentions "category" the code does not reflect it.
* As a result, the logic that updates quality is based on the names of items.
* Adding a new title will likely require changing code. In the case of backstage
  passes, you would need to update code every time.
* All rules are mixed into a single blob of code that makes it hard to follow
  and understand.
* Some business rules are heavily duplicated (misplaced)
* Changing a rule may require an extensive, if not full, overwrite of the code.
* So, because the code and data are not separated, the extensibility and
  maintenance cost are bad and challenging.

Given the challenges the current code presents, we would be better off with rewriting it
to better reflect the domain rules, to be easier to reason about, and to make adding
new rules or changing the existing ones more effortless.

To make sure I won't change or break the system I will update code in three steps:

1. cover existing code with tests
2. refactor code
3. introduce the new changes

As a the main result of refactoring I want the code to reflect category concept from
domain, to have rules clearly separated, and to have quality update logic based on
category instead item name. Let's have a go on the code:

.. code:: python

    from dataclasses import dataclass
    from enum import StrEnum

    # NOTE: introducing Category concept from domain, so that the Quality
    #       update rules are based on item the category instead of name
    class Category(StrEnum):
        REGULAR = "REGULAR"
        AGED = "AGED"
        LEGENDARY = "LEGENDARY"
        BACKSTAGE_PASS = "BACKSTAGE_PASS"


    @dataclass
    class Item:
        name: str
        sell_in: int
        quality: int
        # NOTE: Item gets a new property `category`
        category: Category = Category.REGULAR

        def __repr__(self):
            return "%s, %s, %s" % (self.name, self.sell_in, self.quality)


    # NOTE: a base class for changing item quality over time. It has three methods:
    #       * public "update" that operates on "item" instance and decrease it
    #         quality and sell_in properties
    #       * private "_get_degree" which return the degree of quality change on
    #         the any given moment
    #       * private "_calculate_new_quality" which return quality value and
    #         is responsible for enforcing upper and lower boundaries
    @dataclass
    class BaseRule:

        def _get_degree(self, item: Item):
            return -2 if item.sell_in < 0 else -1

        def _calculate_new_quality(self, item: Item):
            degree = self._get_degree(item)
            quality = item.quality + degree
            if quality < 0:
                quality = 0
            elif quality > 50:
                quality = 50
            return quality

        def update(self, item: Item):
            item.sell_in -= 1
            item.quality = self._calculate_new_quality(item)


    # NOTE: Regular is the same as the base class, for now it only purpose is
    #       to avoid using the base class directly and provide some protection
    #       for future refactoring of the base class
    class RegularQualityDecreaseRule(BaseRule):
        pass


    # NOTE: This rule increases item quality as time goes, please notice that
    #       upper and lower boundaries are still respected
    class ConstantQualityIncreasingRule(BaseRule):
        def _get_degree(self, item: Item):
            return -super()._get_degree(item)


    # NOTE: This rule increases item quality faster as the sell in day
    #       approaches, but drops quality to zero after sell in day
    class FastIncreaseTillSellIn(BaseRule):

        def _get_degree(self, item: Item):
            if item.sell_in < 0:
                return -item.quality
            elif item.sell_in < 5:
                return 3
            elif item.sell_in < 10:
                return 2
            return 1


    # NOTE: This rule is more unusual as it overwrites `update` method,
    #       as the main goal of this rule is to do nothing.
    class QualityConstantRule(BaseRule):
        def update(self, _):
            pass


    # NOTE: Here is the place where Categories meet Rules.
    #       You probably have noticed already that Categories know nothing about
    #       Rules and Rules are not aware of Categories. This decoupling is not
    #       strictly necessary, but it allows for clearer separation of concerns,
    #       making testing and maintenance easier.
    category_rule_mapping = {
        Category.REGULAR: RegularQualityDecreaseRule,
        Category.AGED: ConstantQualityIncreasingRule,
        Category.LEGENDARY: QualityConstantRule,
        Category.BACKSTAGE_PASS: FastIncreaseTillSellIn,
    }


    def get_rule(item: Item):
        return category_rule_mapping.get(item.name, category_rule_mapping[item.category])


    class GildedRose(object):

        def __init__(self, items):
            self.items = items

        def update_quality(self):
            # NOTE: the main entry point function now looks quite simple:
            for item in self.items:
                rule = get_rule(item)()
                rule.update(item)

After refactoring the code code, adding a new category of items is a trivial matter:

.. code:: python

    class Category(StrEnum):
        ...
        # NOTE: adding a new category
        CONJURED = "CONJURED"

    ...

    # NOTE: adding a new rule
    class DecreasingFastRule(BaseRule):
        def _get_degree(self, item: Item):
            return super()._get_degree(item) * 2

    ...

    # NOTE: linking category to the rule
    category_rule_mapping = {
        ...
        Category.CONJURED: DecreasingFastRule,
    }

The main take away of this refactoring is that now it is easier to reason about
the code, and that the code now is more inline with the domain.

As afterword I want to say that I had a lof of fun working on this kata and thank
Emily Bache for creating this repo https://github.com/emilybache/GildedRose-Refactoring-Kata
