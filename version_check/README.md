# Simple Hello World SimDem Script

The demos in this repository are designed to work
with [SimDem](http://github.com/azure/simdem). 

# Version check

The scripts in this project are expected to work against a specific
version of SimDem, the test below ensures that we are testing against
the correct version when running in CI.

```
echo $SIMDEM_VERSION
```

Results:

```
0.7.4-dev
```

# Environment Configuration

For debugging purposes it can be useful to have a dump of the
environment, so here it is:

```
env
```

