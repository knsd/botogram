.. Copyright (c) 2015 Pietro Albini <pietro@pietroalbini.io>
   Released under the MIT license

.. _channels:

~~~~~~~~~~~~~~~~~~~~~
Working with channels
~~~~~~~~~~~~~~~~~~~~~

Telegram Channels provides a way to broadcast a message to multiple users. You
can manually send messages to them with your preferred Telegram client, but
sometimes you would want to send them with a script, or from a bot.

With botogram you can easily do that, without even the need to run the bot.

.. _channels-preparation:

===========
Preparation
===========

In order to start working with channels, you need to get a bot's API key. If
you already have a bot you can skip this step. First of all you need to contact
the `@botfather`_ Telegram account, and then send to it the ``/newbot``
command. After you provide all the information he needs, you will get an API
key, which looks like this fake one::

   123456789:ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghi

The final step is to add your bot as an administrator of the channel. Open your
favourite Telegram client, and go to the administrators' section of your
channel. From it you should add your bot, and then you're ready.

.. _chanels-standalone:

=========================
Manage without a full bot
=========================

Because a bot's initialization is quite an heavy process, you can use a
lightweight API to just interact with channels. First of all you should import
botogram, and then call the :py:func:`botogram.channel` function to get a
channel object:

.. code-block:: python

   import botogram

   chan = botogram.channel("@my_channel", "YOUR_API_KEY")

You need to replace ``@my_channel`` with your channel's public name, and
``YOUR_API_KEY`` with the key you got before. Then you can use all the methods
of the :py:class:`~botogram.Chat` object with the instance you got. For
example, if you want to send a text message you should do:

.. code-block:: python
   :emphasize-lines: 2

   chan = botogram.channel("@my_channel", "YOUR_API_KEY")
   chan.send("Hello world")

.. _channels-bot:

=========================
Manage from a running bot
=========================

If you want to control a channel from your bot, you can use all the methods
which sends messages with the channel name as the user ID. For example, if you
want to forward all the messages your bot receives to the ``@my_channel``
channel, you can do something like this:

.. code-block:: python

   @bot.process_message
   def forward_messages(chat, message):
       message.forward_to("@my_channel")

If instead you want to announce to the ``@my_channel`` channel when someone
cites botogram in a chat, you can do this:

.. code-block:: python

   @bot.message_contains("botogram")
   @botogram.pass_bot
   def we_are_famous(bot, chat, message):
       user = "Someone"
       if message.from_.username is not None:
           user = message.from_.username

       bot.send("@my_channel", "%s mentioned botogram!" % user)
