# jwt_git_example

Managing an NSC project under revision control is quite simple.
Create a repo:

```sh
❯ git clone git@github.com:/aricart/jwt_git_example
Cloning into 'jwt_git_example'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
```

Set NSC to write JWTs to the git repository - not that we have added a directory `project` to contain JWT.
This will allow you to store other artifacts in the repo, without introducing artifacts into the `project` tree.

```sh
nsc env -s /tmp/jwt_git_example/project
+------------------------------------------------------------------------------------------------------+
|                                           NSC Environment                                            |
+--------------------+-----+---------------------------------------------------------------------------+
| Setting            | Set | Effective Value                                                           |
+--------------------+-----+---------------------------------------------------------------------------+
| $NSC_CWD_ONLY      | No  | If set, default operator/account from cwd only                            |
| $NSC_NO_GIT_IGNORE | No  | If set, no .gitignore files written                                       |
| $NKEYS_PATH        | No  | ~/.nkeys                                                                  |
| $NSC_HOME          | No  | ~/.nsc                                                                    |
| Config             |     | ~/.nsc/nsc.json                                                           |
| $NATS_CA           | No  | If set, root CAs in the referenced file will be used for nats connections |
|                    |     | If not set, will default to the system trust store                        |
+--------------------+-----+---------------------------------------------------------------------------+
| From CWD           |     | No                                                                        |
| Stores Dir         |     | /tmp/jwt_git_example/project                                              |
| Default Operator   |     |                                                                           |
| Default Account    |     |                                                                           |
| Root CAs to trust  |     | Default: System Trust Store                                               |
+--------------------+-----+---------------------------------------------------------------------------+
```

From here on, all operations that manage JWT entities will write to the repository directory.
You can commit/update as necessary. Note that JWTs are binary artifats so there's no merging capability.
Let's add some entities

```sh
❯ nsc add operator O
[ OK ] generated and stored operator key "OCUIPREA2CWBCZZWZQYYQBI2WHFIAEQZCP7MCQOR3YTX6DGLGQKS3LJW"
[ OK ] added operator "O"
❯ nsc add account A
[ OK ] generated and stored account key "ABUV3CIFD3AOYSTXRPFIBSGEQ7HW5UYY7SHAUWGJJZBPMKUEPT74BFIY"
[ OK ] added account "A"
❯ nsc add user U
[ OK ] generated and stored user key "UBASEA3SMOYX3SU6HPUKG6GQRFXVEAAKBMJ3IFIN4XCKLODBSH56HY3R"
[ OK ] generated user creds file `~/.nkeys/creds/O/A/U.creds`
[ OK ] added user "U" to account "A"
/tmp/jwt_git_example main ?1 

❯ git add project/O
❯ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   project/O/.nsc
	new file:   project/O/O.jwt
	new file:   project/O/accounts/A/A.jwt
	new file:   project/O/accounts/A/users/U.jwt

```

## What about secrets?

Secrets are stored in a directory designated by the `$NKEYS_PATH` environment variable.
You shouldn't check in secrets - if you must, you should place them in their own directory in the repo.

## Exporting

Typically there two kinds of exporting that you can do:

- To an `nats-account-server`
- To a mem-resolver file

To easily work with a `nats-account-server` - `nsc edit operator --account-jwt-server-url http://someserver:8080/jwt/v1`, you can then use `nsc push -A` to push all accounts to it, or select the specific account you want to push.

To regenerate a mem-resolver file: `nsc generate config --mem-resolver --config-file accounts.conf` and use a NATS configuration import directive to load it. This way you can generate over and over, without having to merge the main server configuration file.




