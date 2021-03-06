.. Copyright (c) 2015 Pietro Albini <pietro@pietroalbini.io>
   Released under the MIT license

.. _shared-memory:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Sharing objects between workers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The botogram's runner is fast because it's able to process multiple messages at
once, and to archive this result botogram spawns multiple processes, called
"workers". The problem with this is, you can't easily share objects between the
workers, because each one lives in a different process, within a different
memory space.

The solution to this problem is shared memory. Shared memory allows you to
store global state without worrying about synchronizing it. It just works as a
standard Python dictionary. Also, each component has a different shared memory
than your bot, so you don't need to worry about conflicts between components.

.. _shared-memory-basics:

How to use shared memory
========================

botogram has an useful decorator, :py:func:`botogram.pass_shared`, which
provides the shared memory's object as first argument of the function. You can
then store everything in it, and those things will be synchronized for you.

Please note that shared memory's object is provided only if the function is
called by botogram itself: if you call it directly, that argument will be
missing.

.. note::

   Synchronization uses pickle under the hood, so you can store in the shared
   memory only objects pickle knows how to serialize. Please refer to the
   official Python documentation for more informations about this.

Here there is a simple example of an hook which uses the shared memory to count
how much messages has been sent:

.. code-block:: python

   @bot.process_message
   @botogram.pass_shared
   def increment(shared, chat, message):
       if "messages" not in shared:
           shared["messages"] = 0

       if message.text is None:
           return
       shared["messages"] += 1

As you can see, first of all the code initializes the ``messages`` key if it
doesn't exist yet. Then it just increments it. Next there is an example of a
command which displays the current messages count calculated by the hook above:

.. code-block:: python

   @bot.command("count")
   @botogram.pass_shared
   def count(shared, chat, message, args):
       messages = 0
       if "messages" in shared:
           messages = shared["messages"]

       chat.send("This bot received %s messages" % shared["messages"])

.. _shared-memory-inits:

Shared memory initializers
==========================

In the example above, a big part of the code is just to handle the case when
the shared memory doesn't contain the ``count`` key, and that's possible only
at startup. In order to solve this problem, you can use the
:py:meth:`botogram.Bot.init_shared_memory` decorator.

Functions decorated with that decorator will be called only the first time you
require the shared memory. This means you can use them to set the initial value
of all the keys you want to use in the shared memory.

For example, let's refactor the code above to use an initializer:

.. code-block:: python

   @bot.init_shared_memory
   def init_shared_memory(shared):
       shared["messages"] = 0

   @bot.process_message
   @botogram.pass_shared
   def increment(shared, chat, message):
       if message.text is None:
           return
       shared["messages"] += 1

   @bot.command("count")
   @botogram.pass_shared
   def count_command(shared, chat, message, args):
       chat.send("This bot received %s messages" % shared["messages"])

As you can see, the code is now clearer, and we can be sure the key we need
will always exist. This can especially be useful if you have a lot more hooks.

.. _shared-memory-components:

Shared memory in components
===========================

Shared memory is really useful while you're developing :ref:`components
<custom-components>`, because it's unique both to your component and the
current bot. This means, you don't have to worry about naming conflicts with
other components, and each bot's data will be isolated from each other if the
component is used by multiple bots.

Using shared memory on a component is the same as using it in your bot's main
code: just use the :py:func:`botogram.pass_shared` decorator to get the shared
memory instance as first argument. To add a shared memory initializer, you can
instead provide the function to the
:py:meth:`botogram.Component.add_shared_memory_initializer` method.
