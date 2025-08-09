# CHALLENGE 1: PATH TRAVERSAL 1
This challenge is about path traversal — basically tricking the server into reading files it’s not supposed to by messing with file paths. The webserver is supposed to only serve files from /challenge/files/, but the way it’s coded, it just slaps the user input straight onto that path and tries to open it.

The key part is this:

requested_path = app.root_path + "/files/" + path return open(requested_path).read()

The path variable is whatever I put after /repository/ in the URL. Since the server doesn’t check if path tries to go up directories (like with ../), I can easily escape from /files/ and grab other files on the system.

I realized I could use ../ to move up the folder tree. So I requested:

GET /repository/../../flag.txt

which the server reads as /challenge/files/../../flag.txt → /flag.txt. That let me access the flag outside the intended folder.
