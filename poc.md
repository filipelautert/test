## POC results

### subtree
* all in 1 project, branch logic is gone
* We'll need to rewrite poms and parts of the build to simplify it, as now now pro build will happen entirely in pro repo.
* we can simplify OSS build a lot (branch logic is gone too)


### fork
* 2 project, branch logic remains
* build logic needs to be moved to pro and still complex as it is today (if we keep it in oss fork we may end up pushing it to oss by mistake)
* we can simplify OSS build a lot (branch logic is gone for it)

### fork into pro
* 1 project, branch logic is gone
* git history may get complicated
* impossible to send back to OSS, one way fork only
