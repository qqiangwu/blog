I am quite confused that what is exactly greedy approach? 

+ Is it methodology just like DP? 
+ Does it have any relation to various recursion based design paradiagms?

Basically, GA and DP both apply to problems with optimal subproblems. They are both subordinated to recursion paradiagms. DP is somehow like brute force, it checks all possibilities, while GA makes a choice immediately based on a greedy choice. DP are guaranteed to produce a correct result, whereas GA not. You must prove that the problem has the greedy property, i.e. local optimal yields global optimal.

More precisely, GA is barely a strategy rather than a design technique. Whenever we need to make a choice, try to make it greedily and to prove that it works. So don't be tied in front of the words GREEDY APPROACH, they are barely a strategy!