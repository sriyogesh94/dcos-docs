---
post_title: Debugging from the DC/OS CLI
feature_maturity: preview
menu_order: 10
---

The DC/OS CLI provides commands to debug services that are not deploying or behaving as expected.

To see full logs, append `--log-level=debug` to any DC/OS CLI command.

For example, to troubleshoot HDFS package installation, use
```
dcos -—log-level="debug" package install hdfs
```
For more information about log levels, consult the [CLI command reference](/docs/1.9/usage/cli/command-reference/) or run `dcos --help`.
