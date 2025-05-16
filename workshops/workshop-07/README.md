## Manage users

The Argo CD CLI provides set of commands to set user password and generate tokens.

**Get Current user logged-in**
```bash
argocd account get-user-info
```

**Get full users list**
```bash
argocd account list
```

**Get specific user details**
```bash
argocd account get --account <username>
```
Note: `-a` in short form of `--account`

**Set user password**
```bash
# if you are managing users as the admin user, <current-user-password> should be the current admin password.
argocd account update-password \
  --account <name> \
  --current-password <current-user-password> \
  --new-password <new-user-password>
```

**Generate auth token**
```bash
# if flag --account is omitted then Argo CD generates token for current user
argocd account generate-token -a <username>
```
**with expire**
```bash
argocd account generate-token --expires-in <duration>
```
Note:
- `-e` in short form of `--expires-in` 
- Duration syntax: 
  - second: `10s`
  - hour: `12h`
  - day: `7d`

**Testing the API token**
```bash
argocd app list --auth-token <token>
```