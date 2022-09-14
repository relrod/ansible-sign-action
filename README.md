# ansible-sign-action

Easily sign Ansible project repositories with `ansible-sign` in a GitHub
Actions workflow.

## What does that mean?

This GitHub Action allows you to easily make use of the
[`ansible-sign`](https://github.com/ansible/ansible-sign) utility in your
workflows to sign your project repository. The signature can then be verified by
AWX/Ansible Automation Controller to ensure that the content has not been
tampered with.

## Scenarios and Using This Action

Most users of `ansible-sign` will be using it alongside AWX/Ansible Automation
Controller to verify the integrity of their Ansible project repositories and
playbooks.

The general idea is that you take your existing Ansible project repository, add
a `MANIFEST.in` file which contains the files you wish to include in signing,
sign the content using `ansible-sign`, and then have AWX/AAC verify the
signature each time it syncs/updates your project repository.

If signature validation fails, it means a file was modified, added to, or
removed from the project since the last time the project was signed.

It would not make much sense to run this action on every push to your
repository. Playing out that scenario - it would mean that every collaborator
with push access to your repository has access to sign the content
(automatically). The only thing this would really prove is that content did not
change between GitHub and the AWX node doing the project update -- and that sync
is already encrypted (using https or ssh git URLs), so there is not much concern
there.

A more interesting scenario is this: Imagine a company or a community maintained
infrastructure in which many members of the organization have commit access to
the project repository, but a small team of people is responsible actually
deploying the changes and ensuring that the infrastructure remains stable. Using
GitHub's "environments" feature, you can add this small team as a reviewer on
the signing workflow. In this way, only a small number of people have
appropriate access to sign the project (and potentially trigger a deployment).

It is worth noting that this moves the responsibility of authorization to
GitHub. If someone gains access to the GitHub account of someone in the signing
team, or otherwise is able to control who is in the signing team, then that
person will also be able to sign repository content.

# Action Options and Sample Usage

|       NAME       |                                  DESCRIPTION                                   |   TYPE   | REQUIRED | DEFAULT |
|------------------|--------------------------------------------------------------------------------|----------|----------|---------|
| `commit-result`  | Describes if the result should be committed and pushed back to the repository. | `bool`   | `false`  | `true`  |
| `gpg-passphrase` | The passphrase for the secret key specified with `gpg-secret-key`, if any.     | `string` | `false`  | `N/A`   |
| `gpg-secret-key` | The private/secret GPG key. This should be stored in a GitHub Secret.          | `string` | `true`   | `N/A`   |

It is recommended that `gpg-secret-key` and `gpg-passphrase` come from secrets
(likely "environment" secrets) stored within GitHub.

Then you can reference them in your workflow:

```yaml
- name: Sign the project
  uses: relrod/ansible-sign-action@main
  with:
    gpg-secret-key: ${{ secrets.GPG_SECRET_KEY }}
    gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
```

Remember that your workflow can make use of the `workflow_dispatch` event to
allow for manual playbook runs/signing. If you trigger the playbook on `push` it
is suggested that you make use of the `environment` job field and have a list of
reviewers set. See the [GitHub
documentation](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
for more information.
