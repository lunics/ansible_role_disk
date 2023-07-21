# Ansible role: CRON

Install and deploy cron jobs and cron vars.

A copy of [robertdebock/ansible-role-cron](https://github.com/robertdebock/ansible-role-cron).

Only available on Archlinux.

## Usage
```yaml
cron_jobs: []
  - name:       description of the task
    day:        1-15          # for the first part of the month
    hour:       23            # in the 23rd hour
    job:        ls -l         # run this command
    minute:     "*/10"        # every 10 minutes
    user:       root          # for a specific user
    cron_file:
    month:
    weekday:
    special_time:
    state:

cron_vars: []
  - name:   PATH
    value:  /usr/bin:/bin:/usr/local/bin
    user:   root

```
