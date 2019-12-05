# ticket-bisect

This is a simple script that is an improvement over native git bisect for large-scale repositories with high commit to ticket density.

**Table of Contents**
- [Background](#background)
- [How To Use](#how-to-use)
- [Bisect Tutorial](#bisect-tutorial)
- [FAQ](#faq)

# Background

Bisecting is often the last resort for identifying offending code because it requires rebuilding, retesting, and is completely naive.

git-bisect performs a bisect to identify changed code via commits; this is the lowest level of granularity for a repository.  The premise of this tool is that for most projects it is almost always sufficient to identify the ticket and then intiutition is sufficient to identify the commit (as opposed to repeatedly bisecting until the commit).  ticket-bisect groups and performs the bisect on a higher abstraction level -- tickets instead of commits -- to provide significant improvements.

(1) There are fewer bisect steps

	This tool is intended for repositories with high commit to ticket density.  Binary search is logarithmic so step reduction is also logarithmic.

If there are 1000 commits over 10 tickets : git-bisect will take 10 steps while ticket-bisect will take 4.

(2) Tickets are the proper level of abstraction

	Pull requests are done at the ticket-level not at the commit-level.  Therefore unless you QA each commit, commits are not guaranteed to be stand-alone.  This means that commits often depend on other commits in the same tickets but sometimes these dependencies are not in commit order -- while the fix would be to properly order the commits, in reality bisect at this point leads to compilation failure and a failure to git-bisect.

Bisecting on tickets is more likely to be compile-valid than bisect on commits.

# How To Use

# Bisect Tutorial

I'd suspect this section isn't necessary, however in the case one doesn't understand how a bisect works / bisect algorithm then this section should help.

The only manual action to perform a bisect is choosing the commit. The algorithm to choose the commit is a binary search, therefore it's simply choosing the proper start point and end point and the commit to test is in the middle (or the start if they're equivalent).

For example for 100 options we'd take #50.

If testing leads us to determine the options we're looking for is between 50 and 100 then next we would take 75.
Otherwise the option is between 1 and 50 and we would take 25.
We continue this binary search until it leads us to one option. For Git Bisect this would lead us to the commit, for Liferay Bisect this would lead us to the ticket.

An example path to identify #37 in 100 options would be 50, 25, 37, 31, 34, 36 and after 36 fails we would identify 37 as the target.

# FAQ

	bisecting is bad

While there are techniques to avoid bisecting, this tool presumes the bisect is necessary and is an improvment for target use-cases over git-bisect.

	Why is this a helper script and not a bisect tool?

Originally intended to be a git bisect override or extend bisect functionality, however I'm satisfied with this simple script. 

Currently it is sufficient for my use case.  It's nice to have a complete tool that walks you through the bisect and placing you at the proper commits and keeping track of history, etc, however I'm doubtful that it would provide enough utility to be worth the investment at this time.  I can do this however it's best as a simple helper script for now.