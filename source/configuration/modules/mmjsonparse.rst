JSON/CEE Structured Content Extraction Module (mmjsonparse)
===========================================================

**Module Name:    mmjsonparse**

**Available since:**\ 6.6.0+

**Author:**\ Rainer Gerhards <rgerhards@adiscon.com>

**Description**:

This module provides support for parsing structured log messages that
follow the CEE/lumberjack spec. The so-called "CEE cookie" is checked
and, if present, the JSON-encoded structured message content is parsed.
The properties are then available as original message properties.

As a convenience, mmjsonparse will produce a valid CEE/lumberjack log
message if passed a message without the CEE cookie.  A JSON structure
will be created and the "msg" field will be the only field and it will
contain the message. Note that in this case, mmjsonparse will
nonetheless return that the JSON parsing has failed.

The "CEE cookie" is the character squence "@cee:" which must prepend the
actual JSON. Note that the JSON must be valid and MUST NOT be followed
by any non-JSON message. If either of these conditions is not true,
mmjsonparse will **not** parse the associated JSON. This is based on the
cookie definition used in CEE/project lumberjack and is meant to aid
against an erroneous detection of a message as being CEE where it is
not.

This also means that mmjsonparse currently is NOT a generic JSON parser
that picks up JSON from whereever it may occur in the message. This is
intentional, but future versions may support config parameters to relax
the format requirements.

Action Parameters
~~~~~~~~~~~~~~~~~

Note: parameter names are case-insensitive.

.. function:: cookie <string>

   **Default**: "@cee:"

   Permits to set the cookie that must be present in front of the
   JSON part of the message.

   Most importantly, this can be set to the empty string ("") in order
   to not require any cookie. In this case, leading spaces are permitted
   in front of the JSON. No non-whitespace characters are permitted
   after the JSON. If such is required, mmnormalize must be used.

.. function:: useRawMsg <boolean>

   **Default**: off

   Specifies if the raw message should be used for normalization (on)
   or just the MSG part of the message (off).

.. function:: container <word>

   **Default**: $!

   Specifies the JSON container (path) under which parsed elements should be
   placed. By default, all parsed properties are merged into root of
   message properties. You can place them under a subtree, instead. You
   can place them in local variables, also, by setting path="$.".

.. function:: message_field <word>

   **Default**: NULL

   (Extension) Specifies the JSON key name for the nested string type JSON message. 
   The string format JSON message is converted to the JSON object and merged
   into the top level JSON object.

.. function:: alt_message_field <word>

   **Default**: NULL

   (Extension) Specifies the JSON key name to keep the raw string type JSON of ``message_field``.  
   If ``message_field`` is not set, alt_message_field is ignored.  

.. function:: compact <boolean>

   **Default**: off

   (Extension) Specifies if the JSON to be parsed contains empty string, array or JSON type object, eliminate it (on)
   or no-op (off).

Check parsing result
~~~~~~~~~~~~~~~~~~~~

You can check whether rsyslogd was able to successfully parse the message by reading the
$parsesuccess variable :

::

  action(type="mmjsonparse")
  if $parsesuccess == "OK" then {
     action(type="omfile" File="/tmp/output")
  }
  else if $parsesuccess == "FAIL" then {
     action(type="omfile" File="/tmp/parsing_failure")
  }

Example
~~~~~~~

This activates the module and applies normalization to all messages::

  module(load="mmjsonparse")
  action(type="mmjsonparse")

To permit parsing messages without cookie, use this action statement::

  action(type="mmjsonparse" cookie="")

To merge the value of "log" into the top level JSON as shown in the input/output example, use this action statement::

  action(type="mmjsonparse" cookie="" message_field="log")

  input
  {"log":"{\"message\":\"Test message\",\"log_level\":\"INFO\"}","time":"2020-05-03T17:43:26.653959-06:00"}
  output
  {"message":"Test message","log_level":"INFO","time":"2020-05-03T17:43:26.653959-06:00"}


To merge the value of "log" into the top level JSON with keeping the original string type JSON 
with the key "original_raw_json" as shown in the input/output example, use this action statement::

  action(type="mmjsonparse" cookie="" message_field="log" alt_message_field="original_raw_json")

  input
  {"log":"{\"message\":\"Test message\",\"log_level\":\"INFO\"}","time":"2020-05-03T17:43:26.653959-06:00"}
  output
  {"message":"Test message","log_level":"INFO","time":"2020-05-03T17:43:26.653959-06:00",
   "original_raw_json":"{\"message\":\"Test message\",\"log_level\":\"INFO\"}"}

To eliminate the empty string, array or JSON type object as sown in the input/output example, use this action statement::

  action(type="mmjsonparse" compact=on)

  input
  {"message":"Test message","field0":"","field1":[],"field2":{}}
  output
  {"message":"Test message"}

The same in legacy format::

  $ModLoad mmjsonparse
  *.* :mmjsonparse:
