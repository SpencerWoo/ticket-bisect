# ticket-bisect

A better git-bisect for repositories with high commit to ticket density.

**Table of Contents**
- [Background](#background)
- [How To Use](#how-to-use)
	- [Setup](#setup)
	- [Usage](#usage)
	- [Additional Notes](#additional-notes)
		- [Ticket Format](#ticket-format)
		- [Bisect Wrapper](#bisect-wrapper)
- [Bisect Tutorial](#bisect-tutorial)
- [FAQ](#faq)

# Background

Bisecting is often the last resort for identifying offending code because it requires rebuilding, retesting, and is completely naive.

[git-bisect](https://git-scm.com/docs/git-bisect) performs a bisect to identify changed code via commits; this is the lowest level of granularity for a repository.  The premise of this tool is that for most projects it is almost always sufficient to identify the ticket and then intiutition is sufficient to identify the commit (as opposed to repeatedly bisecting until the commit).  

(1) There are fewer bisect steps

* This tool is intended for repositories with high commit to ticket density.  Binary search is logarithmic so step reduction is also logarithmic.

If there are 1000 commits over 10 tickets : git-bisect will take 10 steps while ticket-bisect will take 4.

(2) Tickets are the proper level of abstraction

* Pull requests are done at the ticket-level not at the commit-level.  Therefore unless you QA each commit, commits are not guaranteed to be stand-alone.  This means that commits often depend on other commits in the same tickets but sometimes these dependencies are not in commit order -- while the fix would be to properly order the commits, in reality bisect at this point leads to compilation failure and a failure to git-bisect.

Bisecting on tickets is more likely to be compile-valid than bisect on commits.

# How To Use

### Setup

1. Install Python 3.6+
2. Download ticket-bisect.py
3. Setup Alias to run from anywhere

```
tb(){
	python3 path\ticket-bisect.py $@
}
```

This runs python3 which runs ticket-bisect.py and passes in all additional arguments.

4. Run ticket-bisect from any repository directory

```
tb fix-pack-de-57-7010 fix-pack-de-58-7010
```

### Usage
---

If you're unfamiliar with bisecting in general feel free to read [Bisect Tutorial](#bisect-tutorial).

Assuming you're familiar with bisecting this section will detail how to use this helper script.



TODO : explain how to use helper script generated output

### Additional Notes
---

#### Commit Format

| Valid | Invalid |
| --- | --- |
| `TICKET-123 Description` | `Description TICKET-123` |
| `TICKET123 Description` | `TICKET 123 Description` |
| `123-ticket Description` | `Description 123-ticket` |
| `tickethash Description` | `Description tickethash` |

These are invalid because we're taking the first match when split on space -- `^[\w\d]*` -- and expecting it to be the unique idenitifer/ticket.

While these invalid example won't work OOTB, it is quite simple to modify the script so it works with any ticket formatting.

#### Bisect Wrapper

Instead of working with a text file, the Liferay Bisect Wrapper script generates a self-contained HTML file called bisect_log.html, which is an interactive way to keep track of where you are in the bisect process. More information can be found here:

https://github.com/holatuwol/liferay-faster-deploy/tree/master/notmine#liferay-bisect



# Bisect Tutorial

If you've never done a git-bisect, feel free to google or read my explanation of how to think about it below.

Bisecting is when there's a change in behavior and you're interested in identifying that change.  Bisecting is a naive way to find the specific commit that changed the program's behavior.

The only manual action to perform a bisect is choosing the commit. The algorithm to find the offending commit is a binary search.  So we're given the start and end points and we choose the middle commit to test.  Given the result of that test, we move the start to the middle or the end to the middle and then choose the middle commit again of our new range.

To visualize an example we might have 100 options.  The start would be 1, end would be 100, and our middle would be 50.  The start and end are given to us from the beginning (this is how we know we have 100 options) and our only decision is choosing the middle -- 50.

We would test 50 and if that testing determined to be the same as #1 then we would know the option we're interested in is between 50 and 100.  Our next middle would be 75.
Otherwise if testing determined 50 to exhibit the same behavior as #100 then we would know the option we're interested in is between 1 and 50.  Our next middle would be 25.

We repeatedly perform this binary search until it leads us to one option.  For Git Bisect this would lead us to the commit, for Ticket Bisect this would lead us to the ticket.

---

An example path to identify #37 in 100 options would be 50, 25, 37, 31, 34, 36 and after 36 fails we would identify 37 as the target.

# FAQ

	bisecting is bad

While there are techniques to avoid bisecting, this tool presumes the bisect is necessary and is an improvment for target use-cases over git-bisect.

	Why is this a helper script and not a bisect tool?

Originally intended to be a git bisect override or extend bisect functionality, however I'm satisfied with this simple script. 

Currently it is sufficient for my use case.  It's nice to have a complete tool that walks you through the bisect and placing you at the proper commits and keeping track of history, etc, however I'm doubtful that it would provide enough utility to be worth the investment at this time.  I can do this however it's best as a simple helper script for now.