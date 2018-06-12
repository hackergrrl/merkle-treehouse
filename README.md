# merkle-treehouse

> social append-only tree data structure for folks & their communities

## What the--?

- data
- reduce functions


## Example: Track all git forks of a project

Let's say this works by having a node called 'forks', and then expecting all
nodes pointing to it to be a one-liner file with a git remote URL, like
`https://github.com/noffle/merkle-treehouse.git` or `hypergit://abcdef`.

Let's make some test data:

```
$ mt init

$ mt 'forks'
Created node 'forks': HASH

$ echo 'hypergit://abcdef' | mt HASH
Created node linked to HASH: HASH2

$ echo 'http://git.eight45.net/foobar.git' | mt HASH
Created node linked to HASH: HASH3

$ mt list HASH
HASH2
HASH3
```

Now lets write a function that shows all forks for a tree:

```js
var treehouse = require('merkle-treehouse')

var tree = treehouse('./tree')

function getForks (tree, cb) {
  pull(
    tree.refs('/forks'),
    pull.filter(function (node) {
      return (/.*:\/\/.*/.test(node.content)
    }),
    pull.map(function (node) {
      return {
        author: node.author,
        url: node.content.split('\n')[0]
      }
    })
    pull.collect(cb)
  )
}

getForks(tree, function (err, forks) {
  console.log(forks)
})
```

outputs

```
[
  { author: KEY, url: 'hypergit://abcdef' }
  { author: KEY, url: 'http://git.eight45.net/foobar.git' }
]
```

## Example: discussion forum

Discussions can be modeled as a list of initial posts, each with their own
threaded tree of responses. Let's make up some data:

```
$ mit init

$ mt 'forum'
Created node 'forum': HASH

$ cat | mt HASH
Hello world. Anybody out there?
^D
Created node linked to HASH: HASH2

$ cat | mt HASH2
Hey, welcome to the merkle treehouse!
^D
Created node linked to HASH2: HASH3

$ cat | mt HASH2
Thanks!
^D
Created node linked to HASH3: HASH4
```

And now a reducer function that can list root topics and also print discussions:

