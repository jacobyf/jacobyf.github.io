---
layout: post
title: "Set windows environment variable through CMD"
date: 2014-03-07 06:36:00
categories: [coding, skills]

excerpt: "If you want to set windows environment variable through CMD, definitely, you have two ways according to your usage aspects."

author:
  name: Jacob Yang
---

If you want to set windows environment variable through CMD, definitely, you have two ways according to your usage aspects.

# Aspect 1: Use in the context of your process
You can use this bat command to set a variable, when you want you own process use just this time.

    > SET VARIABLE_X = VARIABLE_VALUE
    > ECHO %VARIABLE_X%


# Aspect 2: Use int the system context
Sometimes, you want register a variable that cant be used globally, the only thing you have to do is just change command a little above. Other than use `SET` command, you should use `SETX` command here.

    > SETX VARIABLE_Y = VARIABLE_VALUE
    > ECHO %VARIABLE_Y%



