# Catroweb Auto-Deploy
The script fetches open pull requests from the [Catroweb repository](https://github.com/Catrobat/Catroweb-Symfony) 
using the GitHub REST API v3.

Every pull request gets deployed on a wildcard domain. There are two options to ignore certain pull requests: 
One can ignore certain commit hashes and certain GitHub labels using the configuration file. 
There should already exist a label called **no auto-deploy**, which is in the list of ignored labels.

If a deployment fails, the script remembers that and tries to deploy that commit at most 3 times.

Additionally, the branches `master` and `develop` are deployed. 
To add/remove branches, one needs to edit the database directly.

A PHP index side shows a list of all deployed pull requests and branches.

The script relies on a MySQL/MariaDB database table called `deployment`. 
To create that table, one can use the file *create_deployment_table.sql*.

For every deployment, a different database is created with a different database user and random password. 
The deployment is configured with the password using the *.env.local* file. 
Additionally, a *.env.dev.local* file is created to set *SECURE_SCHEME* to HTTPS.

## Configuration options
In _config.py_ you can change MariaDB credentials 
and the path to the web root folder where all repositories get cloned to.
Additionally, you can change the path of the log file, the list of ignored commit hashes 
and the list of ignored GitHub label ids.

The file _nginx-server-block.template_ is a Python string template for the NGINX site file.
You can use `$label` to insert the label, and `${phpversion}` to insert the selected PHP version. 
Therefore, you have to escape every `$` to `$$`.

The label is a identifier unique to every pull request or branch.
For pull requests, _pr_ concatenated with the number of the GitHub pull request is used, e.g. `pr1234`.
For branches, their (escaped) name is used, e.g. `master`, `develop`. 
Be careful, they are used directly as directory names and are not escaped in database queries.

## Cronjob
The script runs every 15 minutes using the following cronjob:

```
/15 * * * * /usr/bin/flock -w 300 /opt/prdeployer/prdeployer.lock /opt/prdeployer/venv/bin/python3 /opt/prdeployer/prdeployer.py >/dev/null 2>&1
```

[flock](https://linux.die.net/man/2/flock) is used to prohibit multiple concurrent executions.


## Python Packages
The script needs Python 3 and depends on the following packages from PyPI:
 * [requests (v2.23.0)](https://pypi.org/project/requests/)
 * [PyMySQL (v0.9.3)](https://pypi.org/project/pymysql/)

## Author
Script coded in 2020 by [Michael Fürnschuß](https://www.mf.at), member of the Catroweb team.
If you should encounter a problem that you cannot solve on your own, you are welcome to contact me.