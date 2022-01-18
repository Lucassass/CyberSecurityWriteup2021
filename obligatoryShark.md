+++
title = "Killer-Queen CTF: Obligatory Shark"
date = "2021-10-29"
aliases = ["ObligatoryShark"]
[ author ]
  name = "Lucas Sass Rósinberg"
+++

# Obligatory Shark

We are given a wireshark file upon opening the file in Wireshark, we see a single Telnet stream.
We right-click and Follow TCP Stream and see a user logging in with username `user2` and password `33a465747cb15e84a26564f57cda0988`. After testing that the password just wasn't the flag we thought that it is likely just an simple hash like MD5, which we can crack with

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5
```

which immediately spits out the password `dancingqueen`. Wrapped in the flag format, this is the correct flag.

Funny site node the hint which was given later in the CTF to people was good
Hint: youtube link:[Queen - We Will Rock You](https://www.youtube.com/watch?v=-tJYN-eG1zk)
