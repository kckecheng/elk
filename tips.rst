Tips
=====

How to add tags based on filed content with pipelines?
--------------------------------------------------------

If the filed's name is known, it can be used directly. If not, use "message" which holds everything.

::

  filter {
    if [message] =~ /regexp/ {
      mutate {
        add_tag => [ "tag1", "tag2" ]
      }
    }
  }
