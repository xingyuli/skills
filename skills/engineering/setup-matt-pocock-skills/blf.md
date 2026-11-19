# Local files

beyond-local-file projects some files into this working tree and lists those items in `.git/info/exclude`. The copy in this tree is the working copy. The durable copy is the hub file. This repo's git records the other files.

## Tell a projection from a repo file

Run `git check-ignore -v -- PATH`. When the matching line is `.git/info/exclude`, PATH is a projection. `git status` in this repo omits it. That result is the end of the lookup.

When the command names no exclude file, PATH is a normal file in this repo.

## Resolve the hub file

The configuration set is `~/.blf/config` when that file exists. Its `config_file` list names the mapping files. A mapping file's project key is the managed project. Its directory is the mapping file's directory plus that key, or the key itself when the key is absolute. `target` lists the working trees that receive the project. A string value is one target and projects the whole tree. `subpath` lists the items that are projected. A directory item covers the files inside it.

Use the entry whose `target` is this working tree. The hub file is the managed-project directory plus PATH relative to this working tree, including a file inside a directory item. The hub commit runs in the git root that contains that directory.

When no entry covers this working tree, report that and stop.

## Commits

Suggesting a commit message includes two subjects: one for this repo, and one for the hub. List the hub paths beside the hub subject. Stop there. Neither repo is committed while implementing.

When the user asks to commit:

1. Commit this repo with the subject they asked for. When every change is a projection, this repo has nothing to commit. That empty step is success, and the request continues. When the commit fails, stop. The hub stays uncommitted.
2. Take the paths this conversation created or edited for which `git check-ignore -v` names `.git/info/exclude`.
3. Resolve each hub file. Stage it only when its bytes match the working-tree file. When the bytes differ, leave it unstaged and report the path.
4. Commit the staged hub paths. The subject is `{PREFIX}: {the subject they asked for}`. `PREFIX` is the text before the colon on the latest commit in that hub repo that touches the managed-project directory. When that subject has no colon, `PREFIX` is the managed project name.
5. Other dirty paths in the hub repo stay unstaged.
6. Push a repo when the user asks for a push.
