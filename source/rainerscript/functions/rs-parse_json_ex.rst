***************
parse_json_ex()
***************

Purpose
=======

parse_json_ex(str, container, message_field, alt_message_field, compact)

Extension of parse_json.  There is some log collector that sends a nested 
message to rsyslog, where a string format JSON is associated with a well 
known name, e.g., "log" as in '{"log":"{\"message\":\"Test message\"},...}'.  
If the name is set to ``message_field``, parse_json_ex moves the nested message
to the top level.  If ``alt_message_field`` is specified, the original string 
format JSON is kept with the ``alt_message_field`` value.  
If ``message_field`` is set to "", ``alt_message_field`` is ignored.

When the given JSON contains empty string, array or JSON object type, 
they are eliminated if ``compact`` is set to "true".

Example
=======

In the following example, the parsed and processed JSON is placed into the variable $!parsed.
The output is placed in variable $.ret.

.. code-block:: none

   set $.ret = parse_json_ex("{ \"log\":\"{\\\"message\\\":\\\"Test message\\\",\\\"log_level\\\":\\\"INFO\\\"}\",
                                \"time\":\"2018-05-03T17:43:26.653959-06:00\",
                                "key0":"", "key1":[], "key2":{} }", "\$!parsed", "log", "original_raw_json", "true");

The content of $!parsed:

.. code-block:: none

   { "message": "Test message", "time": "2018-05-03T17:43:26.653959-06:00", "log_level": "INFO",
     "original_raw_json": "{\"message\":\"Test message\",\"log_level\":\"INFO\"}" }
