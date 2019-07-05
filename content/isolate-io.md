+++
title = "Isolate I/O"
date = 2019-07-06T12:00:00Z
# template = "micro.html"
+++

Most errors occur when handling user input.  Keep your I/O at the edges of your
application. Validate it. Turn it into well-structured, typed data. Only then
let it near your application. 

Let your business logic stay on the happy path.
