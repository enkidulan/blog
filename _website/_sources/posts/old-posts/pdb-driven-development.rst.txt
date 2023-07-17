.. post:: 02 Dec, 2016
   :tags: python, pdb, debugging
   :category: python
   :author: Maksym Shalenyi
   :excerpt: 2
   :image: 1
   :nocomments:

Pdb Driven Development
======================


Basics of pdb
-------------

The most important command of pdb module that you should know is set_trace method, it enters the debugger at the calling stack frame at a given point in a program, and so far it's the only piece of code that you should not be ashamed to copy-paste:

.. code-block:: python

    import pdb; pdb.set_trace()

When you are in debugger there is a set of command that will help you investigate the scope:

.. code-block:: python

    dir - without arguments, return the list of names in the current local scope. With an argument, attempt to return a list of valid attributes for that object:

    >>> dir(struct) # show the names in the struct module
    ['Struct', '__all__', '__builtins__', '__cached__', '__doc__', '__file__',
     '__initializing__', '__loader__', '__name__', '__package__',
     '_clearcache', 'calcsize', 'error', 'pack', 'pack_into',
     'unpack', 'unpack_from']


    help - Invoke the built-in help system on object, a help page on the object is generated:

    >>> dir(my_list)
    ['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__delslice__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getslice__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__setslice__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
    >>> help(my_list)
    Help on list object:

    class list(object)
     | list() -> new empty list
     | list(iterable) -> new list initialized from iterable's items
     |
     | Methods defined here:
     |
     | __add__(...)
     | x.__add__(y) <==> x+y
     ...
     | sort(...)
     | L.sort(cmp=None, key=None, reverse=False) -- stable sort *IN PLACE*;
     | cmp(x, y) -> -1, 0, 1
     |
     | ----------------------------------------------------------------------
     | Data and other attributes defined here:
     |
     | __hash__ = None
     |
     | __new__ =
     | T.__new__(S, ...) -> a new object with type S, a subtype of T


    inspect - inspect module provides several useful functions to help get information about live objects:

    >>> def f(a, b=1, *pos, **named):
    ... pass
    >>> getcallargs(f, 1, 2, 3)
    {'a': 1, 'named': {}, 'b': 2, 'pos': (3,)}

    >>> getmro(MyClass, 1, 2, 3)
    (, , )

    >>> max = lambda x: 20
    >>> isbuiltin(max)
    False



Also there is a set of pdb command to navigate through code and also for getting information about scope as well:

.. code-block:: python

    w(here) - Print a stack trace, with the most recent frame at the bottom.

    d(own) [count] - Move the current frame count (default one) levels down in the stack trace (to a newer frame).
    u(p) [count] - Move the current frame count (default one) levels up in the stack trace (to an older frame).

    b(reak) [([filename:]lineno | function) [, condition]] - With a lineno argument, set a break there in the current file.

    condition bpnumber [condition] - Set a new condition for the breakpoint, an expression which must evaluate to true before the breakpoint is honored.

    s(tep) - Execute the current line, stop at the first possible occasion

    n(ext) - Continue execution until the next line in the current function is reached or it returns.

    unt(il) [lineno] - Without argument, continue execution until the line with a number greater than the current one is reached.

    c(ont(inue)) - Continue execution, only stop when a breakpoint is encountered.

    l(ist) [first[, last]] - List source code for the current file.

    a(rgs) -Print the argument list of the current function.

    pp expression - Like the print command, except the value of the expression is pretty-printed using the pprint module.

    q(uit) - Quit from the debugger. The program being executed is aborted.


Tricks that are good to know
----------------------------

Pdb is the debugger class.

Use Pdb class to customize behaviour of pdb, exactly you can specify such things as  stdin and stdout, skip list and some others:

pdb.Pdb(completekey='tab', stdin=None, stdout=None, skip=None, nosigint=False, readrc=True)

For me most useful was setting stdin and stdout because it allows to not be depended on application std in/out which in some cases eagerly consumed by app.

RECURSIVE DEBUGGER

That is kind of hidden feature of pdb but still is one that is often used in complex debugging. With debug command you can call any code and step into it with a pdb in a pdb:
.. code-block:: python

    (Pdb++) self.context
    (Pdb++) debug self.context.canSetLayout()
    ENTERING RECURSIVE DEBUGGER
    ((Pdb++))
    --Call--
    [2] >[…]/Products/CMFDynamicViewFTI/ browserdefault.py(183)canSetLayout()
    ((Pdb++))
    183 -> @security.public
    184    def canSetLayout(self):
    186        mtool = getToolByName(self, 'portal_membership')

Through The Web Development
----------------------------

TTW was originally introduced by Plone many years ago and still is widely used  in it even in user friendly manner. Fortunately other tool also use it, the pyramid debug toolbar has very useful exception debugger:
