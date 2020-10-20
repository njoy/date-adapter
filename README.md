# date-adapter

An adapter repository to plug the Howard Hinnant's header-only [date-time](https://github.com/HowardHinnant/date) library into the NJOY21 build system

```console
# This only needs to be done once (per clone)
git remote add date https://github.com/HowardHinnant/date.git

# Do this when you need to update the subtree
git subtree pull --squash --prefix=src date master
```

