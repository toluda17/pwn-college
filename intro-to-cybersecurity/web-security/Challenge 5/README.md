# CHALLENGE 5: CMDi 3
This challenge tries to block injection by wrapping my input in single quotes like this:

ls -l 'user_input'

The single quotes prevent special characters from working inside, because the shell treats everything inside quotes literally — until it finds a matching '. I realized I could break out of the single quotes by injecting a ' to close the current quote early, then add my malicious command, and finally fix the quotes so the whole command stays valid. For example, I sent:

' ; cat /flag.txt # The resulting command looked like:

ls -l '' ; cat /flag.txt #'

The first '' is just an empty argument (from the broken quotes). Then the ; cat /flag.txt runs my command. The # comments out the trailing ' to avoid syntax errors.

This works because the shell needs matching quotes, so I close the initial quote early with a ', inject my command, then comment out the leftover trailing quote with #. This tricks the shell into running my injected command safely.
