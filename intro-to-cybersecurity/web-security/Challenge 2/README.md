# CHALLENGE 2: PATH TRAVERSAL 2
This challenge tries to stop path traversal by stripping leading dots and slashes from the user input path. However, the method used is pretty naive and doesn’t cover all cases.

I noticed that .strip("/.") only removes dots and slashes at the start and end of the string. So if I prepend a word before the ../, the strip doesn’t remove it anymore. For example, sending a path like:

GET /content/bypass../..//flag.txt

lets the ../.. stay intact in the middle, effectively moving up X+1 directories (one more than the number of ../ parts) because of how the path joins.
