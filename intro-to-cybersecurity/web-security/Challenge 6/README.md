# CHALLENGE 6: CMDi 4
This challenge runs a command like:

TZ=user_input date

where the user input sets the timezone environment variable before running the date command. I figured out I could inject a command by ending the TZ= value early and then chaining my own command. Since it’s setting an environment variable, I could do something like:

TZ=foo; cat /flag.txt # This way, the shell runs:

TZ=foo; cat /flag.txt # date. The ';' ends the TZ= assignment, then my cat /flag.txt runs, and # comments out the rest (date) so it doesn’t cause errors. This works because the shell runs the whole line, and TZ=foo; ends the environment variable assignment, letting me run arbitrary commands after it. The # comments out the rest so no syntax errors happen.
