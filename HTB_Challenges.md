# HTB Challenges
## Forensics - MarketDump
I won't go through all the steps, but once we find the string "NVCijF7n6peM7a7yLYPZrPgHmWUHi97LCAzXxSEUraKme" in the middle of all card numbers, it looks out of place. 

Run it through CyberChef at https://gchq.github.io/CyberChef/. You can use "magic" recipe to get the flag. 

## Misc - Canvas
Run the javascript through a deobfuscator, then in the end of the result, we see a "res" string. Change this to get a flag.
